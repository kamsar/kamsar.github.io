title: "Unicorn 3.0 Released"
date: 2015-10-02 11:00:00
categories: Unicorn
---

![patrick you're a bloody genius](http://kamsar.net/index.php/2015/09/Unicorn-3-What-s-new/Unicorn_logo.png)

It's been over a year since the last major release of Unicorn, but I haven't been bored. Today I'm happy to announce the release of Unicorn 3.0 and Rainbow 1.0.

# Unicorn 101

Sitecore development artifacts are both code and database items, such as rendering code and rendering items. As developers, we use serialization to write our database artifacts into source control along with our code so that we have a record of all our development artifacts. [Unicorn](https://github.com/kamsar/Unicorn) is a tool to make serializing items and sharing them across teams and deployed environments easy and fun.

Unicorn takes advantage of keeping serialized items always up to date on the disk, which allows all merging to take place using your source control tool. Modern source control tools such as Git or SVN are capable of automatically merging the vast majority of item changes, which saves you time and reduces the chance of human error when merging.

Unicorn 3 is fresh off the compiler and brings with it a [huge raft of improvements](http://kamsar.net/index.php/2015/09/Unicorn-3-What-s-new/). This version brings its own [serialization format](http://kamsar.net/index.php/2015/07/Rethinking-the-Sitecore-Serialization-Format-Unicorn-3-Preview-part-1/) and [filesystem hierarchy](http://kamsar.net/index.php/2015/08/Reinventing-the-Serialization-File-System-Rainbow-Preview-Part-2/) that are far more friendly to source control and merge conflicts than the default Sitecore format. It's also ridiculously fast - about 50% faster overall than Unicorn 2 or Sitecore serialization APIs.

# What else is new?

In fact there are _lots more_ new features added to Unicorn 3 since the [what's new blog post](http://kamsar.net/index.php/2015/09/Unicorn-3-What-s-new/) was written. Not just minor details either! Read on...

## Transparent Sync
This is a feature that [demanded its own blog post](http://kamsar.net/index.php/2015/10/Unicorn-Introducing-Transparent-Sync/)! Transparent Sync enables Unicorn to sync the serialized items in real time, completely automatically. It does this by using its data provider to directly read the serialized items and inject them into the Sitecore content tree. The items on disk **are** the items in Sitecore: it bypasses the database entirely for transparently synced items.

Seriously, it's magic. [Transparent Sync](http://kamsar.net/index.php/2015/10/Unicorn-Introducing-Transparent-Sync/) might be the best feature of Unicorn 3. You should give it a try!

## New Configuration Architecture

Previously Unicorn was distributed with its default example configuration directly in `Unicorn.config`. This was suboptimal because it encouraged modifying the stock config file and making future upgrades more of a merging challenge than they needed to be. In the new order, Unicorn ships _without_ any configurations defined. There are two [example](https://github.com/kamsar/Unicorn/blob/master/src/Unicorn/Standard%20Config%20Files/Unicorn.Configs.Default.example) configuration-adding [patch files](https://github.com/kamsar/Unicorn/blob/master/src/Unicorn/Standard%20Config%20Files/Unicorn.Configs.NewItemsOnly.example) distributed with the NuGet package which you can duplicate and edit to your desires, and the [README distributed with the NuGet package](https://github.com/kamsar/Unicorn/blob/master/Build/Unicorn.nuget/readme.txt) details what you need to do to get started.

## Control Panel UX Improvements

The control panel has been redesigned to have an improved UX when there are many configurations defined. In the old UI, once more than a couple configurations were defined the page would need to scroll. The new UI reduces the vertical space for each configuration significantly:

{% asset_img newui.png "new control panel" %}

The new control panel also has an experience for selecting only the configurations you wish to sync in a single batch:

{% asset_img newui-multiselect.png "new control panel" %}

For automated builds, you can also copy the URL on the sync button when you've got multiple configurations selected in the control panel to run the same batch at build time. This is great if you've got some configurations that you may not wish to sync during a CI build.

## New Items Only Evaluator

This is an [evaluator](https://github.com/kamsar/Unicorn#evaluator) that changes the behavior of syncing to only write _new_ items from serialization into Sitecore. Existing items with the same ID or items that are not serialized are left alone. This can be useful for example if you're wanting to push some developer-initiated content items up, like metadata or lookup source items, that you want to always exist but once they have been created they become the content editor's to do with as they will. A [sample configuration](https://github.com/kamsar/Unicorn/blob/master/src/Unicorn/Standard%20Config%20Files/Unicorn.Configs.NewItemsOnly.example) that enables the NIO evaluator ships with the NuGet package. Thanks to [Nathanael Mann](https://twitter.com/cardinal252) for the feature request.

## Exclude all children with the predicate

To exclude all children of an item simply add a trailing slash to the `<exclude>` like so:

	<include database="master" path="/sitecore/content">
		<exclude path="/sitecore/content/" />
	</include>

## Visual Studio Control Panel Support

Recently a [plugin for Visual Studio](https://visualstudiogallery.msdn.microsoft.com/64439022-f470-422a-b663-fbb89aaf6e86) was released that enables you to run Unicorn syncs directly within Visual Studio. Unicorn 3 supports this tool out of the box, by simply enabling the `Unicorn.Remote.config.disabled` patch file. 

{% asset_img visualstudio.png "new control panel" %}

The [Unicorn Control Panel for Visual Studio](https://visualstudiogallery.msdn.microsoft.com/64439022-f470-422a-b663-fbb89aaf6e86) requires VS2013 or VS2015. It is developed by [Andrii Snihyr](https://twitter.com/berserkerdotnet) - thank you for the community support!

## TFS Support

Fellow Sitecore MVP and Connective DX'er [Dave Peterson](https://twitter.com/petersondave) has written up a [plugin for Rainbow/Unicorn that integrates it with TFS](https://github.com/PetersonDave/Rainbow.Tfs). TFS 2010 (or server workspaces in 2012 and later) have the unfortunate limitation that all files are locked by default. This interferes with Unicorn's operation as it writes files as items are changed in Sitecore, resulting in errors.

The TFS integration hooks the TFS API to Rainbow's SFS data store, causing it to actually check out the file Unicorn is about to write or delete before acting. You must use a 32-bit IIS app pool when using the plugin, as the TFS API is 32-bit only.

The TFS plugin is available on NuGet; for installation instructions and the latest info [see the README on GitHub](https://github.com/PetersonDave/Rainbow.Tfs/blob/master/README.md)

## Documentation

The documentation in the [README](https://github.com/kamsar/Unicorn/blob/master/README.md), comments in the [stock config files](https://github.com/kamsar/Unicorn/tree/master/src/Unicorn/Standard%20Config%20Files), and verbiage in the actual control panel and logs have all been reviewed and improved. If you get confused by anything, let me know and I'll make the docs better or the error message clearer in the next release.

## Thank you

Unicorn is a project by and for the Sitecore community. I'd like to thank everyone who's contributed to the project in a major or minor way: without you, we wouldn't have this tool today. Thank you!