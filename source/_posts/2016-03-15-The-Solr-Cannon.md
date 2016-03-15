title: 'The Solr Cannon: Rapid Solr Setup for Sitecore'
date: 2016-03-15 14:43:43
categories: Sitecore
---
Have you ever wished that standing up a Solr server for Sitecore was easy? I have. So I made a script to make it so.

The Solr Cannon automates the setup and configuration of Solr on Windows (Note: Linux may be a better production choice if you have the option), including installation as a background service, Sitecore schema installation, core creation, and generating a Sitecore config patch file that will point Sitecore at your shiny new Solr server and its cores.

## What you'll need

* A copy of the [Bitnami Solr Stack](https://bitnami.com/stack/solr) for Windows (I used 5.5.0-1)
* A copy of [the script](https://gist.github.com/kamsar/ef8811bd458603f1e808#file-install-solr-ps1) and the [solr schema file](https://gist.github.com/kamsar/ef8811bd458603f1e808#file-schema-xml) (the schema was generated with Sitecore 8.1 Update 1, you might need to generate your own if it doesn't work)
* PowerShell 3.0 or later

## Let's do this

* Copy the Solr stack and scripts to a folder on the server that will run Solr
* Review the `Install Solr.ps1` script to make sure the variables are to your liking (_project name_, ports, solr stack installer path, etc)
* Open an administrative PowerShell prompt
* Execute `Install Solr.ps1`
* Kick back for a couple minutes
* Bask in the glow of your new Solr server and all of its Sitecore cores

## But what about the rest?

Ok ok, that's not all you need to do in order to configure Solr and Sitecore. 

* Check out [Patrick Perrone's post](http://sitecoreblog.patrickperrone.com/2015/02/sitecore-8-solr-part33-configur-ageddon.html) to help you [swap in Solr](http://sitecoreblog.patrickperrone.com/2015/03/a-script-to-swap-search-in-sitecore.html) as the search provider and configure the Solr IoC container. Note: I'd recommend Windsor for the IoC. Autofac and Ninject were less than awesome install experiences...
* Once you have the Sitecore configuration flipped to Solr, you can grab the `Solr.config` file that the Solr Cannon produced when it installed Solr (written to the script directory). This config patch will:
	* Point Sitecore at your Solr server
	* Make Sitecore look at the correct core names for each index
	* Enable `SwitchOnRebuildSolrIndex` which allows you to rebuild indexes without any index downtime (great for production...or dev)
* Load up Sitecore, open the indexing manager, and rebuild all your indexes. With a bit of luck, you'll be good to go!
 
## Multi-tenant Solr

Suppose you're working on your own machine and you've got more than one dev site that's using Solr. The Solr Cannon scripts can act to create a new config set and cores for several Sitecore installations on the same Solr server. Edit the script to have a different `$ProjectName` set, run it again, and say no when asked if you need to install a Solr server. In this mode, the script will build out a config set and cores for a new Sitecore site, as well as the patch file.


Have fun :)