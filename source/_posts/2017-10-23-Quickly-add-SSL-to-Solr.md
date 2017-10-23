title: Quickly add SSL to Solr
date: 2017-10-23 08:20:39
categories: Sitecore
---
There have been several people recently who I've seen having trouble setting up SSL for their Solr in order to use it with Sitecore 9. So, I present the following gist to you. It's designed to automate the complete setup process of adding SSL to Solr with a self-signed certificate, and trusting that self-signed certificate. For production setups with a real certificate, it should be quite easy to modify.

It's been tested on standalone as well as Bitnami Solr. The script requires Windows 10 to use the `Import-PfxCertificate` cmdlet; if you don't have that you can remove the trust scripting and do it manually.

{% gist c3c8322c1ec40eac64c7dd546e5124de %}

![giphy](https://media.giphy.com/media/IB9foBA4PVkKA/giphy.gif)