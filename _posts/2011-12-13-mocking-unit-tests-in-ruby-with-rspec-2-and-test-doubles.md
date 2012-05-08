---
layout: post
title: Mocking unit tests in Ruby, with Rspec 2 and test doubles
tags:
- Mock
- Rspec
- Ruby
status: publish
type: post
published: true
meta:
  _edit_last: '2'
author: sirsean
---
Mocking in your unit tests is a useful and powerful thing. My experience with it had been with JMock, in Java. But for one of my current projects I've found that I need it with rspec, in Ruby. I didn't find a ton of helpful documentation on the subject, so hopefully this explanation helps someone out.

The concept of this class is that it looks up an SSL certificate serial number in a Redis database, and returns some status information. Those details are mostly irrelevant to the question of mocking, but I thought I'd get it out of the way.

    module R509::Validity::Redis
        class Checker < R509::Validity::Checker
            def initialize(redis)
                raise ArgumentError.new("Redis must be provided") if redis.nil?
                @redis = redis
            end

            def check(serial)
                raise ArgumentError.new("Serial must be provided") if serial.nil? or serial.to_s.empty?

                hash = @redis.hgetall("cert:#{serial}")
                if not hash.nil? and hash.has_key?("status")
                    R509::Validity::Status.new(
                        :status => hash["status"].to_i,
                        :revocation_time => hash["revocation_time"].to_i || nil,
                        :revocation_reason => hash["revocation_reason"].to_i || 0
                    )
                else
                    R509::Validity::Status.new(:status => R509::Validity::UNKNOWN)
                end
            end
        end
    end

If you just write tests for this, and your constructor looks like:

    R509::Validity::Redis::Checker.new(Redis.new)

then you're going to have to actually have a Redis database running for your tests, and you need to have the correct data in it in order for the tests to pass. Obviously, that sucks. You want to avoid that kind of environmental dependency in your unit tests.

Enter mocking. Here's a pair of rspec tests that makes sure it returns an UNKNOWN status if nothing is found:

    it "gets unknown when serial is not found (returns {})" do
        redis = double("redis")
        checker = R509::Validity::Redis::Checker.new(redis)
        redis.should_receive(:hgetall).with("cert:123").and_return({})
        status = checker.check(123)
        status.status.should == R509::Validity::UNKNOWN
    end
    it "gets unknown when serial is not found (returns nil)" do
        redis = double("redis")
        checker = R509::Validity::Redis::Checker.new(redis)
        redis.should_receive(:hgetall).with("cert:123").and_return(nil)
        status = checker.check(123)
        status.status.should == R509::Validity::UNKNOWN
    end

(I have two tests here because I've seen the Redis driver return {} if a Hash isn't found, but I want to make sure my code still works if it returns nil.)

The first key line is:

    redis = double("redis")

It's called "double" because you're creating a "test double" object instead of an actual connection to the Redis database. The test double doesn't actually do anything, but if it receives a method call that you didn't tell it to expect (or if it doesn't receive a method call that you _did_ tell it to expect), it'll give you a failing test.

The next key line is where you tell your test double what to expect (and what it should do):

    redis.should_receive(:hgetall).with("cert:123").and_return({})

Let's break this down. You're telling your test double that it "should_receive" a call to the "hgetall" method (you use the symbol :hgetall for this), "with" the single parameter "cert:123", and it should return an empty hash object {}.

The test double verifies that our Checker#check implementation does actually make this call (and only this call) to the Redis object, and it helpfully returns {}, which then exercises the "nothing found in the database" code path. Thus, we can then check that the returned status should equal R509::Validity::UNKNOWN (which would only happen if we reacted properly to the data we told the test double to return).

And what about testing something else, for example that the serial number _does_ exist in the database? Here's the spec for a certificate that's been revoked:

    it "gets revoked with revocation time and reason" do
        redis = double("redis")
        checker = R509::Validity::Redis::Checker.new(redis)
        redis.should_receive(:hgetall).with("cert:123").and_return({"status" => "1", "revocation_time" => "789", "revocation_reason" => "5" })
        status = checker.check(123)
        status.status.should == R509::Validity::REVOKED
        status.revocation_time.should == 789
        status.revocation_reason.should == 5
    end

Here, we say that our test double should receive the "hgetall" method with the single parameter "cert:123", and it'll return a populated hash with some certificate information (its status, revocation time, and revocation reason). We then verify that the status should be REVOKED, and that the revocation_time and revocation_reason are correctly translated to integers.

Now, so far we've only seen expectations where the method should receive only one parameter. Expecting multiple parameters is exactly what you'd expect: just pass multiple parameters to should_receive. Here's an example of a spec for the R509::Validity::Redis::Writer class, where our test double will expect to receive multiple parameters:

    it "when reason isn't provided" do
        redis = double("redis")
        writer = R509::Validity::Redis::Writer.new(redis)
        redis.should_receive(:hmset).with("cert:123", "status", 1, "revocation_time", Time.now.to_i, "revocation_reason", 0)
        writer.revoke(123)
    end

Here, our test double should receive the "hmset" method, with a slew of parameters (obviously, it maintains the type and order of the parameters, in addition to their values). Note also that we don't specify any "and_return" here, since we don't care about the return value from this method call.

Hopefully that helps to explain rspec mocks, in the interest of getting you to make them yourself. It beats the hell out of having a database running for your unit tests to work.
