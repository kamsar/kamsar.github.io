title: "Blogging with Hexo - a Node.js detour"
date: 2015-04-25 21:07:52
categories: Node.js
---

For the last couple years, I've run my blog from [GitHub Pages](https://pages.github.com/) using a static site generator, specifically [Octopress](http://octopress.org/). Static Site Generators are more or less a basic CMS for technical folks - you define your layout template(s) and then write content that inherit one of those layouts - say, a blog post. When you've authored your post you "bake" the site using the SSG tool, which takes the page layout and combines it with your content items and then spits out static HTML pages and assets. You then can upload the 100% static HTML site to a hosting service - such as GitHub Pages, Heroku, or Azure Websites.

Using a static site generator might seem quite limiting at first - after all, highly dynamic CMS like Sitecore would never work with HTML files and no database. So yes - you give up personalization. You give up any sort of server-side logic. You have to use JS-based commenting (e.g. Disqus, FB comments).

But you gain a lot as well by using such a tool. Ever hacked a static HTML site? Thought not. When was the last major WordPress security hole that demanded you to upgrade now? Yeah - last week. With a staticly deployed site you can also use the full branch and merge power of Git to keep a permanent history of your posts - and even accept pull requests. You'll also never have to worry about backend performance - there is no backend. Authoring static sites can be done in powerful meta-languages, like writing posts in Markdown, CSS with [SASS](http://sass-lang.com/), and JS with [Browserify](http://browserify.org/). And finally, there are completely free hosting services for static sites - such as GitHub Pages - so it can cost nothing to host a static blog.

## Enter Hexo

My previous Octopress blog suffered from an annoying problem: it was a huge pain to set up the generation environment on Windows, because it depended on Ruby, Python, Ruby DevTools, and a host of other very specifically versioned libraries. Not fun - in fact, I resorted to authoring all my posts on OS X. I started to look for something better.

It turns out there are [just a few](https://staticsitegenerators.net/) static site generators out there, in about every programming language known to humanity. I evaluated a number of them and settled on [Hexo](http://hexo.io/), which is written in Node. "But you're a C# developer," you say. In this case the C# options (like [Pretzel](https://github.com/Code52/pretzel)) are significantly less mature - and I like that I can run Hexo on any platform because it's JavaScript.

Setting up Hexo was very simple and took about 5 minutes. No really, the whole set of commands you need are on the [front page of the Hexo site](http://hexo.io/) (well, except "install Node.js first"). I was able to [import all my Octopress posts](http://hexo.io/docs/migration.html#Octopress) nearly unmodified, and tweak the base URL settings to keep my faux-wordpress URL scheme intact.

Then I wrote this post by simply entering `hexo new post "Blogging with Hexo - a Node.js detour"` at a command prompt, opening the file in Sublime Text, and writing my Markdown. I previewed it while writing using `hexo server`, which starts a HTTP server you can see the rendered page on. When I deploy it to the blog, all I'll have to do is `hexo deploy` at a command prompt and all the git magic will be done by Hexo.

Neat, huh?