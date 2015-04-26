---
layout: post
title: "Scoping searches to the current context in Sitecore 7's LINQ"
date: 2013-05-10 22:57:54 -0800
comments: true
categories: 
- Sitecore 
- Lucene
---
As a followup to my <a href="http://kfigy.blogspot.com/2013/04/sitecore-7-linq-gotchas.html">previous post</a> about gotchas with Sitecore 7's LINQ provider, here's another thing to consider.<br><br>
The indexing providers are very greedy about indexing. This means that unlike with traditional querying with Sitecore.Data.Item, where your results are automatically filtered by the context language and latest item version, <i>no such filtering occurs with LINQ</i>. You will receive all versions and all languages unless you specify otherwise.<br><br>
As you might imagine, this can result in unexpectedly large quantities of search results in some cases. It can also be extra sneaky since during development you might only have one version in one language - so you wouldn't even notice the issue.<br><br>
So how do you fix the issue? First, let's talk about versions. The default indexing configuration includes a field called <i>_latestversion</i>.
This is a boolean field that is only set to true for items that are the latest version in their language. We can take advantage of this by implementing a property on our mapped objects and mapping it to this index field like so:<br><br><pre>[IndexField("_latestversion")]
public bool IsLatestVersion { get; set; }</pre>
<br>
Then, when we write a query we want to limit, we simply add a clause to the query:<br><br><pre>.Where(x => x.IsLatestVersion)

// alternatively if you don't want to be strongly typed and have an indexer, you can use
.Where(x=> x["_latestversion"] == "1")</pre>
<br>
Now you'll only get the latest version. Now for languages, which are also pretty simple. If you're inheriting from the SearchResultItem class, you already have a Language property. Otherwise you can add one like so:<br><br><pre>[IndexField("_language")]
public string Language { get; set; }</pre>
<br>
Then, we add the following clause to the query:<br><br>
<pre>.Where(x =&gt; x.Language == Sitecore.Context.Language.Name)</pre>

<p>Now we get results like regular queries. If you're like me, the next question you're asking is "how can I just write this once and forget about it?" For example something like:</p>

<pre>public static IQueryable&lt;T&gt; GetFilteredQueryable&lt;T&gt;(this IProviderSearchContext context)
    where T : MyItemBaseClass
{
    return context.GetQueryable&lt;T&gt;().Where(x =&gt; x.Language == Sitecore.Context.Language.Name && x.IsLatestVersion);
}</pre>

<p>Unfortunately, this seems to be nigh impossible with the current revision of Sitecore 7. The issue has to do with how LINQ resolves expressions involving generic types that are not resolved at compile time. Effectively the expression in the example above converts to:</p>

<pre>.Where(x=> ((T)x).Language == Sitecore.Context.Language.Name)</pre>

<p>Notice the cast to T? That throws the expression parser for a loop. I've been told this will be fixed in a later release of Sitecore 7, but will not be part of the RTM release, so for the moment it looks like we're writing filtering on each query.</p>