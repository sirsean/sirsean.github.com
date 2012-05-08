---
layout: post
title: Local Time Calculator for Java (Android)
tags:
- Android
- Java
- Mobile
status: publish
type: post
published: true
meta:
  _edit_last: '2'
author: sirsean
---
Have you ever noticed that both MLB and NFL only give the starting times of their games in Eastern time? Through their websites, and even through their mobile apps -- which I find especially egregious since your phone knows what timezone it's in.

One of the big features, in my opinion, that my [MLB Scoreboard](https://market.android.com/details?id=com.vikinghammer.mlb.scoreboard.full) app has over MLB's official apps (or any other third party scoreboard apps that I know of) is that it converts the given Eastern times into your local time. I've just started using that same code in my new [NFL Scoreboard](https://market.android.com/details?id=com.vikinghammer.nfl.scoreboard.free) app (which I've decided to make [open source](https://github.com/sirsean/NFL-Scoreboard), for some reason).

I want to be able to use this essentially like so:

    LocalTimeCalculator calculator = new LocalTimeCalculator("12:30", "PM");
    // if I'm in Central time (which I am), this will yield 11:30 AM
    Date localTime = calculator.getLocalTime().getTime();

I want it to return a Calendar instead of a Date so I can easily set the date (year/month/day) if I want to; I do that sometimes, but I didn't feel that it should be part of the local time calculation.

So here's my [LocalTimeCalculator](https://github.com/sirsean/NFL-Scoreboard/blob/master/src/com/vikinghammer/nfl/scoreboard/date/LocalTimeCalculator.java):

    public class LocalTimeCalculator {
        
        private String mTime;
        private String mAmpm;
        
        public LocalTimeCalculator(String time, String ampm) {
            mTime = time;
            mAmpm = ampm;
        }
        
        public Calendar getLocalTime() {
            String[] timeArray = mTime.split(":");
            int hour = Integer.parseInt(timeArray[0]);
            int minute = Integer.parseInt(timeArray[1]);
            
            hour = convertToHourOfDay(hour, mAmpm);
            
            Calendar eastern = Calendar.getInstance(TimeZone.getTimeZone("America/New_York"));
            eastern.set(Calendar.HOUR_OF_DAY, hour);
            eastern.set(Calendar.MINUTE, minute);
            
            Calendar local = Calendar.getInstance();
            local.setTimeInMillis(eastern.getTimeInMillis());
            
            return local;
        }

        private int convertToHourOfDay(int hour, String ampm) {
            if ("AM".equalsIgnoreCase(ampm)) {
                if (hour == 12) {
                    return 0;
                } else {
                    return hour;
                }
            } else {
                if (hour == 12) {
                    return 12;
                } else {
                    return hour + 12;
                }
            }
        }
    }

First, I split the "12:30" time into hour and minute, and then convert the hour to 24-hour time based on the AM/PM field. Then I get a Calendar instance set to Eastern time ("America/New_York"), and set its time. I get a new Calendar instance, which will default to local time,* and set the time on it based on the original Eastern Calendar.

_* On my Android phone, this updated as soon as I changed to a different timezone. It was working fine for me for the first few months of the MLB season before I finally went somewhere else (my trip to Las Vegas for DEFCON) and got to test that it was working in a timezone other than Central. Exactly what I was hoping for._

I don't know if local time conversion was a big reason anybody wanted to use MLB Scoreboard or NFL Scoreboard rather than the official apps, but I do think it's the kind of thing all apps should do. Your phone knows what timezone it's in; you shouldn't have to keep translating times in your head.

It's not complicated, and it makes things better for your users. If you're making Android apps, I think you should be doing this.
