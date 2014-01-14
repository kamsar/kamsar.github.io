---
layout: post
title: "Using MVC 5 with Sitecore 7.1"
date: 2013-12-26 20:52:05 -0800
comments: true
categories: 
---
If you're using something that is already on ASP.NET MVC 5, for example the latest release of Blade, and want to upgrade to Sitecore 7.1 you may be worried that as of 7.1 Sitecore takes a dependency on ASP.NET MVC 4. Thankfully that's actually pretty simple to work around in such a way that both your MVC5 code and Sitecore's MVC4 code can work together.

## Tell Sitecore to use MVC5

First you need to redirect bindings to MVC4 to MVC5. You accomplish this by adding a _binding redirect_ to your Web.config file like so:

    <runtime>
    <assemblyBinding xmlns="urn:schemas-microsoft-com:asm.v1">
      <dependentAssembly>
        <assemblyIdentity name="System.Web.Mvc" publicKeyToken="31bf3856ad364e35" />
        <bindingRedirect oldVersion="1.0.0.0-4.0.0.0" newVersion="5.0.0.0" />
      </dependentAssembly>
     </assemblyBinding>
     </runtime>

Once you've done this, the new SPEAK backend will actually be making calls to the MVC5 `System.Web.Mvc` library - which it appears to be quite happy doing.

## Comment out the extra Razor section definition

In Sitecore 7.1, there is a Web.config added to sitecore/shell/client that contains a definition of the Razor config section. If you're defining the Razor config section yourself on the Web.config - like Blade does - then this is a duplicate declaration and will throw an exception. All you need to do is comment out this duplicate definition and it will fix the issue:

    <!--   <configSections>       
		<sectionGroup name="system.web.webPages.razor" type="System.Web.WebPages.Razor.Configuration.RazorWebSectionGroup, System.Web.WebPages.Razor, Version=2.0.0.0, Culture=neutral, PublicKeyToken=31BF3856AD364E35">
			<section name="host" type="System.Web.WebPages.Razor.Configuration.HostSection, System.Web.WebPages.Razor, Version=2.0.0.0, Culture=neutral, PublicKeyToken=31BF3856AD364E35" requirePermission="false" />
			<section name="pages" type="System.Web.WebPages.Razor.Configuration.RazorPagesSection, System.Web.WebPages.Razor, Version=2.0.0.0, Culture=neutral, PublicKeyToken=31BF3856AD364E35" requirePermission="false" />	   
		</sectionGroup>	    
    </configSections> -->


That's all you need to do to get MVC5 running with Sitecore 7.1. The Web.config in the shell will automatically override whatever Razor settings you may have in your Web.config file and keep SPEAK running when it needs to.