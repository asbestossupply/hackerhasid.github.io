---
layout: post
status: publish
published: true
title: Some EF4 quirks and annoyances
author: noah
author_login: noah
author_email: noah@hackerhasid.com
date: 2010-12-27 15:49:59.000000000 -05:00
categories:
- C#
- Entity Framework
- Testing
tags: []
comments: []
---

I’ve heard a lot of people bash Entity Framework for being incomplete.  I happen to think that EF4 is pretty good for a first version.  It’s just too bad it’s a second.  Especially considering there’s plenty of mature ORMs from which Microsoft could take some more hints.
  
I’m using EF4 code-first which should be a lot like other ORMs out there like, for example, NHibernate (which I used in many previous projects).  NHibernate supports hydration of data into fields.  EF4 can only hydrate properties.  This is more annoying than it may sound.  For example, take the following code snippet:


{% highlight csharp %}
public class Person
{
  public Person()
  {
    _children = new List<Person>();
  }
  private List<Person> _children;
  public ReadOnlyCollection<Person> Children { get { return _children.AsReadOnly(); } }
}

{% endhighlight %}

I use this coding pattering all the time.  I don’t want anyone adding a child willy-nilly!  What if the child already belongs to someone else?  What if the child is older than the parent?  Using this pattern I can have an_ AddChild()_ method that does the necessary validation etc.  NHibernate supports this just fine.  In order to use this with EF4, I have to make some changes:

{% highlight csharp %}
public class Person
{
  public Person()
  {
    childrenBackingStore = new List<Person>();
  }
  virtual List<person> childrenBackingStore { get; set; }
  public ReadOnlyCollection Children { get { return childrenBackingStore.AsReadOnly(); } }
}
{% endhighlight %}


Here are the changes I had to make:


* Rename __children_ to _childrenBackingStore_.&nbsp; This is necessary because EF4 doesn't support special characters like _.
* Refactor `childrenBackingStore` into a property
* Make `childrenBackingStore` protected so that EF4 has the necessary access privileges to set it (EF4 will create a proxy that inherits from `Person`)
* Mark `childrenBackingStore` as `virtual` so that the EF4 proxy can override it and provide lazy loading support


Now the code here doesn’t look much worse than before, but there’s at least one serious side effect to this when it comes to testing.  When mocking `Person` (using a `stub` in RhinoMocks), `Children`will always return `null`.


{% highlight csharp %}
[Test]
[ExpectedException(typeof(NullReferenceException))]
public void StubAPerson_WithRhinoMocks_ChildrenPropertyIsNull()
{
  var person = MockRepository.GenerateStub<Person>();
  var children = person.Children;
  // should throw because of the call to childrenBackingStore.AsReadOnly()
}
{% endhighlight %}


It’s not that the _Person_ constructor doesn’t get called, it does.  It’s just that a stub is really a proxy, a subclass of `Person`, that overrides all the properties.  Unless you stub out the actual property, thereby telling the proxy what you want the property to return, it will just return the type’s default which, for reference types like my `List`, is `null`.


Which makes sense because you’re creating a stub.  That makes testing annoying because you either have to stub out all the properties or create a `PartialMock` instead of a `Stub` which may have other side effects.

{% highlight csharp %}
[Test]
public void StubAPerson_ExplicitlyStubChildrenProperty_ChildrenIsNotNull()
{
  var person = MockRepository.GenerateStub<Person>();
  // for this to compile I have to set childrenBackingStore to protected internal
  // and set the InternalsVisibleTo assembly attribute so my test project can
  // access the internal property
  person.Stub(x => x.childrenBackingStore).Return(new List<Person>());
  Assert.That(person.Children != null);
}
[Test]
public void MockAPerson_UsingPartialMock_ChildrenIsNotNull()
{
  var person = MockRepository.GeneratePartialMock<Person>();
  Assert.That(person.Children != null);
}
{% endhighlight %}

I hope that EF5 (or, if they keep up with the power-of-4 versioning, EF16) supports setting field values!
