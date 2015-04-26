---
layout: post
title: "Synthesis 7.2 released"
date: 2014-07-07 20:33:57 -0700
comments: true
categories: 
- Synthesis
---

Synthesis 7.2 has been released on NuGet. This release includes a number of enhancements to the content search APIs to accomdate more real world situations, as well as several general bug fixes and improvements.

**Synthesis 7.2 only runs on Sitecore 7.2.x due to changes in the content search APIs in 7.2.** It is hypothetically compatible with Sitecore 7.5 as that is supposed to be based on the 7.2 core but I have not tested it. Previous versions of Synthesis are **not** compatible with Sitecore 7.2 because they are compiled against the earlier content search APIs.

So what's changed?

* The `Uri` property on template items has been renamed `ItemUri` to avoid confusion with the similarly named Url property
* Standard item properties related to searches - namely `Statistics` and `Paths` - have been flattened into properties on `IStandardTemplateItem` itself. This allows you to easily use them in content search queries
* Additional `TemplateName`, `Path`, `ParentId`, `AncestorIds`, `DatabaseName`, and `SearchableContent` properties have been added to `IStandardTemplateItem` to support content search queries
* Added `WhereResultIsValidDatabaseItem()` and `TakeValidDatabaseItems()` LINQ extension methods. These allow you to filter index-based result sets and remove any index entries that do not have valid database-based equivalents. Due to the way Synthesis transparently promotes index-based items to database items, this can avoid runtime exceptions during promotion if the index item has no database equivalent or security denies access to the database equivalent.
* Synthesis no longer requires a global registration of its document type mapper now that Sitecore 7.2 supports overriding the mapper via an `ExecutionContext`
* Multiple `ExecutionContext` parameters can now be passed to `GetSynthesisQueryable`
* Support for doing OR and AND searches over collections, such as multilist targets, via the `ContainsOr` and `ContainsAnd` extension methods (original implementation by [@jason_bert](https://twitter.com/jason_bert))
* Integrated [@jorgelusar](https://twitter.com/jorgelusar)'s web test runner to improve visibility of Synthesis' unit tests. Added a lot of tests to cover and document the content search functionality available.
* Added support for using multilist, datetime, and lookup field values from the index without object promotion, if the value is stored in the index.
* The `ignoreStandardFilters` parameter on `GetSynthesisQueryable()` was incorrectly being ignored
* Fixed a bug where `StartupRegenerateProjectPath` entries starting with ~ were not correctly resolved (thanks [@delgadobyron](https://twitter.com/delgadobyron)!)
* Generating templates whose name starts with a number will now work correctly (prefixed with _), courtesy of [@martijn_b0s](https://twitter.com/martijn_b0s)
* Using the `SelectSingleItem()` and `SelectItems()` queries on a template item will no longer cause an error if the query returns no results, courtesy of [@martijn_b0s](https://twitter.com/martijn_b0s)
* A number of hacks to fix issues in the Sitecore search expression parser that were fixed in Sitecore 7.2 have been removed.

As always, thanks to the community members who use and contribute to Synthesis - you all rock. Feel free to hit me up on [Twitter](https://twitter.com/kamsar) or report an issue or pull request on [GitHub](https://github.com/kamsar/Synthesis) if you run into problems.