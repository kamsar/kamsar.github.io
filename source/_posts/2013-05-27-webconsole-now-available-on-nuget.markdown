---
layout: post
title: "WebConsole now available on NuGet"
date: 2013-05-27 22:56:42 -0800
comments: true
categories: Sitecore WebConsole
---
My handy <a href="https://github.com/kamsar/Kamsar.WebConsole">WebConsole library</a> is now available from <a href="https://nuget.org/packages/Kamsar.WebConsole">NuGet</a> for easier installation.<br><br>
This release also fixes an issue where disposal was working in a rather silly fashion on sub-tasks.<br><br>
If you're unaware of this library, it's a very useful thing to use if you find yourself writing "hack pages," "processor pages," "rebuild pages" or other things of the like on a regular basis. Any long running or complex task you'd run from a page can easily get a realtime progress bar, console-style logging, exception handling support, and the ability to break the task up into 'subtasks' with their own tracked progress.<br><br>
In short, you can make your processing pages more informative and even vaguely pretty if you like the console aesthetic, in practically no extra time over simply writing them in the first place. I wrote this for myself, and I use it in a bunch of things :)