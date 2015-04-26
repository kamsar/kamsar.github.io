---
layout: post
title: "Upgrading Sitecore's Password Security with PBKDF2"
date: 2013-09-01 22:41:41 -0800
comments: true
categories: 
- Sitecore 
- Crypto
---
## The problem
A few months ago I read a very interesting article on Ars Technica titled [How I became a password cracker](http://arstechnica.com/security/2013/03/how-i-became-a-password-cracker/). Seriously, go read it. These days it's astonishing how simple it is to brute force even long, random-looking passwords. That 1337-5p33k p@55w0rd? You might as well use dictionary words because there are rules that check for that. Using a passphrase of english words (like [correct horse battery staple](https://xkcd.com/936/))? Well it's easy to test for combinations of dictionary words too, so those are a lot less secure than they look.

## Mitigation
The core problem is that most passwords are stored using hash algorithms such as SHA and MD5 that were originally designed to verify file integrity - so _speed_ was a primary concern. If a hash algorithm is fast, so is brute-forcing passwords that use it. Modern GPUs can compute hashes obscenely fast (spend a bit and you can hit 1 _billion_ hashes per second), so a good way to thwart attackers is simply to use a hash algorithm that is _slow_. Several algorithms exist that are designed to be utilized specifically for passwords (such as [PBKDF2](http://en.wikipedia.org/wiki/PBKDF2) and [BCrypt](http://en.wikipedia.org/wiki/Bcrypt)), and they are all much slower to compute for this very reason.

These algorithms also usually have a "work factor" that allows you to adjust the time it takes to compute the hash, which provides even more future-proofing against even faster password cracking hardware that is yet to be developed.

## When would this affect Sitecore?
To be clear this is not a remotely-exploitable vulnerability. It's also not a Sitecore-specific vulnerability - it affects most all sites running ASP.NET's Membership provider which until very recent providers (Universal Providers) defaults to the SHA1 hash algorithm to store passwords. _In order to exploit what I'm describing here, an attacker would have to gain access to your databases_, which they would then use an offline password cracker like [oclHashcat-plus](http://hashcat.net/oclhashcat-plus/) to break the passwords to login to Sitecore.

## Using PBKDF2 with Sitecore
There's a [class in the .NET framework that implements the PBKDF2 algorithm](http://msdn.microsoft.com/en-us/library/system.security.cryptography.rfc2898derivebytes.aspx) already, but it does so in a way that makes it difficult to add to a membership provider. There's a very nice NuGet package called [Zetetic.Security](http://www.nuget.org/packages/Zetetic.Security) that wraps this functionality into a `KeyedHashAlgorithm` that can be used easily with membership. It's very easy to [install PBKDF2 into a membership provider](http://zetetic.net/blog/2012/7/3/secure-password-hashing-for-aspnet-in-one-line.html) using it - you have to install the NuGet package and change two configurations.

The only problem is that existing users will be unable to login because the existing hashes are all in SHA1 (Sitecore, and membership's, default algorithm) and the membership provider now expects PBKDF2 hashes. If there are only a few users you can just reset passwords and be done with it. But if you have an existing userbase, you probably wish you could allow existing users to keep signing in, and new users (or password changes) to convert to PBKDF2. I've implemented [a prototype extension of the SqlMembershipProvider](https://gist.github.com/kamsar/6407742) that does exactly this by allowing the `ValidateUser` method to attempt SHA1 if PBKDF2 validation fails.

## Other uses
The solution discussed in this post was tested against Sitecore, but is a generic method you could use with any .NET app that uses SQL Membership to store user info. I hope this random crypto rant has been useful to you :)