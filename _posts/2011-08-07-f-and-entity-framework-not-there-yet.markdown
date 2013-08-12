---
layout: post
status: publish
published: true
title: F# and Entity Framework, not there yet
author: noah
author_login: noah
author_email: noah@hackerhasid.com
date: 2011-08-07 04:51:47.000000000 -04:00
categories:
- Uncategorized
tags: []
comments: []
---

Based on [this article](http://blogs.msdn.com/b/visualstudio/archive/2011/04/04/f-code-first-development-with-entity-framework-4-1.aspx) on MSDN I decided to code the data layer for an app in F#.  After a few weeks of hobby time coding, I don’t believe that EF via F# is a viable solution yet.  Here’s why:
  
_NOTE: Windows Live Writer is crashing constantly on me.  I’ve got 2 reasons so far and may update this post with more when Live Writer starts behaving (going to restart at some point this week I suppose) or I find a more stable editor._
  #### 1. No _protected_ keyword
  
Because the Entity Framework’s proxy classes inherit from their base classes (and because they do not support setting private properties of their ancestors), the _protected_ keyword becomes especially important.  _Protected_ becomes a way of creating properties that can be set only by the class itself and by Entity Framework’s proxy classes.  I’m especially fond of this in constructors.  Take the following code snippet:

{% highlight csharp %}
public class Person
{
    public string FirstName { get; set; }
    public string LastName { get; set; }
    public void Person(string firstName, string lastName)
    {
      FirstName = firstName;
        LastName = lastName;
    }
    protected void Person() {}
}
{% endhighlight %}

This requires client code to specify a first and last name for a _Person_ while still allowing Entity Framework to construct a new _Person_ object via its default constructor (a requirement) before setting the properties.  This is also really useful for lists and such – by making them _protected_ you can ensure that they are never null and wrap list inserts inside a method like so:

{% highlight csharp %}
...
protected List<string> people {get; set;}
void AddPerson(string name)
{
    // business logic around the name
    people.Add(name);
}
...
{% endhighlight %}

Since F# doesn’t support the _protected_ keyword, there’s no alternative to generating a public default constructor and exposing your lists getter and setter for all.

#### 2. Awkward _null_ support

When writing functional code, the lack of implicit support for _null_ values is very useful.  However, Entity Framework is used to interact with a database – somewhere where _nulls_ are commonplace and very useful.  I wrote a blog post about one issue with this ([F#, System.Linq extensions and the case of the missing null](http://statichippo.com/archive/2011/07/28/f-system-linq-extensions-and-the-case-of-the-missing.aspx)) but there are others including lack of support for Nullable (in case you think you’re unfamiliar with Nullable, you’re probably not.  You may never have written _Nullable_ in your code -- the ? in C# is syntactic sugar around it.  So if you’ve ever written _int? _in C#, you’ve used Nullable).  F# has it’s own way of achieving something similar to Nullable called Option but it’s based on F# Discriminated Unions and is not supported by Entity Framework.  


The biggest use case for Nullable in Entity Framework in my opinion is with DateTime.  DateTime is a value type and cannot be null.  But null datetime values in the database are a frequent occurrence usually meant to signify that something has never taken place.  Take the following code:

{% highlight csharp %}
public class Email
{
    ...
    public DateTime? DateSent {get; set;}
    ...
}
{% endhighlight %}


In this case, the _Email_ class defines a _DateSent_ property that presumably will be _null_ until the email is actually sent.  This makes a lot of sense in the database and even in C# but doesn’t work out so well in F#.  First of all there’s the syntactic sugar in C#.  Consider the same code written in F#:

{% highlight fsharp %}
open System
type Email =
    ...
    let mutable m_dateSent = new Nullable<DateTime>()
    member this.DateSent with get () = m_dateSent 
                            and set v = m_dateSent <- v
    ...
{% endhighlight %}


Aside from the verbosity, check out this snippet that _doesn’t compile_ (if you’re not familiar with the syntax, I’m using the [F# PowerPack Linq extensions](http://blogs.msdn.com/b/dsyme/archive/2009/10/23/a-quick-refresh-on-query-support-in-the-f-power-pack.aspx)):

{% highlight fsharp %}
let unsentEmails = query <@ seq { for e in context.Emails do if e.EmailSent = null then yield e } @> 
{% endhighlight %}


Can you guess why that won’t compile?  Line 2 checks EmailSent, which is a Nullable against _null_ which is not permitted a _null_ value.  You can get around this by changing that line to 

{% highlight fsharp %}
if e.EmailSent.HasValue = false then
{% endhighlight %}

However, the outputted SQL (using the default, MSSQL connector) from this line is far worse than the equivalent in C# (using the “= null”).  In C#, the expected SQL is outputted – including a _where_ clause that checks for the column against _null _which allows the use of an index in MSSQL (although not for mySQL which is another conversation altogether!).  The F# code, checking the _HasValue_ property of the Nullable,  generates some awful SQL in the _where_ clause using CASE and CAST seemingly just to annoy by preventing the use of indexes:

{% highlight fsharp %}
WHERE 0 = 
    (CASE 
        WHEN ([Extent1].[EmailSent] IS NOT NULL) THEN cast(1 as bit)
        WHEN ([Extent1].[EmailSent] IS NULL) THEN cast(0 as bit)
    END)
{% endhighlight %}