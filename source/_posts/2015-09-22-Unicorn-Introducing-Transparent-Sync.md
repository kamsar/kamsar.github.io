title: "Introducing Transparent Sync in Unicorn 3"
date: 2015-10-02 10:50:00
categories: Unicorn
---

A couple years ago [Alex Shyba](https://twitter.com/alexshyba) told me about an idea he had: what if we used a data provider to map serialized items directly into the content tree, bypassing the database? This idea sounded pretty amazing, and I went nuts and [wrote a prototype](https://github.com/kamsar/Rhino) that did exactly that. 

The prototype, nicknamed Rhino because it has a prominent horn like [certain other mythical beasts](https://en.wikipedia.org/wiki/Unicorn), actually worked fairly well. Unfortunately there are two hard problems in computer science: naming, and cache invalidation. Cache invalidation, specifically using the `FileSystemWatcher` to observe file changes by source control, was unreliable. Because of how core serialization is to development practice, unreliability is not acceptable. Reluctantly, I shelved Rhino and worked on Unicorn 2 instead.

The idea of Rhino stuck around though. The improvements brought around in [Rainbow](https://github.com/kamsar/Rainbow), such as partial item reading and tighter control around storage, enabled working around the limitations that had precluded Rhino from being useful in production. Thus the idea returns as Transparent Sync, part of Unicorn.

Transparent Sync enables Unicorn to sync the serialized items in real time, without developer intervention. It does this by using its data provider to directly read the serialized items from disk and inject them into the Sitecore content tree. The items on disk **are** the items in Sitecore: it bypasses the SQL database entirely. Changes made on disk update nearly instantly in the Sitecore editing interfaces.

Enabling Transparent Sync means you don't need to sync after pulling down others' changes. However in production you probably want to remove the Unicorn data provider because it's not needed. But, without the Unicorn data provider the transparently synced items disappear. That's not a problem, though: you _can_ sync transparently synced configurations in the traditional way to persist them permanently in the database. This is the recommended production deployment method for transparently synced configurations.

## How do we use it?

Transparent sync is turned off by default because you should understand how it works and the implications of its use before enabling it. There is a small startup time penalty for using it: if you've got a SSD this penalty is pretty minimal because the SSD is so fast at random I/O, but a traditional hard drive may prove suboptimal. Transparent sync is excellent for development artifacts like templates and rendering items, but it's  inappropriate if you've got hundreds of thousands of content items. In basic testing I enabled transparent sync for the whole default core and master database (19,228 items) using a SSD and noted about 100ms increase in startup times on average.

Turning transparent sync on is really easy: take the configuration you want to add transparent sync to and put this line in its configuration:

	<dataProviderConfiguration enableTransparentSync="true" type="Unicorn.Data.DataProvider.DefaultUnicornDataProviderConfiguration, Unicorn" />

You can also change the setting in the global `<defaults>` if you want to change the default setting of transparent sync for all configurations. Note: you need to do an initial serialization of a configuration with transparent sync _off_ before you enable it. Otherwise the items in the configuration will seem to disappear from the content editor as transparent sync shows all zero items that are on disk :)

Once Transparent Sync is enabled all you have to do is change items on disk and the updates to them will immediately appear within Sitecore.

## Data Cache

Unicorn 3's [SFS data store](http://kamsar.net/index.php/2015/08/Reinventing-the-Serialization-File-System-Rainbow-Preview-Part-2/) has a data cache capability that can be useful to ensure maximum performance when using transparent sync. The data cache causes items that are read from disk to be stored in RAM for later reuse. This is disabled by default because it requires about as much RAM as the serialization files take on disk, but if you have a small data set and want maximum performance manipulating transparent sync items it is available.

Note that once loaded transparently synced items also go into Sitecore's data caches, so the Unicorn data cache is a second layer cache.

To enable the data cache change `enableDataCache` to `true` on the `targetDataStore` in configuration.

## Anything else?

One thing to be aware of: because it bypasses the normal sync process, transparent sync also bypasses anything that is hooked to sync. This would include things like custom evaluators (like `NewItemsOnlyEvaluator`) and the sync event pipelines. If you are relying on customizations to these, turn transparent sync off for the configurations that use them.

Have fun!