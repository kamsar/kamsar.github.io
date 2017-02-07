title: 'Reinventing the Serialization File System: Rainbow Preview, Part 2'
date: 2015-08-05 18:16:55
categories: Unicorn
---

This is the second post in a series about [Rainbow](https://github.com/kamsar/Rainbow), an advanced serialization library for Sitecore. Part 1, dealing with improving the serialization file format, [can be found here](https://kamsar.net/index.php/2015/07/Rethinking-the-Sitecore-Serialization-Format-Unicorn-3-Preview-part-1/).

# Introducing Serialization File System

Rainbow supports the idea of a _data store_, which is an abstraction of the necessary components to store and retrieve Sitecore items. A data store need not be serialized: Sitecore's database is itself implemented as a data store as far as Rainbow is concerned.

This time we'll be talking about the Serialization File System data store. Serialization File System, or SFS for short, is a pattern for organizing files on disk to represent a Sitecore item tree. It's only a pattern: it depends on a _serialization formatter_, such as [the YAML formatter](https://kamsar.net/index.php/2015/07/Rethinking-the-Sitecore-Serialization-Format-Unicorn-3-Preview-part-1/), to do the actual serialization and deserialization. This means that you can use whatever format you want, without having to reimplement the whole organizational structure. Or if you can't stand SFS you can make your own data store and keep the YAML format :)

# Why do we need SFS?

To understand why we need SFS, let's get down to data structures for a minute here. Windows' file system is essentially a [B-tree](https://en.wikipedia.org/wiki/B-tree). Sitecore's content tree is also essentially a B-tree. So we can map the two together pretty easily, right? **WRONG.**

### Path Length

Sitecore's content tree is effectively infinite (_by definitions of "infinite" that mean "up to 20 levels by default"_) in depth. The Win32 filesystem APIs on the other hand, have a maximum path length of 240 characters. So now we run into a problem where you can have a path in Sitecore that is unrepresentable on the filesystem, because the content path is too long. Whoops. Sitecore's serialization APIs handle this situation, albeit a bit clunkily. They take the item path and apply a trivial hash algorithm to it, then put the item at the root of the serialization tree in a folder named that hash. For example (slightly simplified):

* given `/sitecore/foo/bar/baz/quux/quince` (imagine this is longer)
* `baz` might be written to `c:\serialization\master\sitecore\foo\bar\baz.item`
* but `quux` might go to `c:\serialization\A8CF73\quux.item`
* and `quince` might go to `c:\serialization\C732F1\quince.item` - not even a child of quux

This solution works, but it also means that the hierarchy becomes nearly unintelligible once short-paths start being used. The hash is not a standard algorithm; there is no obvious way to see what it means without reading the .item file within to see what its path is. It also makes merging very unintuitive if the short-pathed items are changed because the file path tells you nearly nothing.

So now we've seen path length problems, but what about a special case of that? What happens if you create an item name that is so long that _it alone becomes too long to fit in the Windows filesystem path limits_? Imagine an item with a 300-character name. Well if this occurs - as can pretty easily with Web Forms for Marketers - the Sitecore serialization APIs all choke. Unicorn 2 fails because it uses the Sitecore APIs. Pretty sure TDS does too, but I could be wrong.

### Filenames are not unique

Sitecore's content tree, and the APIs to retrieve items "by path," are quite misleading. Why? On the Windows filesystem the file name is a unique key. However under the covers of Sitecore the item's ID is the unique key. Duplicate names are totally allowed - which means that getting something "by path" is horribly ambiguous (.5% of the time, but still). But this is the basis upon which the Sitecore standard serialization hierarchy is built: ambiguity. 

**But they made the right choice.** Do you want to merge conflicts in items named by ID on disk? How about deal with that you'd hit the path length limit in about 3 levels with 35+ character file names? Thought not. But this means we have a problem to solve: how do we map _non-unique nodes_ in the database (the item name) onto _unique nodes_ (file names) on the file system.

Sitecore's serialization APIs solve this in a somewhat decent fashion. When an item is serialized the parent paths are evaluated for items of the same name, and if one exists then the item path has the parent ID appended to it. For example:

* given two items with path `/sitecore/foo`
* when you write them to disk they would get a path like `c:\serialization\master\sitecore\foo_d0ec0aa931eb46ecb241d1ca18b4c5b2.item` where each item has its ID postfixed to the filename

Seems legit, right? **Unfortunately it's very broken and can actually corrupt your serialization tree.** Don't believe me? Try this:

* given an item with path `/sitecore/foo`, serialize it
* now we have `c:\serialization\master\sitecore\foo.item`
* next we create another `foo` item with a different ID
* now we'll have `foo.item`, and `foo_d0ec0aa931eb46ecb241d1ca18b4c5b2.item` in the same folder
* so far, so good. but now serialize the original `foo` item again.
* now we have `foo.item` with old data, `foo_d0ec0aa931eb46ecb241d1ca18b4c5b2.item`, AND `foo_2195e766591d4baf8ac63a1efa43526d.item`
* yep. two items on disk for the original foo.item, and a corrupted tree. Bad.

### Pathing bugs in the API

The Sitecore serialization pathing APIs are unfortunately pretty buggy. There are several methods that assume you'd never serialize an item anywhere outside the default serialization folder - and in fact throw an error if you try it. These methods are all static, and thus the only way to change their behavior is by decompiling them and using your own fixed copy of them. That's not in the least suboptimal, and it's quite intentional that Rainbow has zero dependency on the Sitecore serialization APIs.

Tired of hearing me rant about bugs? Me too, how about we talk about solving these problems instead!

# How does SFS work

The SFS data store is capable of storing practically infinite item path depths, as long of an item name as you please, handling duplicate filenames in all cases, and writing to any path you please. SFS is based on the idea of a 'solid' tree, where every node with children must also contain the serialized parent, for example:

* given `/sitecore` as the root
* you could serialize `/sitecore/foo`
* but you could not serialize `/sitecore/foo/bar` _without also serializing_ `foo`

Sitecore serialization does allow for 'sparse' trees where you can have unserialized parents. The astute may be reading this and asking "wtf, do we have to serialize the whole database then?" 

{% asset_img silly.jpg "Stop that, it's silly. Too bad this isn't a python blog." %}

SFS supports _relative trees_ instead of a single monolithic tree that represents an entire database like Sitecore uses. For example:

* given `/sitecore/templates/User Defined` as the root of a tree
* User Defined might be serialized as `c:\rainbows\User Defined\User Defined.yml`

See how Rainbow ignores the relative Sitecore path? This means for deep tree roots your filesystem path length can be much shorter because it doesn't need empty parents, resulting in fewer over-length file paths. It also means that you can browse to the items with fewer clicks in Windows Explorer. Unicorn 3 also allows you to name your trees (which in Unicorn terms map to an `<include>` entry on your predicate), so your serialization folder might resemble:

     c:\rainbow
     	Templates
     		User Defined.yml
     	Funny Giphys
     		Animations.yml
     	Old School Content\
     		Home.yml
     		Home
     			About us.yml

Note how the tree root folder name need not match the root item name (though it does by default, but it's pretty easy to have duplicate names doing that). In the above case the `c:\rainbow\templates` item might be rooted in Sitecore at `/sitecore/templates/User Defined`. _Note: the use of solid trees does preclude some kinds of exclusions, namely anything not path-based because other exclusions could result in a sparse tree._

### Long Path Handling

SFS handles long content paths by using _loopback paths_. These are similar in concept to Sitecore's hashed paths, but unlike hash-paths they actually _transplant_ any children of that path under the loopback path. The loopback paths are also named by the item ID of the parent of the items in the loopback. Let's look at an example (with a contrived very short max path length):

* Given `c:\rainbow\root\some\rather\long\path\length\thing\parent.yml` (ID: 2195e766-591d-4baf-8ac6-3a1efa43526d)
* Suppose that adding 3 characters to the end of "parent" makes the child path over-max-length. So if we add `loopchild` under parent,
* `c:\rainbow\root\2195e766591d4baf8ac63a1efa43526d\loopchild.yml` becomes the child's path, based on the parent item's ID
* If we add `quux` under `loopchild`, it goes to `c:\rainbow\root\2195e766591d4baf8ac63a1efa43526d\loopchild\quux.yml` - under the same loopback, adding a level of human readability hash-paths lack

Loopback paths may loop multiple times for extremely long path lengths. Loopbacks also handle the case where some children are short enough to live under the parent and others' name puts it over the limit into a loopback.

### Duplicate File Name Handling

If you have items of the same name under the same parent in Sitecore, SFS uses a similar approach to Sitecore's APIs with `item_id.yml` as the filename format. However SFS does a lot of _correctness verification_ that Sitecore does not, because SFS generates the paths based on the filesystem and not Sitecore. For example, suppose you had two items of the same name and then _added different children to each_. SFS will actually resolve the path by evaluating down the filesystem paths until it finds the matching path regardless of parentage - and it returns _all_ matches instead of whichever one it feels like. The case of writing items at different times is also handled; SFS checks existing same named items for any with the same ID and reuses the name. So you never get renames for no reason or corrupted trees.

This may sound like a lot of file reading. Yes, it can cause more file reads than Sitecore's approach. However with a smart path to ID cache, and the ability to read _item metadata_ (which in the case of YAML items means only reading the first 4 lines of the file, which is 3x or more faster than reading the whole thing), both dumping and syncing items from SFS is up to 50% faster than Unicorn 2 on the same items.

### Long Item Name Handling

Obscenely long item names are handled by simply _truncating them_ before putting them on the filesystem. A setting controls the maximum name length. Parent items are properly disambiguated using ID suffixes if two differently named items truncate to the same short name. I serialized a whole Sitecore database with max filename length = 5 to test this. So many duplicate names then!

### Reliability

SFS (and the YAML formatting pieces we talked about last time) has 95% code coverage, and all bugs that have been found so far are covered with additional tests.

### Hey, what about an index file to speed up querying?

In fact Rainbow was originally designed to be a general purpose data store, where you could directly query an item by ID, path, template ID, or parent ID. It had in memory indexes that it would maintain, backed variously by a single global index file, and reading the headers of each serialized file. The indexes worked pretty well, in fact. But they had several major problems that caused me to scrap them:

* There's no good way to maintain an index cache, because you may not be the only writer to the index file (e.g. you `git pull` someone else's changes). `FileSystemWatcher` is not 100% reliable, and that would be a necessity to avoid data corruption, which is a big deal when you're talking about serialization.
* A centralized index file precludes the possibility of easily copying items between trees because the index entries would have to go with it
* A filesystem logically organized for a computer and index, where items are stored by ID-based filenames, is nearly unintelligible to a human and merging becomes hairy, and commit errors due to not being able to see the item path easily would be possible

# Can I have it yet?

Why yes, yes you can. But...

{% asset_img lumbergh.jpg "Back up in your *** with the resurrection" %}

...so if you could just go ahead and download the beta, that'd be great.

Rainbow and Unicorn 3 betas are currently available from NuGet (you'll need to enable prerelease packages). For a fresh install it should be as simple as installing the Unicorn package - unless you want to hack around the Rainbow APIs, in which case Unicorn is not required. For upgrading from Unicorn 2, [there's a doc for that](https://github.com/kamsar/Unicorn/wiki/Upgrading-to-Unicorn-3).

It's vaguely stable, but hasn't had extended testing in real life Sitecore development like a final release will have. It no doubt has some bugs. The bugs may be more than minor. Feel free to test it if that doesn't scare you; [send me a ticket on GitHub](https://github.com/kamsar/Rainbow/issues/new) if you find issues. Now's a great time for feature requests too!