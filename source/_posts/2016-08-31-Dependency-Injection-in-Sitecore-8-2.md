title: Dependency Injection in Sitecore 8.2
date: 2016-08-31 17:30:03
categories: sitecore
---

Sitecore 8.2, hot off the presses yesterday, includes built in [dependency injection](https://en.wikipedia.org/wiki/Dependency_injection). If you've been following the internal architecture of Sitecore for very long, you've probably realized that a lot of it is uglier than it needs to be - and less extensible - thanks to the lack of system-wide DI support. It's kind of a big deal. Even if [Nat Mann](https://twitter.com/cardinal252) hates conforming containers ;)

The Sitecore [IoC](https://en.wikipedia.org/wiki/Inversion_of_control) container is based on [Microsoft's Dependency Injection](https://docs.asp.net/en/latest/mvc/controllers/dependency-injection.html) library that was written for .NET Core. It is not the world's most flexible or performant container (nor was it designed to be), but it works and is a decent choice. If you wish you can change the underlying IoC library being used, but we aren't going to cover that today.

Sitecore's built in DI offers two major advantages over the third party dependency injection that most advanced Sitecore teams have been using for a long time:

* Sitecore itself is using it (and thus you can extend Sitecore itself by replacing dependencies)
* You can natively dependency inject into pipeline processors (which you could [sort of already do](https://cardinalcore.co.uk/2014/07/02/sitecore-pipelines-commands-using-ioc-containers/))

## Controller Injection

As with most ASP.NET DI implementations, most dependency resolution will take place in _controllers_. If you've already been using constructor injection with Sitecore, you may have to change absolutely nothing:

	public class FooController : Controller
	{
		private readonly IDependency _dependency;

		public Foo(IDependency dependency) 
		{
			_dependency = dependency;
		}

		public ActionResult Index() 
		{
			return View(_dependency.DoStuff());
		}
	}

Be aware that the built in IoC container has two major limitations when injecting controllers:

* You may only have _one_ public constructor for your controller - or any other registered dependency. This is a good thing, as multiple public constructors are a [DI antipattern](https://cuttingedge.it/blogs/steven/pivot/entry.php?id=97) anyway.
* The controller class must be registered with the container to be resolvable (e.g. you must register it such as `container.AddTransient<FooController>()`).

The latter limitation we can do something nice about: read on and we'll get to that when we talk about registering dependencies :)

## Pipeline Injection

You may also inject dependencies into pipeline processors using the same constructor injection pattern as controllers. Processors are _not injected by default_, presumably for performance. As with controllers, processors may only have one public constructor. I suspect, but have not tried, that this trick would also work with commands.

To inject a processor, simply add `resolve="true"` to its registration. For example:

	<processor type="Foo.BarProcessor, Foo" resolve="true" />

## Web Forms Dependency Injection

{% asset_img grumpy-cat-no.jpg %}

(You can actually do Web Forms DI with Sitecore but I'm not going to tell you how. Quit using Web Forms.)

## Service Locator

You can also resolve dependencies from the Sitecore container using the [Service Locator _antipattern_](http://blog.ploeh.dk/2010/02/03/ServiceLocatorisanAnti-Pattern/). This is where you explicitly ask the container to give you an instance of an object. It's an antipattern, and you should use it as a weapon of last resort, because it tightly couples your class to the IoC container and makes things difficult to test.

There are actually multiple ways you can use Service Locator:

	// the MVC DependencyResolver can be used
	DependencyResolver.Current.GetService<IService>();

	// the Sitecore ServiceLocator can be used
	using Sitecore.DependencyInjection;
	ServiceLocator.ServiceProvider.GetService<IServiceCollection>();

Again don't use these...unless you have no other choice.

# Registering Dependencies

Of course an IoC container is useless if it has no registered dependencies to resolve! Sitecore's container can be configured in multiple ways, all of which involve some level of XML. I heard you groan when you read that ;)

Keep in mind when wiring dependencies that the IoC container is _not multitenant_. Your dependencies are sharing the container with Sitecore's - and if you have more than one site, potentially other sites as well. So don't go expecting to have `IFoo` resolve to different implementations in different sites!

If you get confused and want to see a list of every dependency that is currently registered, along with its scope and type, there's a page for that! Just visit `/sitecore/admin/showservicesconfig.aspx` and there you are. While you're at it, check out the [other handy tools in the admin pages too](https://jammykam.wordpress.com/2016/08/23/sitecore-admin-pages-cheat-sheet-new-tools/).

## Configurators

A configurator is probably what you think of when you consider IoC configuration if you've been using any modern container library. It's a C# class that conforms to an interface where you are given a container object, and expected to wire your dependencies to it. You can register as many configurators as you like in the `<services>` section.

	<configuration xmlns:patch="http://www.sitecore.net/xmlconfig/">
		<sitecore>
			<services>
				<configurator type="MyProject.MyConfiguratorClass, MyProject" />
			</services>
		</sitecore>
	</configuration>

Here's an example configurator implementation that registers a couple dependencies. As with most containers `Transient` and `Singleton` dependencies are available, and I believe `Scoped` as well, but I'm not sure what the exact behaviour of that is in this case.

	using Microsoft.Extensions.DependencyInjection;
	using Sitecore.DependencyInjection;

	namespace MyProject
	{
		public class MyConfiguratorClass : IServicesConfigurator
		{
			public void Configure(IServiceCollection serviceCollection)
			{
				serviceCollection.AddSingleton<IDependency, Dependency>();
				serviceCollection.AddSingleton<IService>(provider => new Service("withFactory"));
			}
		}
	}

**Note**: You cannot use Sitecore Factory conventions when registering configurators, for example setting properties on the configurator with child elements. This is because the Factory also speaks DI now as a fallback, so it'd be like asking the container to resolve itself :)

## Direct Registration

You can also register individual dependencies with XML, just like we did ten years ago! I wouldn't suggest doing this as it is less expressive than a configurator, not type-checked by compilation, and probably marginally slower as well due to having to convert the string to the type for every dependency.

	<configuration xmlns:patch="http://www.sitecore.net/xmlconfig/">
		<sitecore>
			<services>
				<register 
					serviceType="Type.IName, Assembly" 
					implementationType="Type.Name, Assembly" 
					lifetime="Transient" />
			</services>
		</sitecore>
	</configuration>

# Automatic Controller Registration

If you're actually reading this, you may have noticed that I mentioned earlier that you must register every MVC controller you wish to dependency inject with the IoC container. Sounds like a drag, right? Not so fast! Pull out your robe and wizard hat, grab this handy code, and register all your controllers automatically within a configurator:

	public void Configure(IServiceCollection serviceCollection)
	{
		// configurator per project? Use this:
		serviceCollection.AddMvcControllersInCurrentAssembly();

		// configure all the things from on high by convention? Use this (Habitat as the example):
		serviceCollection.AddMvcControllers(
			"Sitecore.Feature.*", 
			"Sitecore.*.Website");

		// you can also pass Assembly instances directly, but why?
		serviceCollection.AddMvcControllers(
			Assembly.FromName("Foo"), 
			Assembly.FromName("Bar"));
	}

And without further ado, here's the code that makes that possible.

{% gist 460528edc3043b16bdccfdbd0ea0c552 %}

Have a nice day!