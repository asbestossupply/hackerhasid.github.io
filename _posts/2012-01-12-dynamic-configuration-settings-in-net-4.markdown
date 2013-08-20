---
layout: post
title: Dynamic configuration settings in .NET 4
categories:
- ASP.NET
- C#
- .NET
---

We have a default implementation over our API that we can customize for clients.  In order to support different feature-sets we have a custom configuration section inside our web.config that looks similar to this:
<!--more-->
{% highlight xml %}
<sites />
  <add name="hackerhasid" />
    <hosts />
        <add name="blog.hackerhasid.com" />
        
        <profile enabled="true" />
    
...
{% endhighlight %}



and this works well for the most part.  But we have some clients that have certain one-off features and settings that aren’t part of our main offering and don’t make sense to be added to our custom _ConfigurationSection_ class.  But it is still important to me to be able to store the associated settings in the same place and I still wanted the settings to be accessible from the same class.

{% highlight xml %}
<sites />
    <add name="hackerhasid">
    ...
    </add>
    <custom myspaceurl="whostillusesmyspace?" />
{% endhighlight %}


I ended up adding a _CustomSettings _property to the _SiteElement _that the  elements map to.  It’s type?  _dynamic._  The API is real simple, you just use the same attribute/element name specified in the web.config to access the value, e.g. _CustomSettings.myspaceUrl_.  In our case we use AutoMapper to map to a poco class which is useful to prevent runtime issues due to fat fingers on the dynamic type (plus AutoMapper provides a test to ensure you mapped all your properties).  I overrode the _OnDeserializeUnrecognizedElement_ hook inherited from the parent _ConfigurationElement _class to set it.  Here’s the code:

{% highlight csharp %}
///
/// Custom settings, set via OnDeserializeUnrecognizedElement
/// 
private dynamic CustomSettings { get; set; }

protected override bool OnDeserializeUnrecognizedElement(string elementName, System.Xml.XmlReader reader)
{
    if (elementName == "custom")
    {
        // create dyamic element to house the settings
        CustomSettings = new DynamicElement(elementName);
        // deserialize settings into dynamic element
        CustomSettings.Deserialize(reader);
        // yes, we've handled the unrecognized element
        return true;
    }
    return base.OnDeserializeUnrecognizedElement(elementName, reader);
}
{% endhighlight %} 


This basically just alters the hander to deal with a  element by using the _DynamicElement_.’s _Deserialize_ method to read the section.  Here’s the code for _DynamicElement_:

{% highlight csharp %}
///
/// Used to interpret custom settings
/// 
public class DynamicElement : System.Dynamic.DynamicObject
{
    #region ConfigElement class
    ///
    /// Used to plug into protected ConfigurationElement methods that need overriding
    /// 
    private class InnerConfigurationElement : ConfigurationElement
    {
        private DynamicElement _parent;
        internal InnerConfigurationElement(DynamicElement parent)
        {
            _parent = parent;
        }

        protected override bool OnDeserializeUnrecognizedAttribute(string name, string value)
        {
            if (_parent.Attributes.ContainsKey(name))
                return false;

            _parent.Attributes.Add(name, value);
            return true;
        }

        protected override bool OnDeserializeUnrecognizedElement(string elementName, System.Xml.XmlReader reader)
        {
            var element = new DynamicElement(elementName);
            element.Deserialize(reader);
            _parent.ChildElements.Add(element);
            return true;
        }

        internal void Deserialize(System.Xml.XmlReader reader)
        {
            DeserializeElement(reader, serializeCollectionKey: false);
        }
    }

    public string Name { get; private set; }
    internal readonly Dictionary<string, string> Attributes = new Dictionary<string, string>();
    internal readonly List<dynamicelement> ChildElements = new List<dynamicelement>();
    private InnerConfigurationElement _configElement { get; set; }
    internal DynamicElement(string name)
    {
        Name = name;
        _configElement = new InnerConfigurationElement(this);
    }

    public override bool TryGetMember(System.Dynamic.GetMemberBinder binder, out object result)
    {
        // search attributes first
        if (Attributes.ContainsKey(binder.Name))
        {
            result = Attributes[binder.Name];
            return true;
        }

        // then search child elements
        var childElement = ChildElements.FirstOrDefault(x => x.Name == binder.Name);
        if (childElement != null)
        {
            result = childElement;
            return true;
        }

        result = null;
        return false;
    }

    public override IEnumerable<string> GetDynamicMemberNames()</string>
    {
        return base.GetDynamicMemberNames();
    }

    public override bool TryConvert(System.Dynamic.ConvertBinder binder, out object result)
    {
        return base.TryConvert(binder, out result);
    }

    public void Deserialize(System.Xml.XmlReader reader)
    {
        _configElement.Deserialize(reader);
    }
}
{% endhighlight %}


it’s a bit longwinded but it’s actually pretty simple.  The _ConfigElement class _region just contains the definition for an _InnerConfigurationElement_ class.  I didn’t want to have to write any xml parsing code since Microsoft provides that out of the box via the _ConfigurationElement_ class.  However, since that class’s entry hook, _DeserializeElement_, is protected I inherited from it and declared an internal _Deserialize_ method that just calls into this one.  I also overrode the _OnDeserializeUnrecognizedAttribute _and _OnDeserializeUnrecognizedElement _methods the latter of which just creates a new _DynamicElement_ and recursively calls the _Deserialize_ method on it.


The rest of the code outside the region is pretty straight forward.  There’s a Dictionary containing attribute name/values and a List containing child elements.  Now that I’m writing this it strikes me perhaps this should be a Dictionary itself to reduce lookup times but I leave that as an exercise to the reader (or maybe I’ll update this post at some point… maybe).  When someone attempts to get the value of a property like _CustomSettings.SomeValue_ I first look in the attributes and then if it doesn’t exist there I look in the child elements.


And there you have it – dynamic configuration settings in .NET 4!
