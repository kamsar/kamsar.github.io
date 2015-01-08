---
layout: post
title: "Quick note: Sitecore MVC partials"
date: 2015-01-07 19:36:25 -0800
comments: true
categories: sitecore
---

Here's a bit of an obtuse trick that lets you use `@Html.Partial()` effectively on Sitecore MVC View renderings without specifying the full path to the partial you want to render. You might think that regular MVC syntax would work here:

	@Html.Partial("_MyPartial")

But you'd be wrong. Instead, you'll get an error that the partial could not be found, and the perplexing message that it tried to search `[path-to-views]/Sitecore/_MyPartial.cshtml`. Where'd that "/sitecore" come from? To understand this, a bit of background on how ASP.NET MVC resolves partials is in order.

The view resolution convention is based on _the controller name_ that executes the view, _not the view's physical location_. This makes sense in a pure ASP.NET MVC environment where every view is the result of a controller. However in a View Rendering, there's just the cshtml. Except that in the background, Sitecore executes the View Rendering using a shim controller - a shim that goes by `SitecoreController`. This is where the `/Views/Sitecore` search path comes from.

So what's the trick? Well, as long as all your views are in a MVC-like hierarchy, you can refer to the partial using the parent path:

	@Html.Partial("../ViewParentFolder/_MyPartial")

This jumps out of the controller rendering folder (always Sitecore) and lets you specify the parent folder directly instead, without resorting to a full absolute path which - especially if you use areas or moved views around later - could break.

In case you're wondering why I'm using native partials instead of `@Html.Sitecore().Rendering()` or `@Html.Sitecore().ViewRendering()` and such, these are static components of a MVC layout that are shared between multiple layouts - so it doesn't make much sense to also register them as dynamic renderings and take the performance hit of that lookup.