title: Dianoga 3.0 is released
date: 2016-10-24 16:42:30
categories: Dianoga
---

I'm happy to announce the release of Dianoga 3.0. This release brings significant improvements compared to previous releases, and is recommended for existing users.

## What's Dianoga?
Dianoga is a tool that optimizes images uploaded to the Sitecore media library to save bandwidth and improve page loading times as a result.

{% asset_img dianoga.jpg "It's also the monster that lives in the Star Wars trash compactor. What?" %}

Dianoga ensures that your site is always serving fully optimised media library images even if you are using Sitecore's dynamic resizing features (for example with [Adaptive Images](https://marketplace.sitecore.net/en/Modules/Sitecore_Adaptive_Images.aspx)). Even if you have already optimized every image uploaded into the media library, after Sitecore performs image processing the result is _not_ optimized (an unfortunate limitation of other image optimization libraries for Sitecore is that they only apply to the original image).

Dianoga is also great for situations where content editors may not be image editing experts and upload images that contain gobs of EXIF data and other nonessential metadata - these are removed automatically before being shown to visitors. All of the optimizations done are lossless (no quantizing, etc) so you lose no image quality.

## What's new in 3.0?

### SVG support
With the rise of [SVG](https://en.wikipedia.org/wiki/Scalable_Vector_Graphics) as a common format for media library media, it became apparent Dianoga needed to support optimizing SVG. Dianoga 3 includes SVG optimization!

SVG media is automatically:
* Optimized for size (the SVG is processed using [SVGO](https://github.com/svg/svgo))
* Gzipped before going in the media cache, and served using the cached gzipped version (reduces size over the wire)
* Apropos configuration to enable SVG support in media library on Sitecore < 8.1 is enabled by default (thanks to Kamruz Jaman and Anders Laub for the blogs e.g. [this](https://jammykam.wordpress.com/2015/11/18/svg-in-media-library-polluting-log-files-with-errors/) and [this](https://laubplusco.net/compress-svg-images-sitecore-media-library/))

### Optimization strategies
Dianoga 1.x used a synchronous optimization technique that resulted in a slower initial image load time, but always served optimized images. Dianoga 2.x instead relied on an asynchronous technique where the first media served would be unoptimized and subsequent hits were optimized, but the response time was never impacted.

It became apparent that both of these strategies have their place, for example you need to be synchronous when uploading to a CDN, and so Dianoga 3 supports _optimization strategies_ that let you choose when to optimize. If you don't like any of the strategies or want to optimize only when sending media to a CDN programmatically, you can also invoke the optimization pipeline directly to optimize precisely when you need to (the `MediaOptimizer` class is what you're after here).

### Pipeline-based optimization
Dianoga is now powered by Sitecore pipelines, providing simple and flexible extension options. The root `<dianogaOptimize>` pipeline runs for all optimizations, and spins off into `<dianogaOptimizeJpeg>` and other similar sub-pipelines for individual file types.

### Optimizer chaining
With Dianoga 2, you could not simply apply more than one optimizer to a file format. For example, you might wish to quantize a PNG (which is lossy), and then also optimize its encoding after quantization to further reduce file size. This is very simple with Dianoga 3 - now each optimizer is just a step in a pipeline. Add or remove optimizers as you wish.

### Media path exclusion
Got a folder of huge photos you don't want optimized because they should be kept pristine for downloads? Have another reason you want to not optimize specific parts of the media library? Great - Dianoga 3 now supports that.

Because optimization is pipeline based, this just takes the form of a processor that aborts optimization when the input is to be ignored. See `Dianoga.ExcludePaths.config.example` which ships with the NuGet package for how to use this.

### Framework version requirements
Dianoga 3 requires .NET 4.5.2 or later to be the target framework of the project it is referenced in. This provides compatibility with Sitecore 8.2. Dianoga 3 requires Sitecore 7.x or later.

## Bugs and fixes
* Asynchronous optimization has been significantly simplified and reliability improved compared to Dianoga 2. File in use issues should be completely eliminated.
* Unit tests have been added to the codebase (I blame Dan Solovay)
* Optimization tools are moved to App_Data so that IIS will never consider serving their files (as opposed to /Dianoga Tools in 2.x and earlier)
* Optimization tools have been updated:
	* libjpeg is now [mozjpeg](https://github.com/mozilla/mozjpeg), which results in better optimization for the web specifically
	* [SVGO](https://github.com/svg/svgo) has been added to optimize SVGs
	* PNG optimization remains unchanged using [PNGOptimizer](http://psydk.org/pngoptimizer), other than now being chainable with the [PNGQuant lossy quantizer](https://pngquant.org/).

## Installing
Dianoga is [available from NuGet](https://www.nuget.org/packages/Dianoga/). You must be using `packages.config` (NuGet 2.x style) package management for Dianoga, because it installs content items to the project. If you wish to use `project.json` (NuGet 3.x style), you must manually install the tools and configuration files because of limitations in NuGet 3.x.

## Upgrading

Easy. Make sure your host project is targeting .NET 4.5.2 (or later - I use it with 4.6.x), and then upgrade your NuGet package.

Have fun!