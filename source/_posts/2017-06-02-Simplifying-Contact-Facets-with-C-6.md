title: 'Simplifying Contact Facets with C# 6'
date: 2017-06-02 10:04:08
categories: 
- Sitecore
- xDB
---

Contact Facets allow you to persist information about visitors into the Sitecore xDB. We're not going to get into the theory behind them in this post; for that go read Pete Navarra's [great blog post](https://sitecorehacker.com/2016/09/03/creating-custom-sitecore-xdb-contact-facets/) that summarizes current practices and how to add facets.

Today we're going to discuss how to syntactically improve the declaration of a contact facet class using syntaxes available in C# 6.0 (VS 2015) and C# 7.0 (VS 2017). It's important to note that the C# version is decoupled from the .NET framework version: the C# 7.0 compiler is perfectly capable of emitting C# 7 syntax to a .NET 4.5-targeted assembly, for instance. So you can use these modern language features as long as you've got the right version of MSBuild or Visual Studio :)

Here's the example Pete uses in his post, which follows [other examples](https://community.sitecore.net/technical_blogs/b/getting_to_know_sitecore/posts/introducing-contact-facets) out there as well:

```csharp
using System;
using Sitecore.Analytics.Model.Framework;
 
namespace SitecoreHacker.Sandbox.Facets
{
    [Serializable]
    public class MarketingData: Facet, IMarketingData
    {
 
        private const string CUSTOMER_ID = "CustomerId";
        private const string SEGEMENT = "Segment"; // sic :p
 
        #region Properties
        public string CustomerId
        {
            get { return GetAttribute<string>(CUSTOMER_ID); }
            set { SetAttribute(CUSTOMER_ID, value); }
        }
 
        public string Segment
        {
            get { return GetAttribute<string>(SEGEMENT); }
            set { SetAttribute(SEGEMENT, value); }
        }
        #endregion
         
        public MarketingData()
        {
            EnsureAttribute<string>(CUSTOMER_ID);
            EnsureAttribute<string>(SEGEMENT);
        }
    }
}
```

As you can see, the facet API requires string keys for the facet values - in this case stored as `const string` - to get and set them. Further, as Pete notes:

> I found out the hard way that the constants defined, the value must equal the actual name of the class property for the same attribute.

Well in C# 6 (VS 2015), there's a syntax for that. The [`nameof`](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/nameof) statement allows you to get the string name of a variable or property. This essentially hands off the management and maintenance of the `const` value to the compiler, instead of the developer.

So we can clean up this example by using `nameof` instead of constants - and get as a bonus refactoring support and compile-time validation of the names:

``` cs
using System;
using Sitecore.Analytics.Model.Framework;
 
namespace Elephant.Sandbox.Facets
{
    [Serializable]
    public class MarketingData: Facet, IMarketingData
    {
        public string CustomerId
        {
            get { return GetAttribute<string>(nameof(CustomerId)); }
            set { SetAttribute(nameof(CustomerId), value); }
        }
 
        public string Segment
        {
            get { return GetAttribute<string>(nameof(Segment)); }
            set { SetAttribute(nameof(Segment), value); }
        }
         
        public MarketingData()
        {
            EnsureAttribute<string>(nameof(CustomerId));
            EnsureAttribute<string>(nameof(Segment));
        }
    }
}
```

Finally if you have C# 7.0 (VS 2017), you can also utilize [expression bodied members](https://blogs.msdn.microsoft.com/dotnet/2017/03/09/new-features-in-c-7-0/) to further clean up the property syntax:

``` cs
using System;
using Sitecore.Analytics.Model.Framework;
 
namespace Rhino.Sandbox.Facets
{
    [Serializable]
    public class MarketingData: Facet, IMarketingData
    {
        public string CustomerId
        {
            // expression bodied members turn the single-expression get 
            // into a lamdba-style syntax, removing the need for braces
            get => GetAttribute<string>(nameof(CustomerId));
            set => SetAttribute(nameof(CustomerId), value);
        }
 
        public string Segment
        {
            get => GetAttribute<string>(nameof(Segment));
            set => SetAttribute(nameof(Segment), value);
        }
         
        public MarketingData()
        {
            EnsureAttribute<string>(nameof(CustomerId));
            EnsureAttribute<string>(nameof(Segment));
        }
    }
}
```

So there - now go forth and put your data in the xDB :)