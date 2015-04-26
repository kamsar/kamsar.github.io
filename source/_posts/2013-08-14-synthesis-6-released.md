---
layout: post
title: "Synthesis 6 released"
date: 2013-08-14 21:48:24 -0800
comments: true
categories: Synthesis
---
Synthesis 6.0.0 has been released to the NuGet gallery. Despite the lofty sounding version bump, this is just a case of [SemVer](http://semver.org/) because Sitecore 7 Update 1 moved a class (`IndexField`) between assemblies. This means binary distribution of Synthesis for Sitecore 7 RTM will not work with Update 1, and distributions for Update 1 will not work with RTM. Joy!

## Only upgrade if you're using Sitecore 7 Update 1!

New things in 6.0.0:

* Fixed [an issue](https://github.com/kamsar/Synthesis/issues/1) where unpublished fields could cause NullReferenceExceptions when accessing HasValue in weird ways
* Added the ability to [customize the interface suffix](https://github.com/kamsar/Synthesis/issues/3) on generated interfaces
* Compiled for NuGet against Sitecore 7 Update 1