---
layout: post
title: "Selecting a Sitecore Rendering Technology"
date: 2013-02-07 12:10:45 -0800
comments: true
categories: Sitecore
---
<br>
Sitecore has a dizzying array of ways you can write renderings. While there is documentation that explains what they are, there’s a lot less about where each kind should be used. This will attempt to explain when you should use each kind of rendering.<br><br>
So what kinds of renderings are there?<br><ul><li>Sublayouts (UserControls/.ascx)</li>
<li>Web controls (WebControl/.cs)</li>
<li>Razor renderings (.cshtml) - Sitecore MVC-style</li>
<li>Razor renderings (.cshtml) - WebForms-style</li>
<li>Method renderings (.cs)</li>
<li>URL renderings</li>
<li>XSLT renderings (.xslt)</li>
<li>Custom - yes, you can write your own rendering provider</li>
</ul>
That's a lot of ways to emit HTML to a page. Let's take a look at each kind and examine their strengths and weaknesses.<br><h2>
Sublayouts (User Controls)</h2>
These are probably the kind of rendering you should be using most of the time unless you have Razor at your disposal. Sublayouts are confusingly named because most of the time they are simply a rendering and not an actual subdivision of a layout. These are basically identical to a traditional .NET User Control - they have a page lifecycle, code behinds, events and all the other traditional Web Forms trappings. They have a relatively HTML-like appearance that makes them sensible to edit if you have HTML/CSS folks collaborating with you, unlike the C#-based renderings.<br><br>
However they also have the same issues as their User Control cousins. Web Forms' at times utterly verbose syntax and confusing event/postback model can introduce bugs. Highly branched markup emission is also very hard in User Controls because the markup is all encoded in the .ascx file, and you have to resort to MultiViews or PlaceHolders and setting a ton of Visible properties to make it work.<br><br><b>Verdict:</b> Use these for relatively static markup emission or places where the Web Forms event model will help you - like custom form renderings.<br><h2>
Web Controls</h2>
Web controls are simply C# classes that derive from the Sitecore WebControl base class. Web controls are perfect if you have to do a rendering whose markup has a lot of branching, for example a list that might have two or three different kinds of elements in the list because you can modularize the rendering into C# methods.<br><br>
On the other hand WebControls can be extremely hard to read if not written in a disciplined manner. There is no obvious HTML emission, so you'll have trouble with HTML/CSS folks when they need to change markup or add a class - it's all in C#. You can also write spaghetti renderings that are very hard to follow how the code runs. You also have to remember that unless you override the GetCachingID() method, your WebControl cannot be output cached.<br><br><b>Verdict: </b>Use these for places where you need tight control over HTML emitted, or have a lot of branches in your markup emission.<br><h2>
Razor Renderings (MVC-style)</h2>
Razor is a templating language traditionally associated with ASP.NET MVC. It dispenses with a lot of the usually unnecessary page lifecycle of a Web Form for a more simple template that is both readable as HTML and allows a decent amount of flexibility in terms of composing branch-heavy renderings. If you're using Sitecore's MVC support it's a no-brainer to use Razor renderings for nearly all purposes.<br><br>
However to use the built in Razor support you must use Sitecore's MVC mode - which means you have to do everything in Razor. You also have to register controllers and views as items, and lose a lot of module support - for example, Web Forms for Marketers cannot presently run in MVC mode. At present this makes it nearly untenable to implement most real world sites using Sitecore's MVC mode.<br><br><b>Verdict: </b>If you've got a Sitecore MVC site, use it.<br><h2>
Razor Renderings (custom)</h2>
There are a couple of third party shared source modules (such as Glass) that have implemented a custom rendering type that invokes the Razor template engine outside of a MVC context. This means you could reap the benefits of a Razor template's syntax, without needing to resort to Sitecore's shaky configuration-over-convention MVC implementation. These are dependent on how you feed the view data to the Razor rendering however, and each implementation works slightly differently.<br><br><b>Verdict:</b> If you're using one of these modules, you'll probably implement most of your renderings in Razor without needing Sitecore MVC<br><h2>
Method Renderings, URL Renderings</h2>
These render output from a C# method that returns a string, and a given URL respectively.<br><b><br></b>
<b>Verdict:</b> There's almost no good use case for either of these rendering types.<br><h2>
XSLT Renderings</h2>
These were once the main type of rendering in use. They use extended XSLT syntax to transform an item "XML" into a rendering. While they can be useful for EXTREMELY simple renderings, they do not scale well to any level of complexity. Most very simple renderings may at some point gain a measure of complexity, which would then mean either very ugly and slow XSLT or a complete rewrite in a different rendering technology. Do yourself a favor and save the rewrite - use something other than XSLT in the first place.<br><br><b>Verdict:</b> Don't use.<br><div>
<br>
Feel free to take these recommendations with a grain of salt. These are my opinions, based on the projects I've worked on. Your project may come across good reasons why I'm absolutely wrong about one of these options. Keep your eyes open :)</div>