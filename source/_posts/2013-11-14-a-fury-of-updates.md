---
layout: post
title: "A Fury of Updates"
date: 2013-11-14 22:30:16 -0800
comments: true
categories: 
- Synthesis
---
Put me on a plane to the Sitecore MVP Summit and I'm going to write software! All of these updates are available on [GitHub](https://github.com/kamsar) for source and NuGet for packages.

# Synthesis.Blade.Ninject

This is an extension to Synthesis.Blade that enables automagical constructor dependency injection to any presenter using the Ninject IoC container. I previously blogged about how to do this (it's not that complex) but now doing DI with Blade is as simple as a NuGet package. Simply wire up Ninject with your dependencies, add some dependencies to a presenter constructor, install this package, and you're good to go. Here's a simple example of a presenter using DI to get a repository dependency:

	public class TestPresenter : SynthesisPresenter<TestModel, ITestTemplateItem>
	{
		private readonly ITestRepository _repository;

		// this constructor parameter will be automatically set by Blade via Ninject
		public TestPresenter(ITestRepository repository)
		{
			_repository = repository;
		}

		public override TestModel GetModel(IView view, ITestTemplateItem item)
		{
			var model = new TestModel();

			model.Frob = _repository.GetFrob(item);

			return model;
		}
	}

Note: It's trivial to implement the same constructor injection with a different IoC container, or without using Synthesis integration. Take a look at the source for Synthesis.Blade.Ninject and it should be pretty obvious what to do :)

Note2: Make sure to use the above Synthesis.Blade 7.0.1 configuration file, or the installation may not work correctly and patch the wrong thing.

# Blade 2.0.1

This is a minor release with enhancements to testability and bug fixes.

- The "Page" property on IView is now marked as obsolete, as it exposes a lot of untestable data to a presenter
- `IView` now has a `ViewContext` property that exposes the `HttpContextBase` for the current request, enabling testable access to request and response data within presenters
- Fixed a bug that caused unexpected presenter resolution when using inheritance in model types (`IView` became covariant in Blade 2, which meant that `IView<BaseClass>` became similar to `IView<DerivedClass>`, and the base class presenter might be selected first in some cases) - fix courtesy of [@gobiner](https://twitter.com/Gobiner)

# Synthesis 7.0.1

This is a minor release that corrects an oversight in 7.0.0 where some of the new field type interfaces did not have setters for their values. This prevented editing items when using field type interfaces. Bug report courtesy of [@gobiner](https://twitter.com/Gobiner).

# Synthesis.Blade 7.0.1

This is a minor release that resolves an issue with the default configuration file if multiple patches were present to the Blade `PresenterFactory`. To upgrade an existing installation, simply edit Synthesis.Blade.config and change the presenter factory patch line to read like so (changing the 'instead' attribute):

    <presenterFactory patch:instead="*" type="Synthesis.Blade.Configuration.SynthesisPresenterFactory, Synthesis.Blade" />

