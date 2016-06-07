title: Synthesis 8.2.1 Released
date: 2016-06-06 19:18:07
categories: Synthesis
---
I am happy to announce that Synthesis 8.2.1 is available on NuGet. This release primarily adds additional features. 

## What's New?

### Improved Multiple Configuration Support

Previously registering multiple configurations in Synthesis was possible but way too hard. Configurations may now register themselves using code, similar to MVC area registrations.

To register a new configuration with Synthesis 8.2.1:
* Add a class to the assembly you want the configuration's model to live in that derives from `SynthesisConfigurationRegistration`
* Implement the abstract members of `SynthesisConfigurationRegistration`. Much deeper customization is also available by overriding other optional members.
* Add a `SynthesisConfigRegistrar` processor to the `initialize` pipeline that is set to scan your assembly, e.g.
```
	<pipelines>
		<initialize>
			<!-- IMPORTANT: Each registrar instance must have a unique hint value for the patch to work correctly. -->
			<processor type="Synthesis.Pipelines.Initialize.SynthesisConfigRegistrar, Synthesis" hint="Hinty McHintface">
				<assemblies hint="list:AddAssembly">
					<meAssembly>My.Assembly.WithModel</meAssembly>
				</assemblies>
			</processor>
		</initialize>
	</pipelines>
```
* That's it! Your configuration will now be activated.

### Auto Friending

{% asset_img bemyfriend.jpg "WHAT DID I EVER DO TO YOU?" %}

Along with simpler configuration registration, allowing your configurations to reference each other's generated classes is also far easier with "Auto-Friending."

To illustrate this, suppose you were using Habitat and in your Project layer you had templates that inherited Feature templates. Without auto-friending (or previously manual friending), the Project would generate duplicate interfaces for the Feature templates. This is a bad thing. But with friending, the Project generated template will simply inherit from the already existing Feature model, extending implicit dependency in the database into explicit dependency at a code level (also a good thing!).

With auto-friending, configurations automatically friend each other in the order they are registered. So for the example above, as long as the Feature's model configuration was registered before the Project's model, everything would Just Work. You can control the order of registration by the order in the `SynthesisConfigRegistrar` assemblies.

### IRenderingContext and improved IoC support (Synthesis.Mvc)

A pattern that I've been using for a while ([as have others](http://www.martywoods.nl/unit-testing-sitecore-mvc-its-easy-when-using-synthesis/)) to improve testability is to make a facade over the `RenderingContext` that turns it into a Synthesis API and removes all `Item` dependencies. This `IRenderingContext` can be registered with your IoC container (to `SitecoreRenderingContext`) and constructor-injected into controller renderings to make `Item`-free controllers that are easy to test without any hacks. Even awesome hacks like FakeDb.

For example:
```
public class FooController : Controller
{
    private readonly IRenderingContext _renderingContext;

    public FooController(IRenderingContext renderingContext) 
    {
        _renderingContext = renderingContext;
    }

    public ActionResult Foo() 
    {
        var dataSource = _renderingContext.GetRenderingDatasource<IExpectedTypeItem>();

        if(dataSource == null) 
        {
            // no datasource set, or datasource is wrong template type (or context item, if no datasource set)
            return Content("Derp.");
        }

        var model = new FooViewModel(dataSource);

        // set other model props here

        // Note that none of this controller directly used Sitecore APIs and thus does not require FakeDb nor HTTP context
        // to have unit tests written against it.

        return View(model);
    }
}```

Additional, more specific interfaces are also available for more specific use cases: `IContextItem`, `IContextDatabase`, and `IContextSite`. These can all be bound to `SitecoreRenderingContext` and provide smaller interfaces for more specific tasks.

### Config Patching by Default

Synthesis now ships with `Synthesis.LocalConfig.config.example`, which is designed to be duplicated to form your own configuration patch. This encourages leaving the default configurations alone, which in turn greatly simplifies upgrading. The documentation and README has also been updated to reflect this.

## Improvements
* The default settings have been improved:
	* Creating model backup files is now disabled by default because it is of questionable utility when using source control
	* The `InterfaceOutputPath` and `ItemOutputPath` settings have been deprecated and merged into a single `ModelOutputPath` because it's silly to emit more than one model file. The separate settings will still operate if you wish to use them.
	* Uncommonly used settings have been removed from Synthesis.config (`SitecoreKernelAssemblyPath`, `SynthesisAssemblyPath`, and `InterfaceSuffix`). They continue to work if set, but are removed for brevity as they are generally not used other than at default values.
* The `SynthesisEditContext` class has been marked obsolete because [the pattern is a bad idea](http://sitecore.alexiasoft.nl/2006/04/03/new-way-of-editing-items/#comment-16)
* WebForms related classes have been marked obsolete because don't use WebForms
* The ability to attempt to automatically rebuild the project containing the model on startup has been removed due to being a generally bad idea
* The `ModelOutputBasePath` setting has been added. This path is prepended to the `ModelOutputPath` for all configurations. The advantage of using this is that for people who work out of webroot, they can use `<sc.variable>` values in a setting (e.g. the out of webroot project location) whereas Sitecore does not expand variables in the `ModelOutputPath`. [#27](https://github.com/kamsar/Synthesis/issues/27)

## Bug Fixes
* Setting max backups to 0 no longer results in an infinite loop and now does not make backup files [#28](https://github.com/kamsar/Synthesis/issues/28)
* Registering the same assembly twice in the type list provider will no longer scan the assembly twice
* Wildcards that do not end in * (e.g. Foo.*.Web) will not operate correctly when adding assemblies to the type list provider

## Upgrading

Upgrading should be as simple as a NuGet upgrade*. If you have customized your Synthesis.config you may need to merge it with the default (or even better make it a patch file).

\* as long as you are using Sitecore 8.1. As with the 8.2.0 release, it is designed only for Sitecore 8.1 due to breaking Sitecore API changes in 8.1. Sorry about that :(

## Thanks!

Thank you to the community members who contributed to this release:

* [Alex Washtell](https://github.com/AlexKasaku)
* [Dana Hanna](https://github.com/thesoftwarejedi)
* [patocl](https://github.com/patocl)
* [Andy Shokry](https://github.com/andyshokry)
* [Martijn Bos](https://twitter.com/martijn_b0s)
* [Mauricio Ulate](https://github.com/mulate)

As always, happy coding!