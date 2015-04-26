---
layout: post
title: "Have two web databases? Don't forget the IndexUpdateStrategy"
date: 2014-01-13 20:12:03 -0800
comments: true
categories: 
- Sitecore 
- Search
---

If you have two web databases in a Sitecore 7 site (e.g. for a pre-production preview or staging site) and you're using Lucene here's something to be aware of:

Simply copying the `Sitecore.ContentSearch.Lucene.Index.Web.config` file, renaming the index and changing the `<locations>` entr(ies) is not sufficient to make your new index work correctly! You'll  notice that if you publish items to that database that the updates don't make it into the index. Why not?

Simply put, the default [`IndexUpdateStrategies`](http://www.sitecore.net/Community/Technical-Blogs/John-West-Sitecore-Blog/Posts/2013/04/Sitecore-7-Index-Update-Strategies.aspx) are confusingly named. The web index defaults to the `onPublishEndAsync` strategy, which makes sense to use for our staging database as well - we want the index to get updated by the event queue after a publish completes. But there's one minor detail: the `onPublishEndAsync` strategy should really be called `onPublishEndAsyncWebdb` or something similar. Let's look at how the strategy is defined:

    <onPublishEndAsync type="Sitecore.ContentSearch.Maintenance.Strategies.OnPublishEndAsynchronousStrategy, Sitecore.ContentSearch">
        <param desc="database">web</param>
        <CheckForThreshold>true</CheckForThreshold>
    </onPublishEndAsync>

Derp - that `<param desc="database">web</param>`! The strategy actually is only watching the event queue for updates from a single database - and that database is not our staging database. The solution is to define a copy of the `onPublishEndAsync` strategy for your staging index and change the database parameter, then change your index declaration to reference that strategy instead:

In app_config\include\sitecore.contentsearch.lucene.defaultindexconfiguration.config (or a patch thereof):

    <onPublishEndAsyncStaging type="Sitecore.ContentSearch.Maintenance.Strategies.OnPublishEndAsynchronousStrategy, Sitecore.ContentSearch">
	  <param desc="database">web_staging</param>
	  <CheckForThreshold>true</CheckForThreshold>
    </onPublishEndAsyncStaging>

In App_Config\Include\Sitecore.ContentSearch.Lucene.Index.Web_Staging.config (or wherever you're defining your staging index), reference the new strategy instead of the existing strategy:

    <strategy ref="contentSearch/indexUpdateStrategies/onPublishEndAsyncStaging" />

Note that if you defined an index that spanned multiple databases you'd need to define multiple strategies for its updates, and reference all of them from the index config - an index can have multiple update strategies.

Hope this is helpful :)