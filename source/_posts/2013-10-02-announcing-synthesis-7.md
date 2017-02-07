---
layout: post
title: "Announcing Synthesis 7"
date: 2013-10-02 22:37:07 -0800
comments: true
categories: Synthesis
---
Synthesis 7 is now [available on NuGet](https://www.nuget.org/packages/Synthesis). This version brings a number of new features and bug fixes largely aimed at supporting unit testing of sites that are built with Synthesis. Version 7 is compatible with Sitecore 7 Update-1 and Update-2.

## Fully mockable generated item classes

Previous versions of Synthesis would generate classes that had concrete field types, such as `RichTextField`, that were strongly tied to the Sitecore item APIs or search APIs. This prevented easily creating mock versions of the class for testing purposes. Synthesis 7 changes that by allowing you to define both a private concrete field type mapping _and_ and public interface mapping, such as this:

    [IndexField("title")]
    public ITextField Title {
        get { return new TextField(/* ... */);
    }

Now each field can be easily mocked with a mocking framework of your choice, enabling you to create totally Sitecore API free instances of Synthesis items. For example, using [Moq](https://github.com/Moq):
	
    var item = new Mock<IFooItem>();
    item.SetupGet(x => x.Title).Returns(new TestTextField("isn't this nice?"));

    string result = item.Object.Title.RawValue; // "isn't this nice?"

But wait! What about those pesky metadata properties like statistics, database, and editing? Those are annoying, huh. Well Synthesis 7 uses adapter classes to make those mockable too.

    public IDatabaseAdapter Database { get; }

It's very easy to add a public interface to a field type mapping; you simply add an interface attribute to the mapping:

    <map field="Single-Line Text" type="Synthesis.FieldTypes.TextField, Synthesis" interface="Synthesis.FieldTypes.Interfaces.ITextField, Synthesis" />

## New testing package

There is now a Synthesis.Testing package on NuGet that contains some dummy field type implementations to help you write cleaner mocks of Synthesis item types. The `TestTestField` class referred to in the Moq example above is part of the testing package.

## Injecting custom field implementations on a per-template-field basis

Previously you were able to map a Synthesis field class to a Sitecore field type. Synthesis 7 enables you to get more specific and map a field class to a specific field on a template. This enables you to provide custom behavior for specific fields.

    <templateFieldMappings hint="raw:AddTemplateMapping">
        <map template="Content Section" field="Content" type="Synthesis.FieldTypes.TextField, Synthesis" interface="Synthesis.FieldTypes.Interfaces.ITextField, Synthesis" />
    </templateFieldMappings>

The template specification can be the template name, full path, or ID interchangeably. The field can be either the field name or field ID.

## Better missing field messages

If you've used Synthesis long, you've probably seen this one. You add a template field, generate your model, and forget to publish the field. Suddenly you get a really helpful "Parameter cannot be null" error that doesn't tell you anything about the field name that actually caused it. Synthesis 7 fixes that problem, and reports the field name that was missing instead of a generic error message.

## Bug fixes

* Modified default behavior of synthesis.axd. In Synthesis 5-6, if the model was detected as synchronized, no option to force a rebuild would be available. Problem is, configuration changes (e.g. changing the root namespace) do not invalidate the template signatures - so there were times when a 'synchronized' model would need to be force rebuilt. In Synthesis 7, the handler always rebuilds the model even if synchronization is detected.
* Synthesis will no longer generate invalid C# code when faced with a template or field name that starts with a number. Leading numbers are automatically prefixed with _ so as to retain valid C#.
* Fix media item URL generation error when `LinkProvider` has `AlwaysIncludeServerUrl` enabled. Previously, links like `/http://foo/~/media/bar` would be generated. Fix contributed by [Dave Peterson](https://github.com/PetersonDave)
* Removed a hack that allowed the LINQ extension `GetSynthesisQueryable()` to work, because it was fixed in Sitecore 7 Update-1.

## Breaking changes

* Due to the new adapter classes, code that uses the Database property may break. The adapter class returns Synthesis types, whereas the Synthesis 6 version returned Sitecore `Item` instances. This enables decoupling the `IDatabaseAdapter` from the Sitecore API.
* If using interfaces for your object properties (which you will need to manually enable if upgrading), the type of those properties will change. This may break code such as view models that contains fields as properties.
* Support for implicit field conversions (e.g. to treat a TextField as a string) have been removed. These do not work when using interfaces, and generally are a bad idea because they can allow unexpected behavior (is an implicit conversion to use a raw value? field renderer?).
* Support for the `FileListField` (File Drop Area) field type has been removed
* Support for the `WordDocumentField` field type has been removed
* The `IFieldMappingProvider` interface signature has changed to enable injecting custom field types on a field level. Custom providers may need minor refactoring.
* _Synthesis.Blade:_ the signature of the `SynthesisPresenter.GetItem()` method has changed from protected to public (a breaking change for all presenters based on this). Unfortunately, this was by far the simplest and cleanest avenue to allow for injecting arbitrary Synthesis item data sources for testing purposes.

## Upgrading

Upgrading to Synthesis 7 is a process that takes a bit of work. 

* First, upgrade the NuGet package to Synthesis 7
* After upgrading the NuGet package, you should remove the field mappings in Synthesis.config to the removed Word Document field and File Drop Area field types. The Word field mapping may not exist depending on what version you're coming from.

        <map field="File Drop Area" type="Synthesis.FieldTypes.FileListField, Synthesis" /> <!-- remove this from Synthesis.config -->

* Next, you will need to regenerate the Synthesis model. Do not build the project at this point, because the model format has changed in Synthesis 7 and the existing model will be full of build errors. Hit /synthesis.axd and force a regeneration, which will remove any build errors not caused by a breaking change as outlined above.
    * If some sort of global handler (such as a `httpRequestBegin` pipeline handler) is causing an error before you even get to synthesis.axd temporarily disable it for the purposes of regenerating the model.
* By default, a Synthesis 7 upgrade will not enable using interfaces for field types (unless you overwrite Synthesis.config during the install, which is not recommended). Should you wish to enable interfaces, simply look at the [default config file](https://github.com/kamsar/Synthesis/blob/master/Source/Synthesis/Synthesis.config) and merge the interface attributes in under `<fieldMappings>` to your config, then regenerate.
* If using Synthesis.Blade, you will need to change the visibility on `GetItem()` for any presenters that use `SynthesisPresenter` as their base class to public, instead of protected.
* Verify that the case of your content search config patch at the bottom of `Synthesis.config` matches the case in your `Sitecore.ContentSearch.Lucene.DefaultIndexConfiguration.config`. The case of the default config changed in Update-2, but depending on how you upgraded yours might not have. See [this blog post](https://kamsar.net/index.php/2013/09/sitecore-7-update-2-and-synthesis-6/) for details as to why this is necesary. If the case is incorrect, Synthesis LINQ queries will not work correctly.
* Fix any remaining build issues caused by breaking changes (if any), and test the site.

## Feedback

I'd love to hear from you if you have trouble or ideas. [Send me an issue on GitHub!](https://github.com/kamsar/Synthesis/issues/new)

As always, the [Synthesis source is available](https://github.com/kamsar/Synthesis) under [the MIT license](http://opensource.org/licenses/MIT), so feel free to hack away - and send me a pull request ;)