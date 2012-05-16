---
layout: post
title: A Rake task to create Mongoid indexes
tags: [Ruby, Mongo, Mongoid, Rake, Sinatra]
author: sirsean
---

I've recently started using Mongoid behind my Sinatra-based web services, and I like it. But I wanted an easy way to create indexes on my Mongo collections, and the options seemed to be a) use Rails, or b) set "autocreate_indexes" to true in mongoid.yml. I don't want to do either of those things.*

_* Notably, "autocreate_indexes" can be unwise because creating indexes can take a while and you don't necessarily want that delay while your service is starting up. It's better to take that hit out of band, like in a Rake task._

Now, Mongoid provides a Rake task to create your indexes if you happen to be using Rails. But if you're not, what can you do? Well, write your own:

    namespace :db do
        task :create_indexes, :environment do |t, args|
            unless args[:environment]
                puts "Must provide an environment"
                exit
            end

            yaml = YAML.load_file("mongoid.yml")

            env_info = yaml[args[:environment]]
            unless env_info
                puts "Unknown environment"
                exit
            end

            Mongoid.configure do |config|
                config.from_hash(env_info)
            end

            MyModels::Thing1.create_indexes
            MyModels::Thing2.create_indexes
            MyModels::Thing3.create_indexes
        end
    end

And you use it as you'd expect:

    rake db:create_indexes[development]

You have to specify the environment, because Mongoid will choose which database to connect to based on that. It supports "development", "test", and "production", which you have to specify in your mongoid.yml file.

It's working well for me so far on two projects. One thing I wish is that I didn't have to list all the model classes, that I could have that determined automatically. Maybe I'll figure that out later.
