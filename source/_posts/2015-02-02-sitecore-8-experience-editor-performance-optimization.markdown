---
layout: post
title: "Sitecore 8 Experience Editor Performance Optimization"
date: 2015-02-02 14:46:19 -0800
comments: true
categories: sitecore
---

Sitecore 8 is pretty nice, but not without its share of rough edges. One persistently annoying issue I came across is that after you compile your solution in Visual Studio, it is *glacially* slow to start up in Experience Editor (formerly Page Editor) mode. Given that most sites built in modern Sitecore fashion are using component based architecture that is Experience Editor centric, this is a big drag on development time.

In this post I'll outline the [story so far](https://twitter.com/kamsar/status/562315734710108160), and some tweaks you can perform to improve the performance of Sitecore 8 after a compilation.

As a baseline, on my i7-3770k, 32GB RAM, and all-SSD system EE 8.0 takes __1:30__ to startup after a compile. Yeah, a minute and a half. Sitecore 6.5 will start in the same situation in __0:21__.

# Option 1: Precompilation

John West and Kevin Obee noted that I should try disabling the SPEAK precompilation features. These are in the `<initialize>` pipeline and registered in `Sitecore.Speak.config` and `ContentTesting\Sitecore.ContentTesting.config`. If you comment these out, you trade having the SPEAK interfaces come up slower the first time (because they are not precompiled) for faster initial startup time (because you skip precompiling).

With those two precompilers commented out, my startup time dropped massively to __0:41__. Still twice as slow as Sitecore 7.2, but oh so worth it when you're compiling all day.

# Option 2: Optimize Compilation

Still unsatisfied with spending 20 seconds compiling unchanged .cshtml SPEAK views every time I compile my application, I started looking around into the actual compilation process. An interesting facet of this issue is that it's *not* triggered by simply restarting the app pool. You must touch an assembly in the bin folder to trigger the slow startup.

A clue came in the form of this [StackOverflow answer](http://stackoverflow.com/a/1419979) from [@pbering](https://twitter.com/pbering). He mentioned enabling the `optimizeCompilations` setting in the web.config. What does this do? Well, off to [MSDN](https://msdn.microsoft.com/en-us/library/ms366723.aspx). If you read this document, it makes it clear that when you change a _top-level file_ (which includes anything in `bin`) it invalidates _all_ dynamically compiled assets because a _dependency of their compilation has changed_. Razor files, such as what SPEAK uses, are dynamically compiled - .NET parses them, compiles them to a C# class, and stores them off in the `Temporary ASP.NET Files` folder.

So essentially what happens is that Sitecore 8 depends on a WHOLE LOT of dynamically compiled SPEAK renderings. When you start it up the first time, ASP.NET compiles them and stores them in its compilation cache. As long as you don't change any dependencies - e.g. dlls, global.asax - .NET can update the cache selectively when the cshtml file is changed, and it's fast. But as soon as a dependency is changed - e.g. your site dll - .NET throws out the whole site's cache and compiles it all again, just to be safe. _With Sitecore 8, this takes 20-60 seconds at least_.

You can instruct .NET to ignore dependency changes and only recompile dynamically compiled files when _the file itself is modified_. This has some potential dangers _if you modify a dependency without changing the dynamically compiled file and make a breaking change_; [see the MSDN article](https://msdn.microsoft.com/en-us/library/ms366723.aspx) for details of the things to be aware of if you enable optimization.

__The good news is, if you turn on `<compilation optimizeCompilations="true">` in your web.config, there is no longer any SPEAK recompiling on startup! Even better, it starts up in 0:13, beating 7.2 by 8 seconds!__

That said, be aware that there are significant issues to be aware of when enabling optimizeCompilations. Read the MSDN article. I have not deeply tested using this setting, other than it removes the startup speed issue. If you run into problems, stopping IIS and clearing the Temporary ASP.NET files folder should clear it up. _Rebuild may not help._

Hope that helps!