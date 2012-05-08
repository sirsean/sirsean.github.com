---
layout: post
title: Announcing Tiny VH
tags: []
status: publish
type: post
published: true
meta:
  _edit_last: '2'
author: sirsean
---
Yesterday is.gd went down (briefly, I guess). I'd been using it to shorten my links, and it was pretty frustrating that it went down. And I've read that some of these URL shortening services are planning to try to monetize their usage. Possibly with interstitial ads. I don't know about you, but I'd find that pretty infuriating.

Therefore, I am here to announce the launch of TinyVH.com, which is a new URL shortening service that I wrote. It's hosted at Dreamhost, and will go down whenever they do. Hopefully they move me to their new cluster soon so I can get the improved speed and reliability. Either way, it's a simple program with minimal demands, and I have no reason to try to monetize it or use the data. I just want a URL shortening service I can trust.

And I don't trust anything more than I trust myself.

I'm already using it for a Wordpress plugin I've written that posts a tweet whenever someone comments on your blog. I'm sure I'll come up with more uses for it. So help yourself.

If you also want to use it automatically, you can make a simple call like this one:

http://tinyvh.com/api.php?url=http://vikinghammer.com

And it'll return a string, a URL that looks like <a href="http://tinyvh.com/8s">http://tinyvh.com/8s</a> ... and then when you visit that link it'll immediately redirect to <a href="http://vikinghammer.com">http://vikinghammer.com</a>. Or whatever URL you happened to enter.

Have fun.
