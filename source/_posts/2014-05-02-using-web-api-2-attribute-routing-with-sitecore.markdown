---
layout: post
title: "Using Web API 2 Attribute Routing with Sitecore"
date: 2014-05-02 19:25:07 -0700
comments: true
categories: sitecore
---

There are a lot of times when developing a Sitecore site that you need to introduce dynamic, API-driven content - such as search autocompletes, single page apps, and other dynamic client-side content. Sitecore itself provides the Item Web API, but sometimes you want a more targeted solution designed for end users to acquire data in a fashion that exposes as little as possible.

Microsoft provides a technology that does this pretty nicely: the aptly named [ASP.NET Web API](http://www.asp.net/web-api). Web API uses a ASP.NET MVC-like controller to easily expose REST services over JSON or XML. I set out to make it work with Sitecore. Of course, [some other folks](http://patrickdelancy.com/2013/08/sitecore-webapi-living-harmony/) had already [got this to work](http://nttdatasitecore.com/Blog/2013/November/Using-Web-API-with-Sitecore-ASPNET-Web-Forms). Unfortunately their work is also in some cases out of date - so let's untangle things.

## How to get Web API working on Sitecore, by version

### Sitecore 6.x or 7.0 (without Sitecore MVC)

If you're using Sitecore 6 or 7.0 without Sitecore MVC enabled, you can use the directions in the previously referenced blog posts to get Web API configured. By default, Sitecore will not respect ASP.NET routing unless Sitecore MVC is enabled so you have to use the solutions these blogs present to make Sitecore ignore routed URLs.

### Sitecore with Sitecore MVC enabled

When Sitecore MVC is enabled (and you cannot turn it off on Sitecore 7.1 and later), you do not need the pipeline handler presented in the existing blog posts about Sitecore and Web API - Sitecore will automatically hand off routed requests to the route handler out of the box.

## Web API versioning

At this time there are two versions of Web API: 

* v1, which released concurrently with ASP.NET MVC 4.
* v2, which released concurrently with ASP.NET MVC 5. 

If you're on Sitecore 6 you'll be using v1, because Sitecore 6 runs on .NET 4.0, and Web API 2 requires .NET 4.5. On Sitecore 7, you have more options. With 7.0 and MVC disabled, you can run v1 or v2. However on Sitecore 7.0 with MVC or 7.1, this adds a dependency on ASP.NET MVC 4 so you are limited to Web API v1 unless you [follow the instructions in this post](/index.php/2013/12/using-mvc-5-with-sitecore-7-1/) to add binding redirects and config modifications to enable MVC 5. The landscape thankfully improves with Sitecore 7.2, as that uses ASP.NET MVC 5.1 which easily adapts to Web API 2. _Note that attribute routing, the point of this post, requires Web API 2_

## Attribute routing?

About time we started talking about the core of this post, eh? Attribute routing is a snazzy new way to register routes for Web API and ASP.NET MVC that was introduced in Web API 2/MVC 5 (and it was adapted from a community developed module for MVC 4). In earlier versions, you had to register routes in a global route table that was generally constructed all in one global class. This made developing relatively ad-hoc data services that provide what amount to an "AJAX backend for a CMS page" somewhat problematic as your dependency of a small part of the site now has to stick its fingers in everyone else's pie by adding a global route.

Attribute routing solves this issue by allowing you to use standard .NET attributes (you know, like `[HttpPost]` on a MVC action method) to define the route to the code. This makes a lot more sense to my way of thinking. Here's an example attribute-routed Web API 2 controller:

{% gist 41b36cbf412b81c1d75a %}

[Microsoft has written pretty detailed documentation](http://www.asp.net/web-api/overview/web-api-routing-and-actions/attribute-routing-in-web-api-2) about Web API 2 attribute routing, which I suggest you review before getting serious about using this stuff.

## Enabling attribute routing in Sitecore

If you read the aforementioned documentation, you're probably already well on your way to enabling attribute routing in Sitecore since it's the same as any other site - all you have to do is register the attribute routes.

The question then becomes how to do that in Sitecore. I'd say the official route is to add a pipeline handler to the `initialize` pipeline that does the registration. I decided to go with a more pure .NET approach, and register it using [WebActivator](https://github.com/davidebbo/WebActivator) - look ma, no config file! I created `App_Start\WebApiConfig.cs` - a pattern which may seem familiar to folks who've recently worked on ASP.NET MVC sites - and registered my attribute routes:

{% gist d7f47ff20e59d8fdd4f5 %}

So now all I have to do is:

* Install WebActivatorEx from NuGet
* Copy WebApiConfig.cs into my project
* Create an attribute-routed controller to do stuff

Pretty simple, and - if you're using Sitecore 7.2 - easy to get set up on as well. For Sitecore before 7.2 there are some potential hoops to jump through to get attribute routing.

Anyhow, I hope this was helpful to someone.
