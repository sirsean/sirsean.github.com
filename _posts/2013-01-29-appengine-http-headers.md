---
layout: post
title: AppEngine HTTP Headers
date: 2013-01-29 11:00:00
tags: [Java, HTTP, Appengine]
author: sirsean
---

So, I'm working on a new Android app* which requires user authentication and storing data to a backend service. Since pretty much everyone with an Android phone has a Google account, and Google is doing a good job of implementing OAuth 2 for me, I figured I'd just use their authentication. [Tim Bray alerted me](https://www.tbray.org/ongoing/When/201x/2013/01/07/Hybrid-Apps) to [the cool new way of authenticating](http://android-developers.blogspot.com/2013/01/verifying-back-end-calls-from-android.html) which doesn't require the user to enter any passwords, and may not even require the user to even click anything.

_* It's a game, with asynchronous multiplayer functionality._

After being confused for a while about how complicated it was supposed to be -- I thought you have to pass in the token from ```GoogleAuthUtil.getToken``` into something else to complete the authentication and pass a cookie back to AppEngine to maintain an HTTP session, when in fact you can just pass the token directly to your service to verify identity -- I was ready to start moving forward.

I decided that I'd pass the token as a custom HTTP header, so I don't have to either put it in the query parameters (which is dangerous even on HTTPS) or pollute the JSON I'm passing to the service. I figured ```X-GoogleAuthToken``` would be a good option for a header name. And this is where our story really begins.

On the local dev service, ```X-GoogleAuthToken``` worked fine, and I was able to authenticate using the token. But when I deployed to production, the value for the header always came through as null.

So I wrote an extremely simple servlet to test out the headers.

	public void doGet(HttpServletRequest req, HttpServletResponse resp)
			throws IOException {
		resp.setContentType("text/plain");
		Enumeration headers = req.getHeaderNames();
		while (headers.hasMoreElements()) {
			String name = (String)headers.nextElement();
			String value = req.getHeader(name);
			resp.getWriter().println(String.format("%s: %s", name, value));
		}
	}

I passed in the following headers*:

    GET /lettergarden HTTP/1.1
    X-Googleauthtoken: blah blah
    Googleauthtoken: test
    Fakeheader: here
    X-Gauthtoken: howdy
    X-Google-Auth-Token: 123
    X-G-Auth-Token: boosh

_* It was supposed to be "X-GoogleAuthToken" and "fakeheader" and "X-Gauthtoken", but I'm using an app called "HTTP Client" that apparently, and I just learned this, title cases the headers before sending them. Seems wrong?_

And on the local server, here's what I got:

    Host: localhost:8888
    User-Agent: HTTP%20Client/0.9.1 CFNetwork/454.12.4 Darwin/10.8.0 (i386) (MacBookPro5%2C4)
    X-Googleauthtoken: blah blah
    Googleauthtoken: test
    Fakeheader: here
    X-Gauthtoken: howdy
    X-Google-Auth-Token: 123
    X-G-Auth-Token: boosh

Okay, that's everything I passed in, plus the Host and User-Agent. Nothing to suggest there are dragons approaching.

Here's what I get after deploying to production:

    Host: some-project-this-isn't-really-the-name.appspot.com
    Fakeheader: here
    Googleauthtoken: test
    X-Gauthtoken: howdy
    X-G-Auth-Token: boosh
    User-Agent: HTTP%20Client/0.9.1 CFNetwork/454.12.4 Darwin/10.8.0 (i386) (MacBookPro5%2C4)
    X-AppEngine-Country: US
    X-AppEngine-Region: il
    X-AppEngine-City: chicago
    X-AppEngine-CityLatLong: 41.878114,-87.629798

The [AppEngine documentation says](https://developers.google.com/appengine/docs/java/runtime#Request_Headers) they'll add X-AppEngine-Country, X-AppEngine-Region, X-AppEngine-City, and X-AppEngine-CityLatLong as a service to you so you can do some location stuff easily. They also say they _will_ remove some headers from requests: Accept-Encoding, Connection, Keep-Alive, Proxy-Authorization, TE, Trailer, Transfer-Encoding.

You may have noticed, however, that it _also_ stripped X-Googleauthtoken and X-Google-Auth-Token from the request. The documentation does not indicate anywhere that AppEngine will strip any HTTP header that starts with "X-Google". And it **does not** strip headers that start with "Google" (which obviously you shouldn't do).

I'd rather Google didn't bogart the entire X-Google\* namespace if they're not even using any part of it. But, frankly, it wouldn't be a big deal at all if they'd just _document that they're doing it_ and _make the dev environment mirror the actual production environment_.

I spent a few hours combing Google search results, AppEngine documentation, and StackOverflow for why this was happening. Maybe, having written this, the next poor sap to be bitten by this won't have to waste so much time.
