---
layout: post
title: F#, System.Linq extensions and the case of the missing null
categories:
- Entity Framework
- F#
- .NET
---

I posted a question to StackOverflow yesterday ([http://stackoverflow.com/questions/6847203/ef4-1-and-f-how-to-deal-with-values](http://stackoverflow.com/questions/6847203/ef4-1-and-f-how-to-deal-with-values)) and will describe the issue and resolution in this post.
<!--more-->

F# is a funny language in that it is a functional (I suppose it’s multi-paradigm but very strong emphasis on functional) language built on the .NET platform which has a very different style as a whole.  I had an interesting issue the other day where I noticed a _null_ value at runtime being returned from a .NET component in a place that F# was certain _null_ values could not exist. 
  
In my case this was Entity Framework.  I followed [this walkthrough](http://blogs.msdn.com/b/jackhu/archive/2011/04/01/f-code-first-development-with-entity-framework-4-1.aspx) to create F# types that mapped to my database using Entity Framework Code First.  I wrote a query that ended in .FirstOrDefault() and here is where the problem started:
  
{% highlight fsharp %}
let result = context
    .SearchResults
    .Where((fun (r:SearchResult) -> r.Program = request.Program))
    .OrderByDescending((fun r -> r.AcquisitionDate))
    .FirstOrDefault()
{% endhighlight %}



Now .FirstOrDefault() is implemented such that it returns either the first element if one exists, or default(T) if not.  Well my _SearchResult_ type is defined in F# and is just a regular class so the _default(typeof(SearchResult))_ is equal to _null_.  However, F# doesn’t allow nulls be default – due to its functional nature it doesn’t by default support _null _values (which I’ve come to really appreciate when I’m writing ‘pure’ F#).  So when the database has no data, what value do you think is bound to _result_?


If you guessed_ null_ you were right.  But I just said that F# doesn’t support _null_.  So the following code does not compile:

{% highlight fsharp %}
match result with
    | null -> () // do nothing since this doesn't compile anyway
    | price -> () // do nothing since this doesn't compile anyway
{% endhighlight %}

the compilation error states

> the type 'SearchResult' does not have 'null' as a proper value


Interesting.  So SearchResult doesn’t support _null_.  Even more interesting is if you comment out the bad code and run it you’ll see that _result_ which is of type _SearchResult_ is indeed bound to _null_.


It seems to me that in a world of uncertainty – a world of mixed paradigms -- F# assumes a convention.  F# assumes that every component is as functionally pure as it is – though it is made to interoperate with components that are known not to be.


It turns out that the fix to this is pretty simple.  As stated in the first paragraph in the [MSDN article on nulls in F#](http://msdn.microsoft.com/en-us/library/dd233197.aspx), F# has an AllowNullLiteral attribute.  I decorated my EF types with this and then the compiler allowed me to compare my _results_ value to _null_.  How imperative!
