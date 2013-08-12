---
layout: post
status: publish
published: true
title: jQuery Accordion - whitespace issue with list
author: noah
author_login: noah
author_email: noah@statichippo.com
wordpress_id: 178
wordpress_url: http://statichippo.com/blog/archive/2008/09/10/jquery-accordion-whitespace-issue-with-list-to-beat-a-dead-horse.aspx
date: 2008-09-10 22:09:04.000000000 -04:00
categories:
- Uncategorized
tags: []
comments: []
---
Choffi has this cool side-bar navigation using an accordion (the idea was taken mainly from apple's website which has a similar thing) that's supposed to look something like:

[caption id="attachment_6" align="alignnone" width="271" caption="This is what it should look like"]![This is what it should look like](http://noahblu.files.wordpress.com/2008/09/1.jpg "What it's supposed to look like")[/caption]

I'm using jQuery 1.2.6 and jQuery UI 1.5.2. Everything looks fine in FF, but when I open it in IE7 it looks really bad (although it works):

[caption id="attachment_7" align="alignnone" width="281" caption="Ugggghhhh"]![What's with all the white space?](http://noahblu.files.wordpress.com/2008/09/y1puuhsvpw8uxcl7ytla7ekypkaekuvnxqqecrhuwgvcobxdom4qkq-rikuricae26m3xa7ow4gjz4.jpg "Ugghhhhhh")[/caption]

After some tinkering, I tracked the issue down to the fact that the Accordion sticks a span element before and after each of my list items (<li>Test</li> becomes <span></span><li>Test</li><span></span>) which is forcing everything down I suppose because my li styles indicate the use of background images. I just LOVE FireBug (https://addons.mozilla.org/en-US/firefox/addon/1843)

Anyway, I edited the ui.accordion.js file to read:
if (!this.options.disableLeftSpan)
$("<span class="ui-accordion-left">").insertBefore(options.headers);</span>
if (!this.options.disableRightSpan)
$("<span class="ui-accordion-right">").appendTo(options.headers);</span>

on lines 42-45 -- basically I just added an if statement before both the left & right spans are added. Now I can make my call like so (notice the disableLeftSpan & disableRightSpan):

$("#nav_accordion").accordion({
active: false,
event: 'mouseover',
fillSpace: true,
disableLeftSpan: true,
disableRightSpan: true
});

and it looks good even in IE7!

I submitted a bug to the jQuery UI team, but until someone integrates these changes, it's easy to do yourself.
