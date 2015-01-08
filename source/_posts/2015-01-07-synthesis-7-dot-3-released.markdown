---
layout: post
title: "Synthesis 7.3 released"
date: 2015-01-07 19:59:09 -0800
comments: true
categories: sitecore synthesis
---

[Synthesis 7.3](https://www.nuget.org/packages/Synthesis) has been released to NuGet and GitHub.

The big new feature in 7.3 is the `Synthesis.Mvc` package, which adds a Synthesis-compatible Sitecore MVC model resolution mechanism. This enables you to create View Renderings that transparently can use a Synthesis type as a model without any additional work:

	@model IFooItem

	<h1>@Html.TextFor(x => x.SomeSitecoreField)</h1>

The Mvc library also includes:

 * Helpers to make it simple to render Synthesis fields (like `TextFor` above). If you've ever used [Blade](https://github.com/kamsar/Blade) these helpers are essentially identical. 
 * Rendering diagnostics, also similar to Blade, which adds HTML comments around renderings with useful data - like whether it was output cached - whenever the site is run with debug compilation enabled. A great tool if you work with frontend devs who never know what rendering markup came from.
 * Altered handling of invalid rendering model types. By default, an exception that takes the whole page down is thrown if a view declares an `@model` and receives something other than that type. This can cause issues if your datasource item is ever an invalid item ID or something similar. Synthesis.Mvc changes this behavior by simply hiding renderings with invalid datasource types; if in preview or edit mode a message is shown in place of the rendering explaining why it's hidden.

 Installing Synthesis.Mvc is easy: Just add the NuGet package to your web project. It consists of a single assembly in /bin and the `Synthesis.Mvc.config` file that registers pipelines in `/App_Config/Include`.

## Bug fixes

 Several bugs were also fixed in Synthesis 7.3:

* The mechanism Synthesis uses to generate media URLs was simplified as Sitecore 6.5 is no longer supported. This makes the media URLs on image and file fields valid in all cases, even when forcing media to have the server in the URL. Thanks to [Jeremy Clifton](https://github.com/jeremyclifton) and [Robert Pate](https://twitter.com/robertpateii) for the PR.
* The Synthesis-wrapped version of `Axes.GetPreviousSibling()` incorrectly called `GetNextSibling()` internally. Thanks to [@ullmark](https://twitter.com/ullmark) for the PR
* If two templates formed an inheritance cycle, generating Synthesis would cause a stack overflow. This is no longer the case, however it will still emit code that won't compile (as the interfaces will create a cycle in C#). Thanks to [@GreyGhostStudio](https://twitter.com/greyghoststudio) for the issue report.
* The constructor of `TestNumericField` incorrectly took an `int?` when it should have been `decimal?`. Thanks to [Robert Hardy](https://github.com/roberthardy) for the PR.

## Upgrading to 7.3

The configuration format for 7.3 has no changes, so upgrading is as simple as upgrading your NuGet packages. The Mvc support is a new separate NuGet package that would need to be installed.