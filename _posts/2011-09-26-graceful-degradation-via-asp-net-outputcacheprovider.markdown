---
layout: post
status: publish
published: true
title: Graceful degradation via ASP.NET OutputCacheProvider
author: noah
author_login: noah
author_email: noah@hackerhasid.com
date: 2011-09-26 00:03:27.000000000 -04:00
categories:
- ASP.NET
tags: []
comments: []
---

Like everyone, we never want our site to go down.  Some pages display data and if something happens to the database we’d like at the very least to display old data instead of an error.  In that vain I wrote a proof of concept that might help.  This is not live and may never go live but it tests well and I thought it was interesting enough to post.
  
The implementation is based on two interconnected pieces – an OutputCacheProvider and an HttpModule.  The OutputCacheProvider, like the built in output cache provider, is responsible for storing and retrieving cached pages.  In addition, however, it keeps cached pages around past their expiration in case the need arises to render them.  The HttpModule’s responsibility is to handle exceptions and by writing out cached versions of the page regardless of their age.
  
The sourcecode is available at [on github](https://github.com/hackerhasid/Graceful-Degradation-Cache-Provider) but I’ll step through the initial check version of the code here to better explain what’s going on.
  ### GracefulDegradationOutputCacheProvider
  
This is just a standard OutputCacheProvider (OutputCacheProviders were introduced with ASP.NET 4.0.  Check out [http://weblogs.asp.net/gunnarpeipman/archive/2009/11/19/asp-net-4-0-writing-custom-output-cache-providers.aspx](http://weblogs.asp.net/gunnarpeipman/archive/2009/11/19/asp-net-4-0-writing-custom-output-cache-providers.aspx) for more info).  The interesting piece to note though is that the standard Get method implemented for OutputCacheProvider just calls another Get method that takes a boolean parameter called _respectExpiration_.  The “standard” Get method just passes in true for this.  Here’s the code:

{% highlight csharp %}
    /// <summary>
    /// Gets an item from the cache by key
    /// </summary>
    /// <param name="key">Key to find</param
    /// <param name="respectExpiration">If true, don't return stale results.  This parameter should be true in regular caching cases and false if we're retrieving from cache when an exception occurred </param>
    /// <returns></returns>
    internal object Get(string key, bool respectExpiration)
    {
        Debug.WriteLine("Cache.Get(" + key + ")");

        CacheItem existing;
        if (!_items.TryGetValue(key, out existing))
            return null;

        if (!respectExpiration)
            return existing.Item;

        if (existing.Expires > DateTime.UtcNow)
            return existing.Item;

        return null;
    }
{% endhighlight %}

as you can see, the _respectExpiration_ flag is used to override the cache expiration and return the value anyway.  This is used by the:

### GracefulDegradationCacheModule


this just catches the HttpApplications’s Error event and attempts to write the cached output out to the response stream.  There are a couple pieces of interest inside the handler:

##### Finding the provider


The handler starts off by finding the GracefulDegradationOutputCacheProvider among the available CacheProviders for the application:

{% highlight csharp %}
    // Find the GracefulDegredationOutputCacheProvider
    foreach (var provider in OutputCache.Providers)
    {
        var inMemoryProvider = provider as GracefulDegradationOutputCacheProvider;
        if (inMemoryProvider == null)
            continue;
{% endhighlight %}

the reason this provider isn’t cached in some field is because this can be modified programmatically and so I just loop every time and attempt to find the provider.

##### Writing the cache out to the response


At first I was using a roundabout method to determine an error threshold with a timer firing and clearing error counts – it was pretty awkward.  When I mentioned it to my peers, Joe ([http://twitter.com/#!/joecianflone](http://twitter.com/#!/joecianflone)) really felt the pain and [BillRob](http://billrob.com/) came up with the much simpler approach of just writing the response out from the cache on error.  Duh!  With a few helpful hints from Bill, here’s what I came up with:

{% highlight csharp %}
    var context = HttpContext.Current;
    var response = context.Response;
    // clear response
    response.Clear();

    // we don't want anyone caching this old response, right?
    response.CacheControl = "no-cache";

    var responseBytes = GetResponseBytes(cacheItem, context);
    // write out response body
    // you can also append some content if you wish,
    // perhaps a floating div that tells the user
    // the page they're seeing is old
    responseBytes.ForEach(x => 
    {
        response.OutputStream.Write(x, 0, x.Length);
    });

    // Don't want to show the error page (though you
    // probably want to alert someone!)
    context.ClearError();
{% endhighlight %}

notice that we explicitly tell clients not to cache this!  Also, as mentioned in the comments, I could see adding some hook here to place some floating div somewhere to let people know what they’re seeing may be stale.  Also, the error is cleared so you probably want to log it somewhere – just because your end user doesn’t see an error doesn’t mean you don’t want to know it happened!
