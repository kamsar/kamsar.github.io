---
layout: post
title: "Automatic Sitecore image optimization with Dianoga"
date: 2014-07-21 19:29:48 -0700
comments: true
categories: 
- Dianoga
---

Have you ever run [Google PageSpeed](https://developers.google.com/speed/pagespeed/) or [WebPagetest](http://www.webpagetest.org) against a Sitecore site and been disappointed with the unoptimized images coming from your Sitecore media library? When I did, I was seeing claims that I could reduce the size of many of my images by 10-30%, simply by removing unnecessary image data and optimizing the image encoding. The tools to accomplish image optimization, such as [jpegtran](http://jpegclub.org/jpegtran/) or [PNGOptimizer](http://psydk.org/pngoptimizer), operate in a lossless fashion - in other words, your images lose zero fidelity but become smaller. On a big website, any reduction in data transfer is a big cost savings on bandwidth as well as a UX improvement due to improved load times. 

With this in mind I set out to make a tool to automatically optimize the output of the Sitecore media library, and [Dianoga](https://github.com/kamsar/Dianoga) was born. Dianoga sits in the ~~trash compactor~~ `getMediaStream` pipeline, which Sitecore uses to transform media items before it writes them to its disk cache. When an image gets processed, we complete the processing by transparently optimizing it before it goes in the cache. No changes are ever made to the actual media item in the database, and you never have to explicitly request optimization.

There are some other [image](http://www.roundedcube.com/Blog/2013/building-the-png-optimizer-module) [optimization](http://mikael.com/2013/08/image-optimizer-module/) modules out there for Sitecore, so how is this one different? First, this one is automatic (no clicking a button in the content editor, or forgetting to). Second, this one optimizes as the media is served to the client, which means that if you're using Sitecore's image resizing functionality (which is perfect for modern responsive techniques like `srcset` and [Adaptive Images](https://marketplace.sitecore.net/en/Modules/Sitecore_Adaptive_Images.aspx) without tons of work) *each size of the image is individually optimized* - which is important because the act of resizing the image deoptimizes it. 

Sound like fun? Free bandwidth savings for no work other than a really difficult installation process? Just kidding - installation is as simple as a zero-configuration NuGet package that doesn't even touch your Sitecore databases. You might want to delete `App_Data/MediaCache` after install so that media is recached with optimizations, but that's all there is to it.

Find Dianoga on NuGet (Sitecore 7+), or you can get the source from [GitHub](https://github.com/kamsar/Dianoga) (source works for Sitecore 6.x). There is also a [readme on GitHub](https://github.com/kamsar/Dianoga/blob/master/README.md) with additional details about logging, troubleshooting, and such.