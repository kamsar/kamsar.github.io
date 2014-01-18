---
layout: post
title: "Unicorn 1.0.3 Released"
date: 2013-06-21 22:50:09 -0800
comments: true
categories: Sitecore Unicorn
---
I've pushed Unicorn 1.0.3 to NuGet. This release fixes a number of bugs, improves logging, and removes some annoyances:

- Deleting a whole tree could cause the app pool to be killed due to a bug in Sitecore's serialization event handler (380479) - this fix expands on a previous workaround to cover recursive deletes
- Fixed an issue where if you changed an item too rapidly after saving or moving it, you could get file read errors or spurious "conflicts" reported due to the async ShadowWriter still being in process.
- "Inconsequential" item saves are now ignored. For example, the Template Editor gleefully changes the revision and last updated date of every template field when it saves, not merely changed ones. Those, as well as rename events where the name does not actually change (again, Template Editor ftw), are ignored and updated files are not written to disk. These fields are also ignored when calculating serialization conflicts on item save.
- Items deleted by Unicorn are now placed in the recycle bin instead of completely removed, so they can be restored if mistakenly deleted.
- Unicorn sync logs are automatically written to the Sitecore log as well (requires dependency upgrade to Kamsar.WebConsole 1.2.2 for the new TeeProgressStatus)