---
layout: post
title: Rails has_secure_password, a reminder
date: 2013-02-20 09:00:00
tags: [Ruby, Rails, has_secure_password, Authentication]
author: sirsean
---

I'm using the new ```has_secure_password``` feature in Rails (I believe it was introduced in Rails 3.1) for user authentication. It uses bcrypt under the hood, which I'd used directly in the past. But ```has_secure_password``` seems to be the cool new way to do this, and it does seem pretty easy.

```has_secure_password``` gives you a bunch of things for free: password hashing and salting, authenticating against the hashed password, and password confirmation validation.

But all the tutorials and explanations I've seen omit a few key details in the name of simplicity. Namely, they don't show you what you need to do for the password confirmation to work, and they don't show you how to validate the password correctly when the User object is updated (ie, they only show you what to do when it's created).

The [RailsCast](http://railscasts.com/episodes/270-authentication-in-rails-3-1) on the topic makes it seem _sooooo_ easy, which is I guess the point, but if you just do what they do you'll end up confused and with something that doesn't work.

Their User model looks like this:

    class User < ActiveRecord::Base
      attr_accessible :email, :password, :password_confirmation
      has_secure_password
      validates_presence_of :password, :on => :create
    end

Note that it only validates the password when the User is created; that's because they want you to be able to change the username later, without having to re-enter the password. But if you add a length restriction to your password (which you should), then later on when the user changes it, the minimum length won't be validated.

Additionally, and I think this is key, the RailsCast shows you the User model, and how to authenticate, but never actually shows how to create a user and have the password\_confirmation validated.

    user = User.new(:email => "my@email.com")
    user.password = "password"
    user.valid? => true

I would've thought that would fail validation, because there's no password\_confirmation set. But this is the key, the thing that apparently isn't mentioned anywhere: **the password confirmation is only validated if you attempt to set it**.

    user = User.new(:email => "my@email.com")
    user.password = "password"
    user.password_confirmation = "otherpass"
    user.valid? => false

    user.password_confirmation = "password"
    user.valid? => true

Okay, so you have to actually set the password\_confirmation or else it won't do anything. But how do you make your validation work when you're updating the user?

Here's a very basic User model that will do that:

    class User < ActiveRecord::Base
      attr_accessible :id, :username, :password, :password_confirmation

      has_secure_password

      validates :username, 
        :presence => true, 
        :length => { :minimum => 3 }, 
        :uniqueness => true

      validates :password,
        :length => { :minimum => 8, :if => :validate_password? },
        :confirmation => { :if => :validate_password? }

      private

      def validate_password?
        password.present? || password_confirmation.present?
      end
    end

Here, it will validate the password and its confirmation [if either the password or the confirmation are set](http://stackoverflow.com/questions/9756652/how-to-disable-password-confirmation-validations-when-using-has-secure-password), or if it's a new User with no password set. This will allow you to update the username without setting the password, but will actually validate the password if you attempt to set it. (And still has the restriction that it only attempts to validate the confirmation if you set password\_confirmation.)
