title: The lazy developer's way to install Sitecore 9
date: 2017-11-01 17:37:11
categories: Sitecore
---

Since Sitecore 9 was released, there's been a lot of talk about the new installation techniques that it necessitates - namely, the move towards infrastructure as code and the Sitecore Install Framework (SIF). It's no secret that installing Sitecore 9 can be a bit more difficult than previous versions, but it really doesn't have to be.

This is the part where you might be expecting me to announce some crazy script I wrote, but not this time because someone else already did the work. So let's address the elephant in the room.

## Solr the easy way

Back in the day [I wrote some scripts to install Solr](https://kamsar.net/index.php/2016/03/The-Solr-Cannon/) using Bitnami. It worked, but I'd always wanted to find the time to make it simpler and less dependent on Bitnami and their notoriously hard to find older versions. Well [Jeremy Davis did exactly what I wanted to do](https://jermdavis.wordpress.com/2017/10/30/low-effort-solr-installs/) and scripted the whole Solr install, locally trusted SSL certificate, and installation as a service. You can also just [skip straight to the gist of the PowerShell you need to run](https://gist.github.com/jermdavis/8d8a79f680505f1074153f02f70b9105).

Seriously, it's awesome and you should use it especially for local dev setups.

A few things I noted when I used it:

* Change the download URLs for Solr and NSSM to be https (encrypted). They work fine that way.
* You must install the Java Runtime Environment (JRE) first and plug in the right version - it won't do it for you
* Make sure to add the `$SolrHost` value to your hosts file before you run the script so that it can resolve with the SSL certificate correctly (it will be bound to that name; don't use `localhost`).

## SIF the easy way with SIFless

SIF is a pretty amazing tool, but it has two shortcomings: one, that it's great for automated infrastructure but not so great for a quick local setup and two, that it doesn't yet have an uninstall feature. Well [Rob Ahnemann wrote a handy GUI for SIF called SIFless](http://www.rockpapersitecore.com/2017/10/introducing-sif-less-for-easy-sitecore-9-installation/) that fixes both of those issues, making quick setups with mostly default settings easy _and_ generating hackable SIF PowerShell scripts that let you do whatever advanced things you want after using the GUI to get started. And it generates uninstall scripts too that get rid of the windows services, solr cores, and other artifacts that are left when you want to tear down that test site.

A few things to be aware of with SIFless:

* Despite the amusing name, SIFless _does_ require SIF to be installed!
* The Solr URL needs to be the path to the Solr admin panel (e.g. not `https://mysolr:8983`, but `https://mysolr:8983/solr`)
* The Solr physical path needs to be to the root of the Solr instance (if it's the right place you'll see 'bin' and 'server' folders; if you used the script above with defaults this would be `C:\solr\solr-6.6.2`)

## Go forth and use Sitecore 9

Using these two tools I went from having no Solr and no Sitecore installations to having a fully operational ~~battle station~~ Sitecore 9 instance with xConnect in about 45 minutes. And that includes debugging my own silly mistakes. I bet you can do it faster. Get thee to a PowerShell console!

![time to get SIFty](https://media.giphy.com/media/cCba2gQCxNpsc/giphy.gif)