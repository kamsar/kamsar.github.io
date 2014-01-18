---
layout: post
title: "Unicorn 1.0.4 Released"
date: 2013-07-17 22:50:09 -0800
comments: true
categories: Sitecore Unicorn
---
I've released [Unicorn](https://github.com/kamsar/Unicorn) 1.0.4 to NuGet. This release fixes bugs that would cause serialization inconsistencies to occur if items having children under Unicorn's control were renamed or moved (the paths stored in the moved children would still be the original path).

The solution is to re-serialize children after moving or renaming, which means Unicorn will not scale very well if you put a giant number of items under it and then move the parent of them all. But then it's not designed for that in the first place - a template with 30-40 subitems should be tolerably quick for a relatively unusual operation.

This fix has been verified using [Rhino.Fsck](https://github.com/kamsar/Rhino/tree/master/src/Rhino.Fsck), a library for verifying the consistency of a serialized tree of items.

[Rhino](https://github.com/kamsar/Rhino) solves this issue slightly more elegantly by simply reading the items from disk, changing the path, and re-writing the `SyncItem`, but it's also lower level. Eventually I'd like to deprecate Unicorn in favor of Rhino.