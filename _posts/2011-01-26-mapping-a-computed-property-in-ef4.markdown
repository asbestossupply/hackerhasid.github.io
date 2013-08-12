---
layout: post
status: publish
published: true
title: Mapping a computed property in EF4
author: noah
author_login: noah
author_email: noah@hackerhasid.com
date: 2011-01-26 04:40:39.000000000 -05:00
categories:
- C#
- Entity Framework
tags: []
comments: []
---

I posed a question a while ago on stackoverflow ([http://stackoverflow.com/questions/4618861/ef4-map-model-defined-function-to-property](http://stackoverflow.com/questions/4618861/ef4-map-model-defined-function-to-property)) about how to map a computed property in Entity Framework 4.  I didn’t get any answers so I figured I’d post what I came up with.

### The Problem
  
Just in case you’re too lazy to click the SO link and read the actual question, here’s the rundown.  Suppose you have a _Product_ and _Order _class defined like so (a bit over-simplified for real ecommerce but you get the idea):

{% highlight csharp %}
public class Product
{
    public int ProductID { get; set; }
    virtual IList<Order> Orders { get; set; }
    public int NumberSold
    {
        get { return Orders.Sum(x => x.Quantity); }
    }
}
public class Order
{
    public int ProductID { get; set; }
    public int Quantity { get; set; }

}
{% endhighlight %}



Now this is less than ideal because the _NumberSold_ property relies on Orders which, if lazy loaded, will be loaded completely into memory (performing a potentially resource intensive SELECT against the database) just to sum the quantities.  What we’d like is a way to just get this _NumberSold_ property back from the database when materializing a _Product_.


### The (good enough) Solution


I wanted some The best way I could find to do this required at worst 1 extra (relatively lightweight) SQL statement.  First, I rewrote the _NumberSold_ property to use a backing store:

{% highlight csharp %}
    protected int? numberSold;
    public int NumberSold
    {
        get
        {
            if (!numberSold.HasValue)
                numberSold = Orders.Sum(x => x.Quantity);
            return numberSold.Value;
        }
    }
{% endhighlight %}

Next, I handled the _ObjectMaterialized_ event on my EF DataContext:

{% highlight csharp %}

  public MyDataContext(string connectionString) : base(connectionString)
  {
    this.ObjectMaterialized += new ObjectMaterializedEventHandler(ObjectMaterialized);
  }
  private void ObjectMaterialized(object sender, ObjectMaterializedEventArgs e)
  {
    var product = e.Entity as Product;
    if (product != null) // the entity retrieved was a product
    {
        product.numberSold = Orders.Where(x => x.ProductID = product.ProductID)
            .Sum(x => x.Quantity);
    }
  }

{% endhighlight %}

I actually did this for a number of entities so I used the strategy pattern so as not to have a dozen if statements.  You can also optimize the code (from a DB perspective) by performing the Sum operation in-memory if the _Orders_ list is already loaded.  Maybe I’ll update this post later with that code but until then I leave that as an exercise for the reader.