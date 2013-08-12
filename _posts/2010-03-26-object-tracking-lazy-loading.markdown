---
layout: post
status: publish
published: true
title: Object tracking != Lazy Loading!
author: noah
author_login: noah
author_email: noah@statichippo.com
wordpress_id: 207
wordpress_url: http://statichippo.com/blog/archive/2010/03/26/object-tracking-lazy-loading.aspx
date: 2010-03-26 17:36:42.000000000 -04:00
categories:
- C#
- Linq
tags: []
comments: []
---

LINQ-to-SQL DataContexts have a boolean property called ObjectTracking which is used to determine whether the framework should exert the overhead necessary to remember previous states.  This defaults to **true** and is used to determine the SQL UPDATE command to run when persisting the object.
  
We have a project with data that’s read-only and so we decided to turn ObjectTracking off (set to **false**).  We figured this was a good idea since, the thinking went, it would save overhead and since we’re not persisting the data anyway (read-only) we’ll have nothing to worry about.  Boy were we wrong.
  
All of a sudden things started breaking!  Some thing.  Not others.  Wierd.  We figured out that it had to do with relationships not being loaded – for instance, if we have **Person** objects that have a collection of **Car** objects, the collection would be empty.
  
Well, apparently this is to be expected.  The MSDN article on [DataContext.ObjectTrackingEnabled](http://msdn.microsoft.com/en-us/library/system.data.linq.datacontext.objecttrackingenabled.aspx) states:
  >    
If ObjectTrackingEnabled is false, [DeferredLoadingEnabled](http://msdn.microsoft.com/en-us/library/system.data.linq.datacontext.deferredloadingenabled.aspx) is ignored and treated as false. In this case, the [DataContext](http://msdn.microsoft.com/en-us/library/system.data.linq.datacontext.aspx) is read-only.
   
(Interestingly enough, the next line states: _“If ObjectTrackingEnabled is true, _[_DeferredLoadingEnabled_](http://msdn.microsoft.com/en-us/library/system.data.linq.datacontext.deferredloadingenabled.aspx)_ is false. In this case, _[_DataContext_](http://msdn.microsoft.com/en-us/library/system.data.linq.datacontext.aspx)_ allows you to load an object graph by using _[_LoadWith_](http://msdn.microsoft.com/en-us/library/system.data.linq.dataloadoptions.loadwith.aspx)_ directives, but does not enable deferred loading.” _which clearly is not the case because it would mean there’s no difference in regards to DeferredLoadingEnabled whether ObjectTrackingEnabled is true or false!  What they **meant** to say was _“If ObjectTrackingEnabled is true **AND** DeferredLoadingEnabled is false…”_)
  
So, I looked at the DeferredLoadingEnabled article and sure enough:
  >    
When the code accesses one of these relationships, null is returned if the relationship is one-to-one, and an empty collection is returned if it is one-to-many. The relationships can still be filled by setting the[LoadOptions](http://msdn.microsoft.com/en-us/library/system.data.linq.datacontext.loadoptions.aspx) property.
   
(In fact, this article clears up the ObjectTrackingEnabled issue above by stating “_[ObjectTrackingEnabled and DeferredLoadingEnalbed are][b]oth are set to true. This is the default._”
  
I guess we didn’t expect that because ObjectTracking and Lazy Loading are two very different things.  Just because I don’t want to track my object does **not **mean I don’t want to enable lazy loading!  This is just one of many quirks in Linq-to-SQL.
