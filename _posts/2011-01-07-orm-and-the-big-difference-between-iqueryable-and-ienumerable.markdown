---
layout: post
status: publish
published: true
title: ORM and the big difference between IQueryable and IEnumerable
author: noah
author_login: noah
author_email: noah@statichippo.com
wordpress_id: 219
wordpress_url: http://statichippo.com/blog/archive/2011/01/07/ORM-and-the-big-difference-between-IQueryable-and-IEnumerable.aspx
date: 2011-01-07 17:47:20.000000000 -05:00
categories:
- Entity Framework
tags: []
comments:
- id: 140
  author: Bill Robertson
  author_email: theman@fdrsucks.com
  author_url: http://billrob.com/
  date: '2011-01-12 22:00:38 -0500'
  date_gmt: '2011-01-12 22:00:38 -0500'
  content: Does that mean when the extension methods fire, it basically calls type.BaseType
    over and over again until it finds a match?
---

There’s somewhat of a pattern in the Linq supported ORM world to create extension methods for filtering.  I’m working on a project right now that uses this pattern.  If you’re not familiar with it, the pattern basically works by creating extension methods for _IQueryable_ that return _IQueryable_ like this_:_
  <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">   <div style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet">     <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: white; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"><span style="color: #606060" id="lnum1">   1:</span> <span style="color: #0000ff">public</span> <span style="color: #0000ff">static</span> IQueryable<order> OrderWasShipped(<span style="color: #0000ff">this</span> IQueryable<order> orders)</order></order></pre></div></div>
<!--CRLF-->

    <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"><span style="color: #606060" id="lnum2">   2:</span> {</pre>
<!--CRLF-->

    <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: white; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"><span style="color: #606060" id="lnum3">   3:</span>     <span style="color: #0000ff">return</span> orders.Where(o => o.Status == OrderStatus.Shipped);</pre>
<!--CRLF-->

    <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"><span style="color: #606060" id="lnum4">   4:</span> }</pre>
<!--CRLF-->



You can now define queries like _Orders.OrderWasShipped()_ which will get translated into SQL and all is well and good.


However, there’s an enormous caveat here that some may not be aware of.  An _IQueryable_ is an _IEnumerable_ and so you may be tempted to change the signature of the methods to take and return an _IEnumerable_.  This way you can pass any type of collection in there to filter it_._


**Don’t**


The main difference between _IEnumerable_ and _IQueryable_ is that, while _IQueryable_ extends _IEnumerable_, it also stores the _Expression_s (not just _Func_s).  Whereas a _Func_ is just a delegate, an _Expression_ is, well, an expression that is able to be traversed at runtime.  And that means it can be translated into SQL.  Because of this difference, the extension methods differ too.  The _.Where_ extension method for an _IEnumerable_ takes a _Func<T, bool>_ whereas the _.Where_ extension method for an _IQueryable_ takes an _Expression<Func<T, bool>>_.  


What this means for you is that if you change your filter extension method to take an _IEnumerable_, you will be using the _IEnumerable_’s _.Where _extension method (since you’ve implicitly converted your _IQueryable_ to an _IEnumerable_), passing in a compiled delegate instead of a traversable expression (they both can be inferred from the same code, in the above case_ o => o.Status == OrderStatus.Shipped_, how it gets compiled depends on which method you call).  And if you do that, your ORM will have no choice but to select **all **of your entities (massive DB hit) and then filter them in memory because it has no idea how to convert that _Func_, that compiled code, into SQL.
