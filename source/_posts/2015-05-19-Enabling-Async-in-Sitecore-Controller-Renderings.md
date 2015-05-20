title: "Enabling Async in Sitecore Controller Renderings"
date: 2015-05-19 21:26:05
categories: Sitecore
---

### IMPORTANT NOTE:
The information in this post unfortunately does NOT work for controller renderings as I had originally thought. It does, however enable you to use async actions outside of Sitecore contexts (e.g. in directly invoked controllers via non-Sitecore routes or Html.RenderAction() on a Sitecore rendering)



For the past few years, the rest of the .NET world outside Sitecore has been adopting the new `async/await` programming model that was introduced in .NET 4.5. This model essentially allows you to free up your worker thread while you're waiting for something else to happen to continue exection. WTF does that mean?

Ok, so imagine you have a very simple web server that has 20 execution threads. This more or less means you can process 20 HTTP requests at the same time before you have to start queuing incoming requests - which kills performance.

Now suppose you're making an expensive database call that takes 5 seconds on each request, and you get 40 requests at once. Without `async`, you have no choice but to send 20 requests into the queue. But with `async`, you can `await` the expensive database call - which frees up your execution thread(s) for someone else to use for actual processing while you wait for the database. So with an async database call and your 40 requests, you'd start processing 20 requests and rapidly start awaiting the database. This would enable the web server to start processing the other 20 queued requests instead of taking up execution threads that are doing nothing but wait! Then imagine if the latter 20 requests needed CPU time instead of database time - the web server CPUs would remain much more loaded over time and get more work done using the same resources.

Sounds good, right? But one of the down-sides of async is that you really need async calls from top to bottom to avoid deadlocks in ASP.NET. In other words, you could write your SQL API or REST client using async APIs all day - but unless the controller you invoke the APIs from is ALSO async, you'll either get a deadlock or very little performance improvement. Unfortunately, Sitecore (as of 8.0 Update-2 anyway) does not support async controller actions. I didn't like this, so I set out to fix that.

Ironically, the problem was very simple and rather hilarious. Sitecore MVC uses its own `ControllerFactory` which essentially wraps the default `IControllerFactory` from ASP.NET MVC and decorates it with some Sitecore-specific stuff. One of the things it does is look for the `IActionInvoker` on each controller, and if one exists wraps that implementation similarly with `SitecoreActionInvoker` to alter the way controller actions are invoked.

Looking around in the ASP.NET MVC source code, you can see that there are two interfaces for action invokers: `IActionInvoker` and `IAsyncActionInvoker`. Guess what? Sitecore's ActionInvoker implements `IActionInvoker` but NOT `IAsyncActionInvoker` - which means that even though the default MVC action invoker it was wrapping supported async invocation, the wrapper's lack of `IAsyncActionInvoker` implementation meant ASP.NET MVC wouldn't use async invocation at all - and would instead throw an error that you cannot return a `Task<ActionResult>` from a synchronous controller. What?!

It's a simple matter to patch Sitecore's `ControllerFactory`, which is registered in the `<initialize>` pipeline:

	<initialize>
		<processor type="Foo.Pipelines.Initialize.InitializeAsyncControllerFactory, Foo.Core" patch:instead="*[@type='Sitecore.Mvc.Pipelines.Loader.InitializeControllerFactory, Sitecore.Mvc']"/>
	</initialize>

	using Sitecore.Mvc.Controllers;
	using Sitecore.Mvc.Pipelines.Loader;
	using Sitecore.Pipelines;
	using ControllerBuilder = System.Web.Mvc.ControllerBuilder;

The class to override the controller factory:

	namespace Foo.Pipelines.Initialize
	{
		/// <summary>
		/// Replaces the standard Sitecore MVC controller factory with one that knows how to do async action invocation.
		/// </summary>
		public class InitializeAsyncControllerFactory : InitializeControllerFactory
		{
			protected override void SetControllerFactory(PipelineArgs args)
			{
				SitecoreControllerFactory controllerFactory = new SitecoreAsyncControllerFactory(ControllerBuilder.Current.GetControllerFactory());
				ControllerBuilder.Current.SetControllerFactory(controllerFactory);
			}
		}
	}

Then override the way the `SitecoreControllerFactory` patches the `IActionInvoker` on the controllers:

	using System.Web.Mvc;
	using System.Web.Mvc.Async;
	using Sitecore.Mvc.Configuration;
	using Sitecore.Mvc.Controllers;

	namespace Foo
	{
		/// <summary>
		/// Patches the normal Sitecore controller factory to enable executing async actions and using async/await
		/// The ActionInvoker that Sitecore MVC wraps the inner action invoker with does not implement IAsyncActionInvoker,
		/// which means ASP.NET MVC does not try to execute it async if needed, and precludes async/await.
		/// </summary>
		public class SitecoreAsyncControllerFactory : SitecoreControllerFactory
		{
			public SitecoreAsyncControllerFactory(IControllerFactory innerFactory) : base(innerFactory)
			{
			}

			protected override void PrepareController(IController controller, string controllerName)
			{
				if (!MvcSettings.DetailedErrorOnMissingAction)
				{
					return;
				}
				Controller controller2 = controller as Controller;
				if (controller2 == null)
				{
					return;
				}

				/* BEGIN PATCH FOR ASYNC INVOCATION (the rest of this method is stock) */
				IAsyncActionInvoker asyncInvoker = controller2.ActionInvoker as IAsyncActionInvoker;

				if (asyncInvoker != null)
				{
					controller2.ActionInvoker = new SitecoreAsyncActionInvoker(asyncInvoker, controllerName);
					return;
				}
				/* END PATCH FOR ASYNC INVOCATION */

				IActionInvoker actionInvoker = controller2.ActionInvoker;
				if (actionInvoker == null)
				{
					return;
				}
				controller2.ActionInvoker = new SitecoreActionInvoker(actionInvoker, controllerName);
			}
		}
	}

And finally, implement an override of `SitecoreActionInvoker` which implements `IAsyncActionInvoker`:

	using System;
	using System.Web.Mvc;
	using System.Web.Mvc.Async;
	using Sitecore.Mvc.Controllers;

	namespace Foo
	{
		/// <summary>
		/// Literally all this does is provider an IAsyncActionInvoker wrapper the same way SitecoreActionInvoker wraps non-IAsyncActionInvokers
		/// This instructs ASP.NET MVC to perform async invocation for controller actions.
		/// </summary>
		public class SitecoreAsyncActionInvoker : SitecoreActionInvoker, IAsyncActionInvoker
		{
			private readonly IAsyncActionInvoker _innerInvoker;

			public SitecoreAsyncActionInvoker(IAsyncActionInvoker innerInvoker, string controllerName) : base(innerInvoker, controllerName)
			{
				_innerInvoker = innerInvoker;
			}

			public IAsyncResult BeginInvokeAction(ControllerContext controllerContext, string actionName, AsyncCallback callback, object state)
			{
				return _innerInvoker.BeginInvokeAction(controllerContext, actionName, callback, state);
			}

			public bool EndInvokeAction(IAsyncResult asyncResult)
			{
				return _innerInvoker.EndInvokeAction(asyncResult);
			}
		}
	}

Then you can go forth and write lovely async controller renderings!

	public async Task<ActionResult> AsyncActionMethod()
	{
		// download a bunch of URLs in parallel with await
		var webClient = new WebClient();

		var urls = new[] {
			"https://google.com",
			"https://bing.com",
			"https://yahoo.com"
		}.Select(url => webClient.DownloadStringTaskAsync(url));

		var contents = await Task.WhenAll(urls);

		// or just await one task
		var google = await webClient.DownloadStringTaskAsync("https://google.com");

		// execution will pick up right here when all the awaited tasks are done - thawing the thread to finish execution

		return View(contents);
	}

Have fun!