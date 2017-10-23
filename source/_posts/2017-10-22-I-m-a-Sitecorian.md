title: I'm a Sitecorian!
date: 2017-10-22 22:01:58
categories: Sitecore
---

I am excited to announce that I am joining the Sitecore product team as a Platform Architect! 

![wth](https://media.giphy.com/media/1jnyRP4DorCh2/giphy.gif)

Now normally this wouldn't merit a whole blog post, and we'd just let the recruiters find out about it on LinkedIn. But I'm sure many folks' next question would be around all the libraries that I maintain and what will happen to them. So let's address the elephant in the room:

![not a thing](https://media.giphy.com/media/y63H09ZvHJdf2/giphy.gif)

## [Unicorn](https://github.com/kamsar/Unicorn), [Rainbow](https://github.com/kamsar/Rainbow), and [Dianoga](https://github.com/kamsar/Dianoga)

These will continue exactly as they are today as independent, community driven projects. I will still be the maintainer. The license will remain MIT.

This also includes the dependency libraries that these projects use (e.g. [Configy](https://github.com/kamsar/Configy), [WebConsole](https://github.com/kamsar/Kamsar.WebConsole), [MicroCHAP](https://github.com/kamsar/MicroCHAP)).

## [Synthesis](https://github.com/kamsar/Synthesis) & [Leprechaun](https://github.com/kamsar/Leprechaun)

Ok hold up: let's first define what Leprechaun _is_ because I haven't publicly spoken about it yet. It's a stable command-line code generator that works from Rainbow serialized items. Kinda like the T4 templates that a lot of people use except that it's better because:

* Uses Roslyn and C# Scripting, so it can run outside Visual Studio (e.g. on a CI server)
* Ridiculously faster than T4
* Has a watch mode that provides instant regeneration when saving templates in Sitecore
* Uses the same configuration system as Unicorn does, so it's familiar and simple to configure

Leprechaun is currently working in production on a couple sites, but does not have complete documentation so it may require a bit more spelunking to use. Currently it supports Synthesis out of the box, but it's easy to add or change code generation templates.

Ok back to what's happening to these projects. For the last year or so it's been difficult to come up with the time and inclination to give Synthesis and Leprechaun the love they deserve. In order to get them that love, I am ceding maintainership to the excellent [Ben Lipson](https://twitter.com/bllipson). Ben is talented developer and Sitecore MVP with a lot of good ideas about where to take these tools. He'll do a great job.

Aside from transferring the repositories to Ben, nothing else is changing.

## Will you be disappearing from the community?

![nope](https://media.giphy.com/media/O9BPkYr89lK2A/giphy.gif)

No. `#venting` 4lyfe.

## What will you be working on at Sitecore?

I'll be on Team X, led by the illustrious [Alex Shyba](https://twitter.com/alexshyba). In other words, if I told you I'd have to kill you.

![actually it's JSS](https://media.giphy.com/media/u4DnvVypp0q8U/giphy.gif)

## `/giphy #magic8ball "Will this be awesome?"`

![yes](https://media.giphy.com/media/3o7abB06u9bNzA8lu8/giphy.gif)