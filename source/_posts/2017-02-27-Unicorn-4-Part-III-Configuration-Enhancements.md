title: 'Unicorn 4 Part III: Configuration Enhancements'
date: 2017-02-27 19:03:31
categories: Unicorn
---

TL;DR: [Unicorn 4 prerelease](https://www.nuget.org/packages/Unicorn.Core/4.0.0-pre03) is on NuGet right now!

Now that that's out of the way, let's talk about another new Unicorn 4 feature: [modular architecture](http://helix.sitecore.net/) friendly configurations. 

{% asset_img everywhere.jpg %}

When [Habitat](https://github.com/Sitecore/Habitat) first launched, I was mildly incredulous at the amount of duplication in its Unicorn configurations. Tons of tiny modules, all of which shared similar but not identical configurations (such as custom root folders) was not really a consideration when multiple configurations were originally conceived. Fast forward to today, and that's a major use case that is more difficult than it needs to be.

Here's an example of a Habitat Unicorn configuration:

```xml
<configuration xmlns:patch="http://www.sitecore.net/xmlconfig/">
  <sitecore>
    <unicorn>
      <configurations>
        <configuration 
        	name="Feature.News" 
        	description="Feature News" 
        	dependencies="Foundation.Serialization,Foundation.Indexing" 
        	patch:after="configuration[@name='Foundation.Serialization']">
          <targetDataStore physicalRootPath="$(sourceFolder)\feature\news\serialization"
                           type="Rainbow.Storage.SerializationFileSystemDataStore, Rainbow" useDataCache="false"
                           singleInstance="true" />
          <predicate type="Unicorn.Predicates.SerializationPresetPredicate, Unicorn" singleInstance="true">
            <include name="Feature.News.Templates" database="master" path="/sitecore/templates/Feature/News" />
            <include name="Feature.News.Renderings" database="master" path="/sitecore/layout/renderings/Feature/News" />
            <include name="Feature.News.Media" database="master" path="/sitecore/media library/Feature/News" />
          </predicate>
          <roleDataStore type="Unicorn.Roles.Data.FilesystemRoleDataStore, Unicorn.Roles" physicalRootPath="$(sourceFolder)\feature\news\serialization\Feature.News.Roles" singleInstance="true"/>
          <rolePredicate type="Unicorn.Roles.RolePredicates.ConfigurationRolePredicate, Unicorn.Roles" singleInstance="true">
            <include domain="modules" pattern="^Feature News .*$" />
          </rolePredicate>
        </configuration>
      </configurations>
    </unicorn>
  </sitecore>
</configuration>
```

It's long and it has a ton of boilerplate that is either identical in every module, or else defined by system conventions (e.g. physicalRootPath). We don't need to be that verbose when using Unicorn 4. When we setup a modular, convention-based system using Unicorn 4 we can start by using _abstract configurations_ to define the conventions of our system:

```xml
<configuration name="Habitat.Feature.Base" abstract="true">
  <targetDataStore physicalRootPath="$(sourceFolder)\$(layer)\$(module)\serialization" />
  <predicate>
    <include name="$(layer).$(module).Templates" database="master" path="/sitecore/templates/$(layer)/$(module)" />
    <include name="$(layer).$(module).Renderings" database="master" path="/sitecore/layout/renderings/$(layer).$(module)" />
    <include name="$(layer).$(module).Media" database="master" path="/sitecore/media library/$(layer).$(module)" />
  </predicate>
  <roleDataStore type="Unicorn.Roles.Data.FilesystemRoleDataStore, Unicorn.Roles" physicalRootPath="$(sourceFolder)\$(layer)\$(module)\serialization\$(layer).$(module).Roles" singleInstance="true"/>
  <rolePredicate type="Unicorn.Roles.RolePredicates.ConfigurationRolePredicate, Unicorn.Roles" singleInstance="true">
    <include domain="modules" pattern="^$(layer) $(module) .*$" />
  </rolePredicate>
</configuration>
```

This configuration defines a configuration that other configurations can _extend_. Because of its `abstract`-ness it is not a Unicorn configuration itself, only a template. Non-abstract configurations may also be extended.

This abstract configuration is also making use of Unicorn 4's ability to do _variable replacement_ in configurations. The `$(layer)` and `$(module)` variables are expanded in the extending configuration and are based on the convention of naming your configurations `Layer.Module`. You can also expand [more than one config per module](https://github.com/kamsar/Unicorn/blob/master/src/Unicorn/Standard%20Config%20Files/Unicorn.CustomSerializationFolder.config.example#L42) and [use your own variables](https://github.com/kamsar/Unicorn/blob/master/src/Unicorn/Standard%20Config%20Files/Unicorn.config#L190). Using our abstract `Habitat.Feature.Base` configuration above, the same `Feature.News` configuration we started with can now be expressed much more simply:

```xml
<configuration 
	name="Feature.News" 
	description="Feature News" 
	dependencies="Foundation.Serialization,Foundation.Indexing" 
	extends="Habitat.Feature.Base">
</configuration>
```

Nice huh? But what if you want to extend or replace a dependency in the inherited configuration? You can do that, too - and using Unicorn 4's element inheritance system you can also do it very cleanly. Unicorn configurations have always been architecturally a set of independent IoC containers. The `<defaults>` node in `Unicorn.config` sets up the defaults, and then each configuration's nodes override and replace the defaults if they exist. This is how you can [deploy only new items](https://github.com/kamsar/Unicorn/blob/master/src/Unicorn/Standard%20Config%20Files/Unicorn.Configs.NewItemsOnly.example) with the `NewItemsOnlyEvaluator` - you're replacing the default evaluator with a different dependency implementation.

Unicorn 4 takes this a step further: with config inheritance, dependencies can be partially extended at an element level. You might have noticed this already in the `Habitat.Feature.Base` configuration, when we did this:

```xml
<targetDataStore physicalRootPath="$(sourceFolder)\$(layer)\$(module)\serialization" />
```

In Unicorn 3, this would have required a `type` attribute. In Unicorn 4, _unless_ you specify a `type` attribute, any attributes you add either replace or add to the default (or inherited) implementation. So instead this kept the same default dependency definition and _changed_ an attribute on it - the `physicalRootPath`. 

If you _do_ specify a `type`, nothing is inherited and it works like Unicorn 3. Thus existing configurations will also work without modification :)

But what about things that have more than just attributes, like the `predicate`'s `include` nodes? You can _append_ elements in the inherited configuration in that case. If we take our `Habitat.Feature.Base` configuration above and extend it like this:

```xml
<predicate>
  <include name="Foo" database="master" path="/sitecore/Foo" />
</predicate>
```

The end result is effectively:

```xml
<predicate type="Unicorn.Predicates.SerializationPresetPredicate, Unicorn" singleInstance="true">
  <include name="Feature.News.Templates" database="master" path="/sitecore/templates/Feature/News" />
  <include name="Feature.News.Renderings" database="master" path="/sitecore/layout/renderings/Feature/News" />
  <include name="Feature.News.Media" database="master" path="/sitecore/media library/Feature/News" />
  <include name="Foo" database="master" path="/sitecore/Foo" />
</predicate>
```
You cannot remove inherited predicate nodes (or other dependencies that use children like `fieldFilter`), so plan accordingly: adding elements only.

And there you have it: with Unicorn 4 you can reasonably simply create serialization conventions for your modules and avoid configuration duplication - or if you're not ready to go modular, you can at least enjoy not needing to have a `type` on most configuration nodes.

## But Wait, There's More 🐘: The Console No Longer Sucks

### In the Control Panel...

The Unicorn console has also received a serious upgrade in Unicorn 4. If you've ever run a sync that changed a large number of items from the Unicorn Control Panel, you may have noticed the browser slow to a crawl and the sync seem to almost stop. The console that underpins Unicorn 3 and earlier started to choke at around 500 lines. 

No longer! Unicorn 4's console has spit out **100,000** lines without a hitch.

### Automated Tools (PowerShell API)

The automated tool console has also received an upgrade. Previously the tool console buffered all the output of a sync before sending it back. This caused problems in certain environments, namely Azure, where TCP connections that don't send any data for more than 4 minutes are terminated. This would cause any long-running syncs in Azure to die unexpectedly.

In Unicorn 4 the automated tool console emits data in a stream just like the control panel console. There's also a heartbeat timer where if no new console entries are made for 30 seconds, a `.` will be sent to make sure the connection is kept active. 

The streaming console also requires updating your `Unicorn.psm1` file - not only will you get defense against TCP timeouts, you'll also be able to see the sync occur in real-time using the PSAPI just like you would from the control panel. No more waiting until it's done to see how things are going :)

## Can I have it yet?

Absolutely. You can find [Unicorn 4.0.0-pre03](https://www.nuget.org/packages/Unicorn/4.0.0-pre03) on NuGet right now!

### How stable is this?

More stable than you might think. Unicorn 4 is largely additions, fixes, and enhancements to the already stable codebase behind Unicorn 3. The core pieces have not changed very much, unless you enable Dilithium and that's optional. The new config inheritance stuff has 97% code coverage. That's not to say it's bug free either. If you find bugs [let me know](https://github.com/kamsar/Unicorn/issues) and I'll fix them :)

### What about installing it?

Installation is just like Unicorn 3: Install the `Unicorn` NuGet package, and follow the directions in the README that will launch on installation to set up configuration(s).

NOTE: [Dilithium](https://kamsar.net/index.php/2017/02/Unicorn-4-Preview-Project-Dilithium/) ships disabled by default. If you want to enable it, make a _copy_ of `Unicorn.Dilithium.config.example` and enable it.

### What about upgrading to it?

If you're coming from classical Unicorn 3.1 or later, upgrading is actually really simple: just upgrade your NuGet package. Unicorn 4 changes nothing about storage or formatting (_except_ that the `__Originator` field is no longer ignored by default), so all existing serialized items are compatible.

Taking advantage of the config enhancements detailed above is also entirely optional: Unicorn 3 configurations are totally readable by Unicorn 4.

If you're invoking Unicorn via its PowerShell API, make sure to upgrade your `Unicorn.psm1` to the Unicorn 4 version to ensure correct error handling with the streaming console.

Have fun!