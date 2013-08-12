---
layout: post
status: publish
published: true
title: Huge EFing Bug!
author: noah
author_login: noah
author_email: noah@statichippo.com
wordpress_id: 212
wordpress_url: http://statichippo.com/blog/archive/2010/06/08/Huge-EFing-Bug.aspx
date: 2010-06-08 18:49:52.000000000 -04:00
categories:
- C#
- Entity Framework
tags: []
comments: []
---

Thanks to Keith (comment #1 at [http://weblogs.asp.net/johnkatsiotis/archive/2010/04/28/huge-ef4-inheritance-bug.aspx#7464062](http://weblogs.asp.net/johnkatsiotis/archive/2010/04/28/huge-ef4-inheritance-bug.aspx#7464062)) for that title, btw.
  
After coming from NHibernate, EF4 is an enormous pain to work with.  Unlike NHibernate, which basically lets you map any relationship you can dream of, EF4 has a lot of issues with inheritance.  Here’s a pretty big one IMO:
  
Take a look at this object hierarchy:
  
[![ObjectHierarchy](http://blog.statichippo.com/images/statichippo_com/WindowsLiveWriter/HugeEFingBug_BA75/ObjectHierarchy_thumb.png "ObjectHierarchy")](http://blog.statichippo.com/images/statichippo_com/WindowsLiveWriter/HugeEFingBug_BA75/ObjectHierarchy.png) 
  
Before I continue, if you think we’re working on an edge case I have to disagree.  One of the tenets of OOP is the idea of encapsulation – there’s no reason a base class shouldn’t be able to encapsulate the logic of a subclass.  For instance, in this case, the _LeaveWork_ method on **Employee** would be abstract, but would be implemented on **SomeoneWhoActuallyWorks **to make sure they don’t leave too early and on **SomeoneWhoDoesNotWork** to allow them to leave at any time.  I know I’m stretching it here with the example, but I wanted something simple that I could test & show you.  I actually am working on a project with this type of hierarchy and EF4 is making my life painful… Take a look:
  ##### Mapping with Entity Framework (NOT)
  
 
  
Here’s an **Employee** table. This should really be all we need.  **Employee**s just have a **FullName** and a type.  We just want to map it to the right classes based on the _EmployeeType_, but that won’t actually get mapped as a property on the class.
  
[![Employee Table](http://blog.statichippo.com/images/statichippo_com/WindowsLiveWriter/HugeEFingBug_BA75/Employee%20Table_thumb.png "Employee Table")](http://blog.statichippo.com/images/statichippo_com/WindowsLiveWriter/HugeEFingBug_BA75/Employee%20Table.png) 
  
So you might be naive and try mapping this relationship like so (in the edmx file):
  
[![Inheritance_NoMiddleMan](http://blog.statichippo.com/images/statichippo_com/WindowsLiveWriter/HugeEFingBug_BA75/Inheritance_NoMiddleMan_thumb.png "Inheritance_NoMiddleMan")](http://blog.statichippo.com/images/statichippo_com/WindowsLiveWriter/HugeEFingBug_BA75/Inheritance_NoMiddleMan.png) 
  
Of course you’d make sure that **Sucker** had a Condition “When _EmployeeType_ = ‘Sucker’” and **Manager** when _EmployeeType_ = ‘Manager’.
  
But that causes a runtime exception on insert:
  >    
Mapping and metadata information could not be found for EntityType 'EFTest.Manager'.
   
You see, the fact that **Manager** and **Sucker** both inherit from **SomeoneWhoActuallyWorks** is actually very important to Entity Framework 4.  I’m not sure why.  I mean if I didn’t specify a middle-man class, what’s the difference?  If it has properties, I’m obviously not trying to map them, so who cares?  Well EF4 cares…  But wait, it gets better (or worse, depending on how you want to look at it)
  
So, your next try might be to map middleman -- **SomeoneWhoActuallyWorks**
  
[![Inheritance_OneMiddleMan](http://blog.statichippo.com/images/statichippo_com/WindowsLiveWriter/HugeEFingBug_BA75/Inheritance_OneMiddleMan_thumb.png "Inheritance_OneMiddleMan")](http://blog.statichippo.com/images/statichippo_com/WindowsLiveWriter/HugeEFingBug_BA75/Inheritance_OneMiddleMan.png) 
  
You can map **SomoneWhoActuallyWorks** to the **Employee** table and set its _Abstract_ property to _True_, this will compile without a problem.  And it will run too.  And inserting a **Manager** or a **Sucker** will work.
  
But this method explodes when you add another branch off of **Employee**:
  
[![FullMappingHierarchy](http://blog.statichippo.com/images/statichippo_com/WindowsLiveWriter/HugeEFingBug_BA75/FullMappingHierarchy_thumb.png "FullMappingHierarchy")](http://blog.statichippo.com/images/statichippo_com/WindowsLiveWriter/HugeEFingBug_BA75/FullMappingHierarchy.png) 
  
This mapping (**SomeoneWhoDoesNotWork** is abstract, btw, and **Owner** is mapped to the **Employee** table with a Condition “Where _EmployeeType_ = ‘Owner’”) won’t even compile:
  >    
Error    1    Error 3032: Problem in mapping fragments starting at lines 47, 66:EntityTypes EFTest.Manager, EFTest.Sucker, EFTest.Owner are being mapped to the same rows in table employees. Mapping conditions can be used to distinguish the rows that these types are mapped to.     
   
Yea…
  
Now you can “fix” this by taking out those middle men.  But deriving **Owner**, **Manager** and **Sucker** from **Employee** directly might have you writing a lot more code, and potentially duplicate code at that.  And it’s a hack.
