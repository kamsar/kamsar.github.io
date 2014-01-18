---
layout: post
title: "Sitecore 7 LINQ gotchas"
date: 2013-04-15 12:59:41 -0800
comments: true
categories: Sitecore Lucene
---
The upcoming Sitecore 7 release brings with it a new "LINQ-to-provider" architecture that effectively allows you to query search indexes using standard LINQ syntax. <a href="http://markstiles.net/Blog/2013/04/05/linq-to-sitecore.aspx" target="_blank">Mark Stiles wrote a pretty good synopsis of the feature</a> that you should probably read first if you're unfamiliar with the basics of how it works. This post won't cover the basics.<br><br>
I've been diving in to the underpinnings of the LINQ architecture and have discovered a number of things that may well cause confusion when people start using this technology. Be warned that this post is based on a non-final release of Sitecore 7, and may well contain technical inaccuracies compared to the final release.<br><h2>
You have to enable field storage to get output</h2>

By default, the values of fields are not stored in the index. If the values are not stored, you can query against the index using custom object mapping (e.g. filters work), but you will not see any field values mapped into results objects. You can define field storage parameters either on a per-field basis, or a per-field-type (e.g. Single-Line Text) in the default index configuration file (in App_Config/Include).

Changes to the storage type require an index rebuild before the storage is reflected.

## LINQ is not C Sharp
Yeah, you heard me right. LINQ may look exactly like C#, but it is not parsed the same way. The lambda expressions you may use to construct queries against Sitecore are compiled into an Abstract Syntax Tree (AST) - sort of a programmatic representation of the code forming the lambda - and that is in turn parsed by the Sitecore LINQ provider.<br><br><i>The code you write into lambdas is not executed as normal C# code is.</i> This is important to remember, because effectively the LINQ provider is simply mapping your query in as simple terms as possible to a key-value query inside of Lucene (or SOLR, etc). For example:<br><br>

	// we'll use this as a complex property type to map into a lucene object<br>
	public class FieldType {
	    public string Value { get; set; }
	    public string SomeOtherValue { get; set; }
	}
	// this will be the class we'll query on in LINQ
	public class MappedClass {
	    [IndexField("field1")]
	    public FieldType Field1 { get; set; }
	}

	// example queries (abbreviated code)
	var query1 = context.GetQueryable<mappedclass>().Where(x=>x.Field1.Value == "foo");
	var query2 = context.GetQueryable<mappedclass>().Where(x=>x.Field1.SomeOtherValue == "foo");

You'd expect query1 and query2 to be different wouldn't you? <b>NOPE.</b> You're not writing C# here, you're writing an AST in C# syntax. The Sitecore LINQ engine takes the shortest path to a key-value query pair. What this really means is that it:

<ol><li>Resolves the Lucene field name of the field you're querying on (in this case, "field1")</li>
<li>Evaluates the operator you used, and the constant value you've assigned to compare to</li>
<li>Constructs a Lucene query expression based on that</li>
</ol>

In effect, you can only ever have one value queried for each property. In the example above both examples are in effect `x.Field1 == "foo"`. The query would be that way even if you did a crazy query like `x.Field1.Foo.Bar.Baz.Boink.StartsWith("foo")` - that would boil down to `x.Field1.StartsWith("foo")`.

There is a facility you can tie into that controls how Sitecore converts types in queries (TypeConverters). Unfortunately, that does not solve the problem of disambiguating properties - the conversion only informs the engine how to convert the constant portion of the query (in this case, the string.

## Sitecore LINQ does not care if the return entity type is mapped to a valid object for it

If you execute a query against a type, say the MappedClass type in the previous example, the LINQ layer will map all results of the query against the MappedClass type. Sounds great, but be careful - it will also map results that may not have the expected template to map to the MappedClass type onto it.

For example, suppose I made a model for the Sample Item template that comes with Sitecore. Then I queried the whole database as SampleItem. Out of my results, probably only two are really designed to be mapped to my SampleItem - the rest will have nulls everywhere. This is potentially problematic if you forget to be specific in your queries to limit the template ID to the expected one.

## You must enumerate the results before the SearchContext is disposed

If you've ever dealt with NHibernate or other lazy-loaded ORMs, this might make perfect sense to you. A typical search query method might look something like this:

	public IEnumerable<MappedClass> Foo()
	{
	    var index = SearchManager.GetIndex("sitecore_master_index");
	    using (var context = index.CreateSearchContext(SearchSecurityOptions.EnableSecurityCheck)) {
			return context.GetQueryable<mappedclass>().Where(x=>x.Name == "foo");
	    }
	}

Can you guess the problem? `IEnumerable` doesn't actually execute until you enumerate the values (e.g. run it through foreach, .ToArray(), etc). If you return the `IEnumerable<mappedclass>`, it cannot be enumerated until the SearchContext has already been disposed of. Which means that will throw an exception!

To avoid this problem you need to either:

<ul><li>Return a pre-enumerated object. Usually the simplest form of this is simply to return the query with .ToArray() at the end, thus enumerating the query into an array before the context is out of scope. <i>Warning: This also means that you should filter as far as possible within the query, including paging, especially with large result sets.</i></li>
<li>Execute all the code within the context's scope. This is probably less desirable as it either means spaghetti code in your codebehind, or a request-level context manager like NHibernate sometimes does.</li>
</ul>

## The object mapper uses cached reflection to set each mapped property

Yes, it uses "the R word." Reflection is great, but it's not all that fast. This is fine if you're running optimized code: you'll want to perform pagination and filtering within Lucene, and avoid returning large quantities of mapped objects. The performance is dependent on both the number of objects and the number of properties you have on each object as each property results in a reflection property set.

I would suggest trying to keep the number of mapped objects returned under 100 in most cases, and/or using output caching to minimize the effect of reflection.

It's also possible to patch the way the mapping code works (I have a reflection-less proof of concept that in relatively unscientific tests is about 50-100x faster than the reflection method. It enforces some code generation requirements however and is not as general purpose or as simple to use as the reflection method.

## It's still awesome
While this post has largely focused on unexpected gotchas around the Sitecore LINQ provider, it's actually pretty darn nice once you get used to its quirks. It's certainly loads easier to use than any previous Lucene integration, and the backend extensibility allows for all sorts of interesting extensions and value storage.