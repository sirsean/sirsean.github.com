---
layout: post
title: ! 'Ruby Dependency Injection: Introducing Dependo'
tags:
- Dependo
- Ruby
status: publish
type: post
published: true
meta:
  _edit_last: '2'
author: sirsean
---
Today, I discovered that I need Dependency Injection in Ruby. Other people who are probably smarter than I am [have said you don't need it](http://weblog.jamisbuck.org/2008/11/9/legos-play-doh-and-programming), because Ruby is so dynamic and you can just do whatever you want to without having the structure provided by dependency injection.

> **DI frameworks are unnecessary.** In more rigid environments, they have value. In agile environments like Ruby, not so much. The patterns themselves may still be applicable, but beware of falling into the trap of thinking you need a special tool for everything. Ruby is Play-Doh, remember! Let’s keep it that way.

Others have apparently taken that to mean that dependency injection is simply unnecessary in Ruby, despite the fact that that's explicitly not what Jamis Buck is saying here.

> So, is there no room for DI in Ruby? There definitely is. I use DI nearly every day in Ruby, but I do not use a DI framework. Ruby itself has sufficient power to represent any day-to-day DI idioms you need.

He talks about the ways he injects his dependencies, using Ruby without any framework:

- Factory method that takes an optional class and instantiates an object for you
- A second factory method that you can override in a subclass for testing
- Pass in either classes or implementations in a constructor

But let's say you've got the following problem:

- You're writing a web app, using Sinatra (or anything else, really)
- You're using an ORM library, like Sequel
    + Your models have methods on them
- You want some sort of logging
    + You want to define just one Logger object and use it in your Sinatra app and your Sequel models

Passing the Logger instance from the Sinatra app to the Sequel models either in a constructor or in each method is an ugly hack, and a non-starter. So what do you do?

Enter: [Dependo](https://github.com/sirsean/dependo)!

Dependo lets you register your Logger object. I do this in my config.ru:

    require "dependo"
    Dependo::Registry[:log] = Logger.new(STDOUT)
    
Then, you can just include everything in the Registry as methods in your Sinatra app, and use them as if they're instance methods:

    class MyApp < Sinatra::Base
        include Dependo::Mixin

        get "/?" do
            log.info "I'm logging!"

            "Hello, world."
        end
    end

You can also include the methods in your Sequel models:

    class MyThing < Sequel::Model
        include Dependo::Mixin
        extend Dependo::Mixin

        def self.do_some_query
            log.info "I'm querying with a class method"
            self
        end

        def perform_some_other_action
            log.info "I'm calling an instance method"
        end
    end

Note that in the model class, we used both **include** and **extend**. We do that so we can get the Dependo functionality in both instance methods (via include) _and_ class methods (via extend).

Now, the Dependo::Registry doesn't care what you put in it. You've already seen that we can put an object in it (like a Logger, as above). You can also put in a number,

    Dependo::Registry[:my_important_number] = 7

or a string,

    Dependo::Registry[:my_important_string] = "seven"

or even a function,

    Dependo::Registry[:my_important_proc] = Proc.new { |x| x + 7 }

or a lambda,

    Dependo::Registry[:my_important_lambda] = lambda { |x| x * 7 }

and you'll use those like so:

    class DemoClass
        include Dependo::Mixin

        def do_things
            puts my_important_number
            puts my_important_string
            puts my_important_proc.call(5)
            puts my_important_lambda.call(6)
        end
    end

_* I don't know how to call a Proc or a lambda in this case without using #call on them. I'd much rather the syntax be something like "my\_important\_lambda(6)" ... so, does anybody have any ideas? Thanks!_

Do I think dependency injection is necessary, even in Ruby? I do. It helps with separation of concerns, so each of your classes can do only the things it cares about; it helps with unit testing, so you can mock your classes' dependencies rather than test them all the way through (as you might in an integration test). You also don't have to muck up your constructors or method definitions; your code's API can stay the way you want it, and having to "include" and/or "extend" a mixin is a small price to pay (I think) for what you get.

I'm using Dependo with some success. Let me know if you do too -- or, more importantly, if you find something about it that sucks, so I can try to fix it.
