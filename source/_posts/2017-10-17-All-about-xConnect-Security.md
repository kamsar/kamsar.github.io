title: All about xConnect Security
date: 2017-10-17 08:57:18
categories: Sitecore
---
Sitecore 9 introduces the new xConnect server layer to the ecosystem. xConnect is an abstracted service layer that Sitecore uses for all its analytics and marketing automation features. If you're using xDB, you'll need an xConnect server if you upgrade to Sitecore 9.

xConnect is noteworthy because it introduces required client certificate authentication for the Sitecore XP server to communicate with xConnect. Certificates are a complex subject, and can fail in any number of less than helpful ways. This post aims to help you understand how certificates work in Sitecore 9, and provide you some tools to diagnose what's wrong when they are not working right. 

## What is TLS?

In order to understand how xConnect works, it's important to understand what's going on: [Transport Layer Security](https://en.wikipedia.org/wiki/Transport_Layer_Security) (TLS). Historically, TLS has also been called SSL.

TLS is a protocol for establishing secure encrypted connections between a _server_ and a _client_. Importantly, the client and server can securely exchange encryption keys in such a way that they cannot be observed by malicious parties that may be watching the exchange.

## Asymmetric vs Symmetric Encryption

To understand how TLS works, it's important to understand the distinction between Asymmetric (also called Public Key) Encryption, and Symmetric Encryption.

If you ever made secret codes as a kid, you've probably used [_symmetric encryption_](https://en.wikipedia.org/wiki/Symmetric-key_algorithm). This is where the sender and receiver both need to know a key to decrypt the message, for example a simple [shift cipher](https://en.wikipedia.org/wiki/Caesar_cipher) where `C` = `A`, `D` = `B`, and so forth. Posession of the secret key lets you read any encrypted message.

[Asymmetric encryption](https://en.wikipedia.org/wiki/Public-key_cryptography) on the other hand uses two different keys: a _public key_ and a _private key_. The public key can be shared with anyone without compromising anything. However a client can use the public key to encrypt a message in such a way that it can only be decrypted _with the server's private key_. In this way, you can receive private encrypted messages from clients you don't share any secrets with - but they can still send the server private messages.

TLS uses _asymmetric encryption_ to transfer an encryption key for _symmetric encryption_, which is used for ongoing data transfer over the encrypted connection. This is done because asymmetric encryption is much much slower than symmetric.

It's important to understand the difference between public and private keys when you set up Sitecore 9, because they need to be deployed to different servers in your infrastructure. A _certificate_ generally includes both a public and private key, however it can also include only a public key.

## xConnect Setup

xConnect uses [mutual authentication](https://en.wikipedia.org/wiki/Mutual_authentication) to secure the connections between it and the Sitecore XP server. This is accomplished using [TLS client certificates](https://en.wikipedia.org/wiki/Transport_Layer_Security#Client-authenticated_TLS_handshake).

If you've worked with SSL certificates before, this is a stronger form of SSL where not only does the client have to trust the server, but the server also has to trust a second certificate issued to the client. In this case, the client is the Sitecore XP server, and the server is the xConnect server. Let's take a look at how this works:

## SSL Server Certificate Negotiation

All SSL connections go through this process, whether xConnect or otherwise. In a standard Sitecore 9 XP installation, the xConnect server will have the _server certificate_ installed. The Sitecore XP server will only have a server certificate if access to Sitecore itself, e.g. for administration, is done via SSL (in which case it will likely be a separate server certificate from xConnect's).

1. Client prepares to make a HTTPS request (e.g. you ask for `https://xconnect`)
1. Client sends a _ClientHello_ message to the server. This proposes encryption standards, among other things.
1. The server replies with a _ServerHello_ message back to the client. This includes the server's _public key_, and the encryption standards that the server has selected from what the client proposed in the _ClientHello_.
1. The client _validates_ the server certificate (e.g. must have correct domain and trusted issuer)
1. A symmetric encryption key is generated and exchanged using the server's public key
1. Now that an encrypted connection is established, a normal HTTP request is sent over the encrypted channel

### What can go wrong with server certificate negotiation

The most common issues are domain mismatches and untrusted certificates. Generally you can diagnose issues with server certificates using a web browser - request the site over HTTPS and review the error shown in the browser. Specifically for xConnect make sure you request the xConnect URL, **not** the Sitecore XP URL, to diagnose this. 

#### Domain Mismatches

A domain mismatch occurs when a certificate's domain does not match the domain being requested. For example, a certificate issued to `sitecore.net` will fail this validation if the site you're requesting is `https://foo.local`. Certificates may also be issued using _wildcards_ (e.g. `*.sitecore.net`). Note that wildcards apply to one level of subdomains only - so in the previous example `sitecore.net` or `foo.sitecore.net` would be valid, but `bar.foo.sitecore.net` would _not_ be. 

If you have a domain mismatch issue, you will need to either get a new certificate (and update the xConnect IIS site(s) to use the new certificate) or change the domain for xConnect to one that is valid for the certificate.

#### Untrusted Certificates

To understand trust issues, it's important to understand how certificates are issued. Certificates are issued by other certificates.

{% asset_img cert-issuer.jpg %}

In fact, certificates can be issued in _chains_ (Xzibit would definitely approve). Trust issues occur when the certificate that issued the _server_ certificate is not considered to be trusted by the _client_. On Windows, trust is established by being included in the _Trusted Root Certification Authorities_ in the machine certificates:

{% asset_img trusted-certs.png %}

Note that to trust a certificate, only the _public key_ for the server certificate must be imported here. If you're using self-signed certificates that issued themselves - like `localhost` in the screenshot - you can add the certificate itself to the trusted root certificates by exporting it and reimporting it into the root certificates. If using a commercially issued certificate, that certification authority's root certificates must be added to the trusted root - in most cases, they are already present.

#### More esoteric errors

There are some less common errors that can also cause server certificate negotiation errors. Commonly servers will be secured against supporting vulnerable [ciphers](https://en.wikipedia.org/wiki/RC4), [hash algorithms](https://en.wikipedia.org/wiki/SHA-1) or [SSL protocol versions](https://en.wikipedia.org/wiki/POODLE). This is most easily accomplished with a tool like [IISCrypto](https://www.nartac.com/Products/IISCrypto). This is a good idea, but if the server and client cannot mutually agree on a supported cipher, hash, and protocol version the connection will fail. If the certificate is trusted and has the correct domain, this would be the next thing to check.

Note that the .NET HTTP client with NetFx versions prior to 4.6.2 defaults to only supporting TLS up to 1.1. Many modern security scripts will disable all TLS protocol versions except for 1.2, which will cause HTTP requests from clients with earlier versions of the .NET framework installed to fail.

## SSL Client Certificate Negotiation

Hopefully now you have a decent idea of how _server_ certificates work. But xConnect also uses _client certificates_. A client certificate enables [_mutual authentication_](https://en.wikipedia.org/wiki/Mutual_authentication). With only a server certificate, the client must decide to trust the server but the server has no way to know if it should trust the client. Enter client certificates.

A client certificate is essentially the opposite of the server certificate. When using a client certificate, the negotiation works similarly to the server certificate, except that when the server sends the `ServerHello` (#3 above) it requests a client certificate in addition to sending its public key. The client then sends the public key of its _client certificate_ back to the server - and then _the server decides whether it should trust the client certificate_.

If the client certificate is not trusted, it is rejected. The rules for validating a client certificate are up to the server and do not necessarily follow the same validation rules as a server certificate on the client. In the case of xConnect:

* The domain/subject on the certificate does not seem to matter to xConnect
* The trusting of the certificate is done using the _thumbprint_ of the certificate (a hash of the certficate which uniquely identifies it)
* The xConnect server must trust the issuer of the client certificate

### What can go wrong with client certificate negotiation

There are a lot of things that can go wrong with the client certificate, moreso than the server certificate. When troubleshooting, make your first step the Sitecore logs - they generally have some basic information about a bad client cert.

#### If you're receiving HTTP 4xx responses

Chances are your client certificate validation failed. This could mean:

* The client certificate is not installed on both the Sitecore XP server and the xConnect server (the xConnect server would only need the public key)
* The client certificate is not considered trusted on the xConnect server
* The certificate _thumbprint_ configured in the xConnect server's `App_Config\ConnectionStrings.config` is missing or incorrect. Note that the thumbprint must be all uppercase with no spaces or colons. If copied from certificate manager, an unprintable character might prefix the thumbprint - check for a hidden character there.
* The certificate _location_ configured in the xConnect server's `App_Config\ConnectionStrings.config` is incorrect. Normally the certificate should be stored in local machine certificates and have a connection string similar to `StoreName=My;StoreLocation=LocalMachine;FindType=FindByThumbprint;FindValue=THUMBPRINTVALUE`.

#### "The certificate was not found"

This indicates one of two things:

* The _thumbprint_ is incorrect in the Sitecore XP server's `App_Config\ConnectionStrings.config` file. Note that the thumbprint must be all uppercase with no spaces or colons. If copied from certificate manager, an unprintable character might prefix the thumbprint - check for a hidden character there.
* The certificate _location_ configured in the Sitecore XP server's `App_Config\ConnectionStrings.config` is incorrect. Normally the certificate should be stored in local machine certificates and have a connection string similar to `StoreName=My;StoreLocation=LocalMachine;FindType=FindByThumbprint;FindValue=THUMBPRINTVALUE`.

#### System.Net.WebException: The request was aborted: Could not create SSL/TLS secure channel.

_As long as the server certificate is valid_, this message is most likely that the Sitecore XP server's IIS app pool user account does not have read access to the client certificate's private key. This access is needed so that the Sitecore XP server can encrypt communications using its client certificate.

To remedy this issue, open your local machine certificates ("Manage computer certificates" in a start menu search). Find the client certificate (normally under `Personal\Certificates`). Right click it, choose `All Tasks`, then `Manage Private Keys...`. You should get a security assignment window like this:

{% asset_img pkey-security.png %}

Next, add your IIS app pool user to the ACLs and grant it `Read` permissions (as above). Remember if you're using AppPoolIdentity (you should be, unless using a domain account for windows auth to SQL), you must select the account by choosing `Local Computer` as the search location, and enter `IIS APPPOOL\MyAppPoolsName` as the account name to find.

Still having issues? Well, you can also use the security audit log to find out which account is failing to get access, then add that account in the key ACLs above:

{% asset_img key-security-audit-failure.png %}

## Local Development Tip

If you work at a Sitecore partner and will have multiple copies of Sitecore 9 running locally, this can cause issues if you issue a dedicated SSL server certificate to each site. This is because a given TCP port (e.g. 443, the default) can only have one SSL certificate bound to it. This precludes having multiple Sitecore 9 instances running together locally _unless they share the same SSL certificate_.

Wildcard certificates are perfect for this job. As long as you use the same top level suffix for all your dev sites (e.g. `*.local.dev`), you can use the same wildcard certificate for your server certificate for all dev sites. Note that IIS' self-signed certificate generator will not create a wildcard certificate for you. You'll have to use something else, like [New-SelfSignedCertificate](https://technet.microsoft.com/en-us/itpro/powershell/windows/pkiclient/new-selfsignedcertificate), to create one.

Important note: You must issue a wildcard for at least two segments of domain for it to be trusted. For example `*.dev` is bad, but `*.local.dev` is good.

Note that _client_ certificates should be unique on each site, only the server certificate should be shared.

## Advanced Debugging with Wireshark

It's possible to watch the SSL negotiation at a TCP/IP level using a network monitor such as [Wireshark](https://www.wireshark.org/). This can provide insights on why your setup is failing when error messages are less than optimal. For example I spent a couple days diagnosing what turned out to be private key security issues. I figured this out by using Wireshark and observing that the client was never sending its client certificate after the server requested it, and figuring out why that was.

To use Wireshark to watch SSL traffic, you'll have to set it up to decrypt traffic. [This guide](https://blogs.technet.microsoft.com/nettracer/2013/10/12/decrypting-ssltls-sessions-with-wireshark-reloaded/) walks you through setting up decryption on Windows with an exported private key.

If you're tracing local dev traffic (e.g. from `localhost` to `localhost`, including using your machine's DNS name) Wireshark will not capture that unless you install [npcap](https://nmap.org/npcap/) instead of the default `pcap` packet capture software. Once `npcap` is installed, tell Wireshark to bind to the `Npcap Loopback Adapter` to see local traffic.

Here is a screenshot of the Wireshark capture where I diagnosed the client certificate security issue:

{% asset_img wireshark-trace.png %}

## Good luck!

Sitecore 9 introduces a lot of new competencies that are required to be an effective developer with it. Certificates are pretty easy to mess up - a single bad character in a thumbprint, or an untrusted certificate and everything can break in a hurry. Hopefully this post will save you some of the headaches I had getting myself started.