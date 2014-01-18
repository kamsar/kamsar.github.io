---
layout: post
title: "Sitecore preview mode and loading jQuery"
date: 2013-10-07 22:35:13 -0800
comments: true
categories: Sitecore
---
It seems like most folks these days are using jQuery on their sites. Sitecore, in fact, uses jQuery for the page editor UI. I discovered an interesting case where it can conflict with your own jQuery version that I thought I'd share.

The scenario where it happens is this:

* You are in preview, page editor, or debugger mode (any mode where the webedit toolbar renders on the site)
* You are loading your jQuery library in the page header and not the footer (note: you should load it in the footer if it's at all possible)

jQuery has a feature called noConflict that is designed to allow you to isolate it from other libraries, such as Prototype, that also claim the "$" global variable. The feature can also be used to run two versions of jQuery on the same page, and Sitecore in fact does exactly this. However, the case that noConflict does not prevent is one where a second copy of jQuery is loaded on top of an existing one.

What happens here is that your jQuery loads in the heading, then Sitecore loads its jQuery and calls noConflict on it. Unfortunately, the loading of Sitecore's JS overwrites the global `jQuery` variable, resulting in this mess:

* `jQuery` = Sitecore jQuery (1.5.5 for SC7 Update-1, pretty old)
* `$` = your jQuery

Now load a jQuery plugin that uses an [IIFE](http://benalman.com/news/2010/11/immediately-invoked-function-expression/) to load itself like:

    (function($) {
        // plugin load code here
    })(jQuery);

Yeah that's right, the plugin invokes its function passing in `jQuery` aliased as `$`. Which means that _the plugin just loaded to Sitecore's jQuery, not yours!_ Now if you invoke the plugin you'll see that it's undefined in your jQuery!

Fortunately it's relatively easy to manually fix this issue. You simply need to:

* Save your jQuery variable immediately after you load it: <br>`<script>var my_jquery = jQuery;</script>`
* Define the `webedit` placeholder explicitly on your layout, so you can control where Sitecore will load its version of jQuery:<br> `<sc:Placeholder runat="server" Key="webedit"/>`
* Set the global jQuery variable back to your saved jQuery immediately after the `webedit` placeholder definition:<br> `<script>jQuery = my_jquery;</script>`
* It's safe to let this code run even when not in preview mode, as it will simply have no effect if Sitecore does not load its jQuery. Of course you can also render it only when `!Sitecore.Context.PageMode.IsNormal` as well if you want clean output.

If you can put your copy of jQuery in the footer, you don't need to resort to any of this hackery. Hope this helps someone :)