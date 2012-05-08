---
layout: post
title: Using Ruby's extend and include to inject functionality into Sequel model classes
tags:
- Ruby
- Sequel
status: publish
type: post
published: true
meta:
  _edit_last: '2'
  _wp_old_slug: ''
author: sirsean
---
I was writing a program that needed to delete some items out of several different tables -- but we want to be safe and keep a record of what was deleted, in case we need to put it back in. Since we're guessing we won't find out what the problems will be, if any, until a lot of new data has been inserted into the database, simply dumping the database and reverting to a backup isn't going to be an adequate solution.

I'm using Ruby here, along with the awesome Sequel ORM library.

My idea is that whenever a record is deleted from our database, I want to create an identical record in a "slum" database. These slums won't be used for anything, hopefully, but if we need to use them they'll have everything -- including the id's -- that we deleted.

At first, my models each had methods that supported this ... and it got pretty cumbersome. Then I remembered, "hey wait a minute, this is Ruby, and I can probably do something awesome!"

Indeed, you can do awesome things.

First, I created a "SlumBuilder" module that adds a class method called "build" to all the classes that **extend** it. In Ruby, when you **extend** a module its classes are included as class methods, and when you **include** a module its classes are included as instance methods. That's an important distinction, because we'll be doing both.

Here's SlumBuilder:

    module SlumBuilder
        def build(original)
            object = self.new
            original.keys.each do |field|
                object[field] = original[field]
            end
            object.save
        end
    end

As you can see, it instantiates a new object of whatever type has extended SlumBuilder, and fills it with all the fields (keys) in the given model object (original).

A Slum model object might look like this:

    class SlumCoolThing < Sequel::Model(:cool_thing)
        extend SlumBuilder
        self.db = $slum_db
    end

That's it. It gets the available fields from the database (thank you Sequel!) and all its functionality from SlumBuilder. We just need to set the database it uses, since we're using two different databases.

The regular model object, the one we'll need to delete some records out of, would look like this:

    class CoolThing < Sequel::Model(:cool_thing)
        include ModelDestroyer
        self.db = $regular_db
    end

Simple enough. As you can see, I went with the convention that the slum's model class is the same as the regular model's class with "Slum" prepended. This is important, as you'll see here in ModelDestroyer:

    module ModelDestroyer
        def slum_class
            Object::const_get("Slum#{self.class}")
        end

        def destroy
            begin
                self.slum_class.build(self)
            rescue
                # failed to build a slum, probably because the class doesn't exist, but we'll just continue on with the deletion
                puts "Failed to build slum: #{self.inspect}"
            end
            self.delete
        end
    end

This is where most of the magic happens. The "slum_class" method uses Ruby's cool "get the constant, like a class, that has the given name" functionality to get the class that has the same name as whichever one has included our ModelDestroyer with "Slum" prepended.

Our "destroy" method gets that Slum class and calls its "build" class method, discussed above, that copies all the fields and saves a new record to the slum database. It then calls Sequel's "delete" method to remove the record from our database.

So if we had a variable, cool_thing, that we pulled out of the CoolThing database and we wanted to delete it, we now have two options:

    cool_thing.delete

Which just removes cool_thing from the database. The other option is:

    cool_thing.destroy

Which removes cool_thing from the database **AND** inserts a perfect copy of it into a separate slum database.

---

You're probably not going to need to do exactly this, with inserting duplicate copies of a row upon deletion. But being able to inject functionality like SlumBuilder and ModelDestroyer into a class is really cool -- we couldn't use inheritance for this, because our model classes already inherit from Sequel::Model.

It's bending my mind right now to think about how I'd do this in Java. I think it might be a bit more verbose ... if it's even possible.
