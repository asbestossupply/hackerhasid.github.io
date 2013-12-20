---
layout: post
status: publish
published: true
title: LINQ to SQL Entities and DDD
author: noah
author_login: noah
author_email: noah@statichippo.com
wordpress_id: 188
wordpress_url: http://statichippo.com/blog/archive/2009/11/08/linq-to-sql-entities-and-ddd.aspx
date: 2009-11-09 03:06:36.000000000 -05:00
categories:
- ASP.NET
tags: []
comments: []
---

 
  
We’re using LINQ-to-SQL (L2S from here on in) for a new project.  L2S has a number of drawbacks, and those are especially underscored when working with a very old and severely ‘lagacy’ database.
  
L2S has a 1:1 relationship between tables and objects and a 1:1 relationship between columns and properties.  That doesn’t work very well if your database doesn’t map to the business objects you wish you had ;).  L2S also has a very tight relationship with your business entities which will make any sort of move away from L2S near impossible the larger your app gets.
  
But requirements are requirements and ours use L2S, so we had to make do.  However, I feared for the life of our project when I noticed L2S objects were scattered all over our BLL.  Since these L2S entities mimic our database 1:1, they are extremely far removed from the business logic we want to implement and there were methods being created to do get information used to bridge gaps and weird stuff like that.
  
I tried out a solution and I’m glad to say it worked (meaning the CTO wants to remove all references to other implementations – I’m extremely happy) -- that’s why I’m finally publishing this post.
  
Here’s how it works:
  
* I created an interfaces that mimics our Domain – not our database.
* I created wrappers for all the L2S entities (sometimes more than 1 wrapper, I’ll get into that in a bit) that wrap the logic required to rewire the L2S entities (basically the DB schema) to the interfaces.
* I created Repositories that abstract away the L2S DataContexts

Now in order to change object relational mappers (say, if we want to use NHibernate in the future), we will only need to create new objects that implement the interfaces and new repositories to access the data.  Yay!

# Multiple Wrappers
  
The reason for the multiple wrappers is that sometimes you have an object that has some self references  (for instance, a category might have a parent category with a parent category etc).   Eg:

{% highlight csharp %}
public class Category
{
  public string< Name { get; }
  public Category Category { get; }
}
{% endhighlight %}

In these cases, you might not need all the information on the parent object, but with the way L2S works you’d be forced to do a full SELECT statement (meaning it includes all the columns) for the parent and parent’s parent etc. and there might be lots of waste there.  


So if you have a table (A) you could have 2 L2S entities, one called FullA with all the columns from A and one called ParentA with just a few columns from the A table.  By creating a relationship between FullA and ParentA and a self reference between ParentA and itself, you bypass this issue.  Then you would have a FullA wrapper which fully implements the interface and a ParentA wrapper which does not.  But since both FullA and ParentA implement IAInterface, you leave yourself open at a later date for using a different (better) Object Relational Mapper.
