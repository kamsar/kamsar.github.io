---
layout: post
title: "Sitecore 7 Update-2 and Synthesis 6"
date: 2013-09-24 22:40:32 -0800
comments: true
categories: Sitecore Synthesis
---
[Sitecore 7 Update-2](http://sdn.sitecore.net/Products/Sitecore%20V5/Sitecore%20CMS%207/Update/7_0_rev_130918.aspx) was released today, and it brought with it a surprise: Synthesis' LINQ query support broke. Well, that's unusual for a Sitecore update.

Fortunately it is easy to fix. In the `Sitecore.ContentSearch.Lucene.DefaultIndexConfiguration.config` config patch file, Update-2 changes all of the element casing from PascalCase to camelCase. So the old XPath `/configuration/sitecore/contentSearch/configuration/DefaultIndexConfiguration` turned into `/configuration/sitecore/contentSearch/configuration/defaultIndexConfiguration`. DOH! Unfortunately for us, XML - and Sitecore config patching - is case senstive. This means the config tweaks made to search in `Synthesis.config` no longer applied!

The [release notes for Update-2](http://sdn.sitecore.net/SDN5/Products/Sitecore%20V5/Sitecore%20CMS%207/ReleaseNotes/webConfig/70_130918.aspx) do not mention this casing change, so if you were manually patching you might _not_ catch this. But the stock `DefaultIndexConfiguraion.config` file available for download on that page, as well as in the zip distro of Update-2, both change a lot of case in the config.

## Fixing the problem

Fortunately it's pretty easy to fix Synthesis running against a re-cased Update-2 config file. At the bottom of your `Synthesis.config` find the <contentSearch> section and replace it with this (or just change :

     <contentSearch>
		<configuration>
			<defaultIndexConfiguration>
				<indexDocumentPropertyMapper type="Synthesis.ContentSearch.SynthesisDocumentTypeMapper, Synthesis" />

				<fields hint="raw:AddComputedIndexField">
					<field fieldName="_templatesimplemented" storageType="yes" indexType="untokenized">Synthesis.ContentSearch.ComputedFields.InheritedTemplates, Synthesis</field>
				</fields>
			</defaultIndexConfiguration>
		</configuration>
	</contentSearch>

## Dealing with versioning

Unfortunately this leaves Synthesis in a less than optimal situation where the default distribution is more or less Sitecore patch-specific. Synthesis 5, released with Sitecore 7 RTM, only works with RTM. Synthesis 6, released with Update-1, only works with Update-1 due to the moving of a Sitecore class in the update. It requires you to find this blog post to get it to work correctly with Update-2 by way of the patch above. The upcoming Synthesis 7 will have configuration by default for Update-2 - which means it will break if installed under Update-1.

Don't get me wrong, I love Sitecore 7, but the search APIs are more than a little unstable at the moment for library authors!