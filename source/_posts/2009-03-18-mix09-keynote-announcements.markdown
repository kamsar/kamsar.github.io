---
layout: post
title: "MIX09: Keynote Announcements"
date: 2009-03-18 10:36:42 -0800
comments: true
categories: .NET
---
<p>Bill Buxton of MS Research led off, looking quite like the professor in Back to the Future. The key to UX, he says, is to balance budget with multiples - multiple ways of accomplishing the same task, that can be chosen between. "The transitions are just as important as the states," he says. Better transition documentation would end up with a better UX implementation of said process in the final product. "Ideation," basically brainstorming, actually lowers usability unless followed by a stage of reflection/iteration on the ideas created.</p>

<p>Expression web 3 is coming. SuperPreview feature can load previews using different browser engines, including Firefox and Safari from within the Expression interface, as well as a side-by-side view with the original comp or overlaying the comp as a transparency. It can also get previews for things like Safari Mac by using a MS cloud service to provide the rendering. It can also firebug-style outline elements while n side-by-side view to compare browsers (ftw!). Free version of SuperPreview only available now, which enables side by side display of multiple versions of IE with no VMs, etc. More on SuperPreview later, after I condense my notes from discussing it with one of the developers on the project.</p>

<p>ASP.NET MVC 1.0 RTM is released.</p>

<p>Some features of Visual Studio 2010 were discussed but I will discuss those in my post about the VS session I was at later in the day.</p>

<p>IIS 7's FTP module getting an update including SFTP support. Much needed, IIS FTP has been pretty terrible for ages now.</p>

<p>Web Platform Installer 2.0 coming out, available from <a href="http://microsoft.com/web">http://microsoft.com/web</a>, which extends on the 1.0 featureset of being able to configure IIS and install Microsoft development tools (SQL/VS express, IIS 7 extensions like URL Rewrite) by adding an "app store for your web server" as ScottGu put it. I think that sounds like a great idea. The Web PI app gallery has an open interface and many common products are already on it - even not traditionally Microsoft apps like PHP and Drupal, as well as open source .NET apps like DotNetNuke and Umbraco.</p>

<p>Silverlight 3 beta is released today. Many many new features to compete with Flash and AIR.</p>

<ul><li>New Codecs: supports H.264/AAC/MP4</li>
<li>GPU acceleration including pixel shaders and 3d perspective transforms on images and video</li>
<li>improved media analytics</li> 
<li>deep linking support</li>
<li>better text quality (ClearType support, including on the Mac)
</li><li>multi-touch support</li>
<li>Silverlight library caching support (i.e. download external code libraries once)</li>
<li>Can run outside a browser similar to AIR, with a solid security sandbox limiting the need for "are you sure xxx?" dialogs</li>
<li>Download size 40k less than Silverlight 2 thanks to code optimizations</li>
<li>"Smooth" streaming using adaptive bitrates depending on network conditions enables several interesting behaviors including lowering the bitrate temporarily to avoid rebuffering a stream, and instant seek in streams by restarting the stream instantly with a low bitrate and kicking it back up as the buffer refills.</li>
<li>DVR-style stop and rewind abilities for live streams</li>
</ul><p>That's all folks. Some exciting stuff coming down the pipe here, with more details in some entries I need to write from my sessions today.</p>