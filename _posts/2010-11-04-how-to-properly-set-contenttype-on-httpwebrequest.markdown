---
layout: post
status: publish
published: true
title: How to properly set ContentType on HttpWebRequest
author: noah
author_login: noah
author_email: noah@statichippo.com
wordpress_id: 217
wordpress_url: http://statichippo.com/blog/archive/2010/11/03/How-to-properly-set-ContentType-on-HttpWebRequest.aspx
date: 2010-11-04 03:01:05.000000000 -04:00
categories:
- C#
tags: []
comments: []
---

I spent hours today debugging an issue where a call to a finicky web service API.  All my hand-built calls in [Fiddler](http://www.fiddler2.com/fiddler2/) worked fine, but my code did not.  I realized I needed to inspect the headers of the outgoing web request to determine the cause and I stumbled upon [this post by Mark Needham](http://www.markhneedham.com/blog/2009/06/24/using-fiddler-with-iis/) about how to configure IIS so that web requests route through Fiddler.
  
The issue turned out to be that the Content-Type header was not being set on the POST to the web service.  Weird, since this line was present:
  >    
request.ContentType = options.ContentType;
   
However, it turns out that the order here is important.  You see, this line was executing after writing the POST data to the request’s RequestStream.  Setting the ContentType property before writing to the RequestStream solved the problem.
