title: Unicorn 4 Released
date: 2017-06-22 12:57:43
categories: Unicorn
---
![unicorn](https://kamsar.net/nuget/unicorn/logo.png)

I'm happy to announce the final release of Unicorn 4.0! Unicorn 4 comes with significant performance and developer experience improvements, along with bug fixes. Unicorn 4 is available from NuGet or [GitHub](https://github.com/kamsar/Unicorn/releases/tag/4.0.0).

# Project Dilithium and Performance

Unicorn 4 is faster - a lot faster. Check out these benchmarks:

![performance benchmarks](https://kamsar.net/index.php/2017/02/Unicorn-4-Preview-Project-Dilithium/perf.png)

The speed increase is due to optimized caching routines, as well as the _Dilithium_ batch processors. Dilithium is an optional feature that is off by default: because of its newness, it's still experimental. I'm using it in production though. Give it a try - it can always be turned off without hurting anything.

For more detail into how Unicorn 4 is faster, and what Dilithium does, [check out this detailed blog post](https://kamsar.net/index.php/2017/02/Unicorn-4-Preview-Project-Dilithium/).

# Improved Modular Configuration

Unicorn 4 features a refactored configuration system that is designed to support [Sitecore Helix](http://helix.sitecore.net/) projects with an improved configuration experience. The new config system is completely backwards-compatible, but now enables _configuration inheritance_, _configuration variables_, and _configuration extension_ so that modular projects can encode their conventions (e.g. paths to include, physical paths) into one base config and all the module configs can extend it.

This drastically reduces the verbosity of the module configurations, and improves their maintainability by allowing conventions to be [DRY](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself). Here's a very simple example of a base conventions configuration:

```xml
<configuration name="Habitat.Feature.Base" abstract="true">
  <targetDataStore physicalRootPath="$(sourceFolder)\$(layer)\$(module)\serialization" />
  <predicate>
    <include name="$(layer).$(module).Templates" database="master" path="/sitecore/templates/$(layer)/$(module)" />
  </predicate>
</configuration>
```

And here's a module configuration that extends it:

```xml
<configuration 
	name="Feature.News" 
	extends="Habitat.Feature.Base">
	<!-- automatically stores items at $(sourceFolder)\Feature\News\serialization -->
	<!-- automatically includes /sitecore/templates/Feature/News -->
</configuration>
```

There's a lot more that you can do with the configuration enhancements in Unicorn 4 too. For additional details, [read this extensive blog post](https://kamsar.net/index.php/2017/02/Unicorn-4-Part-III-Configuration-Enhancements/).

# Sitecore PowerShell Extensions Support

![unicorn + spe](https://kamsar.net/index.php/2017/02/Unicorn-4-Preview-Part-2-SPE-Support/spe.png)

Just about anything you can do with Unicorn can now be automated using Sitecore PowerShell Extensions in Unicorn 4. You can now run Unicorn SPE cmdlets to...

* Run a sync
* Serialize an item or tree of items
* Dump an item to YAML in the console
* [Create a Sitecore package of all the items in a Unicorn configuration](https://kamsar.net/index.php/2017/02/Unicorn-4-Preview-Part-2-5-Generating-Packages-with-SPE/)
* And there's more, all [in this blog post](https://kamsar.net/index.php/2017/02/Unicorn-4-Preview-Part-2-SPE-Support/) - and [this one, about packaging](https://kamsar.net/index.php/2017/02/Unicorn-4-Preview-Part-2-5-Generating-Packages-with-SPE/)

# Console Output Scaling

The Unicorn console has received a serious upgrade in Unicorn 4. If you’ve ever run a sync that changed a large number of items from the Unicorn Control Panel, you may have noticed the browser slow to a crawl and the sync seem to almost stop. The console that underpins Unicorn 3 and earlier started to choke at around 500 lines.

No longer! Unicorn 4's upgraded console has spit out 100,000 lines without a hitch, and it should scale beyond that.

The automated tool console (PowerShell API) has also received an upgrade. Previously the tool console buffered all the output of a sync before sending it back. This caused problems in certain environments, namely Azure, where TCP connections that don't send any data for more than four minutes are terminated. This would cause long-running syncs in Azure to die unexpectedly.

In Unicorn 4 the automated tool console emits data in a stream just like the control panel console. There’s also a heartbeat timer where if no new console entries are made for 30 seconds, a `.` will be sent to make sure the connection is kept active.

The streaming tool console also requires updating your `Unicorn.psm1` file - not only will you get defense against TCP timeouts, you'll also be able to see the sync occur in real time using the PSAPI just like you would from the control panel. No more waiting until it's done to see how things are going :)

# Predicates can Exclude by Template ID or Name

Unicorn 4 can now exclude items from a configuration by template ID, thanks to [Alan Płócieniak](https://alan-null.github.io/2017/03/custom-unicorn-predicate). See also [Alan's original post](https://alan-null.github.io/2017/03/custom-unicorn-predicate) on the technique.

```xml
<include name="Template ID" database="master" path="/sitecore/allowed">
  <exclude templateId="{3B4F2B85-778D-44F3-9B2D-BEFF1F3575E6}" />
</include>
```

You can also exclude items by a regular expression of their name. This enables scenarios such as wanting to include all templates, but exclude all `__Standard values` items.

```xml
<include name="Name pattern" database="master" path="/sitecore/namepattern">
  <exclude namePattern="^__Standard values$" />
</include>
```

The complete grammar for predicates is always in the [predicate test config](https://github.com/kamsar/Unicorn/blob/master/src/Unicorn.Tests/Predicates/TestConfiguration.xml#L32).

# Breaking Changes

Unicorn 4's breaking changes do not break any common use-cases of Unicorn, but review them to see if they affect you.

- The `__Originator` field is now [serialized by default](https://github.com/kamsar/Unicorn/commit/f7d073b255891ec487e9a092fb2fa172ea77b9ac). This enables proper tracking of the origin of items instantiated from branch templates.
- Multithreaded sync support has been removed due to Sitecore bugs preventing realistic use. This was disabled by default already. Dilithium is faster than multithread sync ever was.
- The Rainbow `UseLegacyAttributeFormatting` (formats items in Unicorn < 3.1 format) setting has been removed. The new format is now always used. This has always been off by default.
- Rainbow `FieldComparer`s are no longer activated using the Sitecore Factory, so they only support parameterless constructors (this would only affect custom comparers; the stock ones have always been parameterless)
- Due to the console upgrades, a new `Unicorn.psm1` is required if you are using Unicorn's PowerShell API. This file also now ships in the NuGet package, so you can be sure you're getting the right version for your Unicorn.

# Bug Fixes

- Transparent sync misc fixes (to renaming, saving display names, instantiating branch templates into TpSync areas)
- Renaming an item and changing fields on it in one operation (only possible with Sitecore API) now no longer loses the additional field data in the serialized item
- Improved output and logging, clarified messaging, improved developer experience
- Content editor warnings now handle items in more than one configuration correctly
- Control panel now displays which paths are invalid when initial serialization is blocked due to invalid include paths
- The required password length for user serialization is now configurable, should you really really want to use `b`
- Using Fast Query while any Transparent Sync configuration is active will now log a warning to the Sitecore logs (fast query cannot find transparent sync items, so items may not be returned as expected). This can be disabled if your logs start to fill with spam and you understand the issue.
- PowerShell API now defaults to not write secrets to the console (debug is off by default) for secure-by-default-ness
- Fixed a background exception that could occur when modifying serialized items on disk in rare cases, which could cause the app pool to terminate [#222](https://github.com/kamsar/Unicorn/issues/222)
- The Rainbow data cache now correctly invalidates if an item is moved or renamed on disk after being added to the cache
- Choosing many configurations to sync will no longer push the sync button off the page [#232](https://github.com/kamsar/Unicorn/pull/232)
- The default console logging level for interactive syncing has been changed from Info to Debug, since there is no longer a scaling issue with the console output. This provides better insights into what has been changed on items without needing to see the Sitecore logs

# Upgrading

If you’re coming from classical Unicorn 3.1 or later, upgrading is actually really simple: just upgrade your NuGet package. Unicorn 4 changes nothing about storage or formatting (except that the `__Originator` field is no longer ignored by default), so all existing serialized items are compatible.

If you’re invoking Unicorn via its remote PowerShell API, make sure to upgrade your `Unicorn.psm1` to the Unicorn 4 version to ensure correct error handling with the streaming console.

# Thanks

Thank you to the community members who contributed code and bug reports to this release.

* [chemaxa](https://github.com/chemaxa)
* [Raymond Clemens](https://github.com/Rclemens)
* [Niles11](https://github.com/niles11)
* [Luuk Ostendorf](https://github.com/lordstyx)
* [Alan Płócieniak](https://github.com/alan-null)
* [James Skemp](https://github.com/JamesSkemp)
* [t00ki](https://github.com/t00ki)
* [Wim Vergouwe](https://github.com/WimVergouwe)

# Address

{% asset_img Elephant.jpg "early design for Unicorn 5" %}