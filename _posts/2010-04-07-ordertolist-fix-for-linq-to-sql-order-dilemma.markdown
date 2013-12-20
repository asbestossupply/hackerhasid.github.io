---
layout: post
status: publish
published: true
title: 'OrderToList: Fix for LINQ-to-SQL Order dilemma'
author: noah
author_login: noah
author_email: noah@statichippo.com
wordpress_id: 209
wordpress_url: http://statichippo.com/blog/archive/2010/04/07/ordertolist-fix-for-linq-to-sql-order-dilemma.aspx
date: 2010-04-07 21:04:32.000000000 -04:00
categories:
- C#
- Linq
tags: []
comments: []
---

 
  
**_UPDATE_**: I’ve cleaned up the code a bit and put it up on CodePlex at [http://ordertolist.codeplex.com/](http://ordertolist.codeplex.com/)
  
 
  
A team member recently wanted LINQ-to-SQL to perform a query with a Contains() clause and return the results in the same order.  For example, he was passed a list of People:

`List<int> idsOfPeopleToDisplay = new List<int> { 578, 291, 788, 230, 45 };`

and he wanted to select all People from the database with those IDs, in that order.  The following will result in a non-correctly ordered result set:

`var results = dataContext.People.Where(x => idsOfPeopleToDisplay.Contains(x.ID));`

this is equivalent to:

`SELECT * FROM People WHERE ID in ( 578, 291, 788, 230, 45 );`


The reason the results will not be ordered correctly is because any result with an ID that’s contained in the list will be returned in the order it was found.  This presented a dilemma.  Doing a JOIN onto the ID list was out of the question because of the size of the Article table.


He ended up with a temporary fix.  I believe I went the rest of the way.

## The temporary fix


I wrote a little tester to test my solution against his temporary fix.  This isn’t exactly his code, but it should pretty much perform the same logic:

{% highlight csharp %}
public static List<people> GetSortedPeopleFromList(IEnumerable<int> ids)
{
  using (DatabaseDataContext dataContext = new DatabaseDataContext())
  {
    var people = dataContext.Peoples.Where(x => ids.Contains(x.ID));
    List<people> sortedPeople = new List<people>();</people></people>
    
    foreach (var i in ids)
    {
      sortedPeople.Add(people.Single(x => x.ID == i));
    }
    
    return sortedPeople;
  }
}
{% endhighlight %}

As you can see, I’m retrieving the People from the database and then plucking them out of their list one by one in the order of my IDs.  Not very efficient, I think we can all agree.

## A better solution:

The final solution I came up with was to wrap objects in order to implement IComparable and then do an Array.Sort on them.  I have an extension method to support this called GetSortedPeopleFromList.  It’s used like so:

{% highlight csharp %}
public static List<people> GetSortedPeopleFromList(IEnumerable<int> ids)</people>
{
  using (DatabaseDataContext dataContext = new DatabaseDataContext())
  {
    var people = dataContext.Peoples.Where(x => ids.Contains(x.ID));
    return people.OrderToList(ids, x => x.ID).ToList();
  }
}
{% endhighlight %}

that’s also code from my tester app.  On line 8 you can see the extension method in play here.  The first argument is the list of IDs (which is already sorted to our liking) and the second is a Func that expresses how to retrieve the ID from each element.


The difference is pretty substantial.  I loaded 1k random records (ID, FirstName, LastName, City, State, Zip) into a SQL Express DB and ran some random tests (each pass had a random list of IDs, the IDs are outputted and so are the results of each method in order to confirm that it did, indeed, order the results correctly):


![speedtest](http://statichippo.com/images/statichippo_com/WindowsLiveWriter/OrderToListFixforLINQtoSQLOrderdilemma_F057/speedtest_thumb.png "speedtest")


Notice the bottom lines: 443ms vs 39ms (averaged over 10 iterations) – **_that’s over 11x speed increase!_**  So how does it work?

## OrderToList extension method

{% highlight csharp %}
public static IEnumerable<tobj> OrderToList<TObj, TKey>(this IEnumerable<tobj> list, IEnumerable<tkey> ids, Func<TObj, TKey> getKey)
  {
    var idsList = ids.ToList();
    var wrappedList = list.Select(x => 
    new IComparableWrapper<tobj>(x, idsList.IndexOf(getKey(x)))
  );
  var arr = wrappedList.ToArray();
  Array.Sort(arr);
  return arr.Select(x => x.Item);
}
{% endhighlight %}

This basically does 3 things:

* Wraps each item inside a class called _IComparableWrapper_ (no, it&rsquo;s not an interface and it should probably be renamed)
* Performs an Array.Sort on the result (which will use a Binary sort algorithm)
* Returns an IEnumerable of the original results which are now sorted (that&rsquo;s the _Item_ property of the _IComparableWrapper_)


## Supporting parties:

The _IComparableWrapper_ is a simple class that just wraps an object and implements _IComparable_ (in order to achieve our Binary Sort):

{% highlight csharp %}
public class IComparableWrapper<t> : IComparable</t>
{
  private T _Item;
  public T Item
  {
    get { return _Item; }
  }
  private int _Index;
  public IComparableWrapper(T item, int index)
  {
    _Index = index;
    _Item = item;
  }
  #region IComparable Members
  public int CompareTo(object obj)
  {
    if (!(obj is IComparableWrapper<t>)) throw new InvalidOperationException();

    IComparableWrapper<t> typedObj = (IComparableWrapper<t>)obj;
    return _Index.CompareTo(typedObj._Index);
  }
  #endregion
}
{% endhighlight %}

The _CompareTo_ method compares the _Index_ field of each object, which is just an integer which represents where this item belongs in the list.  Notice in the OrderToList method (above), on line 6 we instantiate a new _IComparableWrapper_ the second constructor argument is just the _Index_ of the Person’s ID in the list of IDs.  The reason I used a Func instead of just passing in an int was to allow for different situations where perhaps the IDs are accessed differently.
