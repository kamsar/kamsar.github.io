---
layout: post
title: "Making Visual Studio 2010 Web.config Transformations Apply on Every Build"
date: 2010-03-25 12:26:39 -0800
comments: true
categories: .NET Build
---
<p>Disclaimer: This was written about Visual Studio 2010 RC. I suspect the same approach will continue to work with the final release but there are no guarantees.</p>

<p>When I first heard about Web.config transformations in Visual Studio 2010 I was excited. Our build process involves keeping a working build in SVN such that our dev servers may merely check out a working copy and go, and the problem of people committing their own, or the dev server's, web.configs had been a problem. Someone else's settings would come to your machine, causing explosions and other hijinks. Wouldn't it be great, I thought, to have each developer keep a delta for their own web.config settings and apply them only on their box, so not only would the commit problem be solved, we could also keep everyone's personal configs safe in SVN.</p>

<p>My problem was that I assumed the transformations would apply more or less at runtime. Their dirty secret is that they're only applied when you <span>publish</span> the site. We don't use VS publishing ever, as our release process is entirely based around SVN tags. This also kills its utility for the situation of having multiple developers with disparate web.configs being able to maintain individual transformations that apply to their computer only.</p>

<p>Feeling disappointed, I set out to remedy the situation. As I knew this was all handled by MSBuild thanks to <a href="http://live.visitmix.com/MIX10/Sessions/FT14">Hanselman's talk at MIX10</a>, I went spelunking in the global targets files. I found the definition of the TransformWebConfig target, which I learned about from <a href="http://blogs.msdn.com/webdevtools/archive/2009/05/04/web-deployment-web-config-transformation.aspx">this blog post</a>, in the C:\Program Files (x86)\MSBuild\Microsoft\VisualStudio\v10.0\Web\Microsoft.Web.Publishing.targets file. With a little fooling around, I began to understand how it works in its essential nature: the Microsoft.Web.Publishing.Tasks.TransformXml task (defined in an assembly in the same directory as the targets file).</p>

<p>Armed with the knowledge of how the task works by example and Reflector, I set out to try and make it do what I wanted: after a build, apply the correct transformation file and <span>overwrite the Web.config</span>. Since there had to be a source file other than the Web.config to transform on, I settled on "Web.generic.config" as a name for the untransformed config.</p>

<p>It turned out to be quite simple to implement. In my project's .csproj file, there lies at the bottom a commented out section with BeforeBuild and AfterBuild tasks already skeletoned out.</p>

<p>I uncommented the AfterBuild task and added the TransformXml task to it, like so:
</p>

	<target name="AfterBuild">
		<TransformXml Source="Web.generic.config"
				 Transform="$(ProjectConfigTransformFileName)"
				 Destination="Web.Config" />
	</target>

<p>Voila - after each build, publishing or no, the Web.generic.config has the active solution configuration's transformation applied to it and copied to the Web.config. Note that unlike the version run during publishing, this one only checks the root Web.config. Subdirectories' config files are ignored. Someone better than I with MSBuild could probably remedy that shortcoming :)</p>

<p>I suspect that with the right importation of tasks into your project file, this solution could be made to work against regular old App.config files as well as Web projects, but I don't do enough app development to dive particularly deeply into it.</p>