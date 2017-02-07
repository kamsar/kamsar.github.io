title: "Unicorn 3: What's new?"
date: 2015-09-14 07:20:00
categories: Unicorn
---


Now that I've finished being confusing with my [last](https://kamsar.net/index.php/2015/09/Rainbow-Part-3-What-is-Rainbow/) [three](https://kamsar.net/index.php/2015/07/Rethinking-the-Sitecore-Serialization-Format-Unicorn-3-Preview-part-1/) [posts](https://kamsar.net/index.php/2015/08/Reinventing-the-Serialization-File-System-Rainbow-Preview-Part-2/) about [Rainbow](https://github.com/kamsar/Rainbow), let's talk about Unicorn 3.

{% asset_img Unicorn_logo.png "new logo!" %}

Unicorn 3 is the product of a year of thought and implementation. The design goal was nothing less than fixing every annoyance or issue that we ran across in daily usage of Unicorn 2. So what's new?

# New serialization format

High on the list of annoyances with daily Unicorn usage was the difficulty of resolving merge conflicts in the Sitecore serialization format. 

The only fix for this was to make a [better format](https://kamsar.net/index.php/2015/07/Rethinking-the-Sitecore-Serialization-Format-Unicorn-3-Preview-part-1/): human readable, easily mergeable, and fixing the daily annoyances of the Sitecore format. Because no tool is an island, the new serialization tools Unicorn 3 uses have been split off into their own library: [Rainbow](https://github.com/kamsar/Rainbow). Rainbow enables others to use the new serialization tools developed for Unicorn 3, without depending on Unicorn.

The details of the new format and why it exists can be found in [part 1 of my Rainbow blog series](https://kamsar.net/index.php/2015/07/Rethinking-the-Sitecore-Serialization-Format-Unicorn-3-Preview-part-1/).

# New file organization

Alongside designing the new serialization format, another problem was to resolve the longstanding limitations and bugs in the way Sitecore's built in serialization stores its file hierarchy. [This was worth a whole blog post in and of itself](https://kamsar.net/index.php/2015/08/Reinventing-the-Serialization-File-System-Rainbow-Preview-Part-2/), but in summary I think those limitations have been eliminated.

The new hierarchy also alters the way the storage tree operates. Whereas Sitecore's model represents an entire database, Unicorn 3 represents a set of subtrees of the database. Depending on the depth of the root paths, this can result in much shorter filesystem paths and fewer over-length path loops.

# Improved user experience

## Improved console messaging

Unicorn 3 has greatly improved messaging, both reducing extra output that isn't necessary as well as vastly better error formatting. This hilarity was a constant annoyance if something went wrong in Unicorn 2 (in this case, invalid serialization format):

{% asset_img u2error.png "seriously? who shipped this." %}

Unicorn 3's version of the same error:

{% asset_img u3error.png "woohoo" %}

## Editor warnings for Unicorn-controlled items

Another common issue is having someone edit an item that Unicorn controls on a deployed server, and having their changes overwritten by the next automated deployment. People asked for a way they could know if an item was "Unicorned" or not. Well, now there is one:

{% asset_img cewarning.png "warning will robinson" %}

The message shown changes depending on the environment as well; this is how it looks when "production mode" is on:

{% asset_img cewarning2.png "warning will robinson" %}

And also, if production mode is enabled if you attempt to save the item:

{% asset_img cewarning3.png "warning will robinson" %}

## Unicorn-enabled serialize commands

Ever used these handy tools on the by-default-hidden Developer tab of the Sitecore ribbon?

{% asset_img partialsync.png "controls" %}

Guess what happens if you use these commands on an item Unicorn 3 controls? You guessed it: you can do __partial syncing__ and __partial reserialization__ using these commands. In Unicorn terms, there is no difference between "update" and "revert": both just mean "sync."

Note: Using the commands on non-Unicorn items results in their default behaviour. Also "update database" and "revert database" do not interface with Unicorn at all, and perform their default actions.

# Performance: 50% more of it

Unicorn 3 has had significant performance profiling applied to it, and is about 10-20% faster than Unicorn 2 or stock Sitecore serialization. On top of that, multithreading is utilized to further increase sync and reserialize speed. Between threading and optimizations, it's generally about 50% faster than Unicorn 2. For maximum performance, using SSD storage is highly recommended as it has a major impact on sync speed.

# Auto-publish synced Items

Unicorn 3 enables efficient auto-publishing of items that a sync has modified, using manual publish queue injection. This option is enabled by default by the `Unicorn.AutoPublish.config` file. You can remove this file to disable the feature if you don't like it.

# Deleted template field handling

In Unicorn 2, deleting template fields could cause havoc. If any other serialized item contained a value for the deleted field, and you failed to reserialize that item after deleting the field, your next sync would be greeted with a big ugly error and you'd have to go manually remove the offending field from the file.

Unicorn 3 fixes this issue, by making this error a warning instead. (We can't just ignore it, because ignoring it would cause syncs that created template fields as well as values for that field to not load the values the first time)

{% asset_img u3warning.png "woohoo" %}

# Versioned to Shared field conversion

Converting a field between shared, versioned, and unversioned by syncing a template was not supported in Unicorn 2. The field itself would change, but the existing values for the field would not move to the correct database table in Sitecore. Unicorn 3 includes the necessary adaptation to migrate stored values when field sharing is changed by a sync.

# Improved configuration scheme

Unicorn 2 used a single configuration file: `Serialization.config`. This was a bit confusing as parts of it were ideally removed when deployed to a CE or CD environment and that was not always obvious.

Version 3 addresses this situation by creating its own `App_Config\Unicorn` folder and splitting the configuration into files that can simply be deleted instead of modified. This folder contains:

* `Unicorn.config`: Contains core configuration, e.g. predicates and dependency setup, can be left intact anywhere.
* `Unicorn.DataProvider.config`: Attaches the Unicorn Data Provider, which handles automatic serialization of changed items to disk. Can be removed for any environment not collecting changes. (usually, any non-dev environment)
* `Unicorn.UI.config`: Adds the Unicorn Control Panel, Partial Sync, editor warnings, and other end-user UI elements. Remove for CD environments.
* `Unicorn.AutoPublish.config`: Causes Unicorn to automatically publish items that it changes during a sync. Remove if you don't want this feature, or on CD environments.
* `Unicorn.Deployed.config.disabled`: If this config is enabled, typically on a deployed CE instance, saving an item added to Unicorn results in a warning confirmation.

Each of these config files also has comments explaining the above as well as what the settings within do.

# Technical Improvements

Unicorn 3 also includes numerous miscellaneous technical improvements.

* No third party dependencies. (that means you, Ninject)
* Improved default predicate configuration settings account for Sitecore 8 additions and common modules.
* Added `SerializationHelper`, a handy class that lets you more easily invoke Unicorn operations such as Sync without a lot of setup, if you want to programmatically use Unicorn.
* Added pipelines to hook to sync events. `unicornSyncBegin` occurs when a configuration sync begins. `unicornSyncComplete` occurs when a configuration sync ends. `unicornSyncEnd` occurs at the end of all syncs. (e.g. after all configurations have synced if batching > 1)
* Thanks to Rainbow, the object model is unified (ALL items are `IItemData` both from Sitecore and serialized). Previously each had their own parallel models.

# System Requirements

Unicorn 3 has been developed on Sitecore 7.2 Update-5, as well as Sitecore 8 Update-5. It should be compatible with all version of Sitecore 7 and 8.

Unicorn 3's code should be easy to adapt to Sitecore 6.2-6.6, however there is no official support for it so you'd want to compile from source.

# Ok, ok. Is it released yet?

No. But it is in an actively updated beta on NuGet that is fairly stable :)