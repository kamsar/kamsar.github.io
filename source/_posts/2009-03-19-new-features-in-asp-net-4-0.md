---
layout: post
title: "New Features in ASP.NET 4.0"
date: 2009-03-19 10:35:22 -0800
comments: true
categories: C#
---
<p>ASP.NET 4.0 is going to introduce a lot of pain-point reducing features, particularly around the efficient delivery of content and standards compliance. Here are the main points as I saw them in Stephen Walther's talk this morning.</p>

<ul><li>FormView, as well as several other table-based controls: RenderTable="false" disables table wrappers</li>
<li>ListView: no longer requires a <layouttemplate>, only an item template. Be careful with empty data sets though, as your wrap tags might be left exposed.</layouttemplate></li>
<li>ViewState can be globally disabled and then selectively enabled on controls using the Control.ViewStateMode="Disabled" (Enabled, Inherit) - it defaults to 'inherit'. This is different than EnableViewState=false in that (1) it inherits and (2) you can re-enable it as needed on specific controls</li>
<li>Control.ClientIdMode property allows you greater control over the ID attribute emitted. Options include "Legacy," [how it is now] "Static," [use what you said] "Predictable," [wasn't defined] or "Inherit" allows overriding the ID value on a control. Can be set in the <pages> web.config element as well but that's probably a bad idea to change except on an as-needed basis to avoid ID collisions in repeating controls. It was noted that the "Legacy" option would probably have its name changed before release.</pages></li>
<li>New Response.RedirectPermanent() creates a 301 permanent redirect as opposed to Response.Redirect()'s 302 temporary redirect</li>
<li>ASP.NET "velocity" distributed caching, allows creating custom cache providers as well as caching on multiple machines</li>
<li>Web.config transforms allow multiple iterations of a web.config to be stored with the application. More about these later.</li>
</ul>