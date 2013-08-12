---
layout: post
status: publish
published: true
title: MVC RenderAction or RenderPartial graceful exception handling
author: noah
author_login: noah
author_email: noah@statichippo.com
wordpress_id: 187
wordpress_url: http://statichippo.com/blog/archive/2009/11/08/mvc-renderaction-or-renderpartial-graceful-exception-handling.aspx
date: 2009-11-09 02:20:33.000000000 -05:00
categories:
- ASP.NET
tags: []
comments: []
---

 
  
We recently had a little dilemma that I’m sure others have had.  We have a lot of areas of our pages that are rendered using RenderPartial or RenderAction (part of [MVC Futures](http://aspnet.codeplex.com/Release/ProjectReleases.aspx?ReleaseId=24471)) and we want graceful handling of exceptions.  Of course exceptions shouldn’t happen, but sometimes they do (if only everyone practiced TDD…), and with 1M hits a day we would like to be able to at least show as much content (and ads ;) as we can – even if something does go wrong.
  ## ViewEngine &amp; IView
  
So I was approached about this issue and my solution was very simple:
  
We created a custom ViewEngine that returns a custom IView (just inherits from the default WebFormView) that wraps its base.Render() call in a try/catch.  
  
Very simple and works like a charm!
