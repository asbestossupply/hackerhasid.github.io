---
layout: post
status: publish
published: true
title: EntityFramework should throw more exceptions!
author: noah
author_login: noah
author_email: noah@statichippo.com
wordpress_id: 210
wordpress_url: http://statichippo.com/blog/archive/2010/05/28/entityframework-should-throw-more-exceptions.aspx
date: 2010-05-28 16:05:50.000000000 -04:00
categories:
- NHibernate
- Entity Framework
tags: []
comments: []
---

I’m working on a new project using .NET4, MVC2 and EntityFramework.  For the last couple years I’ve been using NHibernate as my ORM of choice but there are a lot of similarities between NHibernate and EntityFramework actually so the learning curve is quite small.  There are a number of things that make me long for NHibernate, but one stands out in my mind: lazy loading.  One of my first issues with EF was trying to track down a bug: ClassA has a property reference to ClassB, but the property is always null even when ClassB exists in the database.  After debugging for what seems like hours, I tracked down the issue – the ClassB property on ClassA was not virtual.  This was never a problem with NHibernate because NHibernate threw an exception if the property was not virtual.
 ### Why does the property need to be virtual?
 
Suppose you have two tables in the database, TableA and TableB.  And suppose you map your classes, ClassA and ClassB to those tables.  For lazy loading to work there has to be some proxy – some class of type ClassA that has a property associated with ClassB that when it’s first accessed will do the actual query to get ClassB data.  The way to accomplish this is to create a class, let’s call it ClassAProxy, that inherits from ClassA and overrides the properties, something like this:
 <div class="csharpcode"><pre><span class="lnum">   1:  </span><span class="kwrd">public</span> <span class="kwrd">class</span> ClassA</pre><pre><span class="lnum">   2:  </span>{</pre><pre><span class="lnum">   3:  </span>   <span class="kwrd">public</span> ClassB ReferenceToClassB { get; }</pre><pre><span class="lnum">   4:  </span>}</pre><pre><span class="lnum">   5:  </span>&nbsp;</pre><pre><span class="lnum">   6:  </span>public <span class="kwrd">class</span> ClassAProxy : ClassA</pre><pre><span class="lnum">   7:  </span>{</pre><pre><span class="lnum">   8:  </span>   <span class="kwrd">private</span> <span class="kwrd">bool</span> _IsRetrievedClassB = <span class="kwrd">false</span>;</pre><pre><span class="lnum">   9:  </span>   <span class="kwrd">private</span> ClassB _ReferenceToClassB;</pre><pre><span class="lnum">  10:  </span>   <span class="kwrd">public</span> <span class="kwrd">override</span> ClassB ReferenceToClassB</pre><pre><span class="lnum">  11:  </span>   {</pre><pre><span class="lnum">  12:  </span>     get</pre><pre><span class="lnum">  13:  </span>     {</pre><pre><span class="lnum">  14:  </span>       <span class="kwrd">if</span> (!_IsRetrievedClassB)</pre><pre><span class="lnum">  15:  </span>      {</pre><pre><span class="lnum">  16:  </span>        <span class="rem">// RETRIEVE FROM DB &amp; SET _ReferenceToClassB</span></pre><pre><span class="lnum">  17:  </span>       _IsRetrievedClassB = <span class="kwrd">true</span></pre><pre><span class="lnum">  18:  </span>       }</pre><pre><span class="lnum">  19:  </span>     <span class="kwrd">return</span> _ReferenceToClassB;</pre><pre><span class="lnum">  20:  </span>     }   </pre><pre><span class="lnum">  21:  </span>   }</pre><pre><span class="lnum">  22:  </span>}</pre><pre>&nbsp;</pre><pre /></div>Notice that ClassAProxy has an override on ClassB. That&rsquo;s to see if you&rsquo;re paying attention &ndash; ClassA does not define ReferenceToClassB as virtual, so this wouldn&rsquo;t compile. Change that one line (add &lsquo;virtual&rsquo;) and you get a working lazy loading proxy.<pre><pre>&nbsp;</pre></pre>
### The Takeaway
NHibernate throws an exception if you enable lazy loading but don&rsquo;t set your properties as virtual.&nbsp; Entity Framework just ignores this.&nbsp; That means that you won&rsquo;t know that a property isn&rsquo;t lazy loaded (and will always be null) until you access it.&nbsp; I can&rsquo;t imagine why someone would want this &ldquo;feature&rdquo;!
<style type="text/css">csharpcode, .csharpcode pre</style>
{
font-size: small;
color: black;
font-family: consolas, "Courier New", courier, monospace;
background-color: #ffffff;
/*white-space: pre;*/
}
.csharpcode pre { margin: 0em; }
.csharpcode .rem { color: #008000; }
.csharpcode .kwrd { color: #0000ff; }
.csharpcode .str { color: #006080; }
.csharpcode .op { color: #0000c0; }
.csharpcode .preproc { color: #cc6633; }
.csharpcode .asp { background-color: #ffff00; }
.csharpcode .html { color: #800000; }
.csharpcode .attr { color: #ff0000; }
.csharpcode .alt 
{
background-color: #f4f4f4;
width: 100%;
margin: 0em;
}
.csharpcode .lnum { color: #606060; }

