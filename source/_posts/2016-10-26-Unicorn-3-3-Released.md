title: Unicorn 3.3 Released
date: 2016-10-26 20:54:07
categories: Unicorn
---

{% img /nuget/unicorn/logo.png %}
I'm happy to announce the release of Unicorn 3.3 and Rainbow 1.4. This is primarily a maintenance and bug fix release and is recommended for all users of Unicorn 3.x.

As there was no official announcement for Unicorn 3.2, this will also note what was new in 3.2. Items from 3.2 will be noted as such.

## What's new?

### Improved Configuration Dependencies

Dependencies between configurations have been improved to include _implicit_ dependencies that are created when one configuration includes a path that is a child of another configuration. This simplifies the need to define explicit dependencies in many cases. [This config file](https://github.com/kamsar/Unicorn/blob/master/src/Unicorn/Standard%20Config%20Files/Unicorn.Configs.Dependency.config.example#L32) documents how this works and how to disable it if you need to. [#165](https://github.com/kamsar/Unicorn/issues/165)

### Improved Debugging with PowerShell API

You can now enable diagnostic logging of the CHAP base signature on the client and server side, so if you have issues getting the tool authentication working you can diagnose why. See [this](https://github.com/kamsar/Unicorn/blob/5b99817aa2e49d6c6a8e5ee8729ca9ccd465ea20/doc/PowerShell%20Remote%20Scripting/sample.ps1#L18) and [this](https://github.com/kamsar/Unicorn/blob/5b99817aa2e49d6c6a8e5ee8729ca9ccd465ea20/src/Unicorn/Standard%20Config%20Files/Unicorn.UI.config#L26) for configuration information. [#182](https://github.com/kamsar/Unicorn/issues/182)

### Full Sitecore 8.2 Compatibility

Unicorn and Rainbow now build against and require .NET 4.5.2 for Sitecore 8.2 compatibility. This means that _your project referencing Unicorn must also target framework 4.5.2 or later_, even if not using Sitecore 8.2. Unicorn should work with Sitecore 7 and later. [#175](https://github.com/kamsar/Unicorn/issues/175)

### (3.2) Improved Exclusion Grammar

You can now exclude children of a child of an included path. [#138](https://github.com/kamsar/Unicorn/issues/138)

### (3.2) User and Role Serialization

Unicorn 3.2 introduced YAML-based user and role serialization via the Unicorn.Users and Unicorn.Roles NuGet packages. User and role serialization works similarly to item serialization. To add this capability to your Unicorn installation you simply install the correct NuGet package and the readme will guide you, or you can read the documentation using the links below.

* [Roles README](https://github.com/kamsar/Unicorn/blob/master/Build/Unicorn.Roles.nuget/readme.txt)
* [How to configure roles](https://github.com/kamsar/Unicorn/blob/master/src/Unicorn.Roles/Standard%20Config%20Files/Unicorn.Configs.Default.Roles.config.example)
* [Users README](https://github.com/kamsar/Unicorn/blob/master/Build/Unicorn.Users.nuget/readme.txt)
* [How to configure users](https://github.com/kamsar/Unicorn/blob/master/src/Unicorn.Users/Standard%20Config%20Files/Unicorn.Configs.Default.Users.config.example)

## Bug fixes

{% asset_img fixedbugs.jpg %}

* When deserializing an item that contains a field missing in Sitecore, the field hint is shown instead of just its ID. This aids in tracking down the offending field, since it does not exist to find in the DB. [#169](https://github.com/kamsar/Unicorn/issues/169)
* The `Unicorn.SharedSecret.config.example` has been renamed to include a `z` so that simply renaming it to use does not cause a config load order issue [#181](https://github.com/kamsar/Unicorn/issues/181)
* Fixed an issue with null vs empty field values that could cause an exception in Rainbow [#16](https://github.com/kamsar/Rainbow/issues/16)
* Fixed an issue that would cause missing language data when you removed a version from a tracked item using the Sitecore API within an `EventDisabler` block [#168](https://github.com/kamsar/Unicorn/issues/168)
* Fixed an error that occurred when an item changed template, but the old template was already deleted as part of the same sync process and thus no longer existed [#164](https://github.com/kamsar/Unicorn/issues/164)
* An issue that could cause spurious "Item changed, do you want to overwrite?" popups when saving an item in Experience Editor and using Transparent Sync has been fixed [#163](https://github.com/kamsar/Unicorn/issues/163)
* Fixed an issue that could cause a spurious Unicorn conflict warning when saving items with checkboxes in certain cases [#151](https://github.com/kamsar/Unicorn/issues/151)
* Serialized items with field values that equaled the standard values defined in the database are no longer left as standard values, and are correctly explicitly set to their serialized value. This was a common cause of items that 'synced every time'. [#162](https://github.com/kamsar/Unicorn/issues/162)
* Moving an item between Unicorn configurations will correctly respect exclusions that exclude some children of the target location [#161](https://github.com/kamsar/Unicorn/issues/161)
* Unicorn is now free of uses of `EditContext`, which is [a 10-year-old antipattern](http://sitecore.alexiasoft.nl/2006/04/03/new-way-of-editing-items/#comment-16) [#160](https://github.com/kamsar/Unicorn/issues/160)
* Pathing that escaped from the webroot using `~/` no longer breaks when set as the serialization path [#157](https://github.com/kamsar/Unicorn/issues/157)
* Errors about being unable to send headers after they have already been sent in Sitecore 8.1 Update-3 and later have been resolved [#155](https://github.com/kamsar/Unicorn/issues/155)
* Unicorn Auto Publish now works correctly if it is set to publish to more than one publishing target [#152](https://github.com/kamsar/Unicorn/issues/152)
* Renaming items whose filenames have been truncated on disk (very long db item names) will no longer result in the creation of duplicate files [#146](https://github.com/kamsar/Unicorn/issues/146)
* Items whose only change between db and disk is the `BranchID` are now synced correctly [#144](https://github.com/kamsar/Unicorn/issues/144)
* YAML values containing `\` and `"` are now correctly treated as multiline values so that serialized files are valid YAML. This is a non-breaking change, but you may notice values such as `__Created by` being altered as items are saved. All Unicorn 3.x versions can parse either format without issue. [#143](https://github.com/kamsar/Unicorn/issues/143)
* You can now explicitly add a wildcard item (an item named `*`) to a Unicorn configuration without it being interpreted as a wildcard character. See [test cases](https://github.com/kamsar/Unicorn/blob/master/src/Unicorn.Tests/Predicates/TestConfiguration.xml#L81) [#142](https://github.com/kamsar/Unicorn/issues/142)
* The Unicorn data provider now returns `null` for `ChildIDs` if no children exist to enhance compatibility with the Sitecore data provider's behaviour [#141](https://github.com/kamsar/Unicorn/issues/141)
* A race condition when using multithreaded sync and auto publishing has been fixed [#183](https://github.com/kamsar/Unicorn/issues/183)
* Rapidly clicking a checkbox in the control panel will no longer potentially corrupt the page state [#158](https://github.com/kamsar/Unicorn/issues/158)
* Unicorn and Rainbow both now reference Sitecore from the Official NuGet Feed, so if you need to build from source copying Sitecore assemblies is no longer required. Just clone and build!
* **3.2** Setting exclusions on a configuration that uses Transparent Sync is now an exception (this has never been a supported configuration, just now it's an error too)
* **3.2** Configuration predicates with no include nodes are now an error instead of including all items

## Upgrading

To upgrade to Unicorn 3.3, you need to make sure the project hosting Unicorn and Rainbow is targeting .NET 4.5.2 or later. Then simply upgrade the NuGet package, if you have Unicorn 3.1 or later.

For earlier versions, follow the upgrade instructions on the appropriate launch blog post (e.g. for 3.0 or 3.1) before you upgrade to 3.3.

## Installing

Install the `Unicorn` package from NuGet. A readme will be shown to help you get started after install. You must be using `packages.config` (NuGet 2.x style) package management for Unicorn and Rainbow, because they install content items to the project. If you wish to use `project.json` (NuGet 3.x style), you must manually install the configuration files for Unicorn and Rainbow because of limitations in NuGet 3.x.

## Thanks

As usual Unicorn is not just me, it's a community project. Thank you all for your pull requests, issue reports, and for using my work.

I'd like to specially thank Mark Cassidy for spending a lot of time tracking down - and fixing - some Unicorn issues he was having. All contributors to this release include:

* [Tyler Allen](https://github.com/TAllen)
* [Jason Booth](https://github.com/jason-booth)
* [Mark Cassidy](https://github.com/cassidydotdk)
* [Steve Crabb](https://github.com/steeevec)
* [Jeff Darchuk](https://github.com/JeffDarchuk)
* [Mads Due](https://github.com/MadsDue)
* [Thomas Eldblom](https://github.com/Eldblom)
* [HenkMahieu](https://github.com/HenkMahieu)
* [Ben Lipson](https://github.com/blipson89)
* [mzizka](https://github.com/mzizka)
* [Jawad Sabra](http://sitecore.stackexchange.com/users/324/jawad)
* [Thomas Stern](https://github.com/istern)
* [Tony Wang](https://github.com/tonycwang)
* [wolfeno](https://github.com/wolfeno)
* [Una Verhoeven](https://github.com/eris86)

{% asset_img theroom.gif %}