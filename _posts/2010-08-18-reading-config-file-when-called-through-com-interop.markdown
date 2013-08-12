---
layout: post
status: publish
published: true
title: Reading config file when called through COM interop
author: noah
author_login: noah
author_email: noah@statichippo.com
wordpress_id: 215
wordpress_url: http://statichippo.com/blog/archive/2010/08/18/Reading-config-file-when-called-through-COM-interop.aspx
date: 2010-08-18 21:25:44.000000000 -04:00
categories:
- C#
tags: []
comments: []
---

Seems obvious in retrospect, but gave me some trouble yesterday.  I’m using hMailServer (a nifty, open source SMTP server) which supports scripting via VBScript event handlers.  Now I don’t want to code in VBScript so I wrote a .NET library and was using some COM interop to pass everything through.  However, I wanted my library to read configuration values via the ConfigurationManager class and didn’t know what config file it was looking for!
  
In a .NET app you typically have an App.Config that gets copied to a new name on build – YourApp.exe.config.  I was building a library though, not an executable, and my library was being called through COM, so what should the config file be named?
  
Well after some research (running [procmon](http://technet.microsoft.com/en-us/sysinternals/bb896645.aspx) – man that’s a useful utility!) I found that the config file that ConfigurationManager was looking for used the same naming conventions as if it was called from a .NET app: “calling-executable.exe.config” – in my case “hMailServer.exe.config” in the \hMailServer\bin directory.
  
Good to know!
