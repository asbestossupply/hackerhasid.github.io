---
layout: post
status: publish
published: true
title: EntityFramework should throw more exceptions!
date: 2010-05-28 16:05:50.000000000 -04:00
categories:
- NHibernate
- Entity Framework
tags: []
comments: []
---

I’m working on a new project using .NET4, MVC2 and EntityFramework.  For the last couple years I’ve been using NHibernate as my ORM of choice but there are a lot of similarities between NHibernate and EntityFramework actually so the learning curve is quite small.  There are a number of things that make me long for NHibernate, but one stands out in my mind: lazy loading.  One of my first issues with EF was trying to track down a bug: ClassA has a property reference to ClassB, but the property is always null even when ClassB exists in the database.  After debugging for what seems like hours, I tracked down the issue – the ClassB property on ClassA was not virtual.  This was never a problem with NHibernate because NHibernate threw an exception if the property was not virtual.

### Why does the property need to be virtual?
 
Suppose you have two tables in the database, TableA and TableB.  And suppose you map your classes, ClassA and ClassB to those tables.  For lazy loading to work there has to be some proxy – some class of type ClassA that has a property associated with ClassB that when it’s first accessed will do the actual query to get ClassB data.  The way to accomplish this is to create a class, let’s call it ClassAProxy, that inherits from ClassA and overrides the properties, something like this:

{% highlight csharp %}
public class ClassA
{
  public ClassB ReferenceToClassB { get; }
}

public class ClassAProxy : ClassA
{
  private bool _IsRetrievedClassB = false;
  private ClassB _ReferenceToClassB;
  public override ClassB ReferenceToClassB 
  {
    get
    {
      if (!_IsRetrievedClassB)
      {
        // RETRIEVE FROM DB & SET _ReferenceToClassB
        _IsRetrievedClassB = true
      }

      return _ReferenceToClassB;
    }
  }
}
{% endhighlight %}

Notice that ClassAProxy has an override on ClassB. That's to see if you're paying attention - ClassA does not define ReferenceToClassB as virtual, so this wouldn't compile. Change that one line (add `virtual`) and you get a working lazy loading proxy.

### The Takeaway
NHibernate throws an exception if you enable lazy loading but don't set your properties as virtual. Entity Framework just ignores this. That means that you won't know that a property isn't lazy loaded (and will always be null) until you access it. I can't imagine why someone would want this 'feature'!
