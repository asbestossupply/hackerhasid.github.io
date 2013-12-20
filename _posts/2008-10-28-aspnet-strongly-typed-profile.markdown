---
layout: post
status: publish
published: true
title: ASP.NET Strongly Typed Profile
categories:
- Uncategorized
tags: []
comments: []
---

I'm not a huge fan of the out-of-the-box functionality of ASP.NET Profiles.  I really dislike the way everything is is just serialized as a blob and there's no out-of-the-box alternative.  That aside, I'm using Profiles for an ASP.NET MVC site right now and wanted a way to strongly type the data (and remove the need for string literals).  Here's what I ended up with: 

{% highlight csharp %}
public class StronglyTypedProfile
{
    ProfileBase profile;
    public int MerchantId
    {
        get
        {
            object ob = profile.GetPropertyValue("MerchantId");
            if (ob == null) return -1;
            return (int)ob;
        }
        set
        {
            profile.SetPropertyValue(value);
        }
    }
    public string ForumNick
    {
        get { return (string)profile.GetPropertyValue("ForumNick"); }
        set { profile.SetPropertyValue("ForumNick", value); }
    }
    public string FirstName
    {
        get { return (string)profile.GetPropertyValue("FirstName"); }
        set { profile.SetPropertyValue("FirstName", value); }
    }
    public string LastName
    {
        get { return (string)profile.GetPropertyValue("LastName"); }
        set { profile.SetPropertyValue("LastName", value); }
    }
    public void Save()
    {
        profile.Save();
    }

    private StronglyTypedProfile() {}
    public static StronglyTypedProfile Create(string username)
    {
        StronglyTypedProfile p = new StronglyTypedProfile();
        p.profile = ProfileBase.Create(username);
        return p;
    }
}
{% endhighlight %}


Now I can use it like so: 

{% highlight csharp %}
StronglyTypedProfile profile = StronglyTypedProfile.Create(Username);
profile.FirstName = u.FirstName;
profile.LastName = u.LastName;
profile.Save();
{% endhighlight %}

Very simple, and it works.