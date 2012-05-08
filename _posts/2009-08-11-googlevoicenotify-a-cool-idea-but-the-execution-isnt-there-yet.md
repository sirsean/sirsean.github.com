---
layout: post
title: googlevoicenotify - A cool idea, but the execution isn't there yet
tags: []
status: publish
type: post
published: true
meta:
  _edit_last: '2'
author: sirsean
---
So a friend of mine told me about this cool thing you could do with Google Voice and Prowl: poll Google Voice for new SMS messages and send them to your iPhone with push notifications. It'd be a cool way to cut out all incoming and outgoing text messages -- and thus save some money on your plan.

He told me about [a Python program that'd act as a daemon](http://github.com/mikeyk/googlevoicenotify/tree/master) that polls Google Voice, parses out the text messages, and sends them along to Prowl. Sounds like it'd work great, right? 

Well, there were some problems.

The first problem is that it relies very specifically on a certain HTML schema* from Google Voice -- one that has a div element with a class of "gc-message gc-message-sms" _and_ an id attribute that uniquely identifies a "thread" of conversation. This wouldn't be such a problem ... if the Google Voice "API" actually supplied its information in that format. There is no such thread-wrapping div, and there does not appear to be any unique identifiers for any threads. At first I only had one thread, so I thought that might have been the problem; so I texted my GV account from another phone, creating two threads. There are still no wrapping divs, and still no thread identifiers (unique or not). So ... that's a huge problem.

_* The last commit to this project was on July 31, and I can only assume that the author had it working for himself at the time. Which means that in the last 10-12 days, Google has changed the schema for this page fairly significantly. I wouldn't be at all surprised if it continued to change, perhaps at a fairly rapid pace ... which says to me that this isn't going to be a good idea any time soon._

Anyway, that's a fundamental problem with dealing with Google, and the fact that they don't really give a shit about you. But there's another problem, and that's with the design of the application.

Since Google's "API" doesn't specify which SMS messages are "new," and there's no way to mark them as "read," you get _all_ of them at once every time you poll. Given that you don't want to get a push notification for _every_ text message you've _ever_ received every 30-60 seconds or so, the program has to determine which ones are new. And it doesn't do that very elegantly.

It creates a dict of threads, keyed by the thread id, each of which are an array of text messages. It then pickles this dict and saves it to a file on the filesystem. That has to be pickled and unpickled, obviously, which takes time _and gets slower with each text message you ever receive._ But the dict of threads itself can be accessed in constant time ... that's not true of the array of messages in each thread. In order to query that, you have to use the "message identifier" to see if the message exists in the thread, and that's an array-in call -- linear time! Woo!

Can it get dumber than that? Yes. Yes it can. I haven't told you what the "message identifier" is. Well, Google doesn't supply an actual identifier ... so the author of this program decided that the unique identifier of a text message would be ...

Drum roll please.

    identifier = from_name + ' ' + message_txt

Yes. That's right. I hope nobody you know ever sends you a text message with the same content. Ever. You know, like "Where are you?" or "What?" or something along those lines. I get (and send) those all the time ... and with this program you would only be notified the first time someone _ever_ sends you one. (Unless there's something about the way Google defines "threads" that _guarantees_ that there can never be two messages in a thread that have the same content. I doubt it.) So, yes. Unacceptable.

What amuses me even more is that Google _does_ supply something that can come pretty close to guaranteeing that you can get those repeat messages -- there's a time field, with precision to the minute. So unless someone's sending you messages with the same content multiple times per minute (certainly possible, but you may not _need_ notifications for every one of them, and also your friends are boring), you'll get them.

So it gets slower linearly with the number of text messages you've ever received, and it becomes increasingly likely to drop messages as time goes on. I'm going to go ahead and call bullshit on that.

The way you want to improve this is to create an SQL table with (from, text, time) as the only fields, and make it unique across all three fields. For each message you parse, you first check your database to see if the message exists. If it does, ignore it. If not, insert it and push the notification; lather, rinse, repeat until all the messages have been checked. It ignores the thread problem, and works with the _current_ schema from Google. It also doesn't slow down linearly -- and since your database is local, it'll take a while before those queries start to slow you down (and it'll certainly scale better than reading/writing an ever-growing file and doing in-array operations hundreds of times in a loop.

If I can make some time for this, I'll go ahead and fork the project and make these changes. But if someone else does it before I can, do let me know.
