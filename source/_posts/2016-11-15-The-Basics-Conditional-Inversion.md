title: 'The Basics: Conditional Inversion'
date: 2016-11-15 19:11:10
categories: C#
---

Conditional inversion is a simple but powerful technique that makes your code easier to read. I've noticed that not a lot of people in the Sitecore community seem to know about it, so I thought I'd blog about it.

## Inverting your `if`s

Deeply nested code is pretty hard to read. Let's take this example:

	public class Class1
	{
		private static object Lock = new object();
		private static string Value = false;

		public string GetValue()
		{
			if (Value == null)
			{
				lock (Lock)
				{
					if (Value == null)
					{
						Value = ExpensiveCreateValueMethod();
					}
				}
			}

			return Value;
		}
	}

This is a very common pattern in multithreaded code, the double-check lock. It ensures that the Value is not initialized more than once at the same time by different threads. It's also fairly hard to follow the code: it has a lot of nested blocks, and the information density is not very high.

Let's take a different approach to the same code.

{% asset_img invert.jpg %}

Let's invert our `if` statements so we can _get out of our method as quickly as possible_.

	public class Class1
	{
		private static object Lock = new object();
		private static string Value = false;

		public string GetValue()
		{
			if (Value != null) return Value;
			
			lock (Lock)
			{
				if (Value != null) return Value;
				
				return Value = ExpensiveCreateValueMethod();
			}
		}
	}

With inverted `if` statements we check not for success but for failure. This method has two fewer nesting levels, is four lines shorter, and is significantly more readable because you don't have to traverse a long `if` block to see what code will be executed next on success. Imagine outside a contrived example like this one, where your `if` might be 50 lines long.

So next time you're writing conditionals, think about handling the failure before the success. Instead of:

	public string Foo(string bar) 
	{
		if(bar != null) {
			// do
			// stuff
			return something;
		}

		return null;
	}

Do this:

	public string Foo(string bar) 
	{
		if(bar == null) return null;

		// do
		// stuff
		return something;
	}

Nicer, right?

## Feeling loopy?

{% asset_image loop.gif %}

The same inversion technique is also lovely for loop control. Have you ever written a loop like this?

	foreach (var foo in bar)
	{
		if (foo != null)
		{
			// do
			// stuff
		}
	}

When inverting loop control you make use of the `continue` statement. This is an underused gem that skips the current loop iteration just like `return` skips the rest of a method. We can rewrite the loop above for better readability like so:

	foreach (var foo in bar)
	{
		if (foo == null) continue;
		
		// do
		// stuff
	}

Now if `foo` is null, we just skip the rest of the loop body and go to the next item. Handy, right?

## Bonus points: asserting reference types

A special case of this kind of inversion is the use of the `Sitecore.Diagnostics.Assert` class. When you're designing a method, any reference type parameters to that method _are allowed to be_ `null`. I'll bet you didn't expect someone to pass `null`, right? (I'll bet I didn't either!)

	public void DoStuff()
	{
		// null is usually not overtly passed, e.g. looping over dynamic data
		var stuffFromRestService = new[] { "foo", null, "bar" };
		foreach (var attribute in stuffFromRestService)
		{
			DoAThing(attribute);
		}
	}

	public void DoAThing(string attribute)
	{
		// boom
		attribute.ToLowerInvariant();
	}

Then you get this most favourite error, with a 'helpful' stack trace in the middle of the offending method:
{% asset_image ysod.png %}

Not helpful, right? Well we can handle this more gracefully if we _assert_ that our parameters that are reference types are not null - then we get an exception at the top of the method that is useful and tells us what really happened:

	// using Sitecore.Diagnostics;
	public void DoAThing(string attribute) 
	{
		Assert.ArgumentNotNull(attribute, nameof(attribute));

		// do stuff
		attribute.ToLowerInvariant();
	}

And we get a better error message that tells us the argument name and what happened:
{% asset_img arg.png %}

Pretty handy, right? Now go forth and code!