---
layout: post
status: publish
published: true
title: Response.Flush and the crimes against Page Caching
author: noah
author_login: noah
author_email: noah@statichippo.com
wordpress_id: 203
wordpress_url: http://statichippo.com/blog/archive/2010/02/03/response.flush-and-the-crimes-against-page-caching.aspx
date: 2010-02-03 18:56:08.000000000 -05:00
categories:
- Uncategorized
tags: []
comments: []
---

**Update**: FIX posted [here](http://blog.statichippo.com/archive/2010/03/01/findreplace-on-render.aspx)
  
In a previous post ([http://blog.statichippo.com/archive/2009/11/16/findreplace-on-render-ndash-an-mvc-alternative-to-response-filters.aspx](http://blog.statichippo.com/archive/2009/11/16/findreplace-on-render-ndash-an-mvc-alternative-to-response-filters.aspx "http://statichippo.com/archive/2009/11/16/findreplace-on-render-ndash-an-mvc-alternative-to-response-filters.aspx")), I wrote about how to render a ViewPage to a string before writing it to the OutputStream.  This would be helpful, for instance, to replace certain text with other text (in my case, I used this to replace special tags with other content).  The method used basically worked like this:
     1. Add MemoryStream as a ResponseFilter 
    2. Render output 
    3. Flush the Response so that now the text is in the MemoryStream 
    4. Use a StreamReader to ReadToEnd 
    5. Do the transformations and Write to the Response stream 
   
As it turns out this effectively disabled Page Caching.  We tracked it down to step 3 – Response.Flush.  To me this call looked innocuous, but that’s not how it looks to IIS.  I posted the issue to StackOverflow ([http://stackoverflow.com/questions/2193628/response-flush-breaking-page-caching](http://stackoverflow.com/questions/2193628/response-flush-breaking-page-caching "http://stackoverflow.com/questions/2193628/response-flush-breaking-page-caching")) and user [Womp](http://stackoverflow.com/users/63756/womp) referred me to [this blog post at IIS.NET](http://blogs.iis.net/ksingla/archive/2006/11/16/caching-in-iis7.aspx).
  >    
If some module already flushed the response by the time request reaches UpdateRequestCache stage or if headers are suppressed, response is not cached in output cache module.
   
That sort of makes sense though, since caching the response requires the entire response to be available.
  
I have a solution I’m working on, and I plan to write about it soon.  In the meantime, don’t flush your response if you need page caching.
