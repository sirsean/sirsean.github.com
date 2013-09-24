---
layout: post
title: Mailgun and Go, go-mailgun
date: 2013-09-24 09:00:00
tags: [Go, golang, Mailgun, Email, go-mailgun]
author: sirsean
---

Lately, I've been playing with Go. The language itself is small enough that I quickly got to the point that I needed to write an app in order to feel like I was learning anything more. As it happens, earlier this year, I wrote an app called "MLB Notifier" that monitors the MLB datafeed for changes and emails me* about interesting changes. That was written in Java, and runs on Google App Engine. It seemed like a good opportunity to rewrite an app in Go; I'd get to deal with network communication, JSON parsing, sending email, and a not-insubstantial amount of logic.

_* Note to MLB's lawyers: it only sends emails to me, and nobody else. This is an individual, non-bulk, non-commercial use._

This isn't about MLB Notifier. It's about email. From App Engine, I was able to just send messages from my Gmail account, because it's running on my Google account in Google infrastructure. For some reason, I assumed it wouldn't be an issue to just keep doing that from a VM running somewhere on the internet. But when I tried sending emails via SMTP using my Gmail credentials*, I immediately received an email from Google saying "Suspicious sign in prevented". Seems like someone trying to log into my Gmail account from a server in Indonesia seems weird to Google? Also, it turns out my VM lives in Indonesia.

_* Since this would require me to leave those credentials sitting on that VM, I already didn't like this solution._

Enter Mailgun. My team uses it at work (in Ruby), and it seemed simple and effective. Googling around for how to use it from Go didn't surface any useful results (come on, you guys, I can't be the first person to do this), so I had to figure it out on my own. Mailgun provides solid API documentation, and I was able to convert their Ruby/PHP/Curl examples fairly easily.

They have a very easy endpoint that you hit over HTTPS, which I personally think is nicer than having to actually deal with SMTP myself. The basic process for sending an email is:

* Send POST variables designating the sender, recipient, subject, and message body
* Set the Content-Type to application/x-www-form-urlencoded
* Set HTTP Basic Authentication to log in with api:\[whatever Mailgun says is your API key\]
* Send this request to the special endpoint that Mailgun gives you, which has your username in it

You can see [all the code on GitHub](https://github.com/sirsean/go-mailgun/blob/master/mailgun/mailgun.go), in my new [go-mailgun](https://github.com/sirsean/go-mailgun) project.

I wanted to pass a struct representing my message, which is really easy to define in Go:

```
type Message struct {
    FromName string
    FromAddress string
    ToAddress string
    Subject string
    Body string
}
```

Along with a method to format the from name/address:

```
func (m Message) From() string {
    return fmt.Sprintf("%s <%s>", m.FromName, m.FromAddress)
}
```

So then I just define my ```Send``` method:

```
func Send(message Message) error {
    client := &http.Client{}

```

Here we set up the POST variables based on the Message provided:

```
    values := make(url.Values)
    values.Set("from", message.From())
    values.Set("to", message.ToAddress)
    values.Set("subject", message.Subject)
    values.Set("text", message.Body)
```

And then construct an HTTP request with the Content-Type header and Basic Auth:

```
    request, _ := http.NewRequest("POST", ApiEndpoint, strings.NewReader(values.Encode()))
    request.Header.Set("content-type", "application/x-www-form-urlencoded")
    request.SetBasicAuth("api", ApiKey)
```

Then we send the request (note that Go lets us "defer" closing the body until we return from this method, which is _very_ convenient, in case any of you have experience writing deeply nested try/catch/finally blocks in Java to do the same thing ... actually, I'm sorry I mentioned it, let's just move on):

```
    response, e1 := client.Do(request)
    if e1 != nil {
        fmt.Println("Failed to send request")
        fmt.Println(e1)
        return e1
    }
    defer response.Body.Close()
```

And I read the response here, even though I really don't need it. Although printing it in the logs was useful, at the beginning when I hadn't set the Content-Type and Mailgun's error message told me it couldn't send a message without a "from" parameter. I took that to mean that it wasn't receiving the parameters, and it wasn't long before I figured out it needed the Content-Type.

```
    body, e2 := ioutil.ReadAll(response.Body)
    if e2 != nil {
        fmt.Println("Failed to read response")
        fmt.Println(e2)
        return e2
    }

    fmt.Println(string(body))
    return nil
}
```

So that's easy! But that's if you want to write the method yourself, which I don't intend to do again. Using it is even easier, as demonstrated in this degenerate example app:

```
import (
    "github.com/sirsean/go-mailgun/mailgun"
)

func main() {
    mailgun.ApiEndpoint = "https://api.mailgun.net/v2/YOURNAME.mailgun.org/messages"
    mailgun.ApiKey = "YOURKEY"

    go func() {
        err := mailgun.Send(mailgun.Message{
            FromName: "Foo Bar",
            FromAddress: "foo@bar.test",
            ToAddress: "recipient@bar.test",
            Subject: "This is an example message",
            Body: "It's pretty easy to send messages via Mailgun!",
        })
        if err != nil {
            // you can handle sending errors here
        }
    }()
}
```

I set my endpoint/key once (in my app I do it in ```main.main```, having read the values from a YAML file), and then send the email within a goroutine. In this case it's an anonymous function, but obviously it doesn't have to be; in my MLB Notifier application I name a separate method that calls ```mailgun.Send```, because in that case it makes the code clearer.

So, I was pleased by how simple it was, and I have been pleased with Mailgun's performance delivering the emails. Notably, except for one evening where I was seeing weird behavior:

* (21:42) SD tied it up in the 9th, 2-2
* (21:42) SD broke the tie in the 9th, 3-2
* (22:42) SD tied it up in the 9th, 2-2
* (22:59) SD broke the tie in the 9th, 3-2

That 9th inning took over an hour, and the same team tied it up and took the lead multiple times? I investigated my logic repeatedly, and for the life of me I could not figure out how this could be happening.

And then, I received an email from Mailgun with a link to a [post explaining what happened](http://blog.mailgun.com/post/what-happened-yesterday-and-what-we-are-doing-about-it/). I was satisfied with the explanation and was impressed and pleased that they got right out in front of it without any denials or excuses. So I'll continue to happily use Mailgun to deliver interesting events in baseball games to myself ... unfortunately for them I don't really have to think about it any more, now that it's running smoothly.
