---
layout: post
title: Recursively Passing Blocks
tags: [Ruby, Blocks, dir-util]
author: sirsean
---

Everyone knows how to use Ruby's blocks. ```map``` and ```each```, etc, are among the first things you ever see.

Writing a method that takes a block is something you'll probably eventually have to do, but isn't often covered. So here's a degenerate example on the way to doing what we really want:

    def foo_prepender(target, &block)
        block.call "foo:#{target}"
    end

    foo_prepender("bar") do |thing|
        puts thing
    end

    => "foo:bar"

Our ```foo_prepender``` method takes a parameter and a block, and it calls the block with one parameter. In our example, we call it with a block that just prints out the result.

A while back, I wrote a program that would hunt through a directory (and all its subdirectories) looking for image files to process; I was trying to finally get a handle on my photo collection, which had become more important* now that there are about 15000 pictures of my son sitting on my computer.

_* I should say that it's important to my girlfriend, which makes it very important to me. I've lost photo collections in the past and it hasn't bothered me so much ... if it happens this time it's not going to be great._

Well, when you're dealing with directories on the filesystem, recursion is your friend. Since I wanted to be able to look at all files within a directory and all its subdirectories, I was going to need recursion. And since I wanted a way to do that without assuming anything about image processing, I wanted the image processing code to be in a block ... and the block would have to be called recursively.

    def to_all_files(source_path, &block)
        source = Dir.new(File.expand_path(source_path))
        source.entries.
            select{|x| x[0]!="."}.
            map{|x| File.absolute_path(x, source.path)}.each do |file|
            if File.file?(file)
                block.call file
            elsif File.directory?(file)
                to_all_files(Dir.new(file), &block)
            end
        end
    end

I pass in the path (or a Dir object, that'll work too), along with the block. I look at the absolute path of everything in there, and if it's a file, I pass it along to the block normally (like I showed above).

But if it's a directory, I call this same method recursively, passing the subdirectory and a reference to the block.

When I originally wrote this, I hoped that it'd be easily reusable. This weekend, I had to write another, different program that did some work on the files within a directory, and it turned out that this method was indeed very reusable. So I packaged it up and released it on Github as [dir-util](https://github.com/sirsean/dir-util), so you can go ahead and use it too if you like.
