title: "The Benchmark Show: Sitecore Precompilation"
date: 2016-01-04 20:20:35
categories: Sitecore
---

I'm [mildly obsessed with Sitecore performance](https://kamsar.net/index.php/2015/02/sitecore-8-experience-editor-performance-optimization/), so when I found a new trick I had to see what it could do: [`aspnet_compiler.exe`](https://msdn.microsoft.com/en-us/library/ms229863.aspx). This is a really old tool that was designed to compile your .aspx pages into an assembly - and then later your cshtml as well. This improves startup time because no dynamic compilation needs to take place at runtime.

Getting `aspnet_compiler` to run against Sitecore, by which I mean executing it against an entire deployment ready web root including both your code and the `sitecore` folder, is a bit of a challenge. Because you cannot tell `aspnet_compiler` to ignore anything, and because one of the cshtml files in Sitecore 8.1 Update-1 apparently has a compiler error (for the curious, add `@using Sitecore.ListManagement.Client.Web.UI.Controls.ImportMapTo` to /sitecore/shell/client/Applications/ListManager/Controls/ContactFields/ContactFields.cshtml). You also have to comment out the `@Html.Sitecore()`s in /sitecore/shell/Templates/layout.cshtml (don't worry it's just a template used when you add a new MVC layout).

Once you get the fixes made, running `aspnet_compiler` is as simple as:

```"c:\Windows\Microsoft.NET\Framework\v4.0.30319\aspnet_compiler.exe" -v / -p "c:\path\to\deploy\ready\files" "c:\output\path"```

It will take a while - probably at least a minute - while it compiles every aspx and cshtml in the whole place into assemblies.

## Cool trick. Does it work?

Let's look at some hard data. I took my base Sitecore site (which runs [Performance.config](https://gist.github.com/kamsar/8c9efc80e72e6ada8304)) and executed my deployment scripts over it, then precompiled that to a new folder with `aspnet_compiler`. Then I devised several tests to tease out both the effects of precompilation and whether Sitecore's built in SPEAK precompilation has improved since its 8.0 days - where it could make startup take 140 seconds. The tests were in most cases run both with a cold _ASP.NET Temporary Files_ folder (where .NET puts compiled Razor caches) and with a warm one for comparison. Cold would be the first startup on a new machine, whereas warm would be after an app pool recycle thereafter.

### Test 1: Home Page

This is a simple test of loading the home page, which is a fairly simple Sitecore MVC layout with about 4 view and controller renderings and several partials. All tests begin with the app pool stopped and no `w3wp` executing.

### Test 2: SPEAK Media Library Dialog

The SPEAK media browser (what you get when browsing from an image field, for example) has been historically a pain point performance wise. For this test I loaded the dialog iframe URL into a Chrome tab and measured the time to finish page load with the Chrome developer tools.

## Results

### Home Page (cold)
Baseline (Performance.config): 22.25s
SC Precompiler: 24.8s
Precompiled: 21.5s

As expected the precompiled is marginally fastest. Sitecore 8.1's precompilation function, as we'll see throughout the tests, seems to add 2-4 seconds to startup time. Without preexisting compiled temp files, startup is extremely slow in all cases - unexpectedly so for the supposedly 'precompiled' site. My guess is that it copies the precompiled files to the temp folder, which expends I/O, but that's a guess.

### Home Page (warm)
Baseline: 9.71s
SC Precompiler: 13.97s
Precompiled: 9.51s

With the temporary files warm, startup gets much faster. Once again the Sitecore precompiler is in last place.

### SPEAK Media (cold)
Baseline: 29.6s
SC Precompiler: 33.8s
Precompiled: 22.93s

The effects of precompilation are strong when loading the very complex, partial-heavy SPEAK dialog - coming in 47% faster than the Sitecore default.

### SPEAK Media (warm)
Baseline: 12.2s
SC Precompiler: 16s
Precompiled: 12.82s
SC Precompiler (warmed): 13.23s

Performance is nearly identical once the temporary compiled files are cached the first time for SPEAK. In this case there is also a test for having warmed the Sitecore precompiler and allowed it to finish precompiling in the background, then restarted the cold app pool. 

## Analysis (TL;DR)

Overall the effects of using `aspnet_compiler` prior to deployment are relatively minor in most real world cases. Initial startup is ideally a rare case, and once their caches are heated all options perform quite similarly (as an example, the second hit to the media dialog is ~250ms to finish). For more elastic deployments, such as cloud setups, or for larger clusters it is probably worth the effort since you burn the CPU cycles once and it does significantly improve first hit to SPEAK dialogs as well as marginal overall performance improvements. Rendering heavy Sitecore MVC implementations would also likely see similar gains to SPEAK. In addition to performance, using the `aspnet_compiler` validates the C# in your Razor views - which is a nice plus in the event of a Razor syntax error which would normally be discovered at runtime.

The big downside of precompiling is that it's fairly slow (several minutes on fast hardware, probably much slower otherwise or without SSDs), and that it has to re-copy the whole web root to another output location (lots of I/O). So overall build time on CI may increase significantly.

### Another option: Use the Sitecore precompiler

If you look at the results, _startup time_ with the Sitecore precompiler is always a bit longer. But once the Sitecore precompiler completes its asynchronous run - usually within a couple minutes after startup - the results are very close to a natively precompiled site (see the warm media dialog results). This is overall a much simpler option. It does mean every instance has to compile for itself, but it does not require adaptation of the build process nor long times spent copying files.

And here's the best part: _you can use it for your own views, too_. It's just a Razor precompiler pipeline processor, and you can add one - or more - for your sites' MVC views. For example:

	<processor type="Sitecore.Pipelines.Initialize.PrecompileSpeakViews, Sitecore.Speak.Client">
    	<Paths>/Areas/MySite/Views</Paths> <!-- virtual path to your views -->
    </processor>

This will cause your views to all get precompiled asynchronously on startup - if they are not already - which writes their assemblies into the _Temporary ASP.NET Files_ folder. So you trade a couple seconds of startup time, and an initial spike of CPU cycles, for consistent performance thereafter. However, you lose the protection against syntax errors provided by build-time precompilation.

Next time on the benchmark show: Common Sitecore development tasks, benchmarked on a variety of hardware.
