---
layout: post
status: publish
published: true
title: ASP.NET MVC Captcha
author: noah
author_login: noah
author_email: noah@statichippo.com
wordpress_id: 183
wordpress_url: http://statichippo.com/blog/archive/2008/10/28/asp.net-mvc-captcha.aspx
date: 2008-10-28 21:53:00.000000000 -04:00
categories:
- ASP.NET
tags: []
comments: []
---

 
  
I was looking for an ASP.NET MVC Captcha control and stumbled upon Nick Berardi's ([http://www.coderjournal.com/2008/06/mvc-captcha-for-preview-release-3/](http://www.coderjournal.com/2008/06/mvc-captcha-for-preview-release-3/)).
  
First of all I'm using IIS7 and had trouble getting the HttpHandler to register properly.  I ended up with the following steps to get it to work:
     1. Add <add verb="GET" path="captcha.ashx" validate="false" type="ManagedFusion.Web.Handlers.CaptchaImageHandler, ManagedFusion.Web.Captcha" /> to <httphandlers> in Web.config </httphandlers>
    2. Add <add name="CaptchaImageHandler" verb="GET" path="captcha.ashx" type="ManagedFusion.Web.Handlers.CaptchaImageHandler, ManagedFusion.Web.Captcha" /> to the <handlers> section within <system.webserver> in Web.config </system.webserver></handlers>
    3. Add routes.IgnoreRoute("{handler}.ashx"); to Global.asax 
   ### So that got it working, but there were a few pieces of functionality I felt were missing:
  #### 1) I wanted the ability to style the CaptchaTextBox
  
The site I'm working with is pretty design intensive and I needed a way to inject CSS information into the CaptchaTextBox Html Helper.  I reworked the CaptchaHelper and modified the CaptchaTextBox as well as added some overloads:

{% highlight csharp %}
public static string CaptchaTextBox(this HtmlHelper helper, string name)
{
    return helper.CaptchaTextBox(name, null);
}

public static string CaptchaTextBox(this HtmlHelper helper, string name, Object htmlAttributes)
{
    return helper.CaptchaTextBox(name, ((IDictionary<string, object>)new RouteValueDictionary(htmlAttributes)));
}

public static string CaptchaTextBox(this HtmlHelper helper, string name, IDictionary<String, Object> htmlAttributes)
{
    ModelState state;             
    TagBuilder builder = new TagBuilder("input");             
    builder.MergeAttributes<string, object>(htmlAttributes);             
    builder.MergeAttribute("type", "text");             
    builder.MergeAttribute("name", name);             
    builder.MergeAttribute("id", name);             
    builder.MergeAttribute("value", "");             
    builder.MergeAttribute("maxlength", ManagedFusion.Web.Controls.CaptchaImage.TextLength.ToString());             
    builder.MergeAttribute("autocomplete", "off");             
    if (helper.ViewData.ModelState.TryGetValue(name, out state) && (state.Errors.Count > 0))
    {
        builder.AddCssClass("input-validation-error");
    }           
    return builder.ToString(TagRenderMode.SelfClosing);         
}
{% endhighlight %}

I got the TagBuilder idea from checking out the Reflector on System.Web.MVC.  Pretty cool stuff there.  So now I can use the CaptchaTextBox like so:


`<%= Html.CaptchaTextBox("captcha", new { @class = "field" })%>`

#### 2) I wanted the CaptchaValidationAttribute to invalidate my Model if the captcha isn't valid


(instead of inject a captchaValid with a value of false into my routedata which is what it does off the shelf)


For this I modified the CaptchaValidationAttribute class.  The first thing I did was make add an ErrorMessage string property.  Then I modified the OnActionExecutingContext method to look like this:

{% highlight csharp %}
public override void OnActionExecuting(ActionExecutingContext filterContext)         
{
    // get the guid from the post back             
    string guid = filterContext.HttpContext.Request.Form["captcha-guid"];             

    // check for the guid because it is required from the 
    //rest of the opperation             
    if (String.IsNullOrEmpty(guid))            
    {                 
        filterContext.Controller.ViewData.ModelState.AddModelError(Field, ErrorMessage);                 
        return;             
    } 

    // get values             
    CaptchaImage image = CaptchaImage.GetCachedCaptcha(guid);             
    string actualValue = filterContext.HttpContext.Request.Form[Field];             
    string expectedValue = image == null ? String.Empty : image.Text;   

    // removes the captch from cache so it cannot be used again             
    filterContext.HttpContext.Cache.Remove(guid);

    // validate the captch             
    if (String.IsNullOrEmpty(actualValue) || String.IsNullOrEmpty(expectedValue) || !String.Equals(actualValue, expectedValue, StringComparison.OrdinalIgnoreCase))
    {
        filterContext.Controller.ViewData.ModelState.AddModelError(Field, ErrorMessage);
        return;
    }
{% endhighlight %}

Now I can use the CaptchaValidationAttribute like this: 

{% highlight csharp %}
[CaptchaValidationAttribute()]
public ActionResult Register(FormCollection form)         
{
    // INCREDIBLY OVER-SIMPLIFIED BUT YOU GET THE IDEA             
    if (!ViewData.ModelState.IsValid)             
        return View();
    }         
}
{% endhighlight %}