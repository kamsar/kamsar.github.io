title: "Synthesis Object Mapping Performance"
date: 2015-06-11 06:35:10
categories: Synthesis
---
This morning I saw a post on the [performance of the Glass Mapper by Konstantin Cherkasov](http://sitecorepro.blogspot.com/2015/06/sitecore-and-glassmapper-how-much.html) and thought I'd run a quick test of Synthesis in a similar situation. In Konstantin's testing, he found a `GlassCast` + field access to be nearly 2,000 times slower than using the Sitecore API to access the same field value.

I decided to read the entire contents of the web db, then convert it all to Synthesis items and read the display name field, since they'd all have that. In my case, that worked out to 5148 items - less than Konstantin's test with 20,000 but it was what was handy. Note that with Synthesis, `AsStronglyTypedCollectionOf()` returns a collection of `IStandardTemplateItem`. This doesn't mean it's not doing any fancy mapping, however - this is just the base interface for all items. Individual collection items are still mapped to the appropriate template class; e.g. for an Article, you could do `if(collectionItem is IArticleItem)` and it would be true.

[This gist shows the code I used, and average results](https://gist.github.com/kamsar/f2aa92ef3f63f3c7b931). Synthesis was about 3x slower than the native Sitecore APIs (native: 16-20ms, Synthesis: 54-60ms, sample size 5148 items); still suboptimal, but a heck of a lot less than 2000x. 

The difference is probably mostly due to the fact that Synthesis is a wrapper (internally it reads from an `Item`) instead of a mapper (full POCOs/proxies), so it can be a lot lazier about loading values.