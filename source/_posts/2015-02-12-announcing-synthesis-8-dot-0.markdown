---
layout: post
title: "Announcing Synthesis 8.0"
date: 2015-02-12 21:57:15 -0800
comments: true
categories: synthesis sitecore
---

I am happy to announce the availability of [Synthesis](https://github.com/kamsar/Synthesis) 8.0. Synthesis is an object mapping framework for Sitecore that takes advantage of integrating the code generation with the mapping framework. This enables very high speed object mapping, LINQ-to-indexes support, model sync checking, and deep interface support for template inheritance and unit testability.

## What's new in Synthesis 8.0

### Sitecore MVC Support

There are relatively few reasons to stick with Web Forms any longer for Sitecore development. Synthesis 8 introduces full-scale Sitecore MVC support, including:

* A high-speed model provider for view renderings: you can use `@model IMySynthesisTypeItem` transparently on view renderings
* HTML helpers designed to make it easy to work with Synthesis models and enable Experience Editor (e.g. `@Html.ImageFor(x=>x.MyImageField)`)
* Invalid model types on view renderings are trapped and contained to only the rendering involved, preventing the whole page from exploding due to one rendering failing. Messaging is shown to preview and editing users to show the failed rendering.
* Diagnostic HTML comments are injected around renderings when dynamic debug compilation is enabled, showing the start and end of each rendering, its name, render time, and output cache diagnostics.

The functionality of the `Synthesis.Mvc` package is largely similar to [Blade](https://github.com/kamsar/Blade) combined with [Synthesis.Blade](https://github.com/kamsar/Synthesis/tree/cf6500c2a0949b1fc0a207b81c2771801af41bf9/Source/Synthesis.Blade) except that Blade is for Web Forms and Synthesis.Mvc is for Sitecore MVC. The Synthesis.Mvc [HTML helpers](https://github.com/kamsar/Synthesis/wiki/Using-Synthesis-with-Blade#razor-extensions) are fully compatible with Synthesis.Blade's.

### Multiple configurations

Synthesis now supports more than one model configuration. This enables, for example, generating a separate model per-site for a multi-site installation of Sitecore. The additional configurations are registered similarly to MVC areas, using either WebActivator, the `<initialize>` pipeline, or Global.asax. There is some [basic documentation](https://github.com/kamsar/Synthesis/wiki/Using-Multiple-Configurations) available.

Configurations may also be _friends_, which enables sharing template classes across configurations that are aware of each other. For example, if you have a 'shared' set of templates that multiple sites' templates derive from, making the shared model a 'friend' of your site model will enable your site model to derive from the shared model's interfaces.

### New control panel

Synthesis 8 uses a control panel that is hooked from the `httpRequestBegin` pipeline instead of using a HTTP handler and HTTP module. This enables Synthesis to require no web.config changes at all to install or remove. The control panel is multiple-configuration aware.

This control panel uses a configurable activation URL to allow you to put it where you want it. The default is `/synthesis.aspx`.

## Installation and Upgrade

Synthesis 8 is [available from NuGet](https://www.nuget.org/packages/Synthesis) right now. Initial installation is as simple as installing the `Synthesis` NuGet package (and, if you want MVC support the `Synthesis.Mvc` package). There is a README that opens after installation that helps you get started and configure it.

Upgrading does require merging the [current Synthesis.config](https://github.com/kamsar/Synthesis/blob/master/Source/Synthesis/Synthesis.config) with your existing one. A few settings have been added or modified in this version to support the control panel and multiple configurations. You will also need to remove the Synthesis HTTP handler and HTTP module from web.config if you have them registered. Otherwise upgrade is as simple as upgrading the NuGet package and, if appropriate, installing the MVC package.

Synthesis 8.0 is designed for Sitecore 7.2 and later due to content search API versioning, and was primarily developed and tested on Sitecore 8.0 Update-1.

## Under the hood changes

* Modular generator: The code generator has been refactored and improved into a two-stage design where _metadata_ (e.g. template hierarchy) is generated first, and passed to a code generator. This enables plugging in custom code generators if desired (if you're feeling brave, it should be completely possible to say emit a Glass model from Synthesis' generator). Metadata generation also enables multiple configurations to refer to each other.
* Sitecore-isolated generation classes: All providers involved in generation no longer have any direct hooks to the Sitecore API. This hypothetically enables non-Sitecore database datasources for generation - for example, .item files on disk.
* The default `TypeListProvider` allows for wildcards when specifying assemblies to scan for model types. Useful for cases where you have an assembly per site and don't want to register them all individually.
* The NuGet package now includes a README file that provides some basic directions for getting started with Synthesis, as this has been a previously lacking area of the documentation
* Generated Synthesis models are now less prone to causing SCM merge conflicts. The Synthesis version stamp has been reduced to x.x instead of x.x.x.x on attributes, and the 'generated by .net framework version x' is removed from the header, which could cause problems if one developer had say .18072 and the other .17345; producing constant pointless model changes in the repository.
* `IStandardTemplateItem` now has a `Children` property, which is an alias to `.Axes.GetChildren()`, to make it feel more at home for those used to the `Item` API.

## Bug fixes

* There was a problem with the way the template inheritance cycle rejection was implemented in 7.3.3 that caused multilevel inheritance on templates who had fields named the same as their template would generate invalid models. This has been fixed.
* Setting the value of a `HyperlinkField.Href` with an external URL value (e.g. http://google.com) would cause it to save the link as an internal link type and Sitecore to flag it as a broken link. These are now correctly set as link type external.
* When using LINQ-to-indexes and calling the `GetResults()` extension method, an exception will no longer be thrown. The exception is due to some misbehaving private reflection deep in Sitecore, and the workaround is even more private reflection. The authorities have been notified ;)