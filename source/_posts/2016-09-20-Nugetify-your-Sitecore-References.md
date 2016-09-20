title: Nugetify your Sitecore References
date: 2016-09-20 16:01:17
categories: Sitecore
---

Recently, in a fit of brilliance, Sitecore released a [public NuGet feed](https://doc.sitecore.net/sitecore_experience_platform/developing/developing_with_sitecore/sitecore_public_nuget_packages_faq) that you can use to reference Sitecore assemblies from your projects. While some people have been doing this with home grown packages for years, it's nice to have a stable, official source to get your references from. 

If you're not familiar with how this works, [Jeremy Davis wrote a great post](https://jermdavis.wordpress.com/2016/09/05/the-official-sitecore-nuget-feed-is-here/) about the details of using the official feed.

## Okay so what are you on about?

If you've got a project of reasonable size, especially one using [Helix](http://helix.sitecore.net/), you probably have a bucketload of references to Sitecore assemblies. Manually converting all these references to packages is a bit of a tedious process, and you know what we do to tedious processes around here.

{% asset_img "nugetify.jpg" %}

That's right, we script them.

## How?

Migrating your direct Sitecore references to NuGet is a quite simple process with this script. For obvious reasons, use source control so you have a fallback in case the script doesn't work for your particular setup. It worked for me on several projects, but just in case :)

### 1. Install NuGet.config

Copy this to the root of your solution:

	<?xml version="1.0" encoding="utf-8"?>
	<configuration>
	  <!--
	  Used to specify the default Sources for list, install and update.
	  -->
	  <packageSources>
	    <add key="Official Sitecore" value="https://sitecore.myget.org/F/sc-packages/api/v3/index.json" />
	  </packageSources>
	  
	  <activePackageSource>
	    <!-- this tells that all of them are active -->
	    <add key="All" value="(Aggregate source)" />
	  </activePackageSource>
	</configuration>

This will add the official Sitecore NuGet package feed to your solution. Unlike adding it via Visual Studio, this will also apply for CI and MSBuild-executed builds.

### 2. Copy __Nugetify.ps1__

Copy the following PowerShell script to the root of your solution. __Open it in a text editor and set the correct `$SitecoreVersion` and `$FrameworkVersion`__ for your solution. The NuGet feed has packages for Sitecore 7.0-8.2.

	# Script to convert all sitecore assembly references to Sitecore Public NuGet feed
	# Run from root solution folder (where your packages folder is)
	# execute this script with powershell
	Param
	(
		$sitecoreVersion = '8.2.160729', # NuGet package version to convert to. Format is major.minor.releasedate.
		$frameworkVersion = 'net452' # for 8.2: net452. For 7.0-8.1: net45
	)

	$ScriptPath = $MyInvocation.MyCommand.Path
	$ScriptDir  = Split-Path -Parent $ScriptPath
	$MsbNSString = 'http://schemas.microsoft.com/developer/msbuild/2003'
	$MsbNS = @{msb = $MsbNSString}
	$PackagesConfigFileName = 'packages.config'

	#Create project.json from packages.config
	Write-Host 'Scanning for projects to update...' -ForegroundColor Green
	Get-ChildItem -path '.' -Recurse -Include $PackagesConfigFileName |
		ForEach {
			$PackageFilePath = $_.FullName
			$PackageFileDir = $_.Directory
			Write-Host "Processing $PackageFilePath" -ForegroundColor Green

			# Find existing csproj to match direct references
			$csproj = Resolve-Path "$($_.Directory)\*.csproj"
			$proj = [xml] (Get-Content $csproj)

			# Find existing Sitecore references and NuGet-ify them
			Write-Host "Checking for non-NuGet Sitecore references in $csproj"
			$xpath = "//msb:Reference/msb:HintPath[not(contains(.,'packages\'))]"

			$changedProj = $false
			$sitecoreNuGetPackages = @(Select-Xml -xpath $xpath $proj -Namespace $MsbNS | foreach {
				$node = $_.Node.ParentNode

				$referenceName = $node.Attributes['Include'].Value.Split(',')[0]

	      		# Filter non-NuGet references to transform into NuGet packages
				if($referenceName.StartsWith("Sitecore") `
					-and -not $referenceName.StartsWith('Sitecore.Modules') `
	        		-and -not $referenceName.Contains('WFFM') `
	        		-and -not $referenceName.StartsWith('Sitecore.Forms') `
					-and -not $referenceName.StartsWith('Sitecore.Foundation') `
					-and -not $referenceName.StartsWith('Sitecore.Feature') `
					-and -not ($referenceName.StartsWith('Sitecore') -and $referenceName.EndsWith('Website'))) {

					$changedProj = $true

					Write-Host "NuGet-ifying assembly reference $referenceName"

					# set hintPath to package path
					Push-Location -Path $PackageFileDir
					$hintPathRoot = Resolve-Path "$ScriptDir\packages" -Relative
					Pop-Location

					$hintPath = "$hintPathRoot\$referenceName.NoReferences.$sitecoreVersion\lib\$frameworkVersion\$referenceName.dll"

					$existingHintPath = $node['HintPath', $MsbNSString]
					if($existingHintPath -eq $null) {
						$hint = $proj.CreateElement("HintPath", $MsbNSString)
						$hint.InnerXml = $hintPath
						$foo = $node.AppendChild($hint)
					} else {
						$existingHintPath.InnerXml = $hintPath
					}

					"$referenceName.NoReferences"
				}
			})

			if($changedProj) {
				Write-Host "Saving NuGet-ified references to csproj" -ForegroundColor Yellow
				$proj.Save($csproj)
			} else {
				Write-Host "Found no references to change."
			}

			# Add packages to packages.config
			$packageXml = [xml] (Get-Content $PackageFilePath)

			$sitecoreNuGetPackages | % {
				$packageNode = $packageXml.CreateElement('package');
				$packageNode.SetAttribute('id', $_)
				$packageNode.SetAttribute('version', $sitecoreVersion)
				$packageNode.SetAttribute('targetFramework', $frameworkVersion)

				$foo = $packageXml.DocumentElement.AppendChild($packageNode)
			}

			Write-Host "Updating packages.config with new packages" -ForegroundColor Yellow
			$packageXml.Save($PackageFilePath)
		}

### 3. Run __Nugetify.ps1__

Open a PowerShell and execute `Nugetify.ps1`.

That's it. Open Visual Studio and verify that everything was converted correctly, and you should be good to go.

## What about NuGet 3 + `project.json`?

Another option is to use the NuGet 3.x style package management which is integrated into a `project.json` file that lives next to your csproj files. NuGet 3's major advantage is that the `project.json` both references packages __and__ adds them to your project references. So adding packages does not result in alterations to the csproj file, and upgrading packages is as simple as changing a json file.

Sounds idyllic, right? Well there's one huge downside. Microsoft, in their infinite wisdom, decided that it was not necessary to support content files being deployed into the project the package is installed into. For Sitecore projects, that means packages that come with config files, such as Unicorn, Synthesis, and Glass Mapper, will not install those files into projects using `project.json`. The content-containing packages can still be used, but then it becomes your task to reverse engineer the content they install, add that to your project, and handle upgrades of those content files when the package is upgraded.

For the moment, I wouldn't use `project.json`, but I hope it becomes more tenable in the future. But if you want to use it, I have a script for that too - a script that both nugetifies your Sitecore references __and__ converts __all__ of your packages in `packages.config` to `project.json`. This script is based on [this one](https://github.com/wgtmpeters/nugetprojectjson).

	# Script to generate project.json for all packages.config file in the solution.
	# This script will also migrate non-NuGet Sitecore package references to Sitecore Public NuGet feed

	# Run from root solution folder
	# execute this script with powershell

	# TargetFramework: Use the .NET framework version your projects are targeting, which may NOT be the version Sitecore is built aginst
	# SitecoreVersion: NuGet version you want to convert to for local sitecore assembly references
	Param
	(
	  	[string] $TargetFramework = "net452",
		$sitecoreVersion = '8.2.160729'
	)

	# Filter existing installed NuGet packages to transform versions and such
	function Filter-Packages {
		$input | % {
			$package = $_.Node.id
			$version = $_.Node.version

			# Translate nuget package generator 8.2 package version to official
			if($version -eq "8.2.0.160729") {
				$version = "8.2.160729"
			}

			# Translate 3rd party refs from old SC versions to target public versions
			if($package.Equals('Microsoft.Extensions.DependencyInjection.Abstractions')) {
				$version = '1.0.0'
			}

			if($package.Equals('MongoDB.Driver')) {
				$package = 'mongocsharpdriver'
				$version = '1.11.0'
			}

			# Blacklist these older sitecore nuget generator metapackages, modules, and nonexistant packages
			if($package.EndsWith("-Core") `
				-or $package.EndsWith("-CoreGroup") `
				-or $package.Equals('MongoDB.Bson') `
				-or $package.Equals("Telerik.Web.UI")) {

				return # skip loop
			}

			# Packages that started with Sitecore before should now get NoReferences for sanity (note: you may need to exclude sitecore modules whose name begins in Sitecore here)
			if($package.StartsWith('Sitecore') `
				-and -not $package.StartsWith('Sitecore.FakeDb') `
				-and -not $package.EndsWith('PatchableIgnoreList')) {

				$package = "$package.NoReferences";
			}

			$_.Node.id = $package
			$_.Node.version = $version

			$_
		}
	}

	$ScriptPath = $MyInvocation.MyCommand.Path
	$ScriptDir  = Split-Path -Parent $ScriptPath
	$MsbNS = @{msb = 'http://schemas.microsoft.com/developer/msbuild/2003'}
	$PackagesConfigFileName = 'packages.config'

	#Create project.json from packages.config
	Get-ChildItem -path '.' -Recurse -Include $PackagesConfigFileName |
		ForEach {
			$PackageFilePath = $_.FullName
			$ProjectFilePath = $_.Directory.FullName + '\project.json'
			Write-Host "Processing $PackageFilePath"

			# Find existing csproj to match direct references
			$csproj = Resolve-Path "$($_.Directory)\*.csproj"
			$proj = [xml] (Get-Content $csproj)

			# Find existing Sitecore references and NuGet-ify them
			Write-Host "Checking for non-NuGet Sitecore references in $csproj"
			$xpath = "//msb:Reference/msb:HintPath[not(contains(.,'packages\'))]"

			$changedProj = $false
			$sitecoreNuGetPackages = @(Select-Xml -xpath $xpath $proj -Namespace $MsbNS | foreach {
				$node = $_.Node.ParentNode

				$referenceName = $node.Attributes['Include'].Value.Split(',')[0]

	      # Filter non-NuGet references to transform into NuGet packages
				if($referenceName.StartsWith("Sitecore") `
					-and -not $referenceName.StartsWith('Sitecore.Modules') `
	        -and -not $referenceName.Contains('WFFM') `
	        -and -not $referenceName.StartsWith('Sitecore.Forms') `
					-and -not $referenceName.StartsWith('Sitecore.Foundation') `
					-and -not $referenceName.StartsWith('Sitecore.Feature') `
					-and -not ($referenceName.StartsWith('Sitecore') -and $referenceName.EndsWith('Website'))) {

					$changedProj = $true

					Write-Host "NuGet-ifying assembly reference $referenceName"

					# remove old reference we're NuGet-ing
					[void]$node.ParentNode.RemoveChild($node);

					"$referenceName.NoReferences"
				}
			})

			if($changedProj) {
				Write-Host "Saving NuGet-ified references in $csproj"
				$proj.Save($csproj)
			}

			# Generate project.json
			$file = '{
	  "dependencies": {
	'

	$packages = (Select-xml -xpath '//package' -Path $PackageFilePath | Filter-Packages | % { "    ""{0}"": ""{1}""" -f $_.Node.id,$_.Node.version }) -join ",`r`n"

	$file += $packages;

	$sitecorePackages = (($sitecoreNuGetPackages | % { "    ""{0}"": ""{1}""" -f $_, $sitecoreVersion }) -join ",`r`n")

	# separate the json elements if both converted and sitecore packages exist
	if($packages.Length -gt 0) {
		$file += ",`r`n"
	} else {
		$file += "`r`n"
	}

	$file += $sitecorePackages

	$file += '
	  },
	  "frameworks": {
	    "' + $TargetFramework + '": {}
	  },
	  "runtimes":  {
	      "win-anycpu": {},
	      "win": {}
	  }
	}'

	$file | Out-File $ProjectFilePath

		Remove-Item $PackageFilePath
	}

	Get-ChildItem -path '.' -Recurse -Include '*.csproj' | ForEach {
		$CsProjFilePath = $_.FullName
		$ProjectFilePath = $_.Directory.FullName + '\project.json'

		Write-Host $csProjFilePath

		$proj = [xml] (Get-Content $CsProjFilePath)

		#Remove all references to ..packages files
		$xpath = "//msb:Reference/msb:HintPath[contains(.,'packages\')]"
		$nodes = @(Select-Xml -xpath $xpath $proj -Namespace $MsbNS | foreach {$_.Node})
		if (!$nodes) { Write-Verbose "RemoveElement: XPath $XPath not found" }
		Write-Output 'Reference Nodes found: ' $nodes.Count
		foreach($node in $nodes) {
			$referenceNode = $node.ParentNode
			$itemGroupNode = $referenceNode.ParentNode
			[void]$itemGroupNode.RemoveChild($referenceNode)
		}
		[System.XML.XMLElement] $itemGroupNoneNode = $null
		#Find itemgroup with None Elements, if not found add.
		$itemGroupNoneNodes = @(Select-Xml -xpath "//msb:ItemGroup/msb:None" $proj -Namespace $MsbNS | foreach {$_.Node})
		Write-Output '$itemGroupNoneNode found: ' $itemGroupNoneNodes.Count
		if($itemGroupNoneNodes.Count -eq 0){
			# create itemgroup element for None nodes.
			Write-Output 'Adding ItemGroup for None Nodes'
			$itemGroupNoneNode =  $proj.CreateElement('ItemGroup',$proj.DocumentElement.NamespaceURI)
			$itemGroupNodes = @(Select-Xml -xpath "//msb:ItemGroup" $proj -Namespace $MsbNS | foreach {$_.Node})
			#$itemGroupNodes.Count
			[void]$proj.DocumentElement.InsertAfter($itemGroupNoneNode,$itemGroupNodes[$itemGroupNodes.Count-1])

		}else{
			$itemGroupNoneNode = $itemGroupNoneNodes[0].ParentNode
		}

		#Remove packages.config from ItemGroup
		$nodes = @(Select-Xml -xpath "//msb:ItemGroup/msb:None[@Include='packages.config']" $proj -Namespace $MsbNS | foreach {$_.Node})
		Write-Output 'packages.config Nodes found: ' $nodes.Count
		foreach($node in $nodes) {
			$itemGroupNode = $node.ParentNode
			[void]$itemGroupNode.RemoveChild($node)
		}

		#Remove packages.config from ItemGroup (if it was set to content)
		$nodes = @(Select-Xml -xpath "//msb:ItemGroup/msb:Content[@Include='packages.config']" $proj -Namespace $MsbNS | foreach {$_.Node})
		Write-Output 'packages.config Nodes found: ' $nodes.Count
		foreach($node in $nodes) {
			$itemGroupNode = $node.ParentNode
			[void]$itemGroupNode.RemoveChild($node)
		}

		#Remove build target EnsureNuGetPackageBuildImports from csproj
		$nodes = @(Select-Xml -xpath "//msb:Target[@Name='EnsureNuGetPackageBuildImports']" $proj -Namespace $MsbNS | foreach {$_.Node})
		Write-Output 'EnsureNuGetPackageBuildImports target found: ' $nodes.CountAd
		foreach($node in $nodes) {
			$itemGroupNode = $node.ParentNode
			[void]$itemGroupNode.RemoveChild($node)
		}

		#Add project.json to itemGroup
		if( Test-Path $ProjectFilePath){
			$nodes = @(Select-Xml -xpath "//msb:ItemGroup/msb:None[@Include='project.json']" $proj -Namespace $MsbNS | foreach {$_.Node})
			if($nodes.Count -eq 0){
				$projectJsonNoneNode = $proj.CreateElement("None", $proj.DocumentElement.NamespaceURI)
				$projectJsonNoneNode.SetAttribute("Include","project.json")
				[void]$itemGroupNoneNode.AppendChild($projectJsonNoneNode)
				Write-Output 'Adding None node for project.json'
			}
		}

	    #add PropertyGroup nodes targetFrameworkProfile, CopyNuGetImplementations, PlatformTarget
	    # Find the TargetFrameworkVersion to be used to find the parent PropertyGroup node
	    $xpath = "//msb:PropertyGroup/msb:TargetFrameworkVersion"
	    $nodes = @(Select-Xml -xpath $xpath $proj -Namespace $MsbNS | foreach {$_.Node})
	    if ($nodes.Count -gt 0) {
	        [System.XML.XMLElement] $node = $nodes[0]
	        $propertyGroupNode = $node.ParentNode
	        $nodes = @(Select-Xml -xpath "//msb:PropertyGroup/msb:TargetFrameworkProfile" $proj -Namespace $MsbNS | foreach {$_.Node})
			if($nodes.Count -eq 0){
				$node = $proj.CreateElement("TargetFrameworkProfile", $proj.DocumentElement.NamespaceURI)
				[void]$propertyGroupNode.AppendChild($node)
				Write-Output 'Adding TargetFrameworkProfile node for PropertyGroup'
			}
	        #$nodes = @(Select-Xml -xpath "//msb:PropertyGroup/msb:CopyNuGetImplementations" $proj -Namespace $MsbNS | foreach {$_.Node})
			#if($nodes.Count -eq 0){
			#	$node = $proj.CreateElement("CopyNuGetImplementations", $proj.DocumentElement.NamespaceURI)
	        #    $textnode = $proj.CreateTextNode("true")
	        #    $node.AppendChild($textnode)
			#	[void]$propertyGroupNode.AppendChild($node)
			#	Write-Output 'Adding CopyNuGetImplementations node for PropertyGroup'
			#}
	        $nodes = @(Select-Xml -xpath "//msb:PropertyGroup/msb:PlatformTarget[not(@*)]" $proj -Namespace $MsbNS | foreach {$_.Node})
			if($nodes.Count -eq 0){
				$node = $proj.CreateElement("PlatformTarget", $proj.DocumentElement.NamespaceURI)
	            $textnode = $proj.CreateTextNode("AnyCPU")
	            $foo = $node.AppendChild($textnode)
				[void]$propertyGroupNode.AppendChild($node)
				Write-Output 'Adding PlatformTarget node for PropertyGroup'
			}
	    }

	    # replace ToolsVersion with 14.0
	    $attibutes = Select-Xml -xpath "//@ToolsVersion" $proj -Namespace $MsbNS
	    foreach ($attribute in $attibutes){

	        $attribute.Node.value = "14.0"
	        Write-Output 'Setting ToolsVersion to 14.0'
	    }

		$proj.Save($CsProjFilePath)
	 }


Till next time, happy NuGetting!