---
layout: post
title: "Make Web Forms for Marketers use a regular connection string"
date: 2013-10-9 22:31:11 -0800
comments: true
categories: Sitecore
---
One of my pet peeves about the Sitecore Web Forms for Marketers module is that it manages the forms database connection string in its own config file, instead of the regular ConnectionStrings.config file.

Fortunately this is easy to fix if you want to manage all your connection strings in one place. First, you need to override the default `WFMDataProvider` like so:

{% gist 6903359 %}

Then, you modify the data provider declaration in `App_Config\Include\forms.config` (replace the namespace and assembly with wherever you place the class above in your project):

{% gist 6903413 %}

Finally, simply add a connection string named forms (or whatever name you configured in forms.config above) to your `ConnectionStrings.config` and you're good to go!