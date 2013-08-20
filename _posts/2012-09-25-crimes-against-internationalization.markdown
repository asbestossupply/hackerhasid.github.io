---
layout: post
title: Crimes against internationalization
categories:
- C#
- .NET
- Internationalization
---
What's wrong with this snippet from Authorize.net's .NET SDK?
<!--more-->
{% highlight csharp %}
public CardPresentPriorAuthCapture(string transactionID, Decimal amount)
{
  this.SetApiAction(RequestAction.PriorAuthCapture);
  this.Queue("x_ref_trans_id", transactionID);
  this.Queue("x_amount", amount.ToString());
  ...
{% endhighlight %}
Did you get it yet? Think about that last line, amount.ToString() -- what's that going to do?

Let's see -- what's the meaning of $25.99? Well that's twenty-five dollars and ninety-nine cents. What's the meaning of &euro;25.99? That's not as clear --&nbsp;Europeans use "." where Americans use "," to denote thousands. Maybe&nbsp;it should be read &euro;25.990; almost twenty six thousand Euros! Or perhaps it's a typo and that decimal point shouldn't be there at all, it's meaningless in the position it's in anyway, so perhaps the meaning is twenty-five-hundred and ninety-nine Euros.

Well sure enough, Authorize.net's API assumes that "." denotes decimals and "," denotes thousands. If the thread that calls into their SDK is in a European culture that _.ToString()_&nbsp;call will output _25,99_. But when Authorize.net's API sees that request they parse it as twenty-five-hundred and ninety-nine dollars.

And that is a crime against internationalization. If you're writing an SDK please do not use _.ToString()_&nbsp;without a format provided. In this case, if Authorize.net has just added a couple more characters --&nbsp;_.ToString("0.0") --&nbsp;_&nbsp;the format would stay constant across cultures.
