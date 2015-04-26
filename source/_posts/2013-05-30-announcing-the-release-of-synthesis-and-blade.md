---
layout: post
title: "Announcing the release of Synthesis and Blade"
date: 2013-05-30 22:53:46 -0800
comments: true
categories: Synthesis
---
<p>For years at <a href="http://isitedesign.com/">ISITE Design</a> we've been inventing ways of making Sitecore development easier and more fun. Today I'm happy to announce that some of those tools we've been using internally for ages are now publicly available for everyone to use.<br></p>

<h2>Synthesis</h2>

<p>Synthesis is an object mapping framework that automatically generates strongly typed C# classes to represent your Sitecore templates. While this sort of library <a href="http://glass.lu/">is</a> <a href="https://github.com/hermanussen/CompiledDomainModel">nothing</a> <a href="https://github.com/Velir/Custom-Item-Generator">new</a> in Sitecore circles, Synthesis brings some unique features to the table:</p>

<ul>
<li>A universal object interface for Sitecore 7. The same generated objects that work in database queries also work great in LINQ against an index. If you access data that isn't in the index, they'll even transparently load the item from the database for you.</li>
<li>Automatic LINQ query filtering. You won't have to remember to filter by template, language, and version when you query LINQ with Synthesis.</li>
<li>Generation of an interface hierarchy to represent template inheritance. This brings a whole world of power to you by enabling you to write polymorphic renderings. Nearly all of your interaction can be done by interface.</li>
<li>Automatic checking to see if your generated classes match what's defined in Sitecore</li>
<li>Extensible field API allows you to automatically map custom field types onto a strongly typed version of themselves. Ever dealt with custom XML fields before? This makes them awesome.</li>
</ul>

<p>All of the other expected features of an object mapper are also available, such as:</p>

<ul>
<li>Full support for Page Editor</li>
<li>Support for standard field types such as images, multilists, and rich text fields</li>
</ul>

<h3>In Action</h3>

<p>There are <a href="https://github.com/kamsar/Synthesis/blob/master/README.md">some examples of using Synthesis</a> on the GitHub page.</p>

<h3>Getting Synthesis</h3>

<p>Synthesis is available now <a href="https://nuget.org/packages/Synthesis">on NuGet</a> for a one-click install. You can also see <a href="https://github.com/kamsar/Synthesis">the source</a> on GitHub, or review <a href="https://github.com/kamsar/Synthesis/wiki">the documentation</a>.</p>

<p><i>Which NuGet package should I download?</i> For installing into a web project, use the Synthesis package - it comes with configuration files. If you're referencing it from a library, use the Synthesis.Core package which is binary only.</p>

<h2>Blade</h2>

<p>
While Synthesis handles the mapping side, Blade handles making renderings awesome and has some nice integration points with Synthesis.</p>

<p>Blade uses the <a href="http://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93presenter">MVP pattern</a> to allow you to create very DRY, single-responsibility renderings, allowing you to focus on presentation code and not data access. With Blade, your renderings only request the type of model they expect. For example:</p>

<pre>public partial class SampleSublayout : UserControlView&lt;MyViewModelClass&gt;</pre>

<p>Then Blade steps in and maps the model type onto a Presenter class, which defines how to supply the model type. For example:</p>

<pre>
public class SamplePresenter : SitecorePresenter&lt;MyViewModelClass&gt;
{
    protected override MyViewModelClass GetModel(IView view, Item dataSource)
    {
        var model = new MyViewModelClass();
        model.Example = dataSource.DisplayName;

        return model;
    }
}
</pre>

<p>This enables simple isolation of both views and presenters for writing testable frontends, and looser coupling even if you aren't testing. Don't worry, not every rendering needs a presenter. If you're just using a model type from a Sitecore object mapper such as Synthesis (via the Synthesis.Blade package), Glass, or Compiled Domain Model, you can make that automatically map to the data source item without defining an explicit presenter.</p>

<h3>Ubiquitous Razor 2 Support</h3>

<p>Blade also brings first-class support of the Razor 2 templating language into Sitecore, without using the Sitecore MVC functionality. This is advantageous when developing sites that use modules that are not compatible with the Sitecore MVC rendering technology - you can reap the benefits of MVC-style renderings without the hassle of implementing renderings twice or resorting to XSLT. A Razor rendering with Blade works exactly like a sublayout - only the path is to a cshtml file instead of an ascx. There are also many useful helpers included that simplify things like rendering dates, images, and controlling page editor functionality. For example:</p>

<pre>@inherits RazorRendering&lt;MyProject.MyViewModelClass&gt;
&lt;div&gt;
&lt;h1&gt;@Html.Raw(Model.Example)&lt;/h1&gt;

@Html.TextFor(model => model.SitecoreDataField)
&lt;/div&gt;
</pre>

<p>Blade does its best to feel as much as possible like Sitecore as well as ASP.NET MVC, so it doesn't require digesting lots of unfamiliar ways of doing things. Sitecore features such as Page Editor are fully supported. MVC conventions like automatic HTML encoding, partial views, anonymous types defining HTML attributes, and relative view referencing are all there.</p>

<h3>Awesome Rendering Diagnostics</h3>

<p>Ever spent an hour figuring out why a rendering was busted, only to realize it was output cached without you realizing it? What about having to find a rendering file for a HTML/CSS dev so they can make markup changes? You'll love Blade then. Blade's rendering types emit HTML comments that indicate where a specific rendering begins and ends, as well as if it was output cached, when, and what the cache criteria were. The comments disappear when dynamic debug compilation is disabled, leaving production sites pristine.</p>

<p>Check this out:</p>
<pre><!-- Begin Rendering ~/layouts/Sample Inner Sublayout.ascx -->
&lt;!-- Rendering was output cached at 5/16/2013 9:31:42 PM, ClearOnIndexUpdate, VaryByData, VaryByQueryString --&gt;

&lt;h1&gt;Hello, world!&lt;/h1&gt;
&lt;!-- End Rendering ~/layouts/Sample Inner Sublayout.ascx, render took 0ms --&gt;</pre>

<h3>Full Sitecore 7 Support</h3>

<p>Blade supports the enhancements made to renderings in Sitecore 7. It supports renderings that decache on index update, have multiple items as data sources, index-query based data sources, and even defining your own data source resolution logic. Even better, it removes boilerplate code by automatically normalizing all kinds of data source specifications back into simple Item (or Item[]) instances.</p>

<h3>Getting Blade</h3>

<p>Blade is available now <a href="https://nuget.org/packages/Blade">on NuGet</a> (and its <a href="https://nuget.org/packages/Synthesis.Blade">Synthesis integration package</a>) for a one-click install. Make sure you have <a href="http://visualstudiogallery.msdn.microsoft.com/44a26c88-83a7-46f6-903c-5c59bcd3d35b">Sitecore Rocks</a> configured and connected before installing Blade as it uses <a href="http://vsplugins.sitecore.net/Sitecore-NuGet.ashx">Sitecore NuGet</a> to install some supporting items. You can also see <a href="https://github.com/kamsar/Blade">the source</a> on GitHub, or review <a href="https://github.com/kamsar/Blade/wiki">the documentation</a>.<br><br><i>Which NuGet package should I download?</i> For installing into a web project, use the Blade (and/or Synthesis.Blade) package - it comes with configuration files. If you're referencing it from a library, use the Blade.Core (and/or Synthesis.Blade.Core) package which is binary only.</p>

<h2>Feedback</h2>

<p>I'd love to hear what you think about these libraries or answer questions about them. Feel free to comment below, submit issues on the GitHub repositories (bonus points if there's a PR attached!), tweet <a href="https://twitter.com/kamsar">@kamsar</a>, or yell "hey you!" over the back fence.</p>