title: Become a Sitecore admin without a login
date: 2016-10-24 15:48:51
categories: Sitecore
---

Suppose someone sends you a Sitecore solution to review, and they forgot to send you a username and password. You could ask for one, or you could just make yourself an account with this handy trick that I call "The Shiv." This is the entirety of the trick:

1. Create a `Shiv.aspx` file somewhere under the webroot. It can be named anything.
2. Paste the following code in it

		<%@ Page Language="C#" AutoEventWireup="true" %>
		<%@ Import Namespace="Sitecore.Security.Authentication" %>

		<%
			AuthenticationManager.Login("sitecore\\admin", false, true);
			Response.Redirect("/sitecore/shell");
		%>

3. Hit the page in a browser
4. Boom, you're an administrator
5. **IMPORTANT**: Delete the file. For obvious reasons.

Handy, eh?

You can do a very similar thing using the "Login as administrator" option in [SIM](http://dl.sitecore.net/updater/sim/), however I often find myself in environments without SIM and this code works anywhere.

<iframe src="//giphy.com/embed/Njjakmj3DC7pS" width="480" height="270" frameBorder="0" class="giphy-embed" allowFullScreen></iframe>

This code is also a good security reminder: if someone malicious can upload an arbitrary file somewhere in your webroot that is then executed, _they can upload this shiv-file_ and your security is gone. It doesn't matter if you have encrypted 64-character database passwords, they're in. It doesn't matter if you've locked down TLS and imposed SAML logins, they're in. Game over. So [secure your filesystem](https://doc.sitecore.net/sitecore_experience_platform/setting_up__maintaining/security_hardening/configuring/secure_the_file_upload_functionality) and be awfully wary of accepting users' uploads anywhere on disk.