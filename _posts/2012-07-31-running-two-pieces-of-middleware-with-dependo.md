---
layout: post
title: Running two pieces of middleware with Dependo
tags: [Ruby, Sinatra, Rack, Dependo, Middleware]
author: sirsean
---

Middleware is an awesome feature of Rack -- I can add functionality that wraps around an HTTP call without having to change the actual server.

We use it with some success in our [r509-ca-http](https://github.com/sirsean/r509-ca-http) project. r509-ca-http is a Certificate Authority based on [r509](https://github.com/reaperhulk/r509) that serves over an HTTP REST API. Its functionality is intentionally as simple as possible -- the r509-ca-http project is intentionally not responsible for storing information about certificates' validity or a record of which certificates have been issued. Its only responsibility is issuing and revoking certificates.

But if we're going to actually run a CA, we need it to store validity information, among other things. Enter middleware.

We created a new project, [r509-middleware-validity](https://github.com/sirsean/r509-middleware-validity), responsible for storing validity information (issuance and revocation) about every certificate into a Redis database. Originally, we structured it such that it relied on the server it was wrapping around to have a ```#log``` method on it:

    module R509
        module Middleware
            class Validity
                def initialize(app)
                    @app = app
                end

                def call(env)
                    status, headers, response = @app.call(env)

                    # this will just intercept the calls to /1/certificate/issue and ignore anything else
                    if not (env["PATH_INFO"] =~ /^\/1\/certificate\/issue\/?$/).nil? and status == 200
                        @app.log.info "I intercepted /1/certificate/issue"
                    end

                    [status, headers, response]
                end
            end
        end
    end

In your ```config.ru```, you activate the middleware like so:

    use R509::Middleware::Validity
    server = R509::CertificateAuthority::Http::Server
    run server

And on each call to the server, the middleware will be executed first; you need to send the call along to the server with that ```@app.call(env)``` line, and return the results after you're done. But because we're using that ```@app.log.info``` method call, we're relying on our Sinatra server having a ```#log``` method; since it uses the ```Dependo::Mixin```, and we added a ```Logger``` to the ```Dependo::Registry``` in our ```config.ru```, that method will be available.

Which is pretty cool, and was working -- until we needed to add another piece of middleware. In addition to storing validity information in a database, we _also_ need to store every issued certificate on disk. This is another thing we don't want to add to the HTTP service, so middleware is the perfect place for it. So we created [r509-middleware-certwriter](https://github.com/sirsean/r509-middleware-certwriter) and got it saving every certificate that we issued. It was working great in testing ... and then we tried running the server with _both middlewares active at once_.

    use R509::Middleware::Certwriter
    use R509::Middleware::Validity
    server = R509::CertificateAuthority::Http::Server
    run server

And as soon as we tried to issue a certificate, we got this:

    I, [2012-07-31T14:42:15.260855 #9007]  INFO -- : Writing serial: 1233561620675808731887525784788727751733782628003, Issuer: /C=US/ST=Illinois/L=Chicago/O=Ruby CA Project/CN=Test CA
    NoMethodError: undefined method `log' for #<R509::Middleware::Validity:0x00000100c0dab8>
        /Users/sschulte/code/r509-middleware-certwriter/lib/r509/middleware/certwriter.rb:35:in `rescue in call'
        /Users/sschulte/code/r509-middleware-certwriter/lib/r509/middleware/certwriter.rb:28:in `call'
        /Users/sschulte/.rvm/gems/ruby-1.9.3-p0/gems/rack-1.4.1/lib/rack/lint.rb:48:in `_call'
        /Users/sschulte/.rvm/gems/ruby-1.9.3-p0/gems/rack-1.4.1/lib/rack/lint.rb:36:in `call'
        /Users/sschulte/.rvm/gems/ruby-1.9.3-p0/gems/rack-1.4.1/lib/rack/showexceptions.rb:24:in `call'
        /Users/sschulte/.rvm/gems/ruby-1.9.3-p0/gems/rack-1.4.1/lib/rack/commonlogger.rb:20:in `call'
        /Users/sschulte/.rvm/gems/ruby-1.9.3-p0/gems/rack-1.4.1/lib/rack/chunked.rb:43:in `call'
        /Users/sschulte/.rvm/gems/ruby-1.9.3-p0/gems/rack-1.4.1/lib/rack/content_length.rb:14:in `call'
        /Users/sschulte/.rvm/gems/ruby-1.9.3-p0/gems/rack-1.4.1/lib/rack/handler/webrick.rb:59:in `service'

Turns out that when you're using two middlewares, the second one wraps around the server, and the first one wraps around the second one -- ie, they don't both somehow wrap directly around the server. So in our case, the ```#log``` method exists on the HTTP server so the validity middleware can use it, but it doesn't exist on the validity middleware so the certwriter middleware _can't use it_ and dies.

Here comes [dependo](https://github.com/sirsean/dependo) to the rescue!

    module R509
        module Middleware
            class Validity
                include Dependo::Mixin

                def initialize(app)
                    @app = app
                end

                def call(env)
                    status, headers, response = @app.call(env)

                    log.info "Now I can just use the log method directly"
                
                    [status, headers, response]
                end
            end
        end
    end

Since the server is using ```Dependo::Mixin```, it has a ```#log``` method. If we add ```Dependo::Mixin``` to both our middlewares, they each have a ```#log``` method and can wrap around each other in either direction.

Maybe this is obvious, maybe pointless ... but I hadn't thought of it until I ran into it. This is how I got past it.
