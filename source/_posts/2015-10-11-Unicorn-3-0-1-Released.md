title: Unicorn 3.0.1 Released
date: 2015-10-11 21:05:26
categories: Unicorn
---

The first maintenance release for Unicorn 3.x has been released to NuGet. Unicorn 3.0.1 brings with it a small set of bug fixes and improvements:

* Data provider config patch is compatible with Sitecore Commerce and other modules that add data providers (thanks [@jonnekats](https://twitter.com/jonnekats)!)
* Fixed a bug in Transparent Sync where syncing an item based on a template that only existed in transparent sync (and not the database) would act like the template did not exist. Ironically this was fixed in 3.0.0 but some n00b forgot to merge the config to activate it! (thanks [Kevin Williams](https://github.com/williamsk)!)
* Fixed a race condition that could cause SFS trees to incorrectly reinitialize during a reserialize operation and cause an error.
* Content editor warnings have been improved:
	* The name of the Unicorn configuration that includes the item is now shown so it's easy to trace back why something is included.
	* For transparent sync items, there is an indication of whether the item is on disk only or both on disk and in the database as well. Note that items in both places may well have different field values, there is no equality check.

Due to the fix to Transparent Sync, this update is recommended for anyone using that feature.

Have fun!