title: "Unicorn 3.0.2 Released"
date: 2015-11-03 22:17:49
categories: Unicorn
---

[Unicorn](https://github.com/kamsar/Unicorn) 3.0.2 (and [Rainbow](https://github.com/kamsar/Rainbow) 1.0.1) have been released to NuGet. 

_This release includes some critical fixes to the serialization system, and upgrading is highly recommended for all users of [Unicorn 3](https://kamsar.net/index.php/2015/10/Unicorn-3-0-Released/)._

## Improvements
* Unicorn auto-publish now reports its activities to the sync console, and operates synchronously to guarantee published state for automated builds.
* Guidance on some [Transparent Sync pitfalls](https://github.com/kamsar/Unicorn/wiki/The-Transparent-Sync-Guide) has been added to the wiki.
* The new `Rainbow.SFS.ExtraInvalidFilenameCharacters` setting enables TFS users to successfully serialize branch templates which begin in `$`, a reserved character by TFS. The `Rainbow.TFS.config.example` file is now shipped with Rainbow to illustrate its usage for TFS.
* An [example configuration file](https://github.com/kamsar/Unicorn/blob/master/src/Unicorn/Standard%20Config%20Files/Unicorn.Configs.SerializationFolder.config.example) to demonstrate how to move your serialized items elsewhere has been added.

## Fixes
* **CRITICAL** Saving item versions that were not in the default Sitecore language was not updating the serialized item. This has been fixed. [#76](https://github.com/kamsar/Unicorn/issues/76)
* **CRITICAL** Serialization of significant empty field values, such as unchecked checkboxes when the standard value is checked, has been fixed. [#74](https://github.com/kamsar/Unicorn/issues/74)
* Items that are restored from the Sitecore recycle bin or archive to Unicorn-controlled locations now result in the item being serialized on restore. [#72](https://github.com/kamsar/Unicorn/issues/72)
* Resetting a field to standard values now correctly results in the field value being removed from the serialized item. [#74](https://github.com/kamsar/Unicorn/issues/74)
* Query string order is always respected when syncing multiple configurations at once using the control panel.
* The serialization conflict dialog (if disk doesn't match base item data on save) now formats correctly using HTML for Sitecore 8+
* Using the _Dump Item_ and _Dump Tree_ commands in Sitecore to reserialize partial item trees now correctly serializes an item to all configurations that it may belong to, instead of just the first one.

## Upgrading
Upgrading from Unicorn 3.x is as simple as updating your NuGet packages. If it prompts to overwrite any changed config files, I'd suggest doing so and using a diff with your source control to resolve differences.

If you suspect that you have serialized items that may be missing [significant empty values](https://github.com/kamsar/Unicorn/issues/74), you should reserialize those after upgrading to get their values normalized.

## The Credits

Unicorn isn't just my project. Thanks a lot to all the community members who reported bugs, sent pull requests, and diagnosed issues to help make Unicorn better.

Unicorn 3.0.2 is brought to you by...

[Robin Hermanussen](https://twitter.com/knifecore)
[Thomas Eldblom](https://twitter.com/TEldblom)
[Nathanael Mann](https://twitter.com/cardinal252)
[@GeorgeKarbyn](https://twitter.com/GeorgeKarbyn)
[Kevin Williams](https://twitter.com/williamsk000)
[@amitthakur2014](https://github.com/amitthakur2014)
[@WizX20](https://github.com/WizX20)
[@kamsar](https://twitter.com/kamsar)
...and the rest of #unicorn on Sitecore Slack

As usual, if you run into trouble you can [submit an issue](https://github.com/kamsar/Unicorn/issues/new) on GitHub, hit me up on Sitecore Slack, or [tweet at me](https://twitter.com/kamsar).