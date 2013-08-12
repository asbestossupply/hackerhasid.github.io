---
layout: post
status: publish
published: true
title: If the property's computed, tell me!
author: noah
author_login: noah
author_email: noah@hackerhasid.com
date: 2011-03-08 21:53:33.000000000 -05:00
categories:
- C#
tags: []
comments: []
---

Today’s lesson learned: if a property may or may not be computed, provide a mechanism that allows consumers to check if it is computed.  Seems pretty simple when you hear it but we don’t always design like this.  And I think we probably should most of the time.  I came to this conclusion based on the following problem:
  
My project has a lot of similarities to a shopping cart.  Imagine a product catalog with both a Product and a SubProduct (only one level deep).  Both a Product and SubProduct have a price but price is only required on a Product, not a SubProduct.  If a price is _null_ on a SubProduct, it just inherits its parent Product’s price.  And if the Product price is updated, the SubProduct’s price automatically takes note:

{% highlight csharp %}
class Product
{
    public Price { get; set;}
}
class SubProduct
{
    public Product Product { get; set; }
    protected decimal? _price { get; set; }
    public decimal Price 
    {
        get
        {
            if (_price.HasValue) {
                return _price.Value;
            } else {
                return Product.Price;
            }
        }
        set 
        {
            _price = value;
        }
    }
}

{% endhighlight %}

Now that works well on the frontend of the site.  But when building the administration panel we ran into a problem.  If you want to edit the SubProduct, we autopopulate all the fields as you might expect.  But suppose the Product’s price was $500: if the SubProduct’s __price_ was _null_, we’ll still present the user with a $500 price to edit.  And even if they leave it as is, we’ll save the SubProduct’s __price_ as 500.  And now the SubProduct’s price won’t reflect the current Product’s price.  That’s not good!


I didn’t want to change the behavior of _Price_ though, it’s extremely useful.  And I certainly didn’t want to change the visibility of __price_!  So my first reaction was to start thinking up patterns for wrapping the SubProduct within the admin panel’s assembly in order to allow for properly editing it.  But then I realized I was overthinking the problem.  The problem is not that SubProduct’s __price_ is hidden.  And it’s not that _Price _is computed.  It’s that _Price_ _may or may not be computed_.  More specifically, the problem is that there was no indication whether _Price_ was computed or not.


So I added an _IsPriceInherited_ property that just checks this:

{% highlight csharp %}
public bool IsPriceInherited
{
    get
    {
        return !_price.HasValue;
    }
}
{% endhighlight %}


and then I could easily account for this in the admin panel code.  And after thinking about it, I was struck that informing consumers about computation is just the _polite_ thing to do.  OOP is a great paradigm for _protecting_ code through visibility modifiers, but just because code is safe from outside influence doesn’t mean it can’t be polite.  


And so I learned my lesson, say it with me now, “if a property may or may not be computed, provide a mechanism that allows consumers to check if it is computed.”  Isn’t OOD supposed to mimic real world objects and relationships?  Well, this rule mimics real-world relationship rules: communication is key but that doesn’t mean you should expose your privates.
