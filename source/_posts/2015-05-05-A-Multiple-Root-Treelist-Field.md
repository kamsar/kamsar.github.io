title: "A Multiple Root Treelist Field"
date: 2015-05-05 17:00:27
categories: Sitecore
---
Recently I came across a need to allow selecting items for a TreeList field from several locations - specifically, we needed to be able to choose images OR videos to assign, and those were in different places in the content tree.

This has [been done before](http://optimizedquery.com/2013/02/22/sitecore-field-multi-root-treelist/), but there was altogether too much copied private method in that solution for my taste - and that it appears to not work with Sitecore 7.5 sealed the deal. To the hackmobile!

Turned out to not be too hard to reduce it to one file: Sitecore itself provides a `MultiRootTreeList` control that is used for rendering the insert data source item dialog (which supports multiple roots). With a little creative manipulation of the control tree, here we go:

{% asset_img tree.png "Multi root content tree" %}

And, the code ([also as a Gist](https://gist.github.com/kamsar/33d1245ffdb630b1f126)) - tested on Sitecore 7.2 Update-3 and Sitecore 8 Update-2:

	using System;
	using System.Linq;
	using Sitecore.Diagnostics;
	using Sitecore.Globalization;
	using Sitecore.Shell.Applications.ContentEditor;
	using Sitecore.Text;
	using Sitecore.Web;
	using Sitecore.Web.UI.HtmlControls;
	using Sitecore.Web.UI.WebControls;

	namespace Kamsar.FieldTypes
	{
		/// <summary>
		/// This field type is like a tree list, but you can specify more than one root item to select from (for example, videos or photos)
		/// The data source roots are specified using pipe delimiting just like regular Sitecore Query language
		/// </summary>
		public class MultiRootTreeList : TreeList
		{
			protected override void OnLoad(EventArgs args)
			{
				Assert.ArgumentNotNull(args, "args");
				base.OnLoad(args);

				if (!Sitecore.Context.ClientPage.IsEvent)
				{
					// find the existing TreeviewEx that the base OnLoad added, get a ref to its parent, and remove it from controls
					var existingTreeView = (TreeviewEx)WebUtil.FindControlOfType(this, typeof(TreeviewEx));
					var treeviewParent = existingTreeView.Parent;

					existingTreeView.Parent.Controls.Clear(); // remove stock treeviewex, we replace with multiroot

					// find the existing DataContext that the base OnLoad added, get a ref to its parent, and remove it from controls
					var dataContext = (DataContext)WebUtil.FindControlOfType(this, typeof(DataContext));
					var dataContextParent = dataContext.Parent;

					dataContextParent.Controls.Remove(dataContext); // remove stock datacontext, we parse our own

					// create our MultiRootTreeview to replace the TreeviewEx
					var impostor = new Sitecore.Web.UI.WebControls.MultiRootTreeview();
					impostor.ID = existingTreeView.ID;
					impostor.DblClick = existingTreeView.DblClick;
					impostor.Enabled = existingTreeView.Enabled;
					impostor.DisplayFieldName = existingTreeView.DisplayFieldName;

					// parse the data source and create appropriate data contexts out of it
					var dataContexts = ParseDataContexts(dataContext);

					impostor.DataContext = string.Join("|", dataContexts.Select(x => x.ID));
					foreach(var context in dataContexts) dataContextParent.Controls.Add(context);

					// inject our replaced control where the TreeviewEx originally was
					treeviewParent.Controls.Add(impostor);
				}
			}

			/// <summary>
			/// Parses multiple source roots into discrete data context controls (e.g. 'dataSource=/sitecore/content|/sitecore/media library')
			/// </summary>
			/// <param name="originalDataContext">The original data context the base control generated. We reuse some of its property values.</param>
			/// <returns></returns>
			protected virtual DataContext[] ParseDataContexts(DataContext originalDataContext)
			{
				return new ListString(DataSource).Select(x => CreateDataContext(originalDataContext, x)).ToArray();
			}

			/// <summary>
			/// Creates a DataContext control for a given Sitecore path data source
			/// </summary>
			protected virtual DataContext CreateDataContext(DataContext baseDataContext, string dataSource)
			{
				DataContext dataContext = new DataContext();
				dataContext.ID = GetUniqueID("D");
				dataContext.Filter = baseDataContext.Filter;
				dataContext.DataViewName = "Master";
				if (!string.IsNullOrEmpty(DatabaseName))
				{
					dataContext.Parameters = "databasename=" + DatabaseName;
				}
				dataContext.Root = dataSource;
				dataContext.Language = Language.Parse(ItemLanguage);

				return dataContext;
			}
		}
	}

Have fun!