---
layout: post
title: Introducing rest/api
tags: [Ruby, rest-api]
author: sirsean
---

I have recently been building a modular service-oriented architecture, using Sinatra to create simple, focused HTTP services. Part of the benefit of that is that you can consume one service within another, or consume multiple services in a wholly separate application (and if you find that you need to, you can scale the services separately).

But in order for that to work nicely, you need to be able to call those services easily. So I wrote a little class that wraps the HTTP GET/POST calls, translates to/from JSON as needed, and gives you nice Ruby objects to work with.

For example, say you have a service with the following endpoints:

- /thing/:thing_id
- /thing/create
- /thing/:thing_id/update
- /other_thing/:other_thing_id
- /other_thing/create
- /other_thing/:other_thing_id/update

And you want to consume that service in your program. You **could** spin up HTTP requests every time you want to call them, but that'd kind of suck.

Or, you could use [**rest-api**](https://github.com/sirsean/rest-api).

    module MyThings::API
        class Core
            include REST::API::Base

            attr_reader :thing, :other_thing

            def initialize
                @thing = MyThings::API::Core::Thing.new(self)
                @other_thing = MyThings::API::Core::OtherThing.new(self)
            end
        end
    end

    class MyThings::API::Core::Thing
        def initialize(api)
            @api = api
        end

        def get(thing_id)
            @api.GET("/thing/#{thing_id}")
        end

        def create(param1, param2)
            @api.POST("/thing/create", {
                "param1" => param1,
                "param2" => param2
            })
        end

        def update(thing_id, param1, param2)
            @api.POST("/thing/#{thing_id}/update", {
                "param1" => param1,
                "param2" => param2
            })
        end
    end

    class MyThings::API::Core::OtherThing
        def initialize(api)
            @api = api
        end

        def get(other_thing_id)
            @api.GET("/other_thing/#{other_thing_id}")
        end

        def create(param1, param2)
            @api.POST("/other_thing/create", {
                "param1" => param1,
                "param2" => param2
            })
        end

        def update(other_thing_id, param1, param2)
            @api.POST("/other_thing/#{other_thing_id}/update", {
                "param1" => param1,
                "param2" => param2
            })
        end
    end

To use it, you just need to instantiate your Core API class and set its base_url field (basically, that's where your service is running).

    core = MyThings::API::Core.new
    core.base_url = "http://localhost:4567"
    thing = core.thing.get(thing_id)
    puts thing.param1
    other_thing = core.other_thing.create(param1, param2)
    puts other_thing.param1

And let's say you had another service that you wanted to be able to use in your app (ie, hosted on a different server). You'd create another class that includes ```REST::API::Base``` and hook up your routes, and could just do:

    core2 = MyThings::API::Core2.new
    core2.base_url = "http://localhost:4568"
    another_thing = core2.another_thing.get(another_thing_id)

I'm sure I'll change some things around as I find ways to make it even easier (or, more likely, as I find things about it that suck). But for now, it's good enough for me to use in my projects.

Meanwhile, if you're interested, you can [grab it from Github](https://github.com/sirsean/rest-api).
