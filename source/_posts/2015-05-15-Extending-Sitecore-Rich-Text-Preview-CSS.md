title: "Extending Sitecore Rich Text Preview CSS"
date: 2015-05-15 11:44:16
categories: Sitecore
---

The [other Sitecore Kam](https://jammykam.wordpress.com) wrote an excellent post titled [User Specific or Multi Site Specific CSS styles in Sitecore Rich Text Editor](https://jammykam.wordpress.com/2015/03/11/user-specific-or-multi-site-specific-css-styles-in-sitecore-rich-text-editor/) a few months ago. I noticed one gap in what it did, however: changing the RTE configuration does not change the _preview_ of the rich text shown in the Sitecore content editor interface.

This means that you'd see the site specific styles when in the RTE proper, but not when browsing the item in the content editor.

So here's how to accomplish just that. The RTE preview is rendered by the `sitecore/shell/Controls/Rich Text Editor/Preview.aspx` page. Being a normal Web Form page, we can hack its `Inherits` attribute to something else in order to inject our custom CSS:

	<%@ Page ... Inherits="MyLibrary.MultisiteRichTextEditorPreview, MyLibraryAssembly" %>

Then we have to create a class which inherits the default codebehind class, `Sitecore.Shell.Controls.RADEditor.Preview`:

	using System;
	using System.Web.UI;
	using Sitecore.Shell.Controls.RADEditor;

	namespace MyLibrary
	{
		public class MultisiteRichTextEditorPreview : Preview
		{
			protected override void OnLoad(EventArgs e)
			{
				base.OnLoad(e);

				// replace the method call here with whatever you're using to resolve a custom RTE CSS path(s)
				// note that the query string parameter 'id' is the context item ID here just like in the EditorConfiguration Kamruz blogged about.
				var stylesheetPath = MultisiteRichTextEditorUtility.GetRichTextEditorCssPath();

				if (stylesheetPath != null) Stylesheets.Controls.Add(new LiteralControl(string.Format("<link href=\"{0}\" rel=\"stylesheet\">", stylesheetPath)));
			}
		}
	}

Finally, we need to 'improve' our RTE stylesheet (this is optional, but excellent!):

	* {
		font-family: "Comic Sans MS" !important;
		color: pink !important;
	}

And then we have our newly enhanced rich text preview:

{% asset_img comical.png "HAHAHAHAHA" %}