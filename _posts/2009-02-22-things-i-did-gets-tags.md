---
layout: post
title: Things I Did Gets Tags
tags: []
status: publish
type: post
published: true
meta:
  _edit_last: '2'
author: sirsean
---
Today I put together a quick new feature update for <a href="http://things.vikinghammer.com/">Things I Did</a>. I've been primarily using it at work, to record what I do during the day; but if I wanted to record something I did outside of work it kind of gets lost amongst all the other things.

Obviously, the solution is a tags/labels concept, where each thing can have an arbitrary number of tags. I'd been thinking about it for a while, and hadn't come up with a good way to do it. I was leaning toward a system where you manually add tags, then select them from a list (which checkboxes) when creating a new thing. At best, even in my imagination, it was really cumbersome.

Since the behind-the-scenes implementation should be the same regardless of how the interface is constructed, I decided to go ahead with starting it. I added a simple Tag model and put a ManyToManyField on the Thing class.

While I was playing with getting them connected, I was struck by a much simpler, more text-based interface for tags: #hashtags right there in the thing's text.

I needed a way to extract them from the text, so I added a method to the Thing model:
<pre>def extractHashTags(self):
    hashRegex = re.compile('#([\w\-\_]+)')
    tags = hashRegex.findall(self.text)

    for tag in tags:
        self.text = self.text.replace('#%s' % tag, '').strip()

    return tags</pre>
I'm using a regex that looks for a # followed by letters, hyphens, and underscores, and use findall() to get all of them. (Simply using match() will only get the first one.) I then loop through each tag that was found and remove it from the text, because I don't want to save the tagname as part of the Thing.

But I'm not out of the woods yet, I still need to add something to the add() view to manage the tags on a new Thing. So I replaced the simple thing.save() line with all this:
<pre>thing = Thing(user=user, text=text)
tagTexts = thing.extractHashTags()
thing.save()

for tagText in tagTexts:
    tags = Tag.objects.filter(user=user).filter(text=tagText)
    if tags:
        tag = tags[0]
    else:
        tag = Tag(user=user, text=tagText)
        tag.save()

    thing.tags.add(tag)</pre>
After creating the Thing with the full text (including #hashtags), I extract them and get a list of them. Before I can add the tags (if there are any), I have to save the Thing; ManyToManyField relationships don't work without primary keys. I loop through each of the #hashtags I found and check if they already exist -- if not, I have to create them. Then I add each one to the Thing's list of tags, which automatically saves.

The release process was a little cumbersome; I had to keep track of the newly created tables in apps that were already installed in the settings.py file. It'd be nice if Django had some concept of Rails migrations.

But it's working pretty nicely now. With a bit of use this week I'll be able to tell if I like the new feature.
