---
layout: post
status: publish
published: true
title: BitlyDotNet just got more awesome
author: noah
author_login: noah
author_email: noah@statichippo.com
wordpress_id: 206
wordpress_url: http://statichippo.com/blog/archive/2010/03/11/bitlydotnet-just-got-more-awesome.aspx
date: 2010-03-11 19:17:07.000000000 -05:00
categories:
- C#
- Tooling
tags: []
comments: []
---

[Mike Gleason Jr](http://blog.mikecouturier.com/) has a bit.ly client library written in C# called BitlyDotNet (it’s hosted on [here on Google Code](http://code.google.com/p/bitly-dot-net/)).  If it wasn’t cool enough before, it just got more awesome.
  
I submitted a new version with the following improvements:
     1. Added support for error code 208 ("you have exceeded your hourly rate limit for this method")
    2. Overload to shorten multiple URLs in one shot
    3. Fixed bug that occurred when hourly limit was exceeded (SingleOrDefault throws InvalidOperationException because of multiple errorCode nodes)
   
This library saved me time on a recent project and I’m glad I could contribute.  [Go check it out!](http://code.google.com/p/bitly-dot-net/)
