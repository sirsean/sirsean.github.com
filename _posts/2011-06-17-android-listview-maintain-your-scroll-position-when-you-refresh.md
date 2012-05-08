---
layout: post
title: ! 'Android ListView: Maintain your scroll position when you refresh'
tags:
- Android
- Java
status: publish
type: post
published: true
meta:
  _edit_last: '2'
  _wp_old_slug: ''
author: sirsean
---
Refreshing an Android ListView is a pretty common thing -- but the best way to do it isn't immediately obvious. Here's my progress through the different patterns.

## The Obvious

I figured that when I've downloaded the new list of stuff I want to show, that I could just create a new adapter and stuff it into the list.

    EventLogAdapter eventLogAdapter = new EventLogAdapter(mContext, events);
    mEventListView.setAdapter(eventLogAdapter);

This works, in that the list gets updated. But it always scrolls all the way back up to the top. Sometimes that might be what you want, but most of the time you'll want to maintain your scroll position.

## The Naive

I tried getting pixel-level scroll position using getScrollY(), but it always returned 0. I don't know what it's supposed to do. I ended up going with a solution that got _close_ to maintaining your scroll position.

    int firstPosition = mEventListView.getFirstVisiblePosition();
    EventLogAdapter eventLogAdapter = new EventLogAdapter(mContext, events);
    mEventListView.setAdapter(eventLogAdapter);
    mEventListView.setSelection(firstPosition);

This figures out the first item in the list you can see before resetting the adapter, and then scrolls you to it. This maintains your scroll position _within some unknown/arbitrary range_, and can cause you to jump around in the list a little bit when you refresh.

If you're scrolled halfway through a list item, it'll snap you to the top of it so it's completely visible. Unfortunately, if you're scrolled halfway through a list item, that probably wasn't the one you were paying the closest attention to.

## The Elegant

There had to be a better way!

And, of course, there is. Romain Guy, an Android developer who haunts Stack Overflow and Google Groups dropping golden hints when people ask questions, [pointed out](http://groups.google.com/group/android-developers/browse_thread/thread/2e425f0cca0c0e8b?pli=1):

> The problem is that you are creating a new adapter every time you reload the data. That's not how you should use ListView and its adapter. Instead of setting a new adapter (which causes ListView to reset its state), simply update the content of the adapter already set on the ListView. And the selection/scroll position will be saved for you. 

There are two problems with Romain Guy: 1) he doesn't have a central repository of these hints/answers so I can learn what to do before doing everything wrong first, and 2) they really are just _hints_, in that they point you in the right direction without getting you all the way there.

In this case, yes, updating the adapter without creating a new one every time _will_ maintain your scroll position. Except that you'll frequently get an "IllegalStateException: The content of the adapter has changed but ListView  did not receive a notification. Make sure the content of your adapter is not modified from a background thread, but only from the UI thread."

It turns out that you need to call notifyDataSetChanged() on your adapter after changing its contents; fortunately, that "only from the UI thread" bit was a red herring, because I didn't really want to do any processing that could/should be asynchronous on the UI thread.

I added a refill() method to my adapters:

    public void refill(List<EventLog> events) {
        mEvents.clear();
        mEvents.addAll(events);
        notifyDataSetChanged();
    }

And I call it when my download is complete:

    if (mEventListView.getAdapter() == null) {
        EventLogAdapter eventLogAdapter = new EventLogAdapter(mContext, events);
        mEventListView.setAdapter(eventLogAdapter);
    } else {
        ((EventLogAdapter)mEventListView.getAdapter()).refill(events);
    }

If the list doesn't already have an adapter, then I create one. But if it _does_ have an adapter, then I just refill it. And it maintains my scroll position exactly!
