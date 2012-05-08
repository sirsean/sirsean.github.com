---
layout: post
title: Using function scope to build a jQuery event handler
tags:
- haml
- javascript
- jquery
- poll4.me
status: publish
type: post
published: true
meta:
  _edit_last: '2'
author: sirsean
---
So here's another cool thing from the development of [poll4.me](http://poll4.me) that might interest some people.

When I'm creating or editing polls, I have the option of adding an arbitrary number of answers. I don't want to just leave an infinite number of form fields for the user to potentially fill in if he wants to -- that's as stupid as it is impossible. So they have to be able to take an action that adds a field dynamically.

The first time I did it, on the poll creation screen, it was just a feature; I didn't need to abstract it, so I didn't. But once I needed it on the editing screen too, it was time to abstract. So here's what it looks like.

First the creation screen, where there are three default answer fields and the ability to add more:

    %ol#answers_list
        %li
            %input.answer{ :type => "text", :name => "answers[]", :tabindex => 2 }
        %li
            %input.answer{ :type => "text", :name => "answers[]", :tabindex => 3 }
        %li
            %input.answer{ :type => "text", :name => "answers[]", :tabindex => 4 }
    %p
        %a#add_answer_link{ :href => "#" }="add another answer"
 
 And now the link needs to be connected to its functionality:

     :javascript
        $(document).ready(function() {
            $("#add_answer_link").click(add_answer_callback(3, 5, "#answers_list"));
        });

I'm just adding a click event handler to the link, and it's the result of the add\_answer\_callback() function.

Now, the editing screen, where form fields only exist if they're filled in. I'm keeping track of the proper tabindex with a variable here ... it seems dirty to me so if anyone knows enough about Haml to do this more elegantly I'd like to hear about it.

    %ol#answers_list
        - tabindex = 2
        - @answers.each do |answer|
            %li
                %input.answer{ :type => "text", :name => "answers[]", :tabindex => "#{tabindex}", :value => "#{answer}" }
                - tabindex += 1
    %p
        %a#add_answer_link{ :href => "#" }="add another answer"

And then connecting the event handler is pretty much exactly the same:

    :javascript
        $(document).ready(function() {
            $("#add_answer_link").click(add_answer_callback(#{@answers.count}, #{@answers.count + 2}, "#answers_list"));
        });

Note that instead of using constants for the parameters, I'm using calculated values. You can probably tell from here, but the parameters I'm passing in are: "the 0-based index of the next answer," "the 1-based tabindex of the next form field," and "the jQuery selector to which to append the field." Maybe that wasn't as obvious as I thought. It's a good thing the code has comments (in the source, that is, I'm not bothering with them here).

So I'm calling a function to get the event handler, but the event handler itself is supposed to be a function. This is Javascript now, so that's not hard in the slightest.

    function add_answer_callback(answer_index, tabindex, append_to) {
        return function() {
            $('<li><input id="answer_' + answer_index + '" type="text" class="answer" name="answers[]" tabindex="' + tabindex + '" /></li>').appendTo(append_to);
            $('#answer_' + answer_index).focus();
            answer_index++;
            tabindex++;
        }
    }

This function takes its three parameters and returns a function that takes _no_ parameters and can be used as an event handler. Nice. But the _really cool_ thing about it is that the answer\_index and tabindex variables are incremented inside the event handler (ie, that happens each time you click the link), but since they were parameters on the outer function, they're scoped within the add\_answer\_callback() function, which is only called once (when the page is loaded).

That way, the id and tabindex of each field are set properly as you add more fields, and because you're passing them in it doesn't even matter how many you started with (crucial for the editing screen, of course).
