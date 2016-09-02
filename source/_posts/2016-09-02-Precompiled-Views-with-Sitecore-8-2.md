title: Precompiled Views with Sitecore 8.2
date: 2016-09-02 12:20:43
categories: Sitecore
---

A while ago I wrote a post about [speeding up your views with precompilation](/index.php/2016/01/Benchmark-Show-Precompilation/).

After writing that post I learned about [RazorGenerator](https://github.com/RazorGenerator/RazorGenerator) which [Chris van de Steeg blogged about](http://www.chrisvandesteeg.nl/2010/11/22/embedding-pre-compiled-razor-views-in-your-dll/). RazorGenerator is better in just about every way compared to `aspnet_compiler.exe`. Whereas the `aspnet_compiler` just warms the compiler cache for the current machine, RazorGenerator builds the generated classes for your Razor files _directly into your assemblies_.

With RazorGenerator you can compile once and deploy everywhere. Getting started with it is almost stupidly simple. Seriously, I described the whole process [in one tweet](https://twitter.com/kamsar/status/684823059082027008): Install the `RazorGenerator.MsBuild` NuGet package. That's it. So what does that get you?

* Compile-time view syntax checking. Every deployed a Razor file that broke at runtime after a "successful" build? Well now your builds will _fail_ if your Razor syntax is invalid. Hoorah!
* Your Razor views are compiled into your output assembly (look at it in DotPeek and see the `ASP` namespace)
* It's fast. You probably won't even notice the build time difference.

## But that's not quite all

Right, so how do you get your precompiled views to actually get loaded? ASP.NET MVC doesn't understand how to resolve _classes_ instead of _files_ out of the box. With Sitecore 8.1 and earlier, you could use the `RazorGenerator.Mvc` NuGet package and [wire up its `PrecompiledViewEngine`](https://github.com/RazorGenerator/RazorGenerator/wiki/Using-RazorGenerator.Mvc) to accomplish this.

Sitecore 8.2 modernizes a lot of the MVC implementation under the covers, and part of that is that the Sitecore infrastructure now supports precompiled views natively! (Wondering why the experience editor starts up a lot faster in 8.2? That's why.) It's really easy to use, in fact: all you have to do is tell Sitecore which assemblies to look for precompiled views in and it does the rest. Behold:

	<configuration xmlns:patch="http://www.sitecore.net/xmlconfig/">
	  <sitecore>
	    <mvc>
	      <precompilation>
	        <assemblies>
	          <assemblyIdentity name="My.Project" />
	          <assemblyIdentity name="My.OtherProject" />
	        </assemblies>
	      </precompilation>
	    </mvc>
	  </sitecore>
	</configuration>

**CAVEAT**: The Sitecore precompiler is greedy by default. When it is on, if a class exists for your view, _the physical file for the view is not checked_. You can even delete it. This means that you cannot tweak the cshtml file and have updates be reflected. You can enable timestamp checking if you must by changing this setting:

	<configuration xmlns:patch="http://www.sitecore.net/xmlconfig/">
	  <sitecore>
	    <settings>
		  <!--  MVC: Flag for controlling whether Razor View Engine will look at physical view last modified dates 
            when determining to use a view file from a pre-compiled assembly or from the file system.
            Default: "false"
      	  -->
      	  <setting name="Mvc.UsePhysicalViewsIfNewer" value="false" />
      	</settings>
      </sitecore>
    </configuration>

Because there are (minor) performance implications of doing the timestamp checks, I would advise _enabling_ this setting for development and _disabling_ the setting for production. You should never, ever be changing files on production after a deployment anyway.

## Recap

If you're on Sitecore 8.2, you can get view performance gains by:
* Installing `RazorGenerator.MsBuild` on each Visual Studio project that contains Razor views
* Registering each of those assemblies with the Sitecore precompiled view engine so that it uses your precompiled classes for view rendering


Big thanks to [Kern](https://twitter.com/herskinduk) for setting up the MVC expert panel, without which this feature might well not exist. And we might still have [1:40 startup times](http://kamsar.net/index.php/2015/02/sitecore-8-experience-editor-performance-optimization/) after a compile. :)