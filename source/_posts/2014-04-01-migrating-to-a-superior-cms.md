---
layout: post
title: "Project Platypus: A content migration solution"
date: 2014-04-01 01:22:01 -0700
comments: true
categories: April 1
---

Every so often as a developer we discover new and awesome things. Sitecore is a fairly capable enterprise CMS, but there's another one that is simply better.

It all started when I started working with a beautifully designed, very consistent programming language: PHP. PHP has so many things that .NET could never imagine having, and it's so easy to use. No more pesky ORMs and other silly tools adding layers of complexity on your CMS - with PHP this is all built in and wonderfully easy. 

Once I got started with PHP it was only a short while until I started working with WordPress. I quickly realized that it was perfectly suited to extremely large enterprise multilingual and multisite sites, and that its easy to use page-based model makes it so much easier to develop and maintain than overcomplex and expensive competitors like Sitecore.

Now I set out to design a way to migrate content out of legacy Sitecore sites and into new WordPress instances. Enter Project Platypus. All you have to do is install it on your Sitecore instance, and a few clicks later you'll have a brand new WordPress site with all your content migrated over. It'll even transfer markup and Lucene indexes that are using scSearchContrib - all without any developer intervention.

In order to get people started using Project Platypus, I've made [this short video demonstrating it in action](http://www.youtube.com/watch?v=oHg5SJYRHA0). I'll be posting the source code to GitHub in the next couple days.

Oh, and happy April Fool's Day.