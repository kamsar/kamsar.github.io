title: 'Unicorn 4 Preview, Part 2: SPE Support'
date: 2017-02-06 15:16:03
categories: Unicorn
---

Unicorn 4 will feature full support for [Sitecore PowerShell Extensions](https://www.gitbook.com/book/sitecorepowershell/sitecore-powershell-extensions/details) (SPE) to perform Unicorn actions. If you've ever wanted deep programmatic control over Unicorn (for example "I want to sync Foundation.*"), or if you've got an existing deployment process that's already using SPE Remoting to perform deployment tasks - this is for you.

{% asset_img spe.png %}

To use Unicorn cmdlets in SPE, all that is necessary is to install the SPE package along with Unicorn. Unicorn 4 comes with configuration that remains quiescent until SPE is installed that will automatically enable the Unicorn cmdlets. In case that's not clear enough: SPE is an optional addition and will not be required to use Unicorn 4.

So what can we do with Unicorn cmdlets for SPE?

## Configurations

``` powershell
# Get one
Get-UnicornConfiguration "Foundation.Foo"

# Get by filter
Get-UnicornConfiguration "Foundation.*"

# Get all
Get-UnicornConfiguration
```

The result of `Get-UnicornConfiguration` is an array of `IConfiguration` objects, which you can spelunk (e.g. with their `Name` property) or pass to other cmdlets. Configurations are read only.

## Syncing

Sync cmdlets make use of [Write-Progress](https://msdn.microsoft.com/en-us/powershell/reference/5.1/microsoft.powershell.utility/write-progress) to provide a similar progress bar experience to the Control Panel, albeit a bit less responsive.

``` powershell
# Sync one
Sync-UnicornConfiguration "Foundation.Foo"

# Sync multiple by name
Sync-UnicornConfiguration @("Foundation.Foo", "Foundation.Bar")

# Sync multiple from pipeline
Get-UnicornConfiguration "Foundation.*" | Sync-UnicornConfiguration

# Sync all, except transparent sync-enabled configurations
Get-UnicornConfiguration | Sync-UnicornConfiguration -SkipTransparent

# Optionally set log output level (Debug, Info, Warn, Error)
Sync-UnicornConfiguration -LogLevel Warn
```

For example:

{% asset_img sync.png %}

## Partial Syncing

Sometimes you want to only sync a portion of a configuration. You can do that with PowerShell using `Sync-UnicornItem`.

``` powershell
# Sync a single item (note: must be under Unicorn control)
Get-Item "/sitecore/content" | Sync-UnicornItem

# Sync multiple single items (note: all must be under Unicorn control)
Get-ChildItem "/sitecore/content" | Sync-UnicornItem 

# Sync an entire item tree, show only warnings and errors
Get-Item "/sitecore/content" | Sync-UnicornItem -Recurse -LogLevel Warn
```

## Reserializing

The cmdlet to reserialize is called `Export-UnicornConfiguration` because `Reserialize` is not an [approved verb](https://msdn.microsoft.com/en-us/powershell/reference/5.0/microsoft.powershell.core/functions/get-verb) for a cmdlet :)

``` powershell
# Reserialize one
Export-UnicornConfiguration "Foundation.Foo"

# Reserialize multiple by name
Export-UnicornConfiguration @("Foundation.Foo", "Foundation.Bar")

# Reserialize from pipeline
Get-UnicornConfiguration "Foundation.*" | Export-UnicornConfiguration
```

## Partial Reserializing

Sometimes you want to only reserialize a portion of a configuration. You can do that with PowerShell using `Export-UnicornItem`.

``` powershell
# Reserialize a single item (note: must be under Unicorn control)
Get-Item "/sitecore/content" | Export-UnicornItem

# Reserialize multiple single items (note: all must be under Unicorn control)
Get-ChildItem "/sitecore/content" | Export-UnicornItem 

# Reserialize an entire item tree
Get-Item "/sitecore/content" | Export-UnicornItem -Recurse
```

## Converting to Raw YAML

You can also dump out the raw YAML for an item - or items. The output of `ConvertTo-RainbowYaml` is either a string or array of strings depending on how many items were passed to it. Note that unless `-Raw` is specified, the default field formatters and excluded fields Unicorn ships with are used. These are non-customizable and do not follow Unicorn defaults if changed.

{% asset_img enyaml.png %}

This capability enables casual use of YAML serialization without having to use Unicorn or set up a configuration. It's not a good solution for general purpose synchronization though simply because the nuances of storing trees of items in files are many. Very many. But I'm curious what uses people will find for this :)

``` powershell
# Convert an item to YAML format (always uses default excludes and field formatters)
Get-Item "/sitecore/content" | ConvertTo-RainbowYaml

# Convert many items to YAML strings
Get-ChildItem "/sitecore/content" | ConvertTo-RainbowYaml

# Disable all field formats and field filtering
# (e.g. disable XML pretty printing,
# and don't ignore the Revision and Modified fields, etc)
Get-Item "/sitecore/content" | ConvertTo-RainbowYaml -Raw
```

## Converting from Raw YAML

{% asset_img convertfrom.png %}

In Rainbow the `IItemData` interface is the internal unit of an Item. The `ConvertFrom-RainbowYaml` cmdlet converts raw YAML string(s) into `IItemData` which you can then spelunk as objects or deserialize as needed.

``` powershell
# Get IItemDatas from YAML variable
$rawYaml | ConvertFrom-RainbowYaml

# Get IItemData and disable all field filters
# (use this if you ran ConvertTo-RainbowYaml with -Raw)
$yaml | ConvertFrom-RainbowYaml -Raw
```

## Deserialization

To deserialize items, use `Import-RainbowItem` which takes `IItemData` items in and deserializes them into the Sitecore database. No comparison is done before deserialization, which makes this a bit slower than a full Unicorn sync.

{% asset_img elephant.png %}

As a shorthand `Import-RainbowItem` also accepts YAML strings, however as `IItemData` can represent any sort of item, it is not limited only to deserializing YAML-sourced items.

``` powershell
# Deserialize IItemDatas from ConvertFrom-RainbowYaml
$rawYaml | ConvertFrom-RainbowYaml | Import-RainbowItem

# Deserialize raw YAML from pipeline into Sitecore 
# Shortcut bypassing ConvertFrom-RainbowYaml
$yaml | Import-RainbowItem

# Deserialize and disable all field filters
# (use this if you ran ConvertTo-RainbowYaml with -Raw)
$yaml | Import-RainbowItem -Raw

# Deserialize multiple at once
$yamlStringArray | Import-RainbowItem

# Complete example that does nothing but eat CPU
Get-ChildItem "/sitecore/content" | ConvertTo-RainbowYaml | Import-RainbowItem
```

## Questions?

### Does this mean the existing [PowerShell Remote API](https://github.com/kamsar/Unicorn/tree/master/doc/PowerShell%20Remote%20Scripting) is obsolete?

No. The existing PowerShell API uses Windows PowerShell to provide remote syncing capability and does not require installing _Sitecore_ PowerShell. They serve different parallel purposes, and both are here to stay.

### You have no gifs or memes in this post. Is something wrong?

![you mad?](https://i.giphy.com/qZZAQDUNeXams.gif)

### Can I have a beta yet?

I'll be releasing a beta once I finish the features I have planned. Yep, there's at least one more ;)