title: Controlling Rendering Order in Sitecore MVC
date: 2017-01-13 16:25:47
categories: Sitecore
---

Back in the bad old Web Forms days, rendering a page was a multi-step process. There were `Init`, `Load`, `PreRender`, and `Render`, just to name a few. This was mostly a pain to deal with as it made it hard to know when in the page lifecycle you should place your code to deal with framework side effects. It also made its fair share of inane interview questions.

But it _did_ mean one useful thing: there was a place you could put code where it would always run even if a rendering was output cached, that went before the page actually rendered. Why was this useful? Suppose you needed a certain CSS class on `<body>` when a specific rendering was added to the page. Maybe a rendering would override the `<title>`, or any number of other techniques* which require code to execute prior to page layout composition.

\* most of these techniques are just as bad as Web Forms :P

When MVC became popular, it brought its own single-step rendering pipeline. This is awesome because it's easy to understand: the page rendering starts at the top and works its way down, rendering partials and the like in order. No more "the form value is missing because you added the dynamic field after init so data binding had already run." (Seriously?) Simple.

But in a single step pipeline, there's no room for a later component to signal an earlier component anything. Take this for example:

	// support class
	public class Body
	{
		public static string CssClass { get; set; }
	}

	// layout
	<body class="@Body.CssClass">
		@Html.Sitecore().Placeholder("wrapper")
	</body>

	// inside wrapper, in a controller
	public ActionResult Rendering()
	{
		Body.CssClass = "🐘";
		return View();
	}

What will happen? Well, first the body tag will execute, then the `wrapper` placeholder will be executed. The body class will be set _after_ the `<body>` tag has been rendered: there will be no class on the `body`.

There are two ways I know of to get around this.

## Use a Layout

{% asset_img dawg.jpg %}

A layout? Aren't we already using one of those? Nope: an [MVC layout](https://docs.microsoft.com/en-us/aspnet/core/mvc/views/layout), not a Sitecore layout. In our example above, we could refactor the layout like this:

	// _Layout.cshtml
	<body class="@Body.CssClass">
		@RenderBody()
	</body> 

	// SitecoreLayout.cshtml
	@{
		Layout = "~/Path/To/_Layout.cshtml";
	}

	// the body of this is placed in the @RenderBody() in the MVC layout
	@Html.Sitecore().Placeholder("wrapper")

Here the Sitecore layout inherits the MVC layout by setting the `Layout` property. This also has the side effect that the Sitecore view is executed _before_ the MVC layout. So in our example, now the `SitecoreLayout.cshtml` is rendered (including the rendering which sets the body class), and then the `_Layout.cshtml` renders and places the Sitecore content in the `@RenderBody()`. The body class is thus set how we want.

MVC layouts are also a lovely tool if you have several Sitecore layouts which share a lot of common boilerplate wrapper code - for example, the same meta tags, favicon, or version footer - without duplicating them several times. [Sections](https://docs.microsoft.com/en-us/aspnet/core/mvc/views/layout#sections) enable you to have your Sitecore layout also write to specific places in the MVC layout. For example you might choose to expose a `head` section to enable the Sitecore layout to add CSS files or analytics scripts to the MVC layout `<head>`.

## Reversed Rendering

{% asset_img redpill.jpg %}

The second way to control your renderings' order is to use a technique I call Reversed Rendering. This is a simple trick that relies on the fact that HTML helpers are really just functions. This is a completely valid layout:

	@{
		var wrapper = Html.Sitecore().Placeholder("wrapper");
	}

	<body class="@Body.CssClass">
		@wrapper
	</body>

The `Placeholder()` method returns a `HtmlString` object, which contains the markup for the placeholder. You can take advantage of this to change the order by assigning the placeholder to a variable instead of rendering it directly into the output. Then you can control the order in which things render to make the body content render before the parent tags, thus facilitating communications.

I haven't tried it (and performance wouldn't be great dealing with large strings), but you could also theoretically use this trick to perform HTML post-processing on rendered renderings.

## What about output caching?

Output caching gains performance by not executing the whole rendering pipeline and instead serving a pre-cached static version or a rendering. The problem is that by bypassing the rendering pipeline it also bypasses the code (controller or view) that pushes data up to the layout.

So when using these techniques, make sure to not enable output caching on any renderings that are setting variables used in the layout - or else the variables won't be set as soon as the caching kicks in.

So there you have it. Go forth and have fun!