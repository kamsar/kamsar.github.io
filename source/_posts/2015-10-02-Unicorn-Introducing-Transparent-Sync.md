title: "Introducing Transparent Sync in Unicorn 3"
date: 2015-10-02 10:50:00
categories: Unicorn
---

A couple years ago [Alex Shyba](https://twitter.com/alexshyba) told me about an idea he had: what if we used a data provider to map serialized items directly into the content tree, bypassing the database? This idea sounded pretty amazing, and I went nuts and [wrote a prototype](https://github.com/kamsar/Rhino) that did exactly that. 

The prototype, nicknamed Rhino because it has a prominent horn like [certain other mythical beasts](https://en.wikipedia.org/wiki/Unicorn), actually worked fairly well. Unfortunately there are two hard problems in computer science: naming, cache invalidation, and off by one errors. Cache invalidation, specifically using the `FileSystemWatcher` to observe file changes by source control, was unreliable. Because of how core serialization is to Sitecore development practice, unreliability is not acceptable. Reluctantly, I shelved Rhino and worked on Unicorn 2 instead.

![unicorn logo](http://kamsar.net/nuget/unicorn/logo.png)

The idea of Rhino stuck around. The improvements brought around in [Rainbow](https://github.com/kamsar/Rainbow), such as partial item reading and tighter control around storage, enabled working around the limitations that had precluded Rhino from being useful in production. Thus the idea returns as Transparent Sync, which might just be the best part of Unicorn 3.

Transparent Sync enables Unicorn to sync serialized items in real time, completely automatically. It does this by using its data provider to directly read the serialized items and inject them into the Sitecore content tree. The items on disk **are** the items in Sitecore: it bypasses the database entirely for transparently synced items. Changes made on disk update nearly instantly in the Sitecore editing interfaces.

{% asset_img notsureif.jpg "not sure if witchcraft, or transparent sync" %}

Imagine a scenario where a development team is working with a feature-branch driven workflow, like [GitHub Flow](https://guides.github.com/introduction/flow/). In order to perform a code review when using Transparent Sync, you merely checkout the feature branch under review and your Sitecore is immediately configured with the items that the feature includes.

{% asset_img dontalways.jpg "I don't always do code reviews" %}

## When should we use Transparent Sync?

Transparent Sync is turned off by default because you should understand how it works before enabling it.

Transparent Sync is excellent for development artifacts like templates and rendering items, but it's inappropriate if you've got hundreds of thousands of content items. At startup Unicorn must build an index of metadata, which involves reading the headers of each serialized file. If you have a SSD this penalty is pretty minimal, but a traditional hard drive not so much. In basic testing I enabled transparent sync for the whole default core and master database (19,228 items) using a SSD and noted about 100ms increase in startup times on average.

Because Transparent Sync bypasses the normal sync process, transparent sync also bypasses anything that is hooked to sync. This would include things like custom evaluators (like `NewItemsOnlyEvaluator`) and the sync event pipelines. If you are relying on these customizations, turn transparent sync off for the configurations that use them.

## How do we use Transparent Sync?

Note: you must perform an initial serialization of a configuration with Transparent Sync _off_ before you enable it. Otherwise the items in the configuration will seem to disappear as transparent sync shows all zero items that are on disk!

Turning transparent sync on is really easy: take the `<configuration>` you want to add transparent sync to and put this line in it:

	<dataProviderConfiguration enableTransparentSync="true" type="Unicorn.Data.DataProvider.DefaultUnicornDataProviderConfiguration, Unicorn" />

You can also change the setting in the global `<defaults>` if you want to change the default setting of transparent sync for all configurations.

Once Transparent Sync is enabled all you have to do is change items on disk and the updates will immediately appear within Sitecore.

## Going to production

In production you should remove the Unicorn data provider as it's not usually required and reduces your configuration complexity. However without the Unicorn data provider, transparently synced items disappear: they aren't really in the database at all, they're on disk. For this situation, you _can_ sync Transparent Synced configurations in the traditional way to persist them permanently in the database. This is the recommended method for production deployment.

Have fun!