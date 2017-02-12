title: 'Unicorn 4 Preview Part 2.5: Generating Unicorn Packages with SPE'
date: 2017-02-11 21:09:02
categories: Unicorn
---

[Last time](/index.php/2017/02/Unicorn-4-Preview-Part-2-SPE-Support/) we talked about how Sitecore PowerShell Extensions support was coming to Unicorn 4. This time, we've got a new cmdlet to share.

Over time, many people have asked if there was a way to generate Sitecore packages from Unicorn. The answer has always been no, for many good reasons: packages install slowly, cannot ignore specific fields, or process advanced exclusions like a Unicorn predicate can. This makes them much less safe (and much slower) for deployment purposes compared to a remotely invoked Sync using deployed serialized items.

But there is a great use case for generating packages from Unicorn: authoring modules. As a module author, a method is needed to track the items that belong to your module and also to reliably create Sitecore packages for distribution of your module which contain those items. Unicorn is a natural fit for simply tracking module items, but it has lacked the ability to automatically push updates to release packages like it can to serialized items. This unnecessarily complicates things and reduces release reliability. That's bad.

So when [Michael West](https://twitter.com/MichaelWest101) and [Adam Najmanowicz](https://twitter.com/adamnaj), the authors of Sitecore PowerShell Extensions, asked if there was a way we could export Unicorn configurations to packages my answer was _absolutely._

SPE has long had [packaging support](https://doc.sitecorepowershell.com/appendix/commands/Export-Package.html#examples) built into it, and in fact SPE's release packages are built using SPE. Unicorn packaging support is also implemented through SPE, and here's how it works:

```powershell
# Create a new Sitecore Package (SPE cmdlet)
$pkg = New-Package

# Get the Unicorn Configuration(s) we want to package
$configs = Get-UnicornConfiguration "Foundation.*" 

# Pipe the configs into New-UnicornItemSource 
# to process them and add them to the package project
# (without -Project, this would emit the source object(s) 
#   which can be manually added with $pkg.Sources.Add())
$configs | New-UnicornItemSource -Project $pkg

# Export the package to a zip file on disk
Export-Package -Project $pkg -Path "C:\foo.zip"

```

And when you're done with that, `c:\foo.zip` would contain a package that when installed will contain the entire contents of any Unicorn configuration matching `Foundation.*`.

`New-UnicornItemSource` also accepts parameters to specify package installation options, exactly like SPE's [New-ExplicitItemSource](https://doc.sitecorepowershell.com/appendix/commands/New-ExplicitItemSource.html). This cmdlet is also very similar to how `New-UnicornItemSource` works: each item that is included in the configuration is added to the package as an explicit item source. Doing this also means that the exported package completely respects the Unicorn Predicate, including exclusions of child paths (note that if you specify `-InstallMode Overwrite`, excluded children may be deleted by the package).

## Questions?

### Are the packaged items pulled directly from the serialized items?

No, they are pulled from the Sitecore database because the Sitecore packaging APIs work in `Item`s. So make sure to _sync_ before you generate a package. Unless you're using Transparent Sync in which case the items will already be up to date.

### Should I use this to deploy my site?

No. As mentioned above, packages are a slower and more dangerous method to deploy item updates to your site.

### Does this mean modules will start requiring Unicorn? 🐘

No. Unicorn would only be used in the development of the module, and the build process used to generate plain old Sitecore Packages for module releases. The module itself would need depend on neither Unicorn, Rainbow, or SPE.