title: "Transparent media optimization with Dianoga 2"
date: 2015-07-04 09:20:21
categories: Dianoga
---

I've pushed a significant update to [Dianoga](http://kamsar.net/index.php/2014/07/automatic-sitecore-image-optimization-with-dianoga/), my framework to transparently optimize the size of media library images, including rescaled sizes.

My [original post on Dianoga 1.0](http://kamsar.net/index.php/2014/07/automatic-sitecore-image-optimization-with-dianoga/) explains why you'd want to use it in detail, but as an overview how does saving 5-40% overall on image bandwidth sound? All you have to do is install a [NuGet package](http://www.nuget.org/packages/Dianoga) that adds one dll and one config file to your solution. That's it. You can also find [documentation on GitHub](https://github.com/kamsar/Dianoga/blob/master/README.md).

## What's new in 2.0

Dianoga 2 is faster and more extensible. The key change is that media is now optimized asynchronously, resulting in a near-zero impact on site performance even for the first hit. Dianoga 1.0 ran its optimization in the `getMediaStream` pipeline, which meant it had to complete the optimization while the browser was waiting for the image. For a large header banner, this could be a couple seconds on slower hardware (for the first hit only). Dianoga 2 replaces the `MediaCache` and performs asynchronous optimization _after_ the image has been cached. This does mean that the first hit receives unoptimized images, but subsequent requests after the cache has been optimized get the smaller version.

Extensibility has also been improved in Dianoga 2. Previously, there were a lot of hard-coded things such as the path to the optimization tools and the optimizers themselves. Now you can add and remove optimizers in the config (including adding your own, if you wanted say a PNG quantizer), and move the tools around if desired.

There was [a bug](https://github.com/kamsar/Dianoga/issues/1) in Dianoga 1.0 that resulted in the tools DLL becoming locked, which could cause problems with deployment tools. This has been fixed in 2.0 by dynamically loading and unloading at need. Thanks to [Markus Ullmark](https://twitter.com/ullmark) for the issue report.

## Upgrade

Upgrading from Dianoga 1.0 to 2.0 is fairly simple; upgrade the NuGet package. You can choose to overwrite your Dianoga.config file (recommended), or merge it with the latest in GitHub if you don't want to.

Have fun!