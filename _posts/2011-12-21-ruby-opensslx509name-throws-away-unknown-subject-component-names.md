---
layout: post
title: Ruby OpenSSL::X509::Name throws away unknown subject component names?!
tags:
- C
- OpenSSL
- Ruby
status: publish
type: post
published: true
meta:
  _edit_last: '2'
author: sirsean
---
OpenSSL::X509::Name is a class in Ruby's OpenSSL bindings that lets you deal with the subject line of SSL certificates. It's useful and necessary, though dealing with it can be kind of annoying. But as of Ruby 1.9.3, there's a big bug that threatens to be a deal-breaker for anyone doing significant SSL work in Ruby: its handling of undefined OIDs.

You're accustomed to dealing with subject components by their shortname, things like "CN" or "O" or "C", etc. But underneath the shortname is a different representation; the longname can be something like "1.3.6.1.4.1.311.60.2.1.3" instead.

Now, you're allowed to create an OpenSSL::X509::Name object with an unknown OID like this.

    name = OpenSSL::X509::Name.new [["CN", "vikinghammer.com"], ["1.3.6.1.4.1.311.60.2.1.3", "US"]]

And if you now get the subject line, all is well.

    name.to_s
    #=> "/CN=vikinghammer.com/1.3.6.1.4.1.311.60.2.1.3=US"

But sometimes you want to get that array (like the one you passed into the constructor) back out and deal with it directly. We're doing that in one of our projects, where we want to wrap OpenSSL::X509::Name in a friendlier interface. But what happens if you do it?

    name.to_a
    #=> [["CN", "vikinghammer.com", 12], ["UNDEF", "US", 12]]

**UNDEF**?! What help is that, you guys? I mean, after all ...

    OpenSSL::X509::Name.new name.to_a
    #OpenSSL::X509::NameError: invalid field name
    #    from (irb):8:in `initialize'
    #    from (irb):8:in `each'
    #    from (irb):8:in `initialize'
    #    from (irb):8:in `new'
    #    from (irb):8
    
Yeah, you can't pass the output from OpenSSL::X509::Name#to\_a back into the constructor; if you try, it'll blow up.

But I kind of need to do that; when we parse a CSR or a certificate, it contains an OpenSSL::X509::Name, and we need to be able to read it. When we do that, we read the array out of #to\_a, because parsing the subject line string out of #to\_s is dangerously fragile (because the delimiters, "/" and "=", can be used unescaped in the subject component values).

Enough complaining. What's the solution?

We staged a two-pronged attack on this problem. The first step was to [patch Ruby](https://github.com/reaperhulk/ruby/compare/ruby:trunk...aa668275), to fix [the bug](http://bugs.ruby-lang.org/issues/5787) in its OpenSSL bindings. My teammate [Paul Kehrer](http://langui.sh) did this.

    short_name = OBJ_nid2sn(OBJ_ln2nid(long_name));
    if (strcmp(short_name,"UNDEF") == 0) {
        return_name = &long_name;
    } else {
        return_name = short_name;
    }

It's a fairly simple fix (once you've, you know, gotten deep enough into Ruby's C source). It used to attempt to translate the longname into a shortname, and if no translation was found, it'd just leave the shortname as UNDEF. Instead, he checks for UNDEF and if that's the shortname, he resets it to be the (original) longname. That way, the OID is preserved in the OpenSSL::X509::Name object. Victory!

But ... that'd mean our project would only work on a custom patch of Ruby 1.9.3, and we were hoping to maintain compatibility across both 1.8 and 1.9, and we certainly can't rely on people to use Paul's patch rather than the actual release of Ruby. And even if (hopefully "when") Ruby accepts the patch, that'd mean we'd have to require Ruby (at least) 1.9.4, which is pretty close to unacceptable. We need an interim solution -- where "interim" is taken to mean "useful for at least a few years, potentially indefinitely".

That's where the second prong of our attack comes in. Sanitizing the OpenSSL::X509::Name#to\_a right in Ruby!

    # Sanitize an X509::Name. The #to_a method replaces unknown OIDs with "UNDEF", but the #to_s
    # method doesn't. What we want to do is build the array that would have been produced by #to_a
    # if it didn't throw away the OID.
    class NameSanitizer
        # @option name [OpenSSL::X509::Name]
        # @return an array of the form [["OID", "VALUE], ["OID", "VALUE"]] with "UNDEF" replaced by the actual OID
        def sanitize(name)
            line = name.to_s
            array = name.to_a.dup
            used_oids = []
            undefined_components(array).each do |component|
                begin
                    # get the OID from the subject line that has this value
                    oids = line.scan(/\/([\d\.]+)=#{component[:value]}/).flatten
                    if oids.size == 1
                        oid = oids.first
                    else
                        oid = oids.select{ |match| not used_oids.include?(match) }.first
                    end
                    # replace the "UNDEF" OID name in the array at the index the UNDEF was found
                    array[component[:index]][0] = oid
                    # remove the first occurrence of this in the subject line (so we can handle the same oid/value pair multiple times)
                    line = line.sub("/#{oid}=#{component[:value]}", "")
                    # we record which OIDs we've used in case two different unknown OIDs have the same value
                    used_oids << oid
                rescue
                    # I don't expect this to happen, but if it does we'll just not replace UNDEF and continue
                end
            end
            array
        end

        private

        # get the components from #to_a that are UNDEF
        # @option array [Array<OpenSSL::X509::Name>]
        # @return [{ :index => the index in the original array where we found an UNDEF, :value => the subject component value }]
        def undefined_components(array)
            components = []
            array.each_index do |index|
                components << { :index => index, :value => array[index][1] } if array[index][0] == "UNDEF"
            end
            components
        end
    end

This solution, as you may have noticed, is a little bit more complicated. I'll try to explain.

First, we take advantage of the fact that OpenSSL::X509::Name#to\_s contains **all** the data we need, despite the fact that #to\_a throws it away. So we loop over #to\_a to find all the subject components whose name is UNDEF and record their index in the #to\_a array, as well as their value (we don't need to record their names, because we know they're always UNDEF).

Then, we loop over all those UNDEF subject components, and we determine what the original OID was from the subject line string from #to\_s. For that, we take advantage of the fact that these unknown OIDs will be of the form 1.2.3.4.5, or something like that; ie, our regex looks for anything with digits and periods in the place of the subject component name, _with the same value_ as the undefined subject component.

Note that we also rely on the fact that #to\_s and #to\_a maintain the order of subject components, so as we loop over the unknown OIDs, the current one will always correspond to the first match from the subject line. 

After updating the subject component name in the array, we remove the subject component from the subject line (but only the first occurrence of it), and we record that we've used that OID. Both of those steps are to ensure that if the same OID occurs multiple times, or if two different OIDs have the same value, that we handle that properly and correctly maintain the order of the components.

And now ...

    sanitizer = NameSanitizer.new
    sanitizer.sanitize(name)
    #=> [["CN", "vikinghammer.com", 12], ["1.3.6.1.4.1.311.60.2.1.3", "US", 12]]

    OpenSSL::X509::Name.new(name.to_a)
    #OpenSSL::X509::NameError: invalid field name

    OpenSSL::X509::Name.new(sanitizer.sanitize(name))
    #=> /CN=vikinghammer.com/1.3.6.1.4.1.311.60.2.1.3=US

It works! As long as we pass the original name to the sanitizer, we can get the original OIDs in the array structure we need (the same one returned from #to\_a).

This will work across Ruby 1.8 and 1.9, and if Ruby accepts Paul's patch, it'll also work in 1.9.4; if the bug is fixed, NameSanitizer#undefined\_components will not find any UNDEF subject components, and none of our complex code will ever be touched.

I think this exercise demonstrates a few things:

- It's considerably simpler to fix this kind of thing at a lower level, if possible.
- Even if a bug in your language gets fixed, you should still try to figure out a backwards-compatible fix.
- Use open source languages! It's awesome to have the option of fixing Ruby and submitting a patch.
- If you're working on an open source project, you can write posts like this about it.

This turned a horrifyingly annoying Tuesday evening (when Paul discovered the bug) into a pretty fun Wednesday morning (when we fixed it).
