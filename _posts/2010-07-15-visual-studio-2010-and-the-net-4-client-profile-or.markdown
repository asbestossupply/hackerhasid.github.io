---
layout: post
status: publish
published: true
title: Visual Studio 2010 and the .NET 4 Client Profile (or issues with class libraries
  in VS2010)
author: noah
author_login: noah
author_email: noah@statichippo.com
wordpress_id: 214
wordpress_url: http://statichippo.com/blog/archive/2010/07/15/Visual-Studio-2010-and-the-NET-4-Client-Profile-or.aspx
date: 2010-07-15 18:04:02.000000000 -04:00
categories:
- Uncategorized
tags: []
comments: []
---

If you create a new class library in VS2010 using .NET 4, the default target framework is a new package called .NET 4 Client Profile which is a subset of the full .NET 4 (more info at [http://msdn.microsoft.com/en-us/library/cc656912.aspx](http://msdn.microsoft.com/en-us/library/cc656912.aspx)).  The Client Profile is useful for for streamlining client application installations, but it causes some odd behavior in Visual Studio 2010 with regards to referenced assemblies.  
  
Basically, if you reference another assembly that makes use of references outside the .NET 4 Client Profile, you’ll get full intellisense before you build.  However, your build will fail.  And afterwards, intellisense will disappear for those unsupported assemblies and the IDE will mark them with squiggly lines indicating that they’re unknown classes.  At first it seems as though the dll was deleted or something, but it wasn’t.  It’s just that the default target framework for Class Libraries in Visual Studio 2010 is the .NET 4 Client Profile which doesn’t support something you’re doing.  I assume that the reason for the weird behavior has something to do with the build-as-you-type functionality of VS2010 not paying full attention to this.
  
Anyway, the solution to this is just to go to your project settings (right click on project and select properties) and under the Application tab (top tab) like so:
  
[![Capture](http://blog.statichippo.com/images/statichippo_com/WindowsLiveWriter/Vis.NET4ClientProfileorissueswithclassli_C62C/Capture_thumb.png "Capture")](http://blog.statichippo.com/images/statichippo_com/WindowsLiveWriter/Vis.NET4ClientProfileorissueswithclassli_C62C/Capture.png)
