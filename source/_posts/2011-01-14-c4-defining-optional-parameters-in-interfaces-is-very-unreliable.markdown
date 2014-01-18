---
layout: post
title: "C#4: Defining optional parameters in interfaces is very unreliable"
date: 2011-01-14 12:18:19 -0800
comments: true
categories: .NET
---
See <a href="http://dotnetpad.net/ViewPaste/Tt8j0gjsY0Gwf8PYwBw4CA">this example</a><br>
In short, when the object is treated as the interface the default value from the interface applies. If it's treated as the implementation, the implementation's default applies. But the implementation isn't required to have a default for it at all - in which case the call will fail, unless it's treated as the interface.<br>
It's all very unreliable, and while I'm sure it works this way for good technical reasons under the hood, realistically it doesn't present a very coherent interface.