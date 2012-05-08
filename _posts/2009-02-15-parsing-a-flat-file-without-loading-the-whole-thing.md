---
layout: post
title: Parsing a Flat File Without Loading the Whole Thing
tags: []
status: publish
type: post
published: true
meta:
  _edit_last: '2'
author: sirsean
---
Okay, I want to talk a little bit about parsing a flat file in PHP. Usually when you see something about it online the recommendation for doing it in PHP is to use file() to get the contents of the file in an array, where each line is an element of the array. Then use explode() on each line. Well, as the filesize grows, that method uses up more and more memory until your script starts to blow up.

There must be a better way!

So here, I'm assuming that each line has a single pipe (|) as a delimeter. I want to scan through the file and only grab one line at a time, tossing each line after it's been processed -- and stopping in the middle if we want to. The fscanf() method allows you to just that, using a regex to break up your text however you want to.

Well, let's just say I want to take the line that looks like "field1|field2|field3|field4" and turn it into an array. The fscanf() method's regex allows you to break up the matched string into an array using %[] for each match. Here's the call I use:
<pre>fscanf($file, "%[^\|]|%[^\|]|%[^\|]|%[^\|\n]\n")</pre>
Note that I pass in a file handle received from fopen(), and that I have a \n at the end; without that, fscanf() doesn't know you want to go line by line. The fourth field adds a \n into its match -- that way, we don't have to trim() the last field when we're done.

It now returns an array that looks like [ field1, field2, field3, field4 ]. Which is great. But it does that for every single line in the file, and I need to find out if this is the one I'm looking for before we move along to the next one. In my application, I want to be able to search by any of two fields, so that's what I'm going to demonstrate here.

I don't want to repeat my parse() method, so I pass in a $parseType value in addition to the $key I'm searching for. $parseType will tell parse() which field to use and how to do the comparison.

Here are the methods that I'm using to compare by each field.
<pre>private function field1Parser($line, $key) {
    $field = $line[0];
    $field = preg_replace('/\*/', '.*', $field);
    if (preg_match("/^{$field}$/", $key)) {
        return new MyCrazyObject($line);
    } else {
        return null;
    }
}

private function field2Parser($line, $key) {
    $field = $line[1];
    if ($field == $key) {
        return new MyCrazyObject($line);
    } else {
        return null;
    }
}</pre>
Note here that when we're searching on field1, we replace all the * characters with .* and do a preg_match() on it to check if it's the line we want, whereas on field2 we just compare directly. The latter is much much faster, but there are some cases where we simply need to do the former in order for it to work. Since the parsing method can differ, I need a separate method for each one rather than simply defining which array index to look for.

Here's a simplified version of my parse() method without error checking or caching.
<pre>private function parse($parseType, $key) {
    $file = fopen ($this-&gt;filename, 'r');
    while ($line = fscanf($file, "%[^\|]|%[^\|]|%[^\|]|%[^\|\n]\n")) {
        if ($parseType == MyCrazyParser::FIELD1) {
            $obj = $this-&gt;field1Parser($line, $key);
        } else if ($parseType == MyCrazyParser::FIELD2) {
            $obj = $this-&gt;field2Parser($line, $key);
        } else {
            $obj = null;
        }

        if ($obj != null) {
            fclose($file);
            return $obj;
        }
    }

    fclose($file);
    return null;
}</pre>
Obviously you'll need error checking to validate that the parse type is acceptable, that the file exists, etc. And you're going to want to check if the given key is found in the cache, and cache it when it's found; and also cache that the key was NOT found, so that later on we don't have to read through the entire file every time we're looking for something that isn't found. I took all that stuff out for time/space reasons and because it's simple and not relevant to the actual discussion of parsing. (And I tried saving this thing once and Wordpress bombed out on me and I've had to write the second half of this article including parse() twice, which is extremely frustrating.)

The field1Parser() is 10x slower than the field2Parser(), because of the regex replacing and matching versus simple equality. Which makes it especially necessary to cache. I decided to cache by both keys when an object is found, so that I can come back later and grab it by the other key if I want and not have to worry about searching the file all over again. But we're not really talking about caching here.

I believe using fscanf() in this manner is much better than using the crappy old file()/explode() technique. But I'm not satisfied that this is the best. I'd like to be able to use closures/anonymous functions instead of calling named methods for the different parse types; in fact, the whole parse type technique (requiring a class-level constant for each valid type) seems crufty to me.

If anyone has seen any techniques that are smoother, simpler, or faster, I'd love to know about it.
