---
layout: post
status: publish
published: true
title: NHibernate Generic Dictionary Mapping
author: noah
author_login: noah
author_email: noah@statichippo.com
wordpress_id: 180
wordpress_url: http://statichippo.com/blog/archive/2009/01/14/nhibernate-generic-dictionary-mapping.aspx
date: 2009-01-14 21:42:00.000000000 -05:00
categories:
- NHibernate
tags: []
comments: []
---

"NHibernate: have I told you today how much I love you?"
  
The other day I had a good reason to use a generic dictionary and I though hey, I wonder how I'll get that working with NHibernate...  It's very simple to persist Generic Dictionarys with NHibernate as I found out.  Let's see an example...
  ## **.NET PSEUDO CODE**
  
Take a person and his favorite or least favorite deserts:
  
First we have a Person object
{% highlight csharp %}   
class Person
{
  int PersonId;
  string FirstName;
  string LastName;
}
{% endhighlight %}
   

{% highlight csharp %}
class Desert
{
  int DesertId;
  string Name;
}
{% endhighlight %}
   
Now we want to associate a Person with all the deserts and rate how much he likes them (I know, it's contrived, but it's simpler than the business case I did this with yesterday).  So first we might create an enum for storing how one feels about a particular dessert (we'll call it Rating):

{% highlight csharp %}  
enum Rating 
{
  Love,
  Like,
  Impartial,
  Dislike,
  Disdain
}
{% endhighlight %}
   
And then we might add a generic dictionary to the Person object like so:
  
{% highlight csharp %}
class Person
{
  string FirstName;
  string LastName;
  Dictionary<Desert, Rating> DesertRatings;
}
{% endhighlight %}
   
Looks good.  Now how would we use NHibernate to persist this?  First off, let's make a slight change to the Person object because NHibernate actually uses IDictionary, so that last line of the Person object would look more like:

``IDictionary<Desert, Rating> DesertRatings``

## DATABASE TABLES
  
To persist this data we first create the tables in our database.  Assuming we used the code above, I would probably create 3 different tables: 1 to store the Persons, 1 to store Deserts, and 1 to store the association between Persons and their Desert ratings.  The tables might look like this:
 
_PEOPLE_
* PersonId, int
* FirstName, varchar
* LastName, varchar
    
_DESERTS_ 
* DesertId, int
* Name, varchar
    
_PEOPLEDESERTRATINGS_ 
(PersonId and DesertId will be the primary keys since 1 person can only have 1 rating for a specific desert)
* PersonId, int
* DesertId, int
* Rating, int (this will store the integer representation of the enum)


## NHibernate Mappings
  
Now behold the magic of NHibernate.  Let's take a look at what the Person.hbm.xml file might look like:

```
  <?xml version=”1.0″ encoding=”utf-8″ ?>
  <hibernate-mapping xmlns=”urn:nhibernate-mapping-2.2″ assembly=”MyAssembly” namespace=MyNamespace”>
  <class name=”Person” table=”People”>
  <id name=”PersonId” column=”PersonId” type=”Int32″ unsaved-value=”0″>
  <generator class=”native”></generator>
  </id>
  <property name=”FirstName” column=”FirstName” type=”String” not-null=”true” />
  <property name=”LastName” column=”LastName” type=”String” not-null=”true” />
  <map name=”DesertRatings” table=”PeopleDesertRatings”>
  <key column=”PersonId” />
  <index-many-to-many class=”Desert” column=”DesertId” />
  <element column=”Rating” type=”MyNamespace.Rating” not-null=”true” />
  </map>
  </class>
  </hibernate-mapping>
```
That's all there is too it.  There's a lot that I didn't cover here, and this is NOT meant to be a tutorial on how to design your objects or tables.  However, I think I've shown how amazingly easy it is to persist this kind of thing using NHibernate.  I didn't find anything online about this before I tried it and it worked the first time, but I would have felt more comfortable trying it had I seen someone else do it first.  So hopefully this post helps someone feel comfortable diving right in.
