---
layout: post
title: "New Open Source Projects"
date: 2013-02-07 12:12:13 -0800
comments: true
categories: Unicorn
---
I've recently released several new open source projects on GitHub:<br><br><h2>
<a href="https://github.com/kamsar/Kamsar.WebConsole" target="_blank">Kamsar.WebConsole</a></h2>
<div>
A library that makes it easy to build progress-reporting "console" pages in websites. These are useful for pages that execute long-running processes like search index rebuilds, CMS publishing, etc.</div>
<div>
<a href="/images/20130207webconsole.png" imageanchor="1"><img border="0" height="272" src="/images/20130207webconsole.png" width="640"></a></div>
<div>
An example WebConsole page using the default styling</div>
<div>
<br></div>
<a href="https://github.com/kamsar/Kamsar.WebConsole" target="_blank">https://github.com/kamsar/Kamsar.WebConsole</a><br><h2>
<a href="https://github.com/kamsar/Beaver" target="_blank">Beaver</a></h2>
<div>
<div>
Beaver is a flexible build/configure/deploy system for ASP.NET projects based in PowerShell. It is designed to simplify the task of maintaining builds that may span many target environments (such as dev, QA, production) where each has its own slightly different configuration requirements.</div>
<div>
<br></div>
<div>
Everything in Beaver is based on a consistent, simple idea: the "pipeline." Pipelines are simply a folder that contain "buildlet" files that perform tasks in alphabetical order. Their structure is quite similar to POSIX System V-style init scripts, only instead of a pipeline per runlevel there's a pipeline per build stage. Buildlets can be written in several domain-specific languages (PowerShell, XDT Transform, MSBuild, or even sub-pipelines), and you can implement your own buildlet provider if you wish. Using this architecture allows you to implement small, easily digestible  single-responsibility units of action - as well as gain an immediate understanding of how the build process works by simply looking in a few folders and seeing visually the order things run in and their descriptions.</div>
<div>
<br></div>
<div>
To manage multiple target environments, Beaver implements a powerful inheritance chain of pipelines. Archetypes are a way to extend the global pipelines to perform a specific sort of configuration that might apply to multiple environments - for example, configuring live error handling or hardening a CMS site's content delivery-type servers. Environments then declare themselves as using 0-n archetypes (in addition to being able to extend the global pipelines themselves), allowing them to inherit cross-environment configurations.</div>
<div>
<br></div>
<div>
Beaver was originally designed to support build and deploy operations for the Sitecore CMS. However, Sitecore is merely a set of archetypes - the system can be used for any sort of ASP.NET project.</div>
</div>
<div>
<br></div>
<div>
<a href="https://github.com/kamsar/Beaver" target="_blank">https://github.com/kamsar/Beaver</a></div>
<h2>
<a href="https://github.com/kamsar/Unicorn" target="_blank">Unicorn</a></h2>
<div>
Unicorn is a utility for Sitecore that solves the issue of moving templates, renderings, and other database items between Sitecore instances. This becomes problematic when developers have their own local instances - packages are error-prone and tend to be forgotten on the way to production. Unicorn solves this issue by using the Serialization APIs to keep copies of Sitecore items on disk along with the code - this way, a copy of the necessary database items for a given codebase accompanies it in source control.</div>
<div>
<br></div>
<div>
Unicorn avoids the need to manually select changes to merge unlike some other serialization-based solutions because the disk is always kept up to date by the event handlers. This means that if you pull changes in from someone else's Sitecore instance you will have to immediately merge and/or conflict resolve the serialized files in your source control system - leaving the disk still the master copy of everything. Then if you execute the sync page, the merged changes are synced into your Sitecore database.</div>
<div>
<br></div>
<div>
Installing Unicorn is very simple and using it is nearly automatic as long as you follow the conventions that allow it to work (local databases required).</div>
<div>
<br></div>
<div>
<a href="https://github.com/kamsar/Unicorn" target="_blank">https://github.com/kamsar/Unicorn</a></div>