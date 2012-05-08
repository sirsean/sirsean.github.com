---
layout: post
title: ! 'Emitter Progress: Conversations'
tags:
- Emitter
status: publish
type: post
published: true
meta:
  _edit_last: '2'
author: sirsean
---
I've been making fairly solid progress on Emitter lately. Today's additions were threefold:

- Mentions
- Replies
- Conversations

**Mentions** work exactly the same as they do in Twitter, and aren't worth talking about here.

**Replies** are similar to Twitter, but for now there are still some limitations. For one thing, I haven't worked out how to limit the timelines they go into based on the intersection of followers (though that will almost certainly come soon). Another problem: You can currently reply from your home timeline or a list of a user's emissions, but not from an individual emission. But those are simple enough to do later on.

But the real interesting thing here is **Conversations**. A conversation can start from any original emission; ie, from any emission that isn't in reply to any other emission. When you emit, there's no conversation yet. But once someone replies to you, a conversation is created and your emission becomes the "original" for the conversation.

Anyone who then replies to your original emission _or_ the reply is simply adding to the conversation. This second reply points directly to the emission that it was replying to (whether it was the original or the first reply), as well as to the conversation itself. This data structure allows us to easily get _all_ the emissions in a conversation, or to build out the tree of replies in case we want a threaded display.

For now, I haven't figured out a good way to build a UI for this; I have to think about how I want to show a conversation to the user. But I think showing a full conversation could be a valuable way for people to consume emissions; it's something that you kind of have to maintain in your head when you're using Twitter.

What do you think about conversations?
