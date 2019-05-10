title: 'Build 2019: All the things'
date: 2019-05-10 16:01:11
categories: C#
---

I spent the first part of this week out at [Build 2019](https://www.microsoft.com/en-us/build), and I learned a lot! Here's all the news I saw fit to print from Build in a concise, notesy format.

## .NET Core 3

The next version of .NET Core will be released in September 2019. It will feature a raft of improvements, notably WPF/desktop app support (Windows only), .NET Standard 2.1 (_not going to be supported by .NET 4.x ever_), and C# 8 (_.NET Standard 2.1 required_).

Coinciding with the release of .NET Core 3 will be [dotnetconf](https://www.dotnetconf.net/) from September 23-25, a virtual conference highlighting .NET Core 3.

.NET Core 3.1, the long term support version, is slated to ship in November.
 
## .NET 5 and the Unified BCL

After .NET Core 3 ships, .NET Core is dead. Instead .NET 5 will ship, and it will unify the abstraction of .NET Standard into a universal BCL that can run on any .NET 5 compatible runtime (i.e. Xamarin, Mono, Windows .NET). It will also gain Java and Swift interop capabilities (from Mono/Xamarin) on all platforms. The idea is that .NET 5 will be a singular platform that runs anywhere from mobile devices, to IoT/Raspberry Pi, to desktop apps, to cloud server(less).

Web Forms and WCF will never be ported to .NET Core/.NET 5. Specifically for Web Forms, Blazor will be the recommended migration path.

Following .NET 5, the .NET platform will have yearly releases (.NET 6, 7, 8, …). Alternating years will be LTS versions, in other words 2020's .NET 5 will be supplanted by the LTS .NET 6 in 2021.

[More on .NET 5 here](https://devblogs.microsoft.com/dotnet/introducing-net-5/)

## C# 8

_Note: C# 8 requires compiler changes that need .NET Standard 2.1+. In other words, C# 8 can only be used with .NET Core 3 and later as a consumer!_

The main focus of C# 8 is "robustness." There are a number of new features that support this goal:

#### Async Enumerable
Ever since async/await was shipped in C#, it's been problematic to use it with enumerables because you must await either `Task<IEnumerable<T>>` (thus awaiting the WHOLE enumerable, which loses its lazy enumeration advantages), or `IEnumerable<Task<T>>` which potentially requires awaiting in a loop, which is also suboptimal. It also prevents the use of `yield return` in async methods, which makes them significantly less pretty.

In C# 8, this is fixed by introducing `IAsyncEnumerable<T>`, an asynchronously enumerable enumerable type. This type is enumerated using `await foreach`, i.e. `await foreach(var t in asyncEnumerable) { /* where t is not a task */ }`. The implementation of `IAsyncEnumerable` is simply allowed to `yield return` values, giving the enumerator control over its own internal asynchrony needs, batching, etc.

#### Nullable Reference Types
The `NullReferenceException` is everyone's favorite C# bugbear, and solutions good and bad abound for asserting that method arguments are not null to avoid throwing them (my favorite is `var x = arg ?? throw new ArgumentNullException(nameof(arg));`). In C# 8, you can explicitly declare reference types as nullable, explicitly stating that a method can return - or accept - a null value. Doing this allows the compiler to remove the need for all those assertions, as it can warn you at compile time if you're not checking a nullable type for null before using it. This is an opt-in feature either with `#nullable enable` in a file, or it can be turned on per-project.

```C#
Item? GetItem() {
    // ...
}

void DoStuff() {
    // with nullable reference types on, this will throw a compiler warning
    // because GetItem declared explicitly that it can return null
    var troll = GetItem().Axes.GetDescendants();

    // you can bypass the warning if you know what you're doing (lol) with !
    var explodingTroll = GetItem()!.Axes.GetDescendants();
}
```

#### Range Expressions
A common need is to parse a string or array and split it up into pieces by index; for example "this string's last two characters" or "the first 5 elements in this array." This sort of code is quite vulnerable to naughty data input causing exceptions, for example `"a".Substring(5)` will throw because it isn't 5 characters long.

C# 8 range expressions allow you to concisely and safely (they won't throw if the array is shorter than the slice) express this sort of problem. They work using `^` to anchor the range to "length - x" or "start + x", a spread, and an optional endpoint. A few examples:

```c#
var str = "hello world";
var a = str[0..5]; // 'hello'
var b = str[^1]; // 'd'
var c = str[^1..^4]; // 'ello w'
```

#### Switch Expressions

The `switch` statement receives an upgrade in C# 8 with the ability to assign it directly to a variable, eliminating the need for clumsy `break` statements in every case. It's also possible to use pattern matching with this format (not pictured).

```c#
var result = "hello" switch {
	"hello" => true,
	"goodbye" => false,
	_ => false // default case via a discard
};
// result = true
```

#### Default Interface Implementations

Interfaces can have default implementations for members. This is not intended to kill IoC containers as much as be a tool for API creators to ship additions to public interfaces without breaking existing consumers of that interface. The additional members need only be optionally implemented by downstream consumers, with the defaults used if not overridden.

#### Using Declarations

The `using` statement gets an overhaul to avoid needing a block scope. A using statement is used to prevent forgetting to dispose `IDisposable` resources, but before C# 8 it required a block scope of its own which especially in nested usings made things hard to read. In C# 8, you can define a variable with the `using` keyword and no block scope, and it is implicitly disposed at the end of the current block scope. For example:

```c#
public void Foo() {
    using var file = new FileStream(@"c:\foo.txt");
    // file will be disposed when the Foo() block exits
}
```

[More on C# 8 here](https://docs.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-8)

## TypeScript
In TypeScript 3.4 - currently RC - you can enable _incremental builds_ (via tsconfig, or `--incremental` to the CLI), which allows TS to cache the output of the last build/watch run and essentially 'rehydrate' it during the next build to avoid rebuilding unchanged modules. The upshot of this on the VS Code codebase is that warm build times went from **47 seconds** to **11 seconds**.

On larger TypeScript codebases, using [project references](https://www.typescriptlang.org/docs/handbook/project-references.html) can allow TypeScript to partition compilation units, enabling it to only rebuild changed units in the dependency tree (about like projects and solutions in Visual Studio). This can be used to avoid needing to recompile an entire TypeScript project every time even without incremental builds.

New modernized TypeScript documentation, with content oriented around current TypeScript practices and improved clarity, is in process. Current target is late 2019 to release the new docs.

[More on TypeScript 3.4 here](https://devblogs.microsoft.com/typescript/announcing-typescript-3-4-rc/)

## Live Share
Using Live Share developers can collaborate effectively while remote, with either Visual Studio, VS Code, or both. It's a bit like a code-specific combination of screen sharing and collaborative editing. This includes things like:

* Editing code in a Google Docs-like collaborative realtime editor
* Setting breakpoints, controlling debugger execution that executes on the host's computer
* Reviewing the other dev's localhost ports
* VS Code can live share with Visual Studio
* The viewer need not have a SDK, debugger, or plugins for the host's code to participate - or even the same CPU architecture.
* Works across platforms too, so you could connect using Code on a Mac to VS on Windows and control debugging a windows service, for example.

[More on Live Share here](https://visualstudio.microsoft.com/services/live-share/)

## Visual Studio Code Remote Editing

Code can now connect to a remote system (via SSH or directly to a container) and edit the remote instance as if it were local files. This includes things like installing Code plugins _on the remote environment_ - it's basically connecting to a "headless" VS Code service. For example, you could write Ruby code on a Docker container in AKS from a windows machine running Code...without needing to set up a Ruby dev environment locally or install any Ruby plugins into Code. Or, do .NET Core dev on a remote VM without needing to install the .NET Core SDK locally.

Remote editing really shines when combined with Azure Dev Spaces (read on...).

[More on remote editing here](https://code.visualstudio.com/docs/remote/remote-overview)

## Visual Studio Online (not to be confused with the old name for Azure DevOps) 

* Visual Studio Code in a browser
* Edit code, review PRs, connect to remote development environments (SSH, Containers)
* Works on any browser. Want to review PRs on an iPad? Sure!
* In private preview at the moment

[More on VSO here](https://devblogs.microsoft.com/visualstudio/intelligent-productivity-and-collaboration-from-anywhere/)

## VS Code and VS Tips

* There were excellent sessions on getting more out of Visual Studio and VS Code. If you want to learn something about your tools, these were pretty great. For example:
* You can configure Code to attach multiple debuggers at launch (i.e. Chrome + Node debuggers)
* F1 opens the Code palette, in addition to Ctrl-P
* You can quickly open files in Code from the command palette by removing the > prompt and typing a filename or expression
* Jump to outlined functions, tags, etc the current file, in Code, by entering `@:` in the command palette (i.e. `@:myfunc`)
* IntelliCode (available in Code and VS 16.1+) applies ML models to predict the most commonly used code completions for a given state. For example `if(arrayVar.` might suggest `Length` but `stringVar.` might suggest `Split`. The suggestions model was trained on 2000 of the most popular open source codebases on GitHub, so they're based on actual community practices.
* Visual Studio is aggressively rendering R# irrelevant in current previews. You really might not need it at all in the future. Tons of new refactorings, code cleanup improvements, ability to infer .editorconfig files, et al.

[Code tips session](https://mybuild.techcommunity.microsoft.com/sessions/77039)
[Visual Studio tips session](https://mybuild.techcommunity.microsoft.com/sessions/77044)
[Visual Studio debugger/diags tips](https://mybuild.techcommunity.microsoft.com/sessions/77042)

## Azure Dev Spaces

It's no secret that microservice architectures can be pretty difficult to develop locally. Especially if they tend towards the distributed monolith antipattern ;) Well, Azure decided to do something about that. Probably the coolest demo of the whole event.

Dev Spaces is a prebuilt microservice-oriented workflow for developers based on Azure Kubernetes Service (AKS). The basic concept is that a dev team would share an AKS cluster across their whole team - because developers would probably be working on a few microservices, not the whole galaxy of the system, they can then basically "branch" specific microservices out for personal development, while referencing the rest of the system built from the latest CI build. In other words, there's no need to mock or setup local microservices that you don't care about, because yours runs in AKS and refers to the master build.

Even more bonkers, you can use remote development to debug and auto-deploy files to your personal microservices running in AKS. Pull requests can be made to similarly build in their own namespace, giving faster and more efficient use of build time. Seems like a pretty darn nice experience, with most of the orchestration issues no longer your problem.

[Watch the session](https://mybuild.techcommunity.microsoft.com/sessions/77060)
[Documentation on Dev Spaces](https://docs.microsoft.com/en-us/azure/dev-spaces)

## YAML Builds & Releases for Azure DevOps Pipelines

You can define your Azure DevOps build and release pipelines using YAML files that can be committed to the repository. This allows the build system to be versioned and stable across branches and enables proper testing of changes to the build via PRs. In preview now, release pipelines (in addition to builds) can be defined in YAML. There is also a visual editor that allows generation of YAML for common tasks using a GUI.

Coming soon, pipelines will be able to automatically generate a Kubernetes manifest and Helm chart for any project with a Dockerfile. This will also generate appropriate build YAML to allow building docker images, deploying them to Azure Container Registry, and spinning up the Kubernetes cluster from those images on Azure Kubernetes Service. Looks really easy to use, and a definite lowering of the barrier to entry to Kubernetes deployment. The pipeline doesn't only support Azure k8s either; it can deploy to other container registries or k8s clusters on premise or in other clouds.

## Azure Cognitive Search

Azure search is gaining the ability to apply cognitive services to data being indexed; for example it can index the contents of images as detected by cognitive services, etc. Also the 1000-field limit that Sitecore users love is being investigated and may be raised or eliminated in a near term time frame.

## ML.NET 1.0 & AutoML

A library to build and run machine learning models from within .NET. Can consume several trained model formats, including TensorFlow, which lets you integrate ML models built by a data scientist using Python et al into .NET runtimes. Microsoft is also working on the [ONNX](https://onnx.ai/) model format for interoperable models. While it's capable of training its own models too, it is interesting to see the model promoted where data scientists do their modeling using mainstream ML tools (i.e. Python), and deploy only the trained model to the .NET application. Promising in terms of integrating .NET with mainstream data scientists.

The AutoML toolkit was also announced. AutoML is a nice looking tool for non-data-scientists to take a dataset and automatically discover a good ML algorithm and hyperparameter set to produce an accurate model. Definitely aimed at the backend developer looking to add a splash of ML to their toolset, as opposed to data scientists, but this seems like it could significantly lower the barrier to ML entry for .NET developers.

[More on ML.NET and AutoML here](https://dotnet.microsoft.com/apps/machinelearning-ai/ml-dotnet#automl)

Go forth and code, me hearties.