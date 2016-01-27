title: Unicorn 3.1 Released
date: 2016-01-26 20:58:31
categories: Unicorn
---
Unicorn 3.1 and Rainbow 1.1 are released to NuGet. These updated versions bring several new features and a bunch of fixes. Upgrading is recommended for all users of Unicorn 3.0 due to the fixing of some data issues.

## What's new?

### Configuration Dependencies

Unicorn configurations may now declare dependency relationships between each other. This is useful for example when one configuration may contain templates and another configuration might contain items based on those templates. With a dependency relationship the dependent configurations are guaranteed to always go first when configurations are synced. Transitive relationships (e.g. A -> B -> C) are supported.

Dependencies are defined using [a dependencies attribute](https://github.com/kamsar/Unicorn/blob/master/src/Unicorn/Standard%20Config%20Files/Unicorn.Configs.Dependency.config.example) on the configuration.

Dependencies __do not__ force dependent configurations to sync. If config `A` depends on config `B` and you choose to sync only config `A`, `B` will not be synced. However if you sync _both_ `A` and `B`, then `B` will always sync first due to the dependency.

### Include/Exclude Predicate Enhancements

The predicate system has been significantly overhauled in Unicorn 3.1 to allow more advanced include and exclude scenarios.

* Exclude entries may now use paths relative to the parent `<include>` by omitting a leading slash. This would exclude `/foo/bar`: 

		<include path="/foo">
			<exclude path="bar" />
		</include>


* A new explicit grammar for excluding all children of an include has been introduced. This replaces the previous implicit grammar of adding a trailing slash (which still works but is deprecated).

		<include path="/foo">
			<exclude children="true" />
		</include>

* The explicit children grammar supports `<except>` syntax to exclude all children [except for specific ones](https://github.com/kamsar/Unicorn/blob/master/src/Unicorn.Tests/Predicates/TestConfiguration.xml#L31).

		<include path="/somechildren">
			<exclude children="true">
				<except name="tests" />
				<except name="fests" />
			</exclude>
		</include>

* A new explicit grammar for excluding only children of a specific sub-path has been added. This grammar also supports [`<except>` syntax](https://github.com/kamsar/Unicorn/blob/master/src/Unicorn.Tests/Predicates/TestConfiguration.xml#L41) and relative paths.

		<include path="/foo">
			<exclude childrenOfPath="/foo" />
			<exclude childrenOfPath="bar"> <!-- /foo/bar/* -->
				<except name="quux" /> <!-- include /foo/bar/quux -->
			</exclude>
		</include>

Take a look at the [test predicate configuration](https://github.com/kamsar/Unicorn/blob/master/src/Unicorn.Tests/Predicates/TestConfiguration.xml) for examples of all available grammars.

### Control Panel Tool Authentication

The method used to authenticate automated tools to Unicorn has been updated to be significantly more secure using a [CHAP](https://en.wikipedia.org/wiki/Challenge-Handshake_Authentication_Protocol) implementation. This also comes with a handy [PowerShell module](https://github.com/kamsar/Unicorn/tree/master/doc/PowerShell%20Remote%20Scripting) that you can use to simply invoke the handshake from a build script.

Should you require the legacy shared secret method, that is [still available but it is not the default](https://github.com/kamsar/Unicorn/blob/master/src/Unicorn/Standard%20Config%20Files/Unicorn.UI.config#L24). Security is also now pluggable, if you wanted to use your own implementation.

The remote API ships disabled by default and you must activate it by [generating a random shared secret and adding it to the config](https://github.com/kamsar/Unicorn/blob/master/src/Unicorn/Standard%20Config%20Files/Unicorn.UI.config#L18). This same secret must be known to invoke the API, and unlike the legacy API the secret is never transmitted directly.

### Breaking Changes

Unicorn 3.1 has some breaking changes compared to Unicorn 3.0, so upgrading requires reserializing your items to get them in a consistent format with the latest version. Unicorn 3.1 does understand Unicorn 3.0-formatted items, but Unicorn 3.0 _does not_ understand Unicorn 3.1-formatted items. There will also be some repeated changes in the logs if you sync 3.0 items with 3.1, so usage with 3.0 items is not recommended.

Introducing breaking changes was a hard decision to make, but I think the saner defaults and new capabilities will justify the slightly more difficult upgrade process. So what's breaking?

* The item's database name is now stored in the serialized data ([#7](https://github.com/kamsar/Rainbow/pull/7)). This enables parsing serialized items in contexts where the database cannot easily be inferred, such as Courier.
* Unversioned fields have official support and are nested under the `Language` instead of under a `Version` within the language which was incorrect. Prior to reserializing, syncing a 3.0-formatted item with 3.1 may result in messages that the field has been reset to standard values and then set again. `IItemData` now has an `UnversionedFields` property to store unversioned fields by language.
* The name on serialized fields has been converted from a comment in the YAML into an actual parsed value. Note that this value is just a hint and if a template field is renamed it is NOT updated automatically.
* The default setting of `Rainbow.SFS.ExtraInvalidFilenameCharacters` has been changed to `"$"` to make Rainbow compatible by default with TFS, and the TFS config example that previously set this has been removed.
* The default setting for `Rainbow.SFS.SerializationFolderPathMaxLength` has been increased from `90` to `110` based on real world experience that 90 is too short.
* The default setting for `Rainbow.SFS.MaxItemNameLengthBeforeTruncation` has been reduced from `100` to `30` based on real world experience that 100 was long enough to break the path length limitation for items with near 100 length names.
* Automated deployment tools calling Unicorn have a new security API. See below for details on this, but it's now more secure.

### Other Improvements:

* Syncing dictionary items will now cause the dictionary cache to be cleared at the end of the sync, forcing the synced entries to be loaded.
* The Unicorn Control Panel is now pipeline-based and extensible so that modules may define their own verbs or alter the UI if needed.
* Error messages when serializing and deserializing XML-based fields (layout, rules) have been improved.
* Unicorn and Rainbow both now utilize the C# 6.0 syntax and require the [C# 6.0 compiler](https://www.microsoft.com/en-us/download/details.aspx?id=48159) to build. They still target .NET 4.5.
* Config file comments and documentation have been updated to cover the new features of Unicorn 3.1.

### Bug fixes:

* Renaming or moving items which are serialized on loopback paths will no longer cause the serialized children to be deleted.
* Missing field errors on the root item of a tree now result in a retry and non fatal error as expected.
* A non fatal warning when syncing multiple configurations will no longer result in the sync stopping at the end of the configuration with the non fatal warning.
* When using the New Items Only evaluator, you will no longer receive sync conflict warnings when saving items created by the evaluator.
* Copying items when Transparent Sync is enabled and the source item is not in the Sitecore database now works correctly.
* Empty field values will no longer cause problems when Transparent Sync is enabled.
* XML-based fields (layout, rules) with empty values will no longer cause exceptions. [#6](https://github.com/kamsar/Rainbow/pull/6), [#90](https://github.com/kamsar/Unicorn/issues/90)
* Items installed by Sitecore packages or update packages into Unicorn configuration areas will no longer lose some field values in the serialized items. [#92](https://github.com/kamsar/Unicorn/issues/92)
* Predicate include entries and SFS trees are no longer confused by paths which are different but start with similar bases, such as `/sitecore/content` and `/sitecore/content monkey`
* Plain text Unicorn logs returned to automated tool calls will no longer include HTML-encoded values such as `&gt;`
* Naming items a [Windows reserved file name](https://msdn.microsoft.com/en-us/library/windows/desktop/aa365247(v=vs.85).aspx) such as CON or COM1 will no longer result in an error. [#8](https://github.com/kamsar/Rainbow/issues/8)
* Fields with a blank value in Sitecore that were not included in the serialized item were not being correctly reset to standard values. This has been fixed.

## Upgrading

Ok so there are breaking changes, how do I upgrade? Thankfully it's not too hard.

1. __Before__ you upgrade, run a full sync of all configurations. If some have Transparent Sync enabled, turn that off during the upgrade. (you can reenable after step 4)
2. Upgrade your NuGet packages to Unicorn 3.1. (which will also install Rainbow 1.1)
3. Run a build of your solution, and if you develop out of webroot publish your changes.
4. Using the Unicorn control panel, __Reserialize__ all configurations to flush them to disk as 3.1-format items.
5. If you use the automated tool API to do automated execution of Unicorn, migrate to the [PowerShell module](https://github.com/kamsar/Unicorn/tree/master/doc/PowerShell%20Remote%20Scripting) to invoke the new API.
6. Have fun!

## Contributors

Unicorn 3.1 has seen contributions from many community members. Thank you all for your support, testing, bug reports, and contributions!

* [Mark Cassidy](https://github.com/cassidydotdk)
* [Thomas Eldblom](https://github.com/Eldblom)
* [Andreas Gerhke](https://github.com/agehrke)
* [Oleg Jytnik](https://github.com/OlegJytnik)
* [Chris Paton](https://github.com/patoncrispy)
* [Daniel Rachele](https://github.com/drachele)
* [Lewanna Rathbun](https://github.com/divamatrix)
* [Richard Seal](https://github.com/guitarrich)
* [Andrii Snihyr](https://twitter.com/BerserkerDotNET)
* [Chris van de Steeg](https://github.com/csteeg)
* [George Thomaidis](https://github.com/GeorgeKarbyn)
* [Kevin Williams](https://twitter.com/williamsk000)
* [dashkodo](https://github.com/dashkodo)
* [devstack](https://github.com/devstack)
* [uandrei](https://github.com/uandrei)
