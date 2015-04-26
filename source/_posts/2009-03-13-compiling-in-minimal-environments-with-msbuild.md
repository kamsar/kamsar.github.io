---
layout: post
title: "Compiling in minimal environments with MSBuild"
date: 2009-03-13 12:39:46 -0800
comments: true
categories: Build Scripts
---
I think it's happened to almost everyone: you push web site changes to a dev server, or a staging server, and something breaks. Something that of course only occurs in that remote environment and can't be reproduced on your development instance with all its nice debugging tools.

Suppose that you track it back to an issue that will require recompiling the site to test it. Sure, you could build it locally and manually copy files every time you need to test...and realize you didn't quite fix it, but why do that when there's a better way to recompile a solution without Visual Studio installed: MSBuild.

So what is MSBuild? In short, it's the build runner that Visual Studio uses every time you build a project. It parses the solution or project file and executes the build commands embedded in the XML. It's capable of a <a href="http://msdn.microsoft.com/en-us/library/0k6kkbsd.aspx">whole lot</a>, and makes a great tool to automate nightly builds and lengthy push processes. We tend to use Subversion to accomplish build management by keeping the built DLLs under version control and making configuration changes very minimal across environments, so the build and configuration abilities aren't as awesome when building medium sized websites. It's the ability to parse and compile a solution that is of great interest.

So how do I use it to compile a solution without having a local copy of Visual Studio? Fortunately, that's the easy part. Open a command prompt and type this:

	C:\my-project-path> C:\WINDOWS\Microsoft.NET\Framework\v3.5\msbuild.exe my-solution.sln

* You may need to change the v3.5 to the framework version you're using. You may also need to change Framework to Framework64 if you want to run the 64-bit MSBuild.

Pretty simple eh? That will build the solution using the active configuration from Visual Studio.