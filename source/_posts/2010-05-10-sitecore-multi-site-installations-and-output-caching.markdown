---
layout: post
title: "Sitecore multi-site installations and output caching"
date: 2010-05-10 12:22:42 -0800
comments: true
categories: Sitecore
---
When running Sitecore in a multi-site configuration you may run into an odd issue: output caching may seem to get too greedy and not clear when you'd expect it to.

There's a simple culprit: the default Sitecore setup includes an event handler, `Sitecore.Publishing.HtmlCacheClearer`, that is invoked on the `publish:end` event. This event handler has a list of sites assigned to it, and the default is "website" - great, until you need to have more than one site and publishing doesn't clear your site's output cache. Fortunately it's easy to configure more sites: just add more site nodes to the XML. You cannot however use config includes to allow each site to individually add itself to the list from its own config file.

There's also a nuclear option: you can implement your own event handler that clears all sites' caches. I'm not sure if this would have a detrimental effect on any of the system sites (i.e. shell), but you could exclude it. An example of doing that:

	string[] siteNames = Factory.GetSiteNames();
	for (int i = 0; i < siteNames.Length; i++)
	{
	   SiteInfo siteInfo = Factory.GetSiteInfo(siteNames[i]);
	   if (siteInfo != null)
	   {
			siteInfo.HtmlCache.Clear();
	   }
	}