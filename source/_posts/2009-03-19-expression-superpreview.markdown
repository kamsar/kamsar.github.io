---
layout: post
title: "Expression SuperPreview"
date: 2009-03-19 12:29:31 -0800
comments: true
categories: 
---
<p>Today the release of SuperPreview was announced at the MIX09 keynote. What's that? It's merely a tool that combines the ability to see how a page is rendered in many different browsers (including IE 6, 7, and 8 on ONE MACHINE) in one easy to use tool. It's a kind of hybrid browser screenshot tool (ala <a href="http://browsershots.org/">BrowserShots</a>) but it also captures the outlines of elements similar to the Firebug inspect tool, and provides the ability to compare side by side two browsers or a browser and a comp file. Alternatively you can also overlay one on the other translucently to compare pixel-perfect lineups.</p>

<p>If you've done any kind of HTML/CSS work that required cross-browser compatibility (and if not, now's a great time to learn), you'll realize that this is pretty huge. It was also demonstrated showing a screenshot of Safari Mac using a web service Microsoft will provide to users.</p>

<p>There are a couple blog posts at <a href="http://www.istartedsomething.com/20090318/expression-web-superpreview-cross-browser-testing/">IStartedSomething</a> and <a href="http://www.webdesignerdepot.com/2009/03/microsoft-announces-superpreview-for-ie-browser-testing/">Web Designer Depot</a> that take a pretty deep look with a lot of screenshots into the abilities of SuperPreview.</p>

<p>There are some questions I had about the functionality of SuperPreview and I managed to track down one of the team that wrote it to pepper him with questions.</p>

<ul><li>SuperPreview requires IE8 to be installed currently. There are plans to not make it a requirement (via a web service to render it similarly to the safari mac service). I've heard rumors that IE8 may go RTM at the keynote tomorrow...well today now.</li>
<li>"IE7" mode is the same as IE8 in compat mode, which is 99% the same as 7 according to the devs but differences have been found. Be aware that a real copy of IE7 might be a good thing to keep around, though if you do find a difference please tell the <a href="http://blogs.msdn.com/xweb/default.aspx">Expression team</a>, as they want to hear about it.
</li><li>IE6 mode is supposedly 100% IE6</li>
<li>It's not an interactive preview (it takes screen caps, so its not an alive page)</li>
<li>It waits 500ms for onload js to run before taking the screen cap, so onload scripts that fix display will run</li>
<li>The current preview release only supports IE versions. Later versions will support Firefox, Safari, etc</li>
</ul><p><a href="http://download.microsoft.com/download/5/6/8/568F0D28-0434-4794-B7FC-FB293BCC98FB/SuperPreview_Trial_en.exe">SuperPreview can be downloaded from here</a></p>

<p>UPDATE: Found some great posts on the tech at the Expression team blog. Check <a href="http://blogs.msdn.com/xweb/archive/2009/03/18/superpreview-technology.aspx">this</a> and <a href="http://blogs.msdn.com/xweb/archive/2009/03/18/superpreview-for-internet-explorer.aspx">this</a> for more technical information.</p>