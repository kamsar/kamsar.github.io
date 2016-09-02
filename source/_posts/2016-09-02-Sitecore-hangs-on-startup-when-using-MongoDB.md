title: Sitecore hangs on startup when using MongoDB
date: 2016-09-02 10:52:25
categories: Sitecore
---

I ran across a tricky issue today where a Sitecore content delivery cluster would simply hang on startup. No errors in the logs. Nothing in the windows event logs. The content editing server worked just fine.

Mystifying, right? So it came down to process of elimination: I disabled analytics and removed all Mongo connection strings and the site worked. The obvious answer here would be "it's a firewall issue."

It wasn't. `mongo.exe` could connect to the replica set just fine. The connection was using SSL, so I checked the server certificate. It was valid and trusted. 

Eventually it came down to a few important facts:

* We were using SSL.
* Because we weren't using client certificates, `mongo.exe` had to use `--sslAllowInvalidCertificates`, which meant that it was not verifying the server certificate.
* The Mongo C# driver _does_ validate the server certificate.

But you said the server certificate was totally valid, right? Right - and it was valid.

Except for the sneaky fact that _the certificate had a Certificate Revocation List (CRL) URL embedded in it_. The Mongo C# driver defaults to checking the CRL to make sure the server certificate is not revoked.

Normally that's a good thing, but in this case _the CRL was behind the firewall, and the delivery servers were not_. So the CRL could not be checked, and the server certificate was treated as invalid due to that.

Why this results in straight up hanging the site instead of an error I'm not sure. But that leads to how it got fixed.

## Extending the Mongo Client Settings

The most obvious fix would be to add something to the connection string to disable the CRL check, as it would be undesirable to expose the PKI server that issued the certificate to the DMZ where the delivery servers live. Unfortunately, there is no option to do that in the Mongo connection string format.

Fortunately, There's A Micro-Pipeline For That™. Sitecore exposes the `updateMongoDriverSettings` pipeline, which is empty by default. This pipeline allows you to fool with the [MongoClientSettings](http://api.mongodb.com/csharp/current/html/T_MongoDB_Driver_MongoClientSettings.htm) object and alter configurations that are not available in the connection string.

Here's an example patch to add a processor to said pipeline:

	<configuration xmlns:patch="http://www.sitecore.net/xmlconfig/">
		<sitecore>
			<pipelines>
				<updateMongoDriverSettings>
					<processor type="Me.Pipelines.UpdateMongoDriverSettings.DisableCrlChecks, Me"/>
				</updateMongoDriverSettings>
			</pipelines>
		</sitecore>
	</configuration>

And the processor implementation that alters the `MongoClientSettings` to disable CRL checks:

	using MongoDB.Driver;
	using Sitecore.Analytics.Pipelines.UpdateMongoDriverSettings;

	namespace Me.Pipelines.UpdateMongoDriverSettings
	{
		public class DisableCrlChecks : UpdateMongoDriverSettingsProcessor
		{
			public override void UpdateSettings(UpdateMongoDriverSettingsArgs args)
			{
				args.MongoSettings.SslSettings = new SslSettings { CheckCertificateRevocation = false };
			}
		}
	}

This technique could also be used to diagnose other SSL issues that might involve _invalid_ certificates, because you can also [alter the certificate validity handling](https://stackoverflow.com/a/34735350/1359221) to get diagnostic output.

## Caveat: Sorry, not for you Mongo Session Provider

{% asset_img vader.jpg %} 

Unfortunately if you're using the MongoDB Session State provider, it does not respect this pipeline. In fact, it makes itself highly extensible, by hiding the place to put your options in a private static method! Called by internal methods! 

	private static MongoCollection GetCollection(string connectionString, string databaseName, string collectionName)
	{
		return (MongoCollection) new MongoClient(new MongoUrl(connectionString)).GetServer().GetDatabase(databaseName).GetCollection(collectionName);
	}

So you're hosed if you're using Mongo sessions.