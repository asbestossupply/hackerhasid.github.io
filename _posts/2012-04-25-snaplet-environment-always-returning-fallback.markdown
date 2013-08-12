---
layout: post
status: publish
published: true
title: snaplet-environment always returning fallback
author: noah
author_login: noah
author_email: noah@hackerhasid.com
date: 2012-04-25 00:01:34.000000000 -04:00
categories:
- Uncategorized
tags:
- haskell
comments: []
---
posted this SO question a while back:&nbsp;[http://stackoverflow.com/questions/9879543/snaplet-environment-always-returning-fallback](http://stackoverflow.com/questions/9879543/snaplet-environment-always-returning-fallback)

i was having an issue with the haskell package _snaplet-environment_&nbsp;-- the _lookupEnvDefault_&nbsp;function&nbsp;wasn't picking up the correct environment, always returning the fallback value.

my snaplet.cfg looked like:

{% highlight json %}
app
{
  environments
  {
    production
    {
      config-url = "http://www.google.com"
    }
  }
}
{% endhighlight %}

well, after a _lot_&nbsp;of trial and error i found that i had to modify the config to look like this:

{% highlight json %}
app
{
  environments
  {
    production
    {
      config-url = ""
    }
  }
}
environments
{
  production
  {
    config-url = "http://www.google.com"
  }
}
{% endhighlight %}

this has got to be a bug, right? &nbsp;i guess i'll have to look at the_ snaplet-environment_ code sometime... geez!

UPDATE: well thanks to [@antipax](https://twitter.com/#!/antipax) i've submitted a [pull request](https://github.com/statichippo/Snaplet-Environments/tree/patch-1)
