---
title:  "Getting started with F# and .NET Core"
date:   2017-06-20 15:10:00
description: "Some resources to get you started."
---

# Intro

I recently got interested in the .NET world thanks to my colleagues who are using C# at $WORK. We have several development teams, some on .NET, some on Ruby and some on Javascript. They introduced .Net Core to me and the idea to be able to code on a rock solid platform and be able to collaborate more between the .NET teams and the Ruby team I'm part of appealed to me.
AWS Lambda supports .NET natively and maybe it's possible to extract some of our background processes to F# lambdas.
I'm really fond of functionnal languages like Haskell, OCaml so I took a look at F# to see what I could use it for.
    - Job
    - Haskell OCaml
    - Lambda
    - OSS
    - Cross platform

## Core vs Mono vs Standard

A bit of terminology.

.NET Framework is Windows-only, Mono is a cross-platform implementation of .NET Framework from Xamarin (which Microsoft bought).
Microsoft chose to rewrite .NET to be truly cross-platform and stripped some APIs that were not considered core. The new product is .NET Core. So .NET Core is a smaller cross-platform .NET Framework.
.NET Standard aims to unify a set of APIs that all .NETs (Framework, Mono and Core) must implement.
.NET Native is a new initiative to compile code to native CPU instructions instead of using the CLR to compile to intermediate language.

Since it's quite difficult for a newcomer to find relevant informations with all these .NETs here are some notes on what I understood so far to work with Core. Hope it helps.

# Installation

First get .Net Core from [here](https://www.microsoft.com/net/core) and follow installation instructions.

## The basics

Creating a new project:
```shell
λ mkdir project
λ cd project
λ dotnet new console -lang f#
```

Adding a package:
```shell
λ dotnet add package Newtonsoft.Json
```

Restoring packages:
```shell
λ dotnet restore
```

Building the app:
```shell
λ dotnet build
```

Running the app:
```shell
λ dotnet run
```

Publishing an app:
```shell
λ dotnet publish
```

Publishing with a specific fframework and runtime:
```shell
λ dotnet publish --framework netcoreapp1.1 --runtime osx.10.11-x64
```

.Net Core projects used to have a `project.json` configuration file but it's been replaced by an XML file with a `.fsproj` extension in order to gain better compatibility with `MsBuild`.

Moving to this new format allows to build and restore multiple projects simultaneously thanks to solution files. A solution file is useful when you want to link projects together. When using `dotnet restore` at the solution file level, all packages in that solution will be restored instead of having to do it for every one of them.

Adding a project to a solution:
```shell
λ dotnet sln todo.sln add todo-app/todo-app.fsproj
```

# Tooling

There are a few additional tools you can use.

## Paket

[Paket](https://fsprojects.github.io/Paket/) is a dependency manager for .NET projects. It works with [NuGet](https://www.nuget.org/) packages (like dotnet add packages) and also Git repositories references or HTTP resources. It seems to better solve dependencies issues than the standard NuGet way.

It's quite simple to add to your Core project. Download it for your platorm then `paket init` will generate a paket.dependecies file. `paket install` wil install those dependencies.

## Fake

[Fake](https://fake.build/) is F# Make. Similar to Rake in the Ruby world it's a DSL you can use for all your build or deploy needs for example.

## Forge

[Forge](http://forge.run) is a project/solution management tool. It's useful for creating a project without an IDE but it seems it's not needed for .NET Core projects. If you're working on a Mono project this is a nice addition to your toolbelt.

## Ionide

[Ionide](http://ionide.io/) is a Visual Stidio Code/Atom package for F# development with everything you'd find in a modern IDE like syntax highlighting, autocompletion, type at point... Really an awesome package if you're using these editors.

## Omnisharp

For Emacs users like me [OmniSharp](http://www.omnisharp.net/) is a cross-platform tool that brings Intellisense for a lot of editors (Emacs, Vim, Sublime...). Combined with `fsharp-mode` this makes a pretty nice development environment.

# F# the language

F# looks a lot like OCaml with a few goodies. I'm not going to detail the syntax, just hightlight some nice features. Go check [F# syntax in 60 seconds](https://fsharpforfunandprofit.com/posts/fsharp-in-60-seconds/) for a quick overview of the syntax.

## Operators

The pipe operator `|>` (that Elixir borrowed from F#) lets you pass the result of the left function onto the next function, `<|` takes a function on the left and applies it to a value on the right.

Note: the F# REPL is launched with `fsharpi`, `;;` are needed to end statements in the REPL.

```
> [1..10] |> List.filter (fun n -> n % 2 = 0);;
val it : int list = [2; 4; 6; 8; 10]
```

```
> let double n = n * 2;;
val double : n:int -> int

> printfn "The double of 16 is %d" <| double 16;;
The double of 16 is 32
val it : unit = ()
```

The composition operator `>>` lets you ‘compose’ functions together and the `<<` operator takes two functions and applies the right function first and then the left one.

```
> let triple n = n * 3;;
val double : n:int -> int

> let square n = n * n;;
val square : n:int -> int

> let tripleSquare = triple >> square;;
val tripleSquare : (int -> int)

> tripleSquare 2;;
val it : int = 36
```

```
> let isOdd n = n % 2 = 1;;
val isOdd : n:int -> bool

> [1..10] |> List.filter (not << isOdd);;
val it : int list = [2; 4; 6; 8; 10]
```

## Async and Parallel



## Use C# libs

One of the best benefits of using F# is the amount of quality .NET libraries available. Most of these are written in C# but you can use them in F# quite easily.

    - Suave micro service API
    - Dockerize
    - Deploy

# Tasks Repo
    - explain tasks

# Some nice Resources

There is a really friendly [Slack](https://fsharp.slack.com).
[F# fun profit](https://fsharpforfunandprofit.com) is a superb collection of resources on F#. There is an [ebook](https://www.gitbook.com/book/swlaschin/fsharpforfunandprofit/details) compiling all the posts too.
This [Awesome dotnet core repo](https://github.com/thangchung/awesome-dotnet-core) lists some libs, tools and resources for development with .NET Core but not specifically F#.
Regarding books [Expert F#](https://www.apress.com/us/book/9781484207413) was great and assumes no prior .NET knowledge.
[Ody Mbegbu](https://twitter.com/odytrice) has made a great [video](https://www.youtube.com/watch?v=2xG31sUsCdc) on getting started with .NET Core using F#.

# Conclusion

    - libs issue
    - Core 2 coming
    - What's good about f#
    - What's less good
