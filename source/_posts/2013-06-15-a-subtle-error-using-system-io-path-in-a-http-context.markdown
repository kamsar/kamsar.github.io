---
layout: post
title: "A subtle error: using System.IO.Path in a HTTP context"
date: 2013-06-15 22:52:31 -0800
comments: true
categories: Sitecore
---
Here's a fun little issue that you might eventually run across. Suppose you're writing some code that needs to retrieve the extension from a URL. For example, a URL rewriter or Sitecore pipeline that acts only on certain file types.

You might think, as I would have, "oh, that's built in to .NET - we'll just use `System.IO.Path.GetExtension()`!"

And that would work, almost all of the time. The only issue comes from an internal implementation detail of `IO.Path`: it checks that the path does not contain characters that are invalid in a filesystem path. Specifically, ", <, >, |, and ASCII unprintables (< 32). Well, those characters (except perhaps the unprintables) *are* valid in a URL - so trying to get the extension of a URL path containing these characters will throw a nice fat exception:

    System.ArgumentException: Illegal characters in path.
       at System.IO.Path.CheckInvalidPathChars(String path)
       at System.IO.Path.GetExtension(String path)

Unfortunately there is no way to disable this behavior - which is logical, given the purpose of `IO.Path` as a processor for file system paths, not URLs. Not even first parsing the URL using the `System.Uri` class will fix this, as [this StackOverflow question](http://stackoverflow.com/questions/1105593/get-file-name-from-uri-string-in-c-sharp) suggests. The `LocalPath` property still includes the invalid characters that break `Path.GetFileName()` or `Path.GetExtension()`.

There are a couple of ways I could see solving this problem. The first, and simplest - though possibly prone to security issues, would be to replicate what `Path.GetExtension()` does but omitting the invalid characters check. [Reference](http://www.dotnetframework.org/default.aspx/4@0/4@0/untmp/DEVDIV_TFS/Dev10/Releases/RTMRel/ndp/clr/src/BCL/System/IO/Path@cs/1305376/Path@cs):

    int length = path.Length;
	for (int i = length; --i >= 0; )
	{
		char ch = path[i];
		if (ch == '.')
		{
			if (i != length - 1)
				return path.Substring(i, length - i);
			else
				return String.Empty;
		}
		if (ch == DirectorySeparatorChar || ch == AltDirectorySeparatorChar || ch == VolumeSeparatorChar)
			break;
	}
	return String.Empty;

The second would be to remove any invalid characters prior to calling `Path.GetExtension()`:

    string url = "\"http://mo\"nkeys/foo/bar/\"";
				
	var invalidChars = Path.GetInvalidPathChars().ToHashSet();
	for (int i = url.Length - 1; i >= 0; i--)
	{
		// technically you could use a stringbuilder, but since 99% of the time this won't ever be used, this seems optimized enough
		if (invalidChars.Contains(url[i])) url = url.Remove(i, 1);
	}

	var ext = Path.GetExtension(url);

Personally I like the second solution, since you'd benefit from any upstream fixes in `Path.GetExtension()` in future framework releases.