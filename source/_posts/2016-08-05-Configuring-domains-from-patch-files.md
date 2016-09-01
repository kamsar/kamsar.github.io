title: Configuring domains from patch files
date: 2016-08-05 12:22:53
categories: Sitecore
---

Sitecore domains are logical security groupings, for example the product ships with "sitecore" (backend users) and "extranet" (frontend users). But you do not have to stick with only the domains the product ships with, as usual for Sitecore you can extend and add your own.

Normally adding a domain means editing `App_Config\Security\Domains.config`, but we don't want to do this. Why? Because editing standard Sitecore config files makes it difficult to upgrade Sitecore and introduces error prone file merging. 

What we want to do instead is use [_config patch_ files](http://sitecore-community.github.io/docs/documentation/Sitecore%20Fundamentals/Patch%20Files/). These allow us to add our config completely separately from Sitecore's standard configuration files. But there's a problem when it comes to domains: `Domains.config` does not live in the requisite `<sitecore>` config section so we cannot patch it.

Fortunately there's a little known way around this. Someone at Sitecore implemented a config-based domain manager, but it's not the default - presumably for backwards compatibility. And you _can_ activate that, and add domains to it, using patch files.

So next time you want to add a domain to Sitecore, like say adding a domain for each site to provide logical author role groupings, switch over to the config domain manager. You know you want `MySite\Editor` instead of `sitecore\MySite Editor`. It's easy to do, too.

This patch will activate the config domain manager (the defaults for the config manager already are the same as `Domains.config` so nothing else needs to be registered):

    <configuration xmlns:patch="http://www.sitecore.net/xmlconfig/">
        <sitecore>
            <domainManager defaultProvider="file">
                <patch:attribute name="defaultProvider">config</patch:attribute>
            </domainManager>
        </sitecore>
    </configuration>

And this patch is an example of adding a domain to the config domain manager:

    <configuration xmlns:patch="http://www.sitecore.net/xmlconfig/">
        <sitecore>
            <domainManager>
                <domains>
                    <domain id="MyNewDomain" type="Sitecore.Security.Domains.Domain, Sitecore.Kernel">
                        <param desc="name">$(id)</param>
                        <ensureAnonymousUser>false</ensureAnonymousUser>
                    </domain>
                </domains>
            </domainManager>
        </sitecore>
    </configuration>

And there you have it. Enjoy!