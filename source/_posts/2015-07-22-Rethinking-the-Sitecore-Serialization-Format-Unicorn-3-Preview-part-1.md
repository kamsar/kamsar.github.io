title: "Rethinking the Sitecore Serialization Format: Rainbow Preview, part 1"
date: 2015-07-22 18:23:27
categories: Unicorn
---

If you've worked with Sitecore in a team setting for any length of time, you've probably had to deal with item serialization. Item serialization is nearly a requirement to be effective when you need to share templates, renderings, placeholder settings, custom experience buttons, and all the other Sitecore items stored in the databases that are effectively _development artifacts_, as opposed to content. Being development artifacts, we want to keep them under source control so we can version them, develop feature branches with them, and deploy them using continuous integration to our shared Sitecore installations.

If you've dealt with Sitecore's serialization format in a team environment for any length of time, you've probably started to realize some of its shortcomings. Because,

{% asset_img onedoesnotserialize.jpg "One does not simply serialize without merge conflicts" %}

Let's pick on multilist fields, for example. This is what a multilist looks like in SSF:

	----field----
	field: {E391B526-D0C5-439D-803E-17512EAE6222}
	name: Allowed Controls
	key: allowed controls
	content-length: 194

	{E11BDB3B-1436-4059-90F6-DE2EE52A4EB4}|{D9C54253-37FF-4D64-8894-5373D8799361}|{F118E540-CC75-4AA9-A62B-D6ED9E6F77E4}|{A813194F-32F4-4501-A430-6602ABF73535}|{2F4ADF0B-9633-4EE9-B339-8CA32E2C3293}

Now let's imagine using this in a team environment. Alice creates a new rendering, and needs to add it to placeholder settings so it can be used in Experience Editor. Meanwhile on another branch, Bob adds a different rendering he's made to the same placeholder settings. What happens? _Merge conflict_. And not a simple, easy to solve conflict: one where a very long single line has to be merged by hand - because merging is line oriented. On top of that, you must not forget to _recalculate the content-length_. Oh joy.

Then let's take a look at the data that's stored. Do we need `key` to load the field? Do we need `name` even? Nope, though having a name around makes it easier to understand - we just don't need two. Then how about that the format is endline specific - don't leave home in Git without the special `.gitattributes` to leave those alone, or the files won't be read by the Sitecore APIs.

Here's another gremlin about serialization: it saves fields that aren't important to _development artifacts_ that are version controlled. Yes, I'm talking about you `__Revision`, `__Updated by` and `__Updated`. Certain Sitecore tools - like say the template editor - cause a ton of item saves and they are not picky about if any actual data fields have changed. This means that if I add a `Foo` field to my `Bar` template, the last updated on `Bar`, `Foo`, and every one of `Bar`'s fields gets changed. Even if there is an actual data change, and the data change is auto-mergeable, you can still get conflicts on the statistical fields. _Welcome to annoying merge conflict city, folks!_

{% asset_img whatif.jpg "What if I told you we could solve this" %}

## Part I: The JSON era

I've been at this a while. Version 1 is largely on the junk pile. Why? JSON.

JSON seemed like an obvious candidate format: it's quick to parse, mature, and easy as pie to implement with JSON.NET and a few POCOs. In fact, its performance is quite similar to ye olde `content-length` above. It was certainly a step up, and I was [not the only person](https://github.com/hermanussen/sitecore.customserialization) to think of this idea. I learned a lot and got many ideas from Robin's prototype, not the least of which was that we should _reformat field values to make merging easier_.

Oddly enough it was the idea of field reformatting that made JSON an untenable format. The problem is that merging is line oriented - so we would want to reformat that multilist from the first example with one GUID per line such that merge tools could make sense of it. Well JSON does not allow literal newlines in data values, instead it uses the string literal `\r` and/or `\n`. Unhelpful - but it makes parsing really fast.

## Part II: YAML Ain't Markup Language

With JSON in the bin, I started poking around for formats that had already been created which would support our needs. YAML (the acronym is the title of this section) fit the bill nicely. The design goals of [YAML](http://yaml.org/spec/1.2/2009-07-21/spec.html) are to be a more human readable superset of JSON for things like configuration files and object persistence. Allows multiple lines, human readable, allows lists and nesting - nice.

Well the downside is that because of the flexibility of the format, YAML parsers are on the order of 10-100x slower than JSON parsers. It was slooooow. But the good news is that the YAML-based serialization format I had designed was much, much simpler than the entire YAML specification. So I wrote my own reader and writer that supported only the subset of YAML that was necessary. It was fast. The format was easy to read and understand. It had the ability to add _field formatters_ to make values mergeable (at present, for multilists and layout fields). I wanted to start using it pretty badly in real projects :)

So without further ado, here is the same item that we took the multilist from above, but in YAML:

	---
	ID: 38ddd69e-fb0a-4970-926e-dfb0e5b9a5e1
	Parent: 68e4c671-797d-4a89-8fa6-775926f1381d
	Template: 5c547d4e-7111-4995-95b0-6b561751bf2e
	Path: /sitecore/layout/Placeholder Settings/reductio/ad/absurdum
	SharedFields:
	- ID: 7256bdab-1fd2-49dd-b205-cb4873d2917c
	  # Placeholder Key
	  Value: heading
	- ID: e391b526-d0c5-439d-803e-17512eae6222
	  # Allowed Controls
	  Type: TreelistEx
	  Value: |
	    {E11BDB3B-1436-4059-90F6-DE2EE52A4EB4}
	    {D9C54253-37FF-4D64-8894-5373D8799361}
	    {F118E540-CC75-4AA9-A62B-D6ED9E6F77E4}
	    {A813194F-32F4-4501-A430-6602ABF73535}
	    {2F4ADF0B-9633-4EE9-B339-8CA32E2C3293}
	Languages:
	- Language: en
	  Versions:
	  - Version: 1
	    Fields:
	    - ID: 25bed78c-4957-4165-998a-ca1b52f67497
	      # __Created
	      Value: 20100310T143300
	    - ID: 52807595-0f8f-4b20-8d2a-cb71d28c6103
	      # __Owner
	      Value: sitecore\admin
	    - ID: 5dd74568-4d4b-44c1-b513-0af5f4cda34f
	      # __Created by
	      Value: sitecore\admin
	    - ID: 87871ff5-1965-46d6-884f-01d6a0b9c4c1
	      # Description
	      Value: <p>The heading of a page, above any content renderings.</p>

Notice how the fields' names - nonessential data - are in YAML comments. Present for humans to read, not necessary for the machine. The `Allowed Controls` TreelistEx field is also an example of the YAML multiline format, starting with the pipe and the data on a newline, indented further. YAML uses significant whitespace to define structure, making it easy to read and also not requiring hacks like `content-length` to efficiently parse.

You may notice that only the `Allowed Controls` field has a field type value. This is because the value was formatted with a `FieldFormatter`, so when deserializing the value it uses the type to figure out which formatter to "unformat" the value back into Sitecore with.

The way that languages are structured is also slightly different here: each language has a top level list item, but versions are all grouped hierarchically under their language. So in this format, language is a construct aside from "da-DK #1" like the Sitecore database uses.

We've also used field level ignores to not even store the constantly changing statistics fields (e.g. `__Updated`). This is optional, if you choose to do so, but it does lead to wonderfully compact files on disk.

The YAML format is also completely endline ignorant - it doesn't care if it gets `\n` or `\r\n`.

## Yes, please?

Well sorry - it's not quite done yet, though the YAML part is pretty stable. The YAML format is a component of my upcoming [Rainbow](https://github.com/kamsar/Rainbow) library. Rainbow is essentially a modernized serialization API that aggressively improves both the serialization format _and_ the storage hierarchy, plus providing deep item comparison capabilities.

Rainbow aims to only be an API and has no default configuration or frontend. It will be freely available to use.

I have several projects that I intend to use Rainbow for - maybe you can think of some uses too?
* Unicorn 3.0, being developed alongside it, will use Rainbow for storage, comparison, and formatting needs.
* Rainbow for SPE will enable serializing and deserializing items in YAML using Sitecore Powershell Extensions

At present, Rainbow is alpha-quality. I wouldn't use it unless you want to get your hands dirty, and I might change APIs as needed.

In the next post of this series, I'll go over thinking behind improvements to the standard storage hierarchy to enable infinite depth while still being human readable. But there are still some bugs to fix before I write that ;)