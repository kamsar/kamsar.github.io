---
layout: post
title: "Synthesis 8.1 Released"
date: 2015-02-20 18:39:15 -0800
comments: true
categories: 
- Synthesis
---

Synthesis 8.1 has been released to NuGet. This is a minor feature release that also resolves an issue with the model provider caching in Synthesis 8.0.

## DPI-aware image support

The new `@Html.DpiAwareImageFor()` MVC helper method emits a `srcset` along with the image that enables "retina images" served from the media library. To use it, you basically upload the image 2x (or 3x for the highest DPI devices) larger than it would normally display. Devices that support the higher resolution will automatically acquire the higher resolution version, and lower resolution devices will use the regular version.

Note that this is only for DPI-aware images. The more advanced forms of `srcset` and `picture` that enable viewport-based features ('responsive images') are not supported at this time. Last I checked not well [supported by browsers](http://caniuse.com/#feat=srcset) either.

## WebActivator removed as a dependency

Synthesis 8.0 depended on WebActivator to register its startup tasks. In 8.1, these have moved to the more Sitecorian `initialize` pipeline.

## Improved config schema

The `Synthesis.config` file has been broken into three files: `Synthesis.config` (global model configuration), `Synthesis.ControlPanel.config` (enables the control panel page), and `Synthesis.Startup.config` (enables on-startup sync checks and regeneration). This makes it much easier to deploy Synthesis to a Content Delivery server, as you can simply delete all but `Synthesis.config` and be secure.

## Upgrade Instructions

Synthesis 8.1 is a NuGet upgrade away. However to cover the new configuration changes, you do need to make a couple tweaks to an existing `Synthesis.config`.

- remove `<registerDefaultConfiguration value="true" />` (Synthesis 8.0 only)
- remove the whole `<synchronizationSettings>` node in Synthesis.config (its values now live in `Synthesis.Startup.config`)
- remove `pipelines/httpRequestBegin` in `Synthesis.config` (values now live in `Synthesis.ControlPanel.config`)
- add default configuration registration to Synthesis.config `<pipelines>`:

		<initialize>
			<!-- REGISTER DEFAULT CONFIGURATION
				If this processor is registered, the configuration values below for providers are loaded into a Default Configuration.
				This enables Synthesis to work out of the box if you do not need multiple configurations.
							
				For multiple configurations you may leave this active if you want to keep the default configuration and add your own on top of it,
				or remove it to register only your own configurations. Configurations are registered by calling ProviderResolver.RegisterConfiguration()
				- e.g. in more initialize pipeline processors.
							
				NOTE: Do not register multiple configurations that contain overlapping templates.
			-->
			<processor type="Synthesis.Pipelines.Initialize.RegisterDefaultConfiguration, Synthesis"/>
		</initialize>

- if using the `optimizeCompilations` setting to [speed up Sitecore 8 startup time](http://kamsar.net/index.php/2015/02/sitecore-8-experience-editor-performance-optimization/), turn it off temporarily after the upgrade or clear your temporary asp.net files folder, because the helper structure changed slightly during the upgrade and the cached razor files will be invalid.