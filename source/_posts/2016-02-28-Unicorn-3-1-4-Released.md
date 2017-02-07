title: Unicorn 3.1.4 Released
date: 2016-02-28 20:19:12
categories: Unicorn
---

Unicorn 3.1.4 and Rainbow 1.2 are released to NuGet. This is a maintenance release which contains a few UX improvements and minor bug fixes and improvements.

## What's new?

### Logging Verbosity

The most obvious change is that you can now select a logging verbosity in the control panel. If you've ever run a sync where many, many changes were synced you may have run into the issue where browsers start to crawl when fed that many log messages. Now you can choose to log only warnings or errors for that kind of situation. The Sitecore log files always get full verbosity, so you can see what exactly happened even if it was not written to the browser. Your verbosity setting is stored in a persistent cookie, so your selection is remembered.

Verbosity does not affect automated tools that invoke Unicorn. They always receive full verbosity. ([#112](https://github.com/kamsar/Unicorn/issues/112))

### XML Field Formatting Improved

XML fields are now stored by default with attributes on new lines. This further improves mergeability of Rainbow serialized items by allowing attribute merges. If you hate this idea, you can turn it off in `Rainbow.config`. ([#12](https://github.com/kamsar/Rainbow/issues/12))

Rainbow 1.1:

	<r xmlns:xsd="http://www.w3.org/2001/XMLSchema">
	  <d id="{FE5D7FDF-89C0-4D99-9AA3-B5FBD009C9F3}" l="{0DAC6578-BC11-4B41-960A-E95F21A78D1F}" />
	</r>
	
Rainbow 1.2 (Unicorn 3.1.3+):

	<r xmlns:xsd="http://www.w3.org/2001/XMLSchema">
	  <d
	    id="{FE5D7FDF-89C0-4D99-9AA3-B5FBD009C9F3}"
	    l="{0DAC6578-BC11-4B41-960A-E95F21A78D1F}" />
	</r>
	
This formatting change is completely backwards compatible and no reserialization is required. As items are saved, they will be rewritten to use attributes on newlines.

### Fixes & Minor Improvements

* The UX of the control panel has been streamlined to result in less vertical space for each configuration, a boon for setups like Habitat with lots of configurations.
* Unicorn auto publishing now functions correctly on items with versions in several languages ([#114](https://github.com/kamsar/Unicorn/issues/114))
* Using the "Copy to" function will no longer result in an incorrect serialized parent ID on the copied item. Other forms of duplication were not affected. ([#119](https://github.com/kamsar/Unicorn/issues/119))
* Fixed a case where `NullReferenceException` was incorrectly thrown when using transparent sync ([#117](https://github.com/kamsar/Unicorn/pull/117))
* Fixed an error when duplicating items that did not exist in the database when using transparent sync ([#116](https://github.com/kamsar/Unicorn/issues/116))
* Rainbow YAML formatting is more friendly to third party YAML parsers with the way it escapes values. This does mean that some values that did not previously get wrapped in double quotes (like GUIDs), will now have it going forward. Format is backwards compatible, no reserialize required. Items will upgrade to the new version on save. ([#11](https://github.com/kamsar/Rainbow/issues/11))

## Upgrading

To upgrade simply update your NuGet packages. There are no other steps required for this release **unless coming from [Unicorn 3.0.x](https://kamsar.net/index.php/2016/01/Unicorn-3-1-Released/)**. 

If you wish to completely normalize your items to the latest Rainbow 1.2 format, you may reserialize them. But it is not a requirement, as Rainbow 1.2 can parse 1.1 serialized items.

## Credits

As usual, Unicorn updates are a community effort. Big thanks to the folks below for reporting bugs, sending pull requests, and general encouragement.

* [Andreas Gehrke](https://github.com/agehrke)
* [Dave Peterson](https://github.com/PetersonDave)
* [Mike Carlisle](https://github.com/TheCodeKing)
* [Ruud van Falier](https://github.com/DotTech)
* [Sander Bouwmeester](https://github.com/sbouwmeester)
* [Stanislav Maxymov](https://github.com/stasmaxymov)
