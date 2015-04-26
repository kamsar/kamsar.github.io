---
layout: post
title: "Going to MIX09, or ASP.NET MVC vs the mighty CMS"
date: 2009-03-17 08:38:19 -0800
comments: true
categories: 
---
<p>Well I'll be getting on the plane to go to <a href="http://2009.visitmix.com/">MIX09</a> in a few hours. I'm really looking forward to it, since there are a lot of ASP.NET MVC talks this year by <a href="http://www.hanselman.com/blog">many</a> <a href="http://blog.wekeroad.com/">.NET</a> <a href="http://haacked.com/">people</a> who I have a lot of respect for.</p>

<p>Since I do a lot of <dfn title="Content Management System">CMS</dfn> development ASP.NET MVC has always a bit of an odd proposition technically. CMSes pretty much always want to enforce their own idea of how to program, which tends to be technically about as advanced as putting SQL statements right on the page so their marketing department can make claims about how easy it is to use if you require practically no advanced functionality. It's a constant battle to figure out ways of writing maintainable, "good" code within the confines of a CMS's API, and ASP.NET MVC is definitely one of the tools I'm looking at.</p>

<p>The main problem with integrating ASP.NET MVC into a CMS installation is that almost all CMS like to have a fully hierarchical URL setup; i.e. http://mysite.com/some/long/path/to/rewritten/content.aspx that would match /some/long/path... in the admin interface. The default routing system from what I've been looking at won't really support this setup since what we're really doing is defining what item in the (dynamically created and modifiable) hierarchy to look at as opposed to a (static, defined in code) controller action to invoke. So I think it's likely I'd need to implement my own version of routing, but that brings with it it's own issue that the custom routing then needs to know a lot about the data before passing control off to a controller action to render a page using a specific template renderer (based on the type of page it finds at the location in the hierarchy). I'm still looking around and hopefully can corner one of the MVC guys at MIX and see if they have better ideas :)</p>