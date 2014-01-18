---
layout: post
title: "Making &lt;asp:TextBox&gt; do HTML5 input types"
date: 2010-07-09 10:20:27 -0800
comments: true
categories: .NET
---
I was watching a presentation on HTML5 from one of my coworkers a few weeks ago and we were talking about the new input types in HTML5, like numbers and dates. They're backward compatible with non-HTML5 browsers (which render them as text boxes), but provide very useful UI features particularly to smartphones where the on screen keyboard can change to be more appropriate.
<br>
Anyway, let's just say they're a good idea. A good idea that, much as I like ASP.NET, are not likely to make an appearance any time soon in the core framework. So, I looked around to see if I could figure out a way to patch a standard <asp:TextBox> to render these new HTML5 types. Yes, you MVC types already can do this easily. For the rest of us that are using third party CMS tools, WebForms is still the way of life :)
<br>
I came across a <a href="http://haacked.com/archive/2006/01/18/UsingaDecoratortoHookIntoAWebControlsRenderingforBetterXHTMLCompliance.aspx">post</a> by Phil Haack about a sneaky method of overriding HtmlTextWriter to change the attributes output by WebControls, and repurposed it to accomplish the HTML5-ization. Basically you override the AddAttribute() method and make it ignore the attribute you want to manually handle - in this case, the type attribute of the TextBox. Then I made a new TextMode-style enum that encompasses the HTML5 types and wrote a bit of logic that manually creates the right type attribute. The new enum is exposed in the Html5TextMode property, and the value is proxied back into the default TextMode property if it's compatible with it.
<br>
So how do you use it? It's pretty simple:

	<prefix:Html5TextBox runat="server" Html5TextMode="Tel" ID="telephone" />
	
And the code for the control itself:

	public class Html5TextBox : TextBox
	{
	 /// <summary>
	 /// When using non-HTML5 constructs this mode will be accurate. If using HTML5, it will return SingleLine.
	 /// </summary>
	 public override TextBoxMode TextMode
	 {
	  get
	  {
	   try
	   {
		return (TextBoxMode)Enum.Parse(typeof(TextBoxMode), Html5TextMode.ToString());
	   }
	   catch(ArgumentException) { return TextBoxMode.SingleLine; }
	  }
	  set
	  {
	   Html5TextMode = (Html5TextBoxMode)Enum.Parse(typeof(Html5TextBoxMode), value.ToString());
	  }
	 }

	 /// <summary>
	 /// Sets the text mode of the control including HTML5 text modes such as Num and DateTime
	 /// </summary>
	 public Html5TextBoxMode Html5TextMode
	 {
	  get
	  {
	   object textMode = this.ViewState["5Mode"];
	   if (textMode != null)
	   {
		return (Html5TextBoxMode)textMode;
	   }
	   return Html5TextBoxMode.SingleLine;
	  }
	  set
	  {
	   this.ViewState["5Mode"] = value;
	  }
	 }

	 /// <remarks>
	 /// Adds the normal attributes (since the writer is actually a PatchedHtmlTextWriter from Render() the type attribute won't be rendered),
	 /// then explicitly adds the appropriate type attribute including the HTML5 extensions
	 /// </remarks>
	 protected override void AddAttributesToRender(HtmlTextWriter writer)
	 {
	  base.AddAttributesToRender(writer);

	  string type = null;
	  if (Html5TextMode == Html5TextBoxMode.SingleLine)
	   type = "text";
	  else if (Html5TextMode == Html5TextBoxMode.DateTimeLocal)
	   type = "datetime-local";
	  else type = Html5TextMode.ToString().ToLowerInvariant();

	  if (type != null)
	   (writer as PatchedHtmlTextWriter).AddTypeAttribute(type);
	 }

	 /// <remarks>
	 /// Patches the type of HtmlTextWriter that the control renders to
	 /// </remarks>
	 protected override void Render(HtmlTextWriter writer)
	 {
	  base.Render(new PatchedHtmlTextWriter(writer));
	 }

	 /// <summary>
	 /// A version of HtmlTextWriter that intentionally ignores the "type" attribute when it is added.
	 /// </summary>
	 /// <remarks>
	 /// Technique courtesy of Phil Haack
	 /// http://haacked.com/archive/2006/01/18/UsingaDecoratortoHookIntoAWebControlsRenderingforBetterXHTMLCompliance.aspx
	 /// </remarks>
	 private class PatchedHtmlTextWriter : HtmlTextWriter
	 {
	  internal PatchedHtmlTextWriter(HtmlTextWriter basis) : base(basis) { }

	  public override void AddAttribute(HtmlTextWriterAttribute key, string value)
	  {
	   if(key != HtmlTextWriterAttribute.Type)
		base.AddAttribute(key, value);
	  }

	  public override void AddAttribute(string name, string value)
	  {
	   if(name != "type")
		base.AddAttribute(name, value);
	  }

	  /// <summary>
	  /// Manually adds a type attribute
	  /// </summary>
	  public void AddTypeAttribute(string value)
	  {
	   base.AddAttribute(HtmlTextWriterAttribute.Type, value);
	  }
	 }
	}

	/// <summary>
	/// Extends the TextMode enum with additional HTML5 types
	/// </summary>
	public enum Html5TextBoxMode { MultiLine, SingleLine, Password, DateTime, DateTimeLocal, Date, Month, Time, Week, Number, Range, Email, Url, Search, Tel, Color }