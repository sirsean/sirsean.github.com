---
layout: post
title: Using Dependo to help with testing
tags:
- Dependo
- Rspec
- Ruby
status: publish
type: post
published: true
meta:
  _edit_last: '2'
author: sirsean
---
Last week, I [introduced Dependo](http://vikinghammer.com/2012/01/04/ruby-dependency-injection-introducing-dependo/), my new dependency injection framework for Ruby.

Today, I want to demonstrate something that it makes very easy -- and something that I wouldn't know how to adequately test without it.

In our OCSP responder, we rely on a Redis database and can configure (at startup) whether we want to copy the "nonce"* from the request into the response.

_* The idea of the "nonce" is that it increases the browser's ability to trust a response from the server, because not only is the response signed, the signed response contains the timestamp that was given in the request. It makes it much, much more difficult for a replay attack to work against the service. This is mostly irrelevant to my discussion of Dependo, I just wanted to explain it (a little) so you're not too confused._

But in our unit tests, we need to mock the Redis database (because we don't want to have to actually run a database in order for our tests to pass) and re-configure that "copy nonce" behavior between tests.

First, before each test, we set up our **Dependo::Registry** with the values we're going to want. We clear it out from the previous test, set the Logger to not log anything (we can change that if we want to see logging to help figure something out, but generally we don't want to log anything during tests), mock Redis, set a default "copy nonce" setting (defaults to false here), and read our config file (I'd really rather not do it like this, we'll get around to it later, I hope).

    before :each do
        # clear the dependo before each test
        Dependo::Registry.clear
        Dependo::Registry[:log] = Logger.new(nil)

        # we always want to mock with a new redis
        @redis = double("redis")
        Dependo::Registry[:redis] = @redis

        # default value for :copy_nonce is false (can override on a per-test basis)
        Dependo::Registry[:copy_nonce] = false

        # read the config.yaml
        Dependo::Registry[:config_pool] = R509::Config::CaConfigPool.from_yaml("certificate_authorities", File.read("config.yaml"))
    end

Now, we haven't finished setting up each test. In our **config.ru**, we also add an R509::Ocsp::Signer object to the Dependo::Registry. The Ocsp::Signer is constructed from the other items in the Dependo::Registry that are configured, in production, in the config.ru file. So where did we put it?

    def app
        # this is executed after the code in each test, so if we change something in the dependo registry, it'll show up here (we will set :copy_nonce in some tests)
        Dependo::Registry[:ocsp_signer] = R509::Ocsp::Signer.new(
            :configs => Dependo::Registry[:config_pool].all,
            :validity_checker => R509::Validity::Redis::Checker.new(Dependo::Registry[:redis]),
            :copy_nonce => Dependo::Registry[:copy_nonce]
        )
        R509::Ocsp::Responder
    end

I define it in my spec file's #app method because of the order of execution. That order is:

1. The code in "before :each"
2. The code in the individual test
3. The code in #app

Why is that relevant?

Because, in the case where we want to override the default value of Dependo::Registry[:copy_nonce], we need to be able to do that in the individual test code, and then read it in the #app method where the R509::Ocsp::Signer is built.

    it "copies nonce when copy_nonce is true" do
        @redis.should_receive(:hgetall).with("cert:/C=US/ST=Illinois/L=Chicago/O=Ruby CA Project/CN=Test CA:872625873161273451176241581705670534707360122361").and_return({"status" => R509::Validity::VALID})

        # set to true for this test (this works because the app doesn't get set up until after this code)
        Dependo::Registry[:copy_nonce] = true

        get '/MHsweTBSMFAwTjAJBgUrDgMCGgUABBQ4ykaMB0SN9IGWx21tTHBRnmCnvQQUeXW7hDrLLN56Cb4xG0O8HCpNU1gCFQCY2eXAtMNzVS33fF0PHrUSjklF%2BaIjMCEwHwYJKwYBBQUHMAECBBIEEDTJniOQonxCRmmHAHCVstw%3D'
        request = OpenSSL::OCSP::Request.new(Base64.decode64("MHsweTBSMFAwTjAJBgUrDgMCGgUABBQ4ykaMB0SN9IGWx21tTHBRnmCnvQQUeXW7hDrLLN56Cb4xG0O8HCpNU1gCFQCY2eXAtMNzVS33fF0PHrUSjklF+aIjMCEwHwYJKwYBBQUHMAECBBIEEDTJniOQonxCRmmHAHCVstw="))
        ocsp_response = R509::Ocsp::Response.parse(last_response.body)
        request.check_nonce(ocsp_response.basic).should == R509::Ocsp::Request::Nonce::PRESENT_AND_EQUAL
    end

Without Dependo (or a framework with similar capability), it'd be more difficult to test this behavior. Frankly, I don't know if you could even do it, much less do it elegantly.

So the first time a tough spot like this came up, I found myself glad to have [Dependo](https://github.com/sirsean/dependo).
