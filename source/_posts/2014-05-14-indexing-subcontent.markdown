---
layout: post
title: "Indexing Subcontent"
date: 2014-05-14 20:13:05 -0700
comments: true
categories: sitecore
---

Often in Sitecore solutions we will utilize the page component architecture, where a page is composed of a base item and one or more "subcontent" items that are data sources for renderings that we have added to the page to customize its layout. This grants editors extreme flexibility to customize what is shown on their pages, but causes a major headache if you're trying to have your search index contain the full content of a page because now the content is spread across potentially many backend items.

_Note: While the code presented in this article is mine, the original idea was my coworker Erik Brown's. Unfortunately he has no Twitter or blog that I know of, but if he ever gets one you should follow it :)_

The solution we came up with utilizes Sitecore 7's _computed index fields_ (blog posts: [John West](http://www.sitecore.net/Community/Technical-Blogs/John-West-Sitecore-Blog/Posts/2013/03/Sitecore-7-Computed-Index-Fields.aspx), [Martina Welander](http://www.sitecore.net/Community/Technical-Blogs/Martina-Welander-Sitecore-Blog/Posts/2013/09/Sitecore-7-Search-Tips-Computed-Fields.aspx)), and a handy but less than obvious trick. 

The trick is that the Sitecore indexes can contain more than one value for the same index field. This means that if you want to augment the value of the system `_content` field (which contains all the indexable text fields on the item), all you have to do is define a computed field that _is also named_ `_content`. Now the `_content` index field stores two values: the system value, and your computed value - and it will search on both values when you query it.

Armed with this knowledge I set out to use the layout field on the base item to find all rendering datasources it points to and use the same code that Sitecore uses to create the `_content` value for the main item. This turned out to not be all that hard (though I had the advantage of having done a lot of layout field messing around before):

{% gist d6bfcdd2f3ee7163f0f9 %}

Registering the computed field to augment the main `_content` index field requires a config patch file (note that this targets Lucene; your patch for SOLR or Coveo would be different):

{% gist 7b76515efca7a48f8911 %}

Another handy tweak for indexing this kind of item is to use [Nick Wesselman](https://twitter.com/techphoria414)'s [handy code to cause the main item to be updated in the index whenever any of its datasource items are modified](http://www.techphoria414.com/Blog/2013/November/Sitecore-7-Computed-Fields-All-Templates-and-Datasource-Content). Nice work!

Hope you find this helpful :)