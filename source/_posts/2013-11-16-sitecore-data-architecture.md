---
layout: post
title: "Sitecore Data Architecture"
date: 2013-11-16 22:28:41 -0800
comments: true
categories: Sitecore
---
## Introduction

There are many times where you need to extend the Sitecore data architecture in some way. One of the most obvious, and most used, is attaching an event handler to one of the item events (e.g. item:saved, item:renamed) but this is not by any means the only way to manipulate items before they get to the database.

This blog post came out of a discussion I had with Alex Shyba about the DataEngine, a few hours on a plane, and a lot of decompilation. I'm going to attempt to show what happens when an item gets saved and where you can plug into that.

## The entry point

The most high level API is of course the `Item` class. Saving an item using this API is very simple:

	using(new EditContext(item))
	{
		item["MyField"] = "newValue";
	}

When the `EditContext` goes out of scope, this causes `item.Editing.AcceptChanges()` to be invoked, and our journey down the rabbit-hole begins there.

## The Item Manager

Our next step is the highest level item data API in Sitecore: the `ItemManager`. At first blush `ItemManager` appears to be a rather ugly giant static class, with tons of manipulaton methods: `SaveItem`, `AddVersion`, etc. However, it's not as bad as it looks. `ItemManager` is basically a static facade around the current `ItemProvider`.

## The Item Provider

Here's where things get interesting. You can register _multiple_ item providers in the config under `itemManager/providers`. While this appears to provide a lot of power as an extension point - being able to plugin your own high level provider - it unfortunately seems to be rather ungainly. If multiple providers are registered you can access them directly using `ItemManager.Providers`, but the other `ItemManager` methods only execute against the default provider. You _could_ replace the default provider with your own, but this is less than ideal as it can lead to contention for whose item provider gets used if more than one module wants to extend using it.

Extension points aside, the `ItemProvider` is the most broad data API Sitecore provides. It deals in high level objects and abstracts lower level concepts like databases and item definitions away. Most all methods in the `ItemProvider` result in invocations of the next lower level construct, the `DataEngine`.

*UPDATE*: [Nick Wesselman](https://twitter.com/techphoria414) pointed out that some queries will bypass the Item Provider and go directly to the Data Engine. [This gist](https://gist.github.com/techphoria414/7506347) has some examples of the types of things that bypass the Item Provider. In light of this I'd say the item provider is generally not a good candidate for extending things.

## The Data Engine

`DataEngine` (generally accessed by `databaseObject.DataManager.DataEngine`) is a very interesting component for extension. Unlike the item provider, the `DataEngine` is database-specific. This is also the layer at which item event handlers (e.g. item:saved) are processed, but more interestingly is also a place where you can hook events in an uninterruptible fashion. Regular event handlers, such as `item:saved`, are vulnerable to being disabled when someone scopes an `EventDisabler`. Generally this is good, as events are disabled for performance reasons during things like serialization bulk load operations. But occasionally, such as in [Unicorn](http://github.com/kamsar/Unicorn), you need an event handler that cannot be disabled. Having a Unicorn event not fire would likely mean data being lost.

The internal architecture of `DataEngine` works something like this:
- An engine method gets invoked
- The engine creates and initializes an `DataEngineCommand<>` object, using a stored prototype of the command
- The `DataEngineCommand` has its `Execute` method invoked.

The `EngineCommand` is a generic type, and each action that the engine can take has its own implementation - for example, there is a `Sitecore.Data.Engines.DataCommands.SaveItemCommand` class. Each of these command types is bootstrapped by a prototype system on the `DataEngine` itself. The engine, by way of its `Commands` property, stores copies of each command type. When a new command is needed, the existing prototype command is cloned (`Clone()`), initialized (`Initialize()`), and then used to execute the action. These prototypes are settable, including via configuration, so this allows you to inject your own derived implementation of each command and thus inject 'event handler like' functionality into the `DataEngine`.

Item Buckets utilizes this type of functionality injection, replacing the `AddFromTemplatePrototype` with its own extended implementation. This is the relevant section of Sitecore.Buckets.config:

      <database id="master" singleInstance="true" type="Sitecore.Data.Database, Sitecore.Kernel">
        <Engines.DataEngine.Commands.AddFromTemplatePrototype>
          <obj type="Sitecore.Buckets.Commands.AddFromTemplateCommand, Sitecore.Buckets" />
        </Engines.DataEngine.Commands.AddFromTemplatePrototype>
      </database>

This same functionality could be used to inject non-disableable event handler type functionality into other events, for example by replacing the `SaveItemPrototype`. Unfortunately this method also suffers from possible contention issues if multiple modules wished to patch these commands, so be careful when using them.

## The `DataEngineCommand`

Once the `DataEngine` creates and executes a command, the command generally does two things:
* Pass its task down to the Nexus `DataApi` to be handled
* Fire off any normal item event handlers (item:saved), as long as events are not disabled

Most of the methods on `DataEngineCommand` are virtual so you could extend them. The most obvious candidate, of course, is the `DoExecute` method that performs the basic action of the command. 

Further down the rabbit-hole we arrive to the Nexus APIs.

## The Nexus `DataApi`

The Nexus assembly, due to it containing licensing code I believe, is the only obfuscated assembly in Sitecore. This makes following what the data API is doing rather difficult, but I'm pretty sure I traced it back out as a call to the `databaseItem.DataManager.DataSource` APIs, which thankfully are not obfuscated. It's worth noting that the Nexus APIs appear to also be using a _separate_ command-class based architecture internally. Given the sensitive nature of the Nexus assembly however, I would look elsewhere for extension points.

## The `DataSource`

The `Sitecore.Data.DataSource` appears to largely be a translation layer between slightly higher level APIs, where things like `Item` are used, and the low level API of the data providers (which use constructs like IDs and `ItemDefinition` instead). The data source is very generic and does not appear to be designed for extension, which is fine because we can extend both above and below it.

Methods within `DataSource` eventually make calls down to static methods on the `DataProvider` class.

## The Data Provider

There are two faces to the `DataProvider` class. The internal static methods that the `DataSource` is invoking are helper methods that invoke non-static methods on all of the data providers attached to the database. (Yes, you can have multiple data providers within the same database, which may not even look at the same backend database at all...hehe) The actual data provider instances are the lowest level data APIs in Sitecore. They deal with primitive objects and have a lot of unspoken rules about them that make them tougher to implement than most other extension points. However they are also the most powerful extension point there is. Using a data provider you can manipulate the content tree in nearly any fashion for example [Rhino](http://github.com/kamsar/Rhino), a data provider that makes serialized item files on disk appear to be real content items in Sitecore.

## You're still reading this?

Hopefully this is a useful post for some crazy nuts like myself who like to bend the guts of Sitecore. It's definitely a long and byzantine road from saving an item to it getting to the database. Rather amazing that Sitecore is as fast as it is with all of these layers. Part of me suspects that some are baggage from Sitecore 4 for backwards compatibility ;)