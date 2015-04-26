---
layout: post
title: "Announcing Blade 2"
date: 2013-10-02 22:27:07 -0800
comments: true
categories: Blade
---
I'm happy to announce that [Blade v2 is now available from NuGet](https://www.nuget.org/packages/Blade). There are some very nice new features in this release.

## MVC4 Razor APIs instead of RazorEngine

Blade 2.0 replaces the RazorEngine library with directly calling MVC4 APIs. Doing this is much cleaner (and faster!) than the previous Razor implementation. This does introduce some minor breaking changes so read up before you upgrade.

* Useful errors with source context if there's a syntax error or runtime error in a razor file. This was a major issue with previous versions of Blade that made razor errors difficult to debug.
* Full support, including IntelliSense, for using `@model` to declare the model type in razor views instead of the much more verbose `@inherits RazorRendering<TModel>` previously required. The old way still works fine too.
* The `BladeHtmlHelper` is gone, replaced by the normal MVC `HtmlHelper` - note that pieces of the helpers that do not make sense in a pseudo-MVC context, such as `ActionLink`, may behave unpredictably.
* View resolution follows the conventions of the MVC view engines that are registered. This means that path resolution is a bit different:
    * Html.Partial() will require absolute paths (`~/Foo/Bar/Baz.cshtml`) unless you are storing your Razor files in `/Views` like MVC does
    * When statically including Razor renderings with the `RazorTemplate` web control, you must add `.cshtml` on the end of view paths, whereas previously that was optional
    * Razor layout paths are unchanged from previous behavior: they can be relative, and you need .cshtml on them
    * You should be able to plug in whatever view resolution logic you desire by adding in your own view engines in Global.asax just like with MVC sites

## Form support and model binding!

Presenters can now handle postback in a clean fashion by using ASP.NET MVC model binding to bind posted values to your ViewModel automatically. There is also explicit support for handling XHR (AJAX) postbacks separately, enabling easy creation of simple callbacks to the same page that return JSON data.

* MVC model validation attributes (e.g. `[Required]`) and validation controls are fully supported for Razor views, as are MVC HTML helpers (`@Html.TextBoxFor(x=>x.MyStringProperty)`, for example). User Controls and Web Controls support validation, but they do not have `ModelState` to pass it around to so you'd need to manually parse the validation result in the presenter for those.
* To support postback, if using a `SitecorePresenter` or `SynthesisPresenter`, simply override the `HandlePostBackWithModelBinding` method. The method is invoked if HTTP POST is requested, and model binding is automatically executed. Similar methods exist for XHR and non-automatic-modelbinding for both.
* The postback handling occurs immediately after the OnLoad event just like web forms usually does for submit event handlers. You can assume that `GetModel()` will have already been called on the presenter.
* Adding custom `ModelBinders` in Global.asax, just like in MVC, should work perfectly.

## Unit testable presenters

In Blade itself, the `SitecorePresenter` has been modified to allow injection of a custom `Item` as the data source of the presenter (via the `SetDataSource()` method), enabling you to decouple it from data source resolution logic. However, you probably should not be testing with `Item` instances, as those require a lot of Sitecore API presence. To get around this issue, Synthesis.Blade's `SynthesisPresenter` enables injection of a custom Synthesis item interface directly into its `GetItem` method for completely Sitecore-independent tests! Check out this trivial example of testing a SynthesisPresenter, using Moq:

    var item = new Mock<ITestTemplateItem>();
    var view = new Mock<IView>();

    // todo: setup mocks appropriately

    var presenter = new TestPresenter();

    var model = presenter.GetModel(view.Object, item.Object);

    // todo: assert something about the resultant model

### Testing limitations

Views are not particularly testable due to them using the MVC Razor engine, which requires a web context to execute. Hypothetically you could write tests against them using a web-based test runner, but it seems that most people do not recommend testing the view due to the relative fragility of the tests involved.

## Upgrading

To upgrade from Blade 1.5 to Blade 2.0, simply upgrade the NuGet package. You should definitely test all of your razor views after the upgrade to make sure they are running correctly against the new MVC-based razor implementation. If you are not using razor, the upgrade should be pretty much seamless.

Because Blade uses XDT-transformation-based installation routines, you will need NuGet 2.6 or later to install it.

If you are also using Synthesis with Blade, you will need to upgrade to Synthesis/Synthesis.Blade 7.0 to be compatible with Blade 2.0.

## Feedback

I'd love to hear from you if you have trouble or ideas. [Send me an issue on GitHub!](https://github.com/kamsar/Blade/issues/new)

As always, the [Blade source is available](https://github.com/kamsar/Blade) under [the MIT license](http://opensource.org/licenses/MIT), so feel free to hack away - and send me a pull request ;)