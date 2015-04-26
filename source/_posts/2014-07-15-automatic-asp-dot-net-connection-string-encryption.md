---
layout: post
title: "Automatic ASP.NET connection string encryption"
date: 2014-07-15 17:29:06 -0700
comments: true
categories: 
- Sitecore
---

Encrypting your connection strings is a good practice as it obfuscates your database connection information from an attacker who has gained filesystem access to your web server. It's a bit of a pain to implement if you're wanting to keep all of your build happening on a CI server however, as encrypting it requires a executing `aspnet_regiis` on the target server because the encryption is keyed by the machine key of the destination server.

The other day I was reading [Thomas Stern's article](http://blog.istern.dk/2014/05/01/encrypting-and-securing-your-sitecore-connenction-strings/) about encrypting Sitecore connection strings and it got me to thinking of ways to make connection string encryption fully automatic with minimal configuration. Turns out it's actually quite easy.

There are actually quite nice [C# APIs to encrypt and decrypt](http://www.codeproject.com/Tips/598863/EncryptionplusDecryptionplusConnectionplusStringpl) so I set to try and make the application self-encrypt on startup using these APIs. That way you can deploy an unencrypted configuration file from CI, and as soon as the app starts up (which is immediately in my case since we run some HTTP calls to it as part of the deploy) it will encrypt itself using the local machine key.

To use this code example you will need to install the [WebActivator NuGet package](https://www.nuget.org/packages/WebActivatorEx/) that allows it to register itself as an application startup task. You can also just call the `AutoEncrypt` method in `Global.asax` for the same effect, but WebActivator makes it completely self-contained.

This example only runs encryption if `<compilation debug="false">` is in the web.config (so for local debugging you keep yourself decrypted), and the section has not already been encrypted. It is also fully compatible with the default `App_Config\ConnectionStrings.config` that Sitecore uses as the config reader is smart enough to save the right file.

{% gist ef3a9f3b7ef1f9d491d5 %}

Hope that's helpful.