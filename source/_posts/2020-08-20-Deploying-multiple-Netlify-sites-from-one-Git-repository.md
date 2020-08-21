title: Deploying multiple Netlify sites from one monorepo
date: 2020-08-20 20:30:32
categories: Jamstack
---

[Netlify](https://www.netlify.com/) is an incredibly easy to use and powerful host for static sites. While this blog is not hosted there, I did [deploy it to Netlify](https://hilarious-parrot.netlify.app/), no joke in less than 5 minutes. With automatic deployment of updates on commit. Check em out.

Anyhow today I had a monorepo that has more than one site that I wanted to deploy to Netlify. The repo looked something like this:

```
MyRepo
└───sites
    ├───charlie.com
    └───nick.org
```

Netlify [supports this](https://www.netlify.com/blog/2019/10/09/launching-monorepo-support-for-netlify-sites/) but it's not super well documented how to accomplish it without a little sleuthing.

Monorepo support works by setting the `Base directory` of each site's configuration to point at the relative path to your site root:

{% asset_img "gui.png" %}

As it says on the tin, this essentially sets the path to `cd` to before starting the build commands. (Find it here: `https://app.netlify.com/sites/<your-site-name-here>/settings/deploys`)

But there's one cool thing the description forgets to mention: You can also configure Netlify sites using a [`netlify.toml`](https://docs.netlify.com/configure-builds/file-based-configuration/) file, making your configuration versioned in Git. This gets really useful to control the whole build stack from one place: configuring the build commands, redirects, setting up lambda functions. Netlify usually expects `netlify.toml` in the root of the repository. However, the _Base directory_ setting also changes where the `netlify.toml` is expected to live. If we do this:

```
MyRepo
└───sites
    ├───charlie.com
    │   └───netlify.toml
    └───nick.org
        └───netlify.toml
```

...then we get the lovely capability to both monorepo our sites, and also version our Netlify configuration. Awesome!