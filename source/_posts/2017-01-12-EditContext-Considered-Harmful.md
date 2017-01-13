title: EditContext Considered Harmful
date: 2017-01-12 17:52:42
categories: Sitecore
---

If you've worked with Sitecore for very long, you have probably needed to update an item's content programmatically. A pattern in common usage for this is to use the `EditContext` class, for example:

	Item item;
	using(new EditContext(item))
	{
		item["FieldName"] = "A new value";
	}

Unfortunately, it has a fatal flaw. A fatal flaw that Jakob Christensen [pointed out _in 2006_](http://sitecore.alexiasoft.nl/2006/04/03/new-way-of-editing-items/#comment-16). Unfortunately it's still in wide use, and is not marked as obsolete. Hopefully this post can help change that!

The problem lies in the way the `using` statement works. It is effectively equivalent to this:

	var context = new EditContext(item);
	try
	{
		item["FieldName"] = "A new value";
	}
	finally
	{
		context.Dispose();
	}

An astute observer will notice that due to the `finally` semantic, the `EditContext` is _always disposed_, regardless of whether an exception occurs. Take this code for example:

	Item item;
	using(new EditContext(item))
	{
		item["FieldA"] = "Sample";
		item["FieldB"] = GetFieldValue();
		item["FieldC"] = "A new value";
	}

	public string GetFieldValue() 
	{
		throw new ArgumentException("Let's address the elephant in the room.");
	}

In this example `GetFieldValue()` throws an exception. This will cause an immediate drop through to the `finally` of the `using` statement, which executes whether an exception is thrown or not. Which will _commit_ the partial item change to `FieldA` without `FieldB` or `FieldC`, leaving the Sitecore item in an inconsistent state. Which you probably do not want.

## So what _should_ we use?

The correct way to edit an item is with the `Editing` property, for example:

	Item item;
	item.Editing.BeginEdit();
	item["FieldA"] = "Sample";
	item["FieldB"] = GetFieldValue();
	item["FieldC"] = "A new value";
	item.Editing.EndEdit();

Note how `EndEdit()` will _not_ be called if an exception occurs. This prevents Sitecore from writing any field changes to the database (changes are queued up and committed all together when `EndEdit()` is called).

This syntax is however a bit hard to read, because it is difficult to parse the code compared to the neat indentation of `EditContext`. I would like to propose an alternative syntax, taking advantage of the fact that you can use _arbitrary braces_ in C#:

	Item item;

	item.Editing.BeginEdit();
	{
		item["FieldA"] = "Sample";
		item["FieldB"] = GetFieldValue();
		item["FieldC"] = "A new value";
	}
	item.Editing.EndEdit();

There! Now it reads nearly like `EditContext`, but does not suffer from its downsides :)