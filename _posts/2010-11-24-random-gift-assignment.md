---
layout: post
title: Random Gift Assignment
tags:
- Ruby
status: publish
type: post
published: true
meta:
  _edit_last: '2'
  _wp_old_slug: ''
author: sirsean
---
This year, my family is thinking about changing the way they exchange gifts between each other. The basic idea is that rather than having every single person give a gift to every other person, each person would only give a gift to one other person. Saves money, makes things easier, seems like a positive all around. But how to choose who gives something to whom?

Enter the Gifter program. It takes a list of names, assigns each one as giving a gift to one other, and outputs the assignments. Each name only appears once as a gift-receiver, and it doesn't allow any name to be assigned to itself -- it kind of defeats the purpose if you get to give a gift to yourself, doesn't it?


    class Gifter
        def initialize(names)
            @names = names
        end

        def unused_names(all_names, used_names)
            all_names - used_names
        end

        def random_name(names)
            names[rand(names.size)]
        end

        def random_name_excluding(names, exclude)
            if names.size == 1 and names[0] == exclude
                raise "Cannot assign #{exclude} to self"
            end
            name = self.random_name(names)
            while name == exclude
                name = self.random_name(names)
            end
            name
        end

        def try_to_give
            gifts = {}
            @names.each do |name|
                gifts[name] = self.random_name_excluding(self.unused_names(@names, gifts.values), name)
            end
            gifts
        end

        def give
            if @names.size == 1
                raise "You can't exchange gifts if you're all by yourself, you poor lonely fool!"
            end
            loop do
                begin
                    return try_to_give
                rescue
                end
            end
        end
    end

    names = File.read("names").split("\n")

    gifter = Gifter.new(names)
    gifts = gifter.give

    gifts.keys.each do |giver|
        puts "#{giver}: #{gifts[giver]}"
    end

It reads the names out of a file, expecting that each name is on its own line. Then it attempts to assign each name to one other name, and if it gets to the end and finds that its only option is to assign a name to itself, it throws away all the assignments and tries again. That can happen a few times, but the likelihood of it happening repeatedly, to the point where you'd notice the program running slowly, is pretty astronomical. So I'm not worried about it.

We'll see if we end up doing it this way. But if you want to try it with your family (or group of friends), I guarantee this program is easier than trying to come up with a fair and random assignment by hand.
