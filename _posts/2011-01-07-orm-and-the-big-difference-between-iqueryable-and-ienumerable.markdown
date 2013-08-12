---
layout: post
status: publish
published: true
title: ORM and the big difference between IQueryable and IEnumerable
author: noah
author_login: noah
author_email: noah@hackerhasid.com
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

There’s somewhat of a pattern in the Linq supported ORM world to create extension methods for filtering.  I’m working on a project right now that uses this pattern.  If you’re not familiar with it, the pattern basically works by creating extension methods for `IQueryable` that return `IQueryable` like this:

{% highlight csharp %}
public static IQueryable OrderWasShipped(this IQueryable<Order> orders)
{
  return orders.Where(o => o.Status == OrderStatus.Shipped);
}
{% endhighlight %}

You can now define queries like `Orders.OrderWasShipped()` which will get translated into SQL and all is well and good.


However, there’s an enormous caveat here that some may not be aware of.  An `IQueryable` is an `IEnumerable` and so you may be tempted to change the signature of the methods to take and return an `IEnumerable`.  This way you can pass any type of collection in there to filter it.


**Don’t**


The main difference between `IEnumerable` and `IQueryable` is that, while `IQueryable` extends `IEnumerable`, it also stores the `Expression`s (not just `Func`s).  Whereas a `Func` is just a delegate, an `Expression` is, well, an expression that is able to be traversed at runtime.  And that means it can be translated into SQL.  Because of this difference, the extension methods differ too.  The `.Where` extension method for an `IEnumerable` takes a `Func<T, bool>` whereas the `.Where` extension method for an `IQueryable` takes an `Expression<Func<T, bool>>`.  


What this means for you is that if you change your filter extension method to take an `IEnumerable`, you will be using the `IEnumerable`’s `.Where` extension method (since you’ve implicitly converted your `IQueryable` to an `IEnumerable`), passing in a compiled delegate instead of a traversable expression (they both can be inferred from the same code, in the above case `o => o.Status == OrderStatus.Shipped`, how it gets compiled depends on which method you call).  And if you do that, your ORM will have no choice but to select **all **of your entities (massive DB hit) and then filter them in memory because it has no idea how to convert that `Func`, that compiled code, into SQL.
