---
layout: post
status: publish
published: true
title: Find/Replace on Render - an MVC alternative to Response Filters
date: 2009-11-16 22:55:28.000000000 -05:00
categories:
- ASP.NET
- C#
tags: []
comments: []
---

**UPDATE** 02/03/2010: You might NOT want to use this method as it screws up Page Caching: [http://blog.statichippo.com/archive/2010/02/03/response.flush-and-the-crimes-against-page-caching.aspx](http://blog.statichippo.com/archive/2010/02/03/response.flush-and-the-crimes-against-page-caching.aspx)
  
This post is sort of in response to another post of mine entitled [Rendering ViewPage to random stream (or not)](http://blog.statichippo.com/archive/2009/09/16/rendering-viewpage-to-random-stream-or-not.aspx).  Back then I was looking for this solution.  Now I found it!
  
If you’re not familiar with Response Filters, read [the MSDN article](http://msdn.microsoft.com/en-us/library/system.web.httpresponse.filter.aspx) and [this quick example at aspnetlibrary.com](http://aspnetlibrary.com/articledetails.aspx?article=Use-Response.Filter-to-intercept-your-HTML).  Basically, Reponse Filters are used to filter output before it gets sent downstream.  This is useful if you want to, say, replace certain special characters.
  ## Background & the existing Response Filter solution
  
I’m working on a Content Management System that does just this.  Let’s say you want to create a page break in your page, you might have a  tag inside the content.  This is a simple example, but there are more complex examples too, such as displaying Commerce info based on information in content.
  
The way this was done before was via Response Filters, but I wanted these special tags to be handled as each Page/Partial was rendered.  There were a few reasons for this:
     1. Using response filters meant storing the HTML to render outside of an ASPX page. I have seen people render ASPX pages for this kind of thing, but considering the logic for rendering a special tag in a Controller 'felt' right, I didn't want to go this route 
    2. The existing solution was using a separate filter for _each_ tag. What that meant was that before the Response was sent downstream, it was converted into a String numerous times in order to do a Find/Replace (one time for each tag filter). Keep in mind that one tag might render another tag, so there is a bit of recursive goodness going on here too. Now this could also have been refactored so that a single Filter would do the Find and hand over the Replace to other classes, but, again, I wanted these tags to be rendered through a Controller in-pipeline with the pages that contain them. 
   ## IView, in-pipeline rendering solution
  
At first I thought it would be a pretty simple fix.  I’d just create an IView that renders to a string and then do a find/replace.  That’s basically what I did, however, this is harder than it sounds.

{% highlight csharp %}
public class MyView : WebFormView
{
    string _Masterpath;
    string _ViewPath;
    public MyView(string viewPath) : this(viewPath, null) { }
    public MyView(string viewPath, string masterpath) : base(viewPath, masterpath)
    {
        _Masterpath = masterpath;
        _ViewPath = viewPath;
    }
    
    public override void Render(ViewContext viewContext, System.IO.TextWriter writer)
    {
        string response = RenderToString(viewContext, writer);
        
        // WebFormView doesn't care about the writer arg
        
        var outputTextWriter = viewContext.HttpContext.Response.Output; 
        MyTagParser.RenderTags(response, viewContext, outputTextWriter);


    }

}
{% endhighlight %}


I created a class called MyView that inherits from WebFormView, since that’s the default we’re using anyway.  You’ll notice the above code is missing the RenderToString() method.  That’s what I had the hardest time on.


After some failed attempts, I did a quick google search and found [this post on stackoverflow](http://stackoverflow.com/questions/483091/render-a-view-as-a-string).  The question was how to render a View to a String and [Tim Scott](http://stackoverflow.com/users/29493/tim-scott) had a great idea, which I ended up using.  I just wish I thought of it!


The idea is basically to create a new MemoryStream and use it as your Response Filter.  Render your View the normal way, but now it will be rendered to the MemoryStream instead of the Response.  Now you can create a StreamReader to read the contents of the MemoryStream into a string.


Here’s that method:

{% highlight csharp %}
private string RenderToString(ViewContext viewContext, TextWriter writer)
{
    string responseString = null;
    var response = viewContext.HttpContext.Response;
    response.Flush();
    var oldFilter = response.Filter;           
    Stream filter = null;
    try
    {
        filter = new MemoryStream();
        response.Filter = filter;
        base.Render(viewContext, writer);
        response.Flush();
        filter.Position = 0;
        var reader = new StreamReader(filter, response.ContentEncoding);
        responseString = reader.ReadToEnd();
    }
    finally
    {
        if (filter != null) filter.Dispose();
        
        response.Filter = oldFilter;
    }


    return responseString;
}
{% endhighlight %}


So, now that the I’m getting a string, it’s time for the Find/Replace code.  That’s in the MyTagParser class with its helper methods (but missing the main, RenderTags method, below):

{% highlight csharp %}
public class MyTagParser
{
    public static IDictionarystring, ITagParser> TagParsers = new Dictionarystring, ITagParser>(StringComparer.CurrentCultureIgnoreCase);

    const int MAX_RECURSION = 50;
    private const string k_RECURSION_COUNT = "tag_parser_recursion_count";

    private static Dictionarystring, string> GetTagParameters(string tag)
    {
        Dictionarystring, string> paramDictionary = new Dictionarystring, string>();

        // Get param/value pairs

        Regex parameterRegex = new Regex("([a-z0-9]+)=\"([\\w\\s#|]*)\"", RegexOptions.IgnoreCase);

        foreach (Match paramMatch in parameterRegex.Matches(tag))
        {
            string paramName = paramMatch.Groups[1].Value;
            string paramValue = paramMatch.Groups[2].Value;
            paramDictionary.Add(paramName.ToLower(), paramValue);
        }

        // Get params with no value pair (e.g. "notable" in )
        Regex novalueRegex = new Regex("\\s\\b([a-z0-9]+)\\b(?!=)", 
        RegexOptions.IgnoreCase);
        foreach (Match paramMatch in novalueRegex.Matches(tag))
        {
            string paramName = paramMatch.Groups[1].Value;
            paramDictionary.Add(paramName.ToLower(), null);
        }

        return paramDictionary;
    }

    private static ITagParser FindCorrespondingTag(string tagName)
    {
        ITagParser ziffTag = null;

        if (TagParsers.ContainsKey(tagName)) return TagParsers[tagName];

        return null;
    }


    private static int IncrementRecursionCount(ViewContext viewContext)
    {
        IDictionary httpItems = viewContext.HttpContext.Items;

        if (!httpItems.Contains(k_RECURSION_COUNT)) httpItems[k_RECURSION_COUNT] = 0;

        int recursionCount = (int)httpItems[k_RECURSION_COUNT];

        httpItems[k_RECURSION_COUNT] = ++recursionCount;

        return recursionCount;
    }

    private static void WriteTextToOutput(string origString, TextWriter writer, int startOffset, int endOffset)
    {
        int length = endOffset - startOffset;
        string str = origString.Substring(startOffset, length);
        writer.Write(str);
    }


    // RenderMyTags() method below
}
{% endhighlight %}

Since these tags can be nested, we don’t want infinite loops.  I have a key that’s stored in the HttpContext Items to prevent this – if our count gets beyond MAX_RECURSION, we won’t process any more tags.


And finally, the RenderMyTags method does all the manly work:


{% highlight csharp %}
public static void RenderMyTags(string origString, ViewContext viewContext, TextWriter writer)
{
    if (string.IsNullOrEmpty(origString)) return;
    int recursionCount = IncrementRecursionCount(viewContext);
    int renderOffset = 0;

    Regex specialTagRegex = new Regex("]*>((.*?))?", 
    RegexOptions.IgnoreCase);
    MatchCollection specialTagMatches = specialTagRegex.Matches(origString);
    foreach (Match match in specialTagMatches)
    {
    // Write the text before this tag
    WriteTextToOutput(origString, writer, renderOffset, match.Index);

    if (recursionCount <= MAX_RECURSION)
    {
        string fullTag = match.Value;
        string tagName = match.Groups[1].Value;
        string innerText = string.Empty;
        if (match.Groups.Count == 4 /* has closing tag */)
        innerText = match.Groups[3].Value;
        int trueMatchIndex = match.Index + renderOffset;
        ITagParser tagParser  = FindCorrespondingTag(tagName);

        if (tagParser != null)
        {
            Dictionarystring, string> parameters = GetTagParameters(fullTag);
            tagParser.RenderTag(viewContext, fullTag, innerText, parameters);
        }
    
        renderOffset = match.Index + match.Length;
    }

    // Write the remaining text
    WriteTextToOutput(origString, writer, renderOffset, origString.Length);
}
{% endhighlight %}


Regex blows up with NULL strings, so line 4 just checks for that.  Line 7 keeps renderOffset – this helps us keep track of where we are in the string we’re processing.  Imagine the following string:

> SOME TEXT  SOME MORE TEXT

We want to write out the SOME TEXT, then render the SPECIALTAG, then again write the SOME MORE TEXT.


The rest is pretty self explanatory.
