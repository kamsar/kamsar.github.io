---
layout: post
title: "Editing hidden template fields in the page editor"
date: 2013-08-14 22:44:01 -0800
comments: true
categories: Sitecore
---
Recently on a project I came across a need to allow editors who were using the Sitecore Page Editor to modify some fields that had no logical place on the actual editor screen. In this case, the fields were relating to page SEO - meta keywords and description among other things. Since these are in the head of the page, there's no good place to stick an edit frame or custom experience button - they aren't even part of a rendering placed on the page.

While poking around the WebEdit customization options, I hit on the idea of adding a WebEdit ribbon button that would act as a custom experience button would, and load a subset of the item fields in a content editor window. It turns out this was actually relatively easy. 

In the Core database, the default WebEdit ribbon is defined under `/sitecore/content/Applications/WebEdit/Ribbons/WebEdit/Page Editor`. Under this item are items for each 'chunk' in the ribbon, and under that each command button, like this:

<img src="/images/20130814tree.png" alt="tree" width="185" height="253" class="alignnone size-full wp-image-751" />

To implement your own command, you just need to add a Small Button or Large Button item underneath a Chunk - new or existing - to place your button on the ribbon. The "SEO" chunk in the above screenshot is an example of what you want to end up with. 

The key piece of all of this is how to have your button actually load the field editor, and telling it what fields to load. You do this by configuring the Click field on the button to run the `webedit:fieldeditor` command like so:

    webedit:fieldeditor(fields=Browser Title|Description|Keywords, command={007A0E9E-59AA-48BB-84F2-6D25A8D2EF80})

The fields argument is pretty easy to understand - a pipe delimited list of fields to edit, just like a WebEdit Custom Experience Button has (this command is in fact the one used by custom experience buttons). The command parameter I am not sure what it is used for. Internally it seems to need to be an item in the core db that exists - I used the GUID of my button item, and that worked fine.

And here's what you end up with when you're done, for a very small amount of work:

<img src="/images/20130814result.png" alt="result" />

This worked for me on Sitecore 7 (RTM), but I suspect the technique would work great on earlier versions of Sitecore as well.