title: 'Unicorn 4 Preview, Part 1: Project Dilithium'
date: 2017-02-01 20:51:13
categories: Unicorn
---

Not too long ago I was considering the future of Unicorn. Rather naïvely, I couldn't think of all that much that needed fixing. I wondered if there would ever be a Unicorn 4. 

Now since you've read the title of this blog post, something tells me you don't think that attitude lasted very long. It didn't. Because there's a pretty simple truth about Unicorn, even with all the [optimizations I put in to Unicorn 3](http://kamsar.net/index.php/2015/09/Unicorn-3-What-s-new/#Performance-50-more-of-it):

{% asset_img tdh.jpg %}

There were two primary causes of this:

1. The Sitecore API is slow and not suited to dealing with large quantities of items
2. Unicorn was originally designed to optimize for a few configurations, and Helix was blasting the number of active configurations to large numbers

I found that the sync time was becoming annoyingly long. Long enough that it was a distraction, long enough that a piddly 20% faster from more profiling just wasn't good enough.

What to do, what to do.

## Introducing Project Dilithium

I started on a crazy quest to speed up syncing performance. Sitecore's recently added [Publishing Service](https://stephenpope.github.io/publishing) gained drastic speed gains in publishing by leveraging direct database access. Publishing, like syncing, is a batch-oriented operation. The Sitecore API is a single-item-oriented API. It does no optimizations, aside from caching, when you want more than one item. I resolved to try out SQL and see how it went. It went. See for yourself:

{% asset_img perf.png %}

(benchmarks on i5-3570k@4GHz, SSDs, freshly restarted IIS + SQL, average of 3 runs each)

{% asset_img whoa.jpg %}

### So what is [Dilithium](http://bit.ly/2kjLK50)?

There are two pieces of the Dilithium project: SQL and Serialized. Both operate in a similar fashion: before a sync operation begins, all items that will be involved in that sync are read in a single big batch either from SQL or disk. The actual sync then occurs against the pre-batched items. In case you're wondering, the graph above _includes_ the batching times for Dilithium - no cheating :)

The SQL version takes advantage of the `Descendants` table to grab all predicate-included items for every configuration in a single huge SQL query (this does mean that `FastQueryDescendantsDisabled` must be turned off). These items are then indexed and prepared for fast access. In this way, the entire data contents of the stock `core` database can be read in about 1500ms, compared to the Sitecore API's 8 seconds.

The serialized version skips all the checks that usually are needed to traverse a Serialization File System tree - expensive operations that guarantee that the children you're getting all belong to the same item ID, for example. It's essentially reading all files sequentially and caching them for sync just like the SQL version.

When both DiSerialized and DiSQL are used together, they enable _parallel loading_ of both sources at once. This reduces the load time quite significantly, as you can see on the shortest bar on the chart above.

### Questions?

#### Direct SQL, are you nuts?

Yes. Yes I am. Directly querying the Sitecore database is a terrible idea in almost every situation...except this. The schema for content has also been very stable for several major revisions of the product. And Stephen Pope said it was ok :)

#### Is it any faster adding or modifying items?

No. I'm not crazy enough to also do direct SQL writes, so all modifications go through normal Sitecore item saving APIs and perform pretty much the same as Unicorn 3. This also means that they're mature, well-tested code - Unicorn 4 under the hood is not much different from v3, except for Dilithium.

#### Can I turn it off?

Heck yes. You can enable or disable Dilithium on a per-configuration basis. It also works just fine if you sync Dilithium and standard configurations together.

Serialized items and formats are identical between Dilithium and normal, so you can also enable and disable at will with no migration process.

#### Will this eat up all my RAM?

If you're syncing gigabytes of media items...yes, definitely. For more normal developer and light content items, nope. DiSQL does not read blob data, so if you become RAM constrained turn off DiSerialized first.

Note that the precaches are disposed of immediately after the sync completes, so memory spikes are temporary.

#### Let's address the 🐘 in the room, how is Unicorn 4 60% faster than 3 without Dilithium?

Actually pretty simple: eliminating a cache bombing that was done between syncing configurations that was needed for Transparent Sync. Well, it wasn't needed if you're syncing a non-Transparent configuration. Doing that meant that sites like Habitat with a ton of small non-Transparent configurations perform a _lot_ faster.

#### Any other performance tricks?

In my daily development I use a mix of Transparent Sync for developer items (templates, renderings), and standard sync for content items. Since Transparent Sync is disk-based anyway there is little point to running a sync on Transparent Sync configurations when you're just doing local development. As such, there's now an option to _skip_ Transparent Sync-enabled configurations when you sync all. This is an option in the PowerShell API in Unicorn 4, as well as an option in the control panel (same place as log level).

Depending on how many Transparent Sync configurations you have, your time savings can be significant from enabling this. You should not however do this when deploying transparent configurations to a shared server.

## Can I have it?

Not yet, it still needs some additional testing and vetting before I call it alpha. If you're interested in being a brave early tester, get ahold of me on Sitecore Slack :)

## Is this the only new thing in Unicorn 4?

Nope. I've got some more tricks up my sleeve, so stay tuned for some more preview blogs coming soon&trade;

![](http://i.giphy.com/26uf80oUzpxWmuTOo.gif)