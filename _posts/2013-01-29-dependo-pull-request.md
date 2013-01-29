---
layout: post
title: Dependo Pull Request
tags: [Ruby, Dependo, Open Source]
author: sirsean
---

Last night, I got my first pull request on GitHub from someone I don't know. Pretty cool.

As you may know, [Dependo is a very simple dependency injection library for Ruby](/2012/01/04/ruby-dependency-injection-introducing-dependo/), which just overrides the ```#method_missing``` method and lets you inject methods onto an object just by mixing in the Dependo module.

Well, apparently someone wanted to be able to see if methods have been injected onto the object. I'd been doing that by calling ```Dependo::Registry#has_key?```, but this new way is better.

    module Mixin
        def respond_to?(key, include_private=false)
            if Dependo::Registry.has_key?(key)
                true
            else
                super(key, include_private)
            end
        end
    end

So I quickly accepted the pull request. I probably wouldn't have thought of this without some guy finding my library, using it, and contributing the idea and making my project better.

In case you needed another reason to open source your stuff, this is a great example of why it's a good thing.

Thanks, [Tomas](https://github.com/tmattia)!
