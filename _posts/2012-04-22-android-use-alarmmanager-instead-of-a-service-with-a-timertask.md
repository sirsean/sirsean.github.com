---
layout: post
title: ! 'Android: Use AlarmManager instead of a Service with a TimerTask'
tags:
- Android
- battery
- Java
- Mobile
status: publish
type: post
published: true
meta:
  _edit_last: '3'
author: sirsean
---

In building the live scoring widget for MLB Scoreboard, I needed to be able to reload the scores, repeatedly, when the app isn't open. Since Android lets you start a Service that runs in the background, that seemed like the best way to go.

I started the Service both when your phone first booted and each time you opened the app (to make sure it was always available in the background). I used a TimerTask and a Handler to run the "download the scores and update the widget" code periodically; I limited how often it actually did the downloading work by checking when the game actually started and only downloading scores once per day (in the morning) when the game wasn't playing, but then loading it every minute during the game. I thought that would help save the battery a little bit.

I was wrong. Because of the way TimerTask works in a Service, it was still executing code very frequently even thought it wasn't actually downloading anything. As a result, the phone kept waking up to check if it should download anything, which meant it could never sleep long enough to save any battery. With the early prototype of my widget sitting on my phone, I ran out of power even more quickly than my battery's waning capacity normally would.

Enter [**AlarmManager**](http://developer.android.com/reference/android/app/AlarmManager.html). Rather than starting a Service that runs in the background always, _this_ is what you want to use to schedule your code to run when your app isn't open.

Crucially, it gives you the option of waking up the phone to execute your code or just _waiting until the user wakes up the phone_ before executing the scheduled code.

Here's how you schedule a receiver.

    Intent alarmIntent = new Intent(context, FavoriteTeamDownloadReceiver.class);
    PendingIntent pendingIntent = PendingIntent.getBroadcast(context, 0, alarmIntent, PendingIntent.FLAG_UPDATE_CURRENT);
    AlarmManager alarmManager = (AlarmManager)context.getSystemService(Context.ALARM_SERVICE);
    alarmManager.set(AlarmManager.RTC, Calendar.getInstance().getTimeInMillis(), pendingIntent);

**Note:** The code I want to run is a **BroadcastReceiver** called **FavoriteTeamDownloadReceiver**. By using PendingIntent.FLAG\_UPDATE\_CURRENT, if I schedule another Intent before the previous one executes, the extras in its bundle will be overridden with the new values. That's not important here, but it's what I want usually.

You grab the AlarmManager by calling **getSystemService(Context.ALARM_SERVICE)**, you never want to instantiate it yourself.

And by scheduling it with **AlarmManager.RTC**, I can specify the time it should run but if the phone is sleeping, the intent will not run _until the phone is woken up_ by some other means. It doesn't wake up to run my code. (If you _want_ it to wake up to run your code, like if you were actually making an alarm clock or something, you could use **AlarmManager.RTC_WAKEUP**.)

Of course, right now, this wouldn't work. You need to actually write your BroadcastReceiver:

    public class FavoriteTeamDownloadReceiver extends BroadcastReceiver {
        @Override
        public void onReceive(final Context context, Intent intent) {
            // do the thing in here
            // including figuring out the next time you want to run 
            // and scheduling another PendingIntent with the AlarmManager
        }
    }

And in your **AndroidManifest.xml** file, you need to tell the system that you're going to be doing this:

    <receiver android:name=".service.FavoriteTeamDownloadReceiver" android:process=":remote" />

Now you're up and running. When your phone is in your pocket, nothing will happen; it doesn't wake up to check if it should download anything, and doesn't use any extra battery. When the phone wakes up, it will run the most recently scheduled intent that should have executed, and by the time you unlock the phone your home screen widget will already have been updated.

Pretty useful. And this seems like the right way to do it.
