title: Synthesis 8.2.2 Released
date: 2016-11-18 14:12:34
categories: Synthesis
---

I am happy to announce the release of [Synthesis](https://github.com/kamsar/Synthesis) 8.2.2. 

Synthesis is an object mapping framework for Sitecore that enables developing more reliable and maintainable sites in less time than traditional Sitecore development. It is a strongly typed template object generator that is easily understandable for developers with either a Sitecore or traditional .NET background.

## What's new?

### Automatic Model Regeneration

Synthesis now ships by default with event handlers that automatically regenerate your model classes when templates are changed, renamed, moved, or deleted. Whereas before you'd have to manually request a regeneration, now you can just save the template and within 2-3 seconds your model classes have been updated.

Automatic Model Regeneration turns itself off if `<compilation debug="false">` is set in the web.config. You can choose to disable it in published scenarios either this way or by deleting its `Synthesis.AutoRegenerate.config` file when deploying. [#37](https://github.com/kamsar/Synthesis/issues/37)

### Hyperlink Tag Body Rendering

Previously Synthesis' MVC helpers did not have a way to render a hyperlink field with an arbitrary HTML body within (e.g. an image, etc). This has been rectified in 8.2.2, with the new `BeginHyperlinkFor` helper. [#31](https://github.com/kamsar/Synthesis/issues/31)

	@using(Html.BeginHyperlinkFor(m => m.HyperlinkField)) {
		<h1>Woohoo!</h1>
		<img src="homer.gif">
	}

### Config Files In Their Own Folder

The Synthesis configuration files have been moved to `App_Config/Include/Synthesis` for clarity. The NuGet package upgrade will take care of the migration.

### .NET Framework 4.5.2

Synthesis 8.2.2 requires that the project it is installed on be targeting the .NET Framework 4.5.2 or later. This enables full Sitecore 8.2 compatibility. Synthesis 8.2.2 should work on Sitecore 8.1 or later.

## Bug Fixes

* Model source files whose contents have not changed in the current regeneration are now not rewritten to disk. This prevents their timestamp from changing and thus triggering a need to rebuild their host project. When using modular architecture, this can greatly reduce build times. [#32](https://github.com/kamsar/Synthesis/issues/32)
* Fixed a bug when using Solr indexes where Synthesis' regenerate process would throw an exception and possibly result in an incomplete model [#34](https://github.com/kamsar/Synthesis/issues/34)
* Specifying a model output path in a directory that does not exist will now create that directory instead of erroring [#35](https://github.com/kamsar/Synthesis/issues/35)
* The content search configuration has been adapted to register the Synthesis `_templatesimplemented` computed field in such a way that it works with 8.1 and 8.2's new registration scheme, as well as with Solr indexes. Note that the Synthesis content search integrations (querying) do not otherwise support Solr still.
* You can now disable the generation of Content Search elements in your models if you wish, using the `EnableContentSearch` setting. The default value is set in `Synthesis.config`. This will enable using models with fewer project reference requirements, if you do not need the search integrations.

## Upgrading

If coming from Synthesis 8.2.x, the upgrade is via a simple NuGet upgrade. For earlier versions, consult the instructions on the release posts for the versions between what you're on and where you're going.

## Thanks

Thanks to the community members who contributed to this release.

* [Peter Holm](https://github.com/zimpy)
* [rolek](https://github.com/rolek)