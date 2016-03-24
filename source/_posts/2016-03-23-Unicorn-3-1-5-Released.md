title: Unicorn 3.1.5 Released
date: 2016-03-23 21:31:57
categories: Unicorn
---

Unicorn 3.1.5 and Rainbow 1.3 are released to NuGet right now. This is a maintenance release. Upgrading is recommended, particularly if you are using Transparent Sync.

## New Features

### Link DB and Indexing support

The new `<syncConfiguration>` dependency (see `Unicorn.config`) enables configurations to opt in to having synced items be updated into the link database and/or the search indexes after being changed by a sync.

This improves Unicorn's compatibility with content syncing scenarios that are less developer focused, at the expense of reduced sync performance.

Transparent Sync also supports these flags, and items changed _while Sitecore is alive to see them_ will have their links and indexes taken care of if set to do so. However if the IIS App Pool is not active during the file changes, nothing is updated.

### Per-configuration concurrency

Unicorn 3 has always supported multithreaded sync and reserialize, but it has been disabled since release due to a concurrency bug in the Sitecore Template Engine that results in sync deadlocks when it is used to sync template item changes.

3.1.5 brings the ability to set the max thread count per-configuration (using the same `<syncConfiguration>` as indexing support) so that for configurations that are not syncing templates (or on Sitecore prior to 8.0U2) you may again enable concurrency. Threaded syncing can be 30-50% faster than non threaded.

### Control panel UX improvements

Unicorn 3.1.5 brings a 'Sync all' button to the control panel, and a few minor UI tweaks such as relegating the log verbosity to an options modal.

## Bug Fixes
* Checkboxes inherited through several layers of standard values will no longer result in the derived template being reset to base standard values
* Auto-publish no longer will miss publishing unversioned fields in certain cases
* Renaming or moving items with transparent sync enabled and data cache turned on no longer results in a corrupted data cache
* Change Template now works correctly with transparent sync
* Unicorn now understands content items that are using Sitecore 8.1 language fallback without creating extra versions on the serialized copy
* The log verbosity feature has had several issues fixed that would result in logs incorrectly not being shown at the selected level
* Unicorn's Micro IoC container now understands int constructor parameters
* Adding or changing serialized template items when using transparent sync will now correctly clear the template engine, preventing an error
* Automated build powershell examples are now more flexible when invoked in unusual path locations
* In the control panel, the batch action buttons will no longer 'overlay' configuration checkboxes in certain situations
* Logging of unversioned field updates will no longer incorrectly show a version number alongside the update in the logs
* Certain unusual situations when syncing templates and items of that template will no longer throw a null reference due to stale caches

## Upgrading

Upgrading from 3.1.4 is just a NuGet upgrade away. There are some config changes, so overwrite files and merge anything custom back in. Or even better use config patches and leave the default config alone :)

Note that Unicorn 3.1.5 changes the storage format for checkbox fields so that _unchecked_ checkboxes are stored as '0' instead of Sitecore's standard blank value. This makes sync more reliable, however it does means that the first sync after serializing a new item with an unchecked checkbox will result in a 'change' as the zero value is pushed back into the Sitecore database. Checkboxes that are already serialized with blank values will continue to work just fine as they are.

# Credits

Unicorn is a team effort of the Sitecore community. I'd like to thank the following folks who contributed to this release:

[Dana Hanna](https://github.com/thesoftwarejedi)
[Kasper Gadensgaard](https://github.com/Gadensgaard)
[Mark Cassidy](https://github.com/cassidydotdk)
[Mike Carlisle](https://github.com/TheCodeKing)
[Søren Kruse](https://github.com/Krusen)
[WizX20](https://github.com/WizX20)

And extra special thanks to Robin Hermanussen, Richard Seal, Alex Washtell, Mark Cassidy, and everyone else I missed for tirelessly answering questions in #unicorn on Sitecore Slack when I'm busy or not around!