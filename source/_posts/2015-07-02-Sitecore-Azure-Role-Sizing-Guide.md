title: "Sitecore Azure Role Sizing Guide"
date: 2015-07-02 19:44:20
categories: Sitecore
---

It seems like everyone is hot to get Sitecore on [Azure](http://azure.microsoft.com/) these days, but there's not a whole lot of guidance out there on how Sitecore scales effectively in an Azure environment. Since I get asked the "what size instances do we need?" or "how much will Azure cost us?" question a lot, I figured some testing was in order.

The environment I tested in is a real actual customer website on Sitecore 7.2 Update-4 (not yet launched but in late beta). The site has been relatively optimized with normal practices - namely, appropriate application of output caching and in-memory object caching. The site scores a B on [WebPageTest](http://www.webpagetest.org/) - due to things out of my control - and is in decent shape. We are deploying to Azure Cloud Services ([PaaS](https://en.wikipedia.org/wiki/Platform_as_a_service)), using custom PowerShell scripting - _not_ the Sitecore Azure module (we have had suboptimal experiences with the module). We're using SQL Azure as the database backend, and the Azure Cache session provider. Note that the Azure Cache session is not supported for xDB (7.5/8) scenarios.

## The Tests

I wanted to determine the effects of changing various cloud scaling options relative to real world performance, but I also didn't want to spend days benchmarking. So I settled on two test scenarios:

* Frontend performance. This is a JMeter test that hits the site with 10,000 requests to the home page, with 1,000 threads at once. The idea is to beat the server into submission and see how it does.
* Backend performance. I chose Lucene search index rebuilding, as that is a taxing process on both the database and server CPUs, and it conveniently reports how long it took to run. Note that you should probably be using Solr for any xDB scenarios or any search-heavy sites.

These are not supposed to be 'end of story' benchmarks. There are no doubt many bones to pick with them, and they're certainly not comprehensive. But they do paint a decent picture of overall scaling potential.

## Testing Environment

The base testing environment consists of an Azure Cloud Service running two A3 (4-core, 7.5GB RAM, HDDs) web roles. The databases (core, master, web, analytics) are SQL Azure S3 instances running on a SQL Azure v12 (latest) server.

Several role variants were tested:
* 2x A3 roles, which are 4-cores with 7GB RAM and HDD local storage (~$268/mo each)
* 2x D2 roles, which are 2-cores (D-series cores are '60% faster' per Microsoft) 7GB RAM and SSD local storage (~$254/mo each)
* 2x D3 roles, 4-cores 14GB RAM and SSD local storage (~$509/mo each)
* 2x D4 roles, 8-cores 28GB RAM and SSD local storage (~$1018/mo each)

In addition, I tested the effect of changing the SQL Azure sizes on the backend with the D4 roles. I did not test the frontend in these cases, because that seemed to scale well on the S3s already.
* S3 databases, which are 100DTU "standard" (~$150/mo each)
* P1 databases, which are 125DTU "premium" (~$465/mo each)
* P2 databases, which are 250DTU "premium" (~$930/mo each)

## Results

Our first set of tests are comparing the size of the web roles in Azure. It is quite likely that these results would also be loosely applicable to IaaS virtual machine deployments as well.

### Throughput

{% asset_img rps.png "Throughput Graph" %}

The throughput graph is about as expected: add more cores, get more scaling. The one surprise is that the D2 instance, with 4 total cores at a higher clock and SSD disk, is able to match the A3 with 8 total cores (D2 and A3 cost a similar amount). 

Keep in mind that all of these were using the same SQL Azure S3 databases as the backend - for general frontend service, especially with output caching, Sitecore is extremely database-efficient and does not bottleneck on lower grade databases even with 16 cores serving. 

Note that I strongly suspect the bandwidth between myself and Azure was the bottleneck on the D4 results, as I saw it burst up to more like 500 during the main portion of the test.

### Latency

{% asset_img latency.png "Latency Graph" %}

Latency continues the interesting battle between A3 and D2. The A3 pulls out a minor - within margin of error - victory on average but the SSD of the D2 gives it a convincing performance consistency win with the 95% line over 1 second faster. Given that A3 and D2 cost a similar amount, it seems D2 would be a good choice for budget conscious hosting - but if you can afford Sitecore, you should probably stretch your licenses to more cores per instance.

The D3 is probably the ideal general purpose choice for your web roles. 

The latency numbers on the monstrous 8-core D4 instances tell the tale that my 1,000 threads and puny 100Mb/s bandwidth were just making the servers laugh.

These graphs illustrate the latency consistency you gain from a D-series role with SSD:

{% asset_img a3graph.png "A3 Latency Graph" %}
{% asset_img d3graph.png "D3 Latency Graph" %}

These two graphs plot the latency over time during the JMeter test. You can see how the top one (A3) is much less consistent thanks to its HDDs, whereas the lower one (D3) is smoother, indicating fewer latency spikes. The latency decreases in the final stage of the test due to some JMeter threads having completed their 10-request run earlier than others, so less server load is in play.

### Bandwidth

{% asset_img bandwidth.png "Bandwidth Graph" %}

Bandwidth is another measure of how much data is being pushed out. Once again, the lesser gain than you might expect from D4 was due to my downstream connection becoming saturated as the monstrous servers [laughed](https://www.youtube.com/watch?v=CTjolEUj00g).

### Backend: Rebuild all indexes

For this test, I rebuilt the default sitecore_core_index, sitecore_master_index, and sitecore_web_index indices.

{% asset_img backend.png "Backend Graph" %}

Here we see the limitations of the S3 databases creep in during our D4 testing. Up to D3 seems to be limited by CPU speed, but the D4 is actually slower until we both up the max index rebuild parallelism _and_ go to P2 databases to feed the monstrous 8-core beast. 

Note that with SQL Azure, the charge is per-db - so if you wanted to kit out core, master, and web on P2s you'd be paying near $3,000/month just for the DBs. At that point it might be worth considering using IaaS SQL or looking at the [Elastic Database Pools](https://azure.microsoft.com/en-us/documentation/articles/sql-database-elastic-pool/) to spread around the bursty load of many databases.

## Final Words

Overall Sitecore is a very CPU-intensive application where you can generally get away with saving a few bucks on the databases in favor of more compute, especially in content delivery. As in most applications, SSDs make a significant performance improvement to the point where I would suggest Azure D3 instances for nearly all normal Sitecore in Azure deployments, unless you need [Real Ultimate Power](http://www.realultimatepower.net/) from a D4 - or for the truly ridiculous a 16-core D14 ;)

For general frontend scaling the SQL Azure S3 seems to be the best price/performance until you go past quad cores, or past two web roles (one would assume 4x D3 would be relatively similar to 2x D4). Once you have 16 cores serving the frontend, you'll be bottlenecked below P2 databases at least for index rebuilding. Publishing probably has similar characteristics.

As always when designing for scale, the first step should be to optimize your code and caching. For example, before I did any caching optimizations I did the same JMeter test on the site. The A3s only put out 40 requests/sec - after output caching and tweaks, they did 144 requests/sec. That's not to say the site was slow pre-optimization: [TTFB](https://en.wikipedia.org/wiki/Time_To_First_Byte) was about 220ms without load. But once you add load, even small optimizations make a huge difference.