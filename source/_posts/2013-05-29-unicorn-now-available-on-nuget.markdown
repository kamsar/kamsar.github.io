---
layout: post
title: "Unicorn now available on NuGet"
date: 2013-05-29 22:55:09 -0800
comments: true
categories: Sitecore Unicorn
---
<p><a href="https://github.com/kamsar/Unicorn">Unicorn</a>, a library I wrote to handle automatically serializing Sitecore content items into a source control system, is now <a href="https://nuget.org/packages/Unicorn/">available on NuGet</a>.</p>

<p>Be sure to review the post-installation steps in the <a href="https://github.com/kamsar/Unicorn/blob/master/README.md">GitHub README</a> file - if used incorrectly, Unicorn can be dangerous to your items.</p>

<p>In other news, I began a <a href="https://github.com/kamsar/Unicorn/tree/extensibility">major refactoring of Unicorn</a> this weekend that will bring it a ton of new features mainly in the form of extensibility. Don't want it to delete orphaned items? Custom comparison to determine updating? Implement your own <i>Evaluator</i>. Want to serialize to SQL? Use a custom serialization format that ignores versions? Sweet - make a <i>SerializationProvider</i>. Want to filter serialization based on anything you can imagine (like the rules engine? an Excel spreadsheet? presence of a version in a specific language?)? That's where a <i>Predicate</i> comes in.</p>

<p>At the moment "Unicorn 2.0" is not yet fully stable. I plan to add a few more things to it such as detecting duplicate item IDs during processing and possibly a "what-if" mode. I'll be working on it time permitting :)</p>