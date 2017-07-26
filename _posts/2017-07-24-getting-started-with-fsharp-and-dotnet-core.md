---
title:  "Getting started with F# and .NET Core"
date:   2017-07-24 21:10:00
description: "My adventures in the modern .NET world."
tags: ["F#", ".NET Core", "fsharp", "dotnet core"]
---

**TLDR**: F# and .NET Core are really cool, you should give them a try.

I recently got interested in the .NET world thanks to my colleagues who are doing C# at $WORK. My last experience with Microsoft technologies was during my school years doing ASP Classic and it was not a great one but .NET Core is now a multi-platform, open-source .NET and it has a functional language: F#.

F# was designed by [Don Syme](https://twitter.com/dsyme). It's open-source and maintained by the [F# Software Foundation](http://fsharp.org) and individual contributors with some working at Microsoft.

I'm really fond of functional languages like Haskell or OCaml so I took F# for a test-drive to see what I could benefit from it.


# Core vs Mono vs Standard

A bit of terminology first.

**.NET Framework** is Windows-only and is what you usually think about what .NET is.

**Mono** is a cross-platform implementation of .NET Framework from Xamarin (which Microsoft bought).

Microsoft chose to rewrite .NET to be truly cross-platform and stripped some APIs that were not considered core. The new product is **.NET Core**. So .NET Core is a smaller cross-platform .NET Framework.

**.NET Standard** aims to unify a set of APIs that all .NETs (Framework, Mono and Core) must implement.

**.NET Native** is a new initiative to compile code to native CPU instructions instead of using the CLR to compile to intermediate language.

Unfortunately resources for using F# with .NET Core are still very sparse, most examples are tailored for the .NET Framework or Mono so here are some notes on what I learned so far. Hope it helps.

# Installation

First get .Net Core from [here](https://www.microsoft.com/net/core) and follow installation instructions.

#### The basics

.NET Core provides a great command line tool: `dotnet`.

Creating a new project:
{% highlight shell %}
λ mkdir project
λ cd project
λ dotnet new console -lang f#
{% endhighlight %}

Adding a package:
{% highlight shell %}
λ dotnet add package Newtonsoft.Json
{% endhighlight %}

Restoring (installing) packages:
{% highlight shell %}
λ dotnet restore
{% endhighlight %}

Building the app:
{% highlight shell %}
λ dotnet build
{% endhighlight %}

Running the app:
{% highlight shell %}
λ dotnet run
{% endhighlight %}

Publishing an app:
{% highlight shell %}
λ dotnet publish
{% endhighlight %}

Publishing with a specific framework and runtime:
{% highlight shell %}
λ dotnet publish --framework netcoreapp1.1 --runtime osx.10.11-x64
{% endhighlight %}

.NET Core projects used to have a `project.json` configuration file but it's been replaced by an XML file with a `.fsproj` extension in order to gain better compatibility with `MsBuild`.

Moving to this new format allows to build and restore multiple projects simultaneously thanks to solution files. A solution file is useful when you want to link projects together. When using `dotnet restore` at the solution file level, all packages in that solution will be restored instead of having to do it for every one of them.

Adding a project to a solution:
{% highlight shell %}
λ dotnet sln todo.sln add todo-app/todo-app.fsproj
{% endhighlight %}


# Tooling

There are a few additional tools you can use besides `dotnet` CLI.

#### Paket

[Paket](https://fsprojects.github.io/Paket/) is a dependency manager for .NET projects. It works with [NuGet](https://www.nuget.org/) packages (like dotnet add packages) and also Git repositories references or HTTP resources. It seems to better solve dependency issues than the standard NuGet way.

It's quite simple to add to your Core project. Download it for your platorm then `paket init` will generate a paket.dependecies file. `paket install` will install those dependencies.

#### Fake

[Fake](https://fake.build/) is F# Make. Similar to Rake in the Ruby world it's a DSL you can use for all your build or deploy needs for example.

#### Ionide

[Ionide](http://ionide.io/) is a Visual Studio Code/Atom package for F# development with everything you'd find in a modern IDE like syntax highlighting, autocompletion, type at point... Really an awesome package if you're using these editors.

#### Omnisharp

For Emacs users like me, [OmniSharp](http://www.omnisharp.net/) is a cross-platform tool that brings Intellisense for a lot of editors (Emacs, Vim, Sublime...). Combined with `fsharp-mode` this makes a pretty nice development environment.

#### Forge

[Forge](http://forge.run) is a project/solution management tool. It's useful for creating a project without an IDE but it seems it's not needed for .NET Core projects. If you're working on a Mono project this is a nice addition to your toolbelt.

#### Web stuff

In C# you would usually use ASP.NET Core (aka Rails.NET) or eventually [Nancy](http://nancyfx.org) a Sinatra-like framework for your web services needs. It's possible to use them with F# but a better alternative exists: [Suave](https://suave.io/).

[Tamizh Vendan](https://twitter.com/tamizhvendan) has written a good book on Suave: [F# Applied](http://products.tamizhvendan.in/fsharp-applied/).

# F# the language

F# looks a lot like OCaml with a few goodies. I'm not going to detail the syntax, just highlight some features of the language that I really like.

Go check [F# syntax in 60 seconds](https://fsharpforfunandprofit.com/posts/fsharp-in-60-seconds/) for a quick overview of the syntax.

#### Use C# libs

One of the best benefits of using F# is the amount of quality .NET libraries available. Most of these are written in C# but you can use them in F# quite easily.

Since it's my first attempt to learn .NET programming, I found that learning a bit of C# can be helpful to understand how to use libraries and stuff. The [C# 7 and .NET Core: Modern Cross-Platform Development](https://www.packtpub.com/application-development/c-7-and-net-core-modern-cross-platform-development-second-edition) book was an easy read and simple enough approach to learn more about the ecosystem.

#### Smooth Operators

The pipe operator `|>` (that Elixir borrowed from F#) lets you pass the result of the function on the left onto the other function as its first argument.

**Note**: the F# REPL is launched with `fsharpi`, `;;` are needed to end statements in the REPL.

{% highlight ocaml %}
> [1..10] |> List.filter (fun n -> n % 2 = 0);;
val it : int list = [2; 4; 6; 8; 10]
{% endhighlight %}

 `<|` takes the function on the left and applies it to the value on the right.

{% highlight ocaml %}
> let double n = n * 2;;
val double : n:int -> int

> printfn "The double of 16 is %d" <| double 16;;
The double of 16 is 32
val it : unit = ()
{% endhighlight %}

The composition operator `>>` lets you ‘compose’ functions together.

Let's define two functions `triple` and `square`:
{% highlight ocaml %}
> let triple n = n * 3;;
val double : n:int -> int
{% endhighlight %}

{% highlight ocaml %}
> let square n = n * n;;
val square : n:int -> int
{% endhighlight %}
We can compose the two functions like:
{% highlight ocaml %}
> let tripleSquare = triple >> square;;
val tripleSquare : (int -> int)
{% endhighlight %}

{% highlight ocaml %}
> tripleSquare 2;;
val it : int = 36
{% endhighlight %}

The `<<` operator takes two functions and applies the function on the right before the one the left of the operator.

{% highlight ocaml %}
> let isOdd n = n % 2 = 1;;
val isOdd : n:int -> bool

> [1..10] |> List.filter (not << isOdd);;
val it : int list = [2; 4; 6; 8; 10]
{% endhighlight %}

#### Async and Parallel

Asynchronous workflows can be easily constructed using `async { ... }`. It's also possible to have some computations composed in parallel using the `fork-join` combinator `Async.Parallel`.

{% highlight ocaml %}
let task1 = async {
        do! Async.Sleep 1000
        printfn "task1 finished!"
        return 5
    }

let task2 = async {
        do! Async.Sleep 500
        printfn "task2 finished!"
        return 2
    }

[task1; task2]
|> Async.Parallel
|> Async.RunSynchronously
|> printfn "%A"

{% endhighlight %}
outputs:
{% highlight ocaml %}
task2 finished!
task1 finished!
[|5; 2|]
val task1 : Async<int>
val task2 : Async<int>
val it : unit = ()
{% endhighlight %}

This example executes both async tasks `task1` and `task2` concurrently and wait for both to finish before printing the results.

#### Mailboxes

Another concurrency primitive available in F# is mailboxes. F# can natively handle message-based concurrency thanks to its mailbox processors. It somewhat brings Erlang's actor model to F#.

Basically actors are lightweight constructs that have a queue and can send messages to other actors and process incoming messages sent to them from the queue.

This great [article](http://www.codemag.com/article/1707051) by [Rachel Reese](https://twitter.com/rachelreese) explores the subject if you want to dig deeper.


# Some additional resources

It's quite hard to find good resources updated for **.NET Core** with examples so I've setup a [Github Repository](https://github.com/julienXX/GettingStartedWithFSharpAndDotNetCore) with some basic tasks I needed to do like using a database or parsing JSON. I'll try to update it whenever I manage to do something that was difficult to find existing resources for.

There is a really friendly [Slack](https://fsharp.slack.com) that you should join.

[F# for fun and profit](https://fsharpforfunandprofit.com) is a superb collection of resources on F#. There is an [ebook](https://www.gitbook.com/book/swlaschin/fsharpforfunandprofit/details) compiling all the posts too.

This [Awesome dotnet core repo](https://github.com/thangchung/awesome-dotnet-core) lists some libs, tools and resources for development with .NET Core but not specifically F#.

Regarding books [Expert F#](https://www.apress.com/us/book/9781484207413) is great and assumes no prior .NET knowledge.

[Ody Mbegbu](https://twitter.com/odytrice) has made a nice intro [video](https://www.youtube.com/watch?v=2xG31sUsCdc) to getting started with .NET Core using F#.

# Conclusion

F# is a nicely designed functional language (that can do OOP too but I didn't try it yet). Most of the resources are newbie friendly and avoid words like `functors` and `monads`. In general I find them more geared toward practical usage which is pretty nice sometimes.

The main issue with .NET Core at the moment (1.x) is that there are still a lot of missing APIs which lead to incompatible libraries like [FSharp.Data](http://fsharp.github.io/FSharp.Data/) but the good news is that the upcoming .NET Core release later this year (2.x) will bring something like 20000 more APIs and most of the libs should be compatible or easily adaptable then.

For me F# is a great language with a robust eco-system. It has all the FP features that matters the most to me: a strong type system, pattern-matching and good concurrency primitives. Its interoperability with C# could be a gateway to more functional programming in my company. I'm currently rewriting some of our slow Ruby services with it and I wish this will demonstrate its value to my co-workers.

Hope this article helped clearing up a bit how to work with F# and .NET Core. Don't hesitate to post your questions or remarks [here](https://github.com/julienXX/julienxx.github.com/issues).
