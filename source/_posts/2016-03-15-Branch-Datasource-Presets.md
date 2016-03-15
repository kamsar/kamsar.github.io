title: Branch Datasource Presets
date: 2016-03-15 14:39:23
tags: Sitecore
---

Ever stored rendering component data source items under a page? For example:

	/sitecore/content/Page
	/sitecore/content/Page/Datasources/ComponentDataSourceItem

It's a good practice and it seems to work quite well for page specific components. I use it all the time.

But have you ever wished branch templates understood that pattern in a logical fashion? What if this:

	/sitecore/templates/branches/Foo/$name
	/sitecore/templates/branches/Foo/$name/Datasources/ComponentDataSourceItem

...expanded out to have the branch create the hierarchy __and__ re-link the layout details on the instiantiated branch item to point to the right relative child data source item, instead of the child item under the branch template?

Doing this lets you use branch templates to create preset rendering hierarchies, __including__ page specific data source items.

Sound good? Well you've found the right place to [get the code to do just that](https://github.com/kamsar/BranchPresets). This requires Sitecore 8 or above with the pipeline-based item provider to operate.