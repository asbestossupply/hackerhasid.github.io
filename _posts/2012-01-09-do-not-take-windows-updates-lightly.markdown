---
layout: post
status: publish
published: true
title: Do not take Windows Updates lightly
author: noah
author_login: noah
author_email: noah@hackerhasid.com
date: 2012-01-09 04:39:05.000000000 -05:00
categories:
- Windows
- ASP.NET
- .NET
tags: []
comments: []
---

Recently we experienced an odd issue; on an intermittent basis authenticated users would see a non-authenticated view of the page they were on.  A little debugging revealed that each server behaved well in isolation but when the load balancer was in the mix moving requests between servers things got wonky.  Using his mad curl skillz, @andrewmglenn determined that the authentication cookie set from one of our servers was not playing nicely with the others.  This turned out to be a two way street – one server didn’t like the cookies set from the other servers and the others didn’t like the cookie set from this one.
  
It turned out this one server had a security update installed not long before we experienced the issue.  The link in Windows Update points to [http://support.microsoft.com/kb/2656351](http://support.microsoft.com/kb/2656351) and mentions something about a “vulnerability that would allow an unauthenticated remote attacker to compromise your system”.  The update included updated versions of numerous important files including System.Web.dll and aspnet_wp.exe.  But it takes 2 clicks from the support url to actually get to a page with real information on the exploit and the solution taken in the security update.  Over at [http://technet.microsoft.com/en-us/security/bulletin/ms11-100](http://technet.microsoft.com/en-us/security/bulletin/ms11-100) towards the bottom of the page in the FAQ section there’s a mention of the workaround taken by the update: “The update addresses this vulnerability by correcting how the ASP.NET Framework authenticates users.”
  
So it seems that this server is generating and verifying authentication cookie values using a new algorithm!
  
The moral of the story is clear and is an important lesson for small shops: do not take Windows Updates lightly.
