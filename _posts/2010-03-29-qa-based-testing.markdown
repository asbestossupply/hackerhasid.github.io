---
layout: post
status: publish
published: true
title: QA Based Testing
author: noah
author_login: noah
author_email: noah@statichippo.com
wordpress_id: 208
wordpress_url: http://statichippo.com/blog/archive/2010/03/28/qa-based-testing.aspx
date: 2010-03-29 07:06:00.000000000 -04:00
categories:
- Uncategorized
tags: []
comments: []
---

Over the last couple weeks I’ve spoken to a few people about something I’m calling _QA Based Testing_, and I figured it would be a good time to write about it.
  
There’s been a lot of attention around Test Driven Development in the last couple years.  TDD does take some practice to get used to and it requires team participation.  If your boss tells you to “do it right now” while he looks over your shoulder waiting to alert the business that it’s ready, you might have a hard time practicing TDD ;).
  
But there’s another way to get your code coverage up, and it doesn’t require practice or team participation.  In fact, it’s very helpful for working with legacy systems.  The idea behind _QA Based Testing_ is that the moment QA fails a certain part of the app, the first thing to do is write a test.  The test should fail because it should produce the issue that QA had a problem with.  Then fix your code.  Red-Green-Resubmit to QA.
  
At my current company I’ve used this with great success.  First of all, I don’t always do TDD.  And second of all, we’re building on top of a legacy system that constantly surprises us with new issues.  Even modules I built with TDD have failed to account for certain situations because I just could not think of all the possible (invalid) data permutations we would have to account for.  As issues arise, the code coverage increases.  And it requires no getting used to and no team participation.  
  
There’s really no excuse for not testing your code!
