Trivial
# Efficient VSCode Workflow in Julia

## Introduction
Julia is a programming language designed to solve the "two language problem". This refers to the practice of using a quick interactive programming language, such as Python, to rapidly prototype a solution and then rewriting the prototype in a compiled language, such as C++ or Rust, for good runtime performance. Keeping the two versions of the same program in sync is a significant software engineering challenge. And, it's more difficult to learn and program in two languages than in one.

Julia promises the best of both worlds: quick interactive development and good runtime performance in a single language. If you set up your Julia environment properly then Julia largely delivers on this promise. Unfortunately, some of the default environment settings do not lead to the most efficient and glitch free workflow. Most of the settings and features for efficient workflow are documented, somewhere, but it's hard to find a single document that includes all the information you need to set up your environment properly.

This document is an attempt to address this issue. It will show you how to set up your Julia and VSCode environment to minimize annoying delays and ensure smooth interaction of the programming tools in the Julia ecosystem.

This is not a tutorial on the Julia programming language. If that's what you want look [here](https://julialang.org/learning/tutorials/) instead. 

An intermediate level of programming experience is necessary to follow these instructions. You should be comfortable working with command line interfaces. If you are an absolute beginner you may find it difficult to correctly carry out these instructions. If you use Julia primarily as a scripting language, or you do all your work in Jupyter or Pluto notebooks then this document will be overkill for you.

## Beginner problems with Julia

The biggest, most annoying, and perhaps unexpected problem beginners encounter is long startup time of your project in the REPL, the interactive window which parses your text input and executes it as Julia code. 

If your project use several large packages it can take a long time to load the packages *every* time you start a new REPL session. A package I worked on, OpticSim.jl, takes 98 seconds to load on my computer (OpticSim uses *many* packages). After using the environment settings described later in this document OpticSim load time was reduced to 866ms, a 113x speedup.

The fundamental cause of this delay is that the current Julia compilation system does not cache all the compiled code generated when you first load a package. Code may be unnecessarily recompiled every time you start a new REPL session, even when the source code of the package hasn't changed.

This startup latency has steadily declined over the last several years, and is likely to decline further soon. But for now it's a problem you have to work around.

Another common beginner problem is figuring out how to organize your code so that Julia tools such as the VSCode IDE, the package manager, and the Revise package, work well together. These tools make assumptions about code organization that are not well documented, or at least not well documented in one place. If your code is organized according to these assumptions the tools mostly interact seamlessly. If not then the Julia programming experience can be confusing and frustrating.

Finally, there are tips and tricks that are broadly useful but whose documentation is scattered and difficult to find. The last section of the document has short descriptions of several. 

## Set up your Julia environment
You should perform the environment setup steps in the order they appear in this document. Some functionality will not work if you change this order.
### Get an account on Julia Discourse
Don't skip this step in your eagerness to begin programming. Eventually you will hit a problem this guide won't help you fix. When this happens Julia Discourse is the place to go for answers.

Whether you are a beginner or an expert you should get an account on the Julia [discourse](https://discourse.julialang.org/) forum. The Julia community is friendly and helpful and you are likely to get an answer to your question within 24 hours. Read [this](https://discourse.julialang.org/t/please-read-make-it-easier-to-help-you/14757) before posting.

Here are tips for posting to maximize the chance of someone answering your question. Whenever possible post a complete working piece of code that others can copy and paste into the REPL and execute. This will greatly increase the odds of you getting help. When you include code in the markdown text of your post enclose it within ``` ```. This will make the code format better. 

The simplest way to create a posting is to copy and paste the full output of the REPL including the entire error message. To copy at the REPL you need to use ctrl-shift-c rather than ctrl-c. Similary to paste into the REPL you need to use ctrl-shift-v rather than ctrl-v. Here's a typical posting (to see how this was done look at the raw markdown in Workflow.md):

This code doesn't work. Help!
```
julia> x^2=3
ERROR: syntax: "2" is not a valid function argument name around REPL[29]:1
Stacktrace:
 [1] top-level scope @ REPL[29]:1
```

It can also be helpful to include the version of Julia you are using, and what packages you have loaded. Type `versioninfo()` at the REPL to get the Julia language version number. To get the list of packages enter the package manager mode by typing `]` at the REPL and then typing `st`. Here's typical output:

```
julia> versioninfo()
Julia Version 1.8.3
Commit 0434deb161 (2022-11-14 20:14 UTC)
Platform Info:
  OS: Windows (x86_64-w64-mingw32)
  CPU: 32 × AMD Ryzen 9 7950X 16-Core Processor
  WORD_SIZE: 64
  LIBM: libopenlibm
  LLVM: libLLVM-13.0.1 (ORCJIT, znver3)
  Threads: 32 on 32 virtual cores
Environment:
  JULIA_EDITOR = code.cmd
```

```
(Differentiation) pkg> st
Project Differentiation v1.0.0-alpha.1
Status `C:\Users\seatt\source\Differentiation\Project.toml`
  [5ae59095] Colors v0.12.10
  [b552c78f] DiffRules v1.12.2
  [a2cc645c] GraphPlot v0.5.2
  [bd48cda9] GraphRecipes v0.5.12
  [f526b714] GraphViz v0.2.0
  [86223c79] Graphs v1.7.4
  [b964fa9f] LaTeXStrings v1.3.0
  [e9d8d322] Metatheory v1.3.5
  [77ba4419] NaNMath v1.0.1
  [91a5bcdd] Plots v1.38.0
  [6c8a4c8a] RelevanceStacktrace v0.1.8
  [c5292f4c] ResumableFunctions v0.6.1
  [295af30f] Revise v3.4.0
  [276daf66] SpecialFunctions v2.1.7
  [90137ffa] StaticArrays v1.5.11
  [d1185830] SymbolicUtils v0.19.11
  [0c5d862f] Symbolics v4.13.0
⌅ [8ea1fca8] TermInterface v0.2.3
  [f8b46487] TestItemRunner v0.2.1
  [1c621080] TestItems v0.1.0
  [b4f28e30] TikzGraphs v1.4.0
  [8dfed614] Test
Info Packages marked with ⌅ have new versions available but compatibility constraints restrict them from upgrading. To see why use `status --outdated`
```

Sometimes the error message, or your code sample, is very long and you don't want to clutter your posting with too much text. You can use the `<details>...</details>` markdown command to hide these details in drop down menus:

<details>
   <summary> This text will be displayed as a short summary of what is enclosed by the details block </summary>
[comment]: # You need a blank line between the summary line and the line beginning with ``` or the text won't format properly in markdown

```
julia> versioninfo()
Julia Version 1.8.3
Commit 0434deb161 (2022-11-14 20:14 UTC)
Platform Info:
  OS: Windows (x86_64-w64-mingw32)
  CPU: 32 × AMD Ryzen 9 7950X 16-Core Processor
  WORD_SIZE: 64
  LIBM: libopenlibm
  LLVM: libLLVM-13.0.1 (ORCJIT, znver3)
  Threads: 32 on 32 virtual cores
Environment:
  JULIA_EDITOR = code.cmd
```
</details>

### Install Julia
You can install Julia manually but don't; instead install [juliaup](https://github.com/JuliaLang/juliaup) and then use `juliaup` to install Julia for you. Follow the instructions at the link to install juliaup.

After you have installed `juliaup` open a command window and type:

```
juliaup update
```

This should install the latest version of Julia. In the future you will use the same command to upgrade to the latest version of Julia. This is all you need from `juliaup` for now but if you are curious what other options are available type `juliaup` at the command line and then hit enter.

Once you have installed Julia open a command window and type `julia` at the prompt. This should start an interactive Julia REPL session and display something like this:
```
               _
   _       _ _(_)_     |  Documentation: https://docs.julialang.org
  (_)     | (_) (_)    |
   _ _   _| |_  __ _   |  Type "?" for help, "]?" for Pkg help.
  | | | | | | |/ _` |  |
  | | |_| | | | (_| |  |  Version 1.8.2 (2022-09-29)
 _/ |\__'_|_|_|\__'_|  |  Official https://julialang.org/ release
|__/                   

julia> 
```
If the Julia REPL doesn't start or your prompt doesn't look anything like this go [here](https://github.com/JuliaLang/juliaup) for troubleshooting suggestions.

### Create a basic startup.jl file
Now that Julia is installed you should create a `startup.jl` file. The instructions in the `startup.jl` file will be executed at the beginning of every interactive Julia session and ensure that important packages are automatically loaded in your interactive REPL environment. 

If you are a less experienced programmer the file you will create in this section is all you will need. More advanced programmers will want to also look at the Advanced setup.jl file section.

On linux the startup file is located at `~\.julia\config\startup.jl`. On Windows the file will be at `C:\Users\YourUserName\.julia\config\startup.jl` Create the `config` directory if it doesn't already exist. Create a file named `startup.jl` in this directory and copy this text to it. 

```
using Pkg

let
    pkgs = ["OhMyREPL", "Revise"]
    for pkg in pkgs
        if Base.find_package(pkg) === nothing
            Pkg.add(pkg)
        end
    end
end

using OhMyREPL

try
    using Revise
catch e
    @warn "Error initializing Revise" exception=(e, catch_backtrace())
end
```
This will automatically load the packages Revise and OhMyREPL every time you start a Julia REPL. Revise is a package that tracks changes to Julia source files in the background. When Revise detects a file change it automatically updates the dynamic state of any interactive REPL sessions you have running. Programming Julia without it is too painful to contemplate. 

The OhMyREPL package enhances the REPL by adding support for context sensitive color text and color parenthesis matching. This makes it is easier to write and edit code at the REPL.

If you are a beginner you won't need more packages in your `startup.jl` file until you have much more experience with the language. You can skip the next section and go to the Set up VSCode section.

### Advanced startup.jl file
This `startup.jl` file adds packages to help you manage github repos and to benchmark and debug code. If you are not familiar with github or have never benchmarked code you should skip this section.

Create this `setup.jl` instead of the basic `startup.jl` file:
```
using Pkg

let
    pkgs = ["BenchmarkTools", "OhMyREPL", "Revise", "PkgTemplates", "Infiltrator"]
    for pkg in pkgs
        if Base.find_package(pkg) === nothing
            Pkg.add(pkg)
        end
    end
end

using BenchmarkTools
using OhMyREPL
using PkgTemplates
using Infiltrator

try
    using Revise
catch e
    @warn "Error initializing Revise" exception=(e, catch_backtrace())
end
 
ENV["JULIA_EDITOR"] = "code.cmd"
```
* [PkgTemplates](https://github.com/JuliaCI/PkgTemplates.jl) adds functionality that is especially useful if your code is on a github repo.
* [Infiltrator](https://github.com/JuliaDebug/Infiltrator.jl) is a low overhead way of inserting breakpoints in your code. Julia has an interactive debugger but it can be painfully slow if your code does significant computation. Infiltrator is fast but not as full featured as the debugger.
* [BenchmarkTools](https://github.com/JuliaCI/BenchmarkTools.jl) adds tools for benchmarking code. These tools will help you find and fix inefficient code.

### Other startup packages to consider
* [InteractiveCodeSearch](https://github.com/tkf/InteractiveCodeSearch.jl) is great for finding the source locations of functions from the REPL. It requires the installation of peco, which is easy on Linux but more complicated on Windows.
* [AbbreviatedStackTraces](https://github.com/BioTurboNick/AbbreviatedStackTraces.jl) shrinks stack traces to reduce irrelevant text. 

Don't add too many packages to your `startup.jl` file. Each package takes time to load and this overhead is incurred every time you start the REPL.

## Set up VSCode
This document assumes you will be using VSCode as your Julia IDE. It has the best support and features of the available IDE's. With VSCode installed you will rarely need to leave the IDE since the Julia language extension adds support for an interactive REPL panel, notebooks, and plotting.

Install VSCode from here https://code.visualstudio.com/Download. Start VSCode and click on the extension manager. Type `julia` in the extension manager search window and install the Julia language extension.

The remaining setup instructions in this section are executed from within VSCode. You will need a running instance of VSCode to make these changes.

### Set `Auto-Save` to `onFocusChange`
This setting will make the view of your source in the VSCode text editor and the dynamic state of the interactive REPL session more consistent. One of the disadvantages of interactive programming systems is that the source text you are editing in VSCode and the definitions of functions, constants, etc., in the interactive session window can get out of sync. You change a function definition in the text editor and save the file but forget to evaluate the new definition in the interactive window so it still has the old definition. This can be very confusing.

The combination of Revise and auto save on focus change can almost completely eliminate this problem. Whenever the cursor moves out of the text editor window the file will be saved. Revise will detect this change and update the interactive environment in the REPL. What you see in the text editor will be a close match to what is defined in the REPL. 

Type `ctrl-shift-p` to open the command palette. Type `settings` in the command palette search window and select `Preferences:Open Settings (UI)`. Type `Files:Auto Save` in the settings search bar. On the drop down menu select `onFocusChange`. This will configure VSCode to automatically save your files when the cursor moves from a text editor window to a Julia command prompt window. 

There are some changes Revise will not update for you: 
* changes to a `struct` definition
* changes to a `const`

Revise will warn you when you make changes like this. If you want them updated in the REPL you must kill your current REPL and start a new one.
 
### Set `Julia:Num Threads`
By default VSCode starts Julia with 1 thread enabled. To automatically enable the full number of threads available on your machine open the command palette (ctrl-shift-p), type `settings`in the command palette search bar and select `Preferences:Open Settings (UI)`. Type `julia:num` in the settings search bar. Select `Edit in settings.json` in the `Julia:Num Threads` option. Type this line into the settings.json file:
```
   "julia.NumThreads": "auto"
```
### Create your first project
Projects are not required for writing and executing Julia code but I strongly recommend that you use projects for all your Julia code. Many of the Julia tools assume that you have your code organized as projects and won't work well, or sometimes at all, if you don't have a project. Your workflow will be smoother and less error prone if you use projects.

As you become more experienced you will want to learn [more](https://pkgdocs.julialang.org/v1/) about packages and how they are managed. For now you can treat the package infrastructure largely as a black box; you just need to know how to create one.

Open a command window in a directory where you want to put your package. Type `julia` at the command window prompt. Once the REPL starts type `]` to enter package manager mode. Let's assume your package is called Package1. Type `generate Package1` and hit enter. The package manager will create a new directory called `Package1`.

To edit the package in VSCode select the open folder menu item (File->Open Folder) and navigate to the directory `Package1`. Do not navigate to the `src` subdirectory contained inside the `Package1` directory; VSCode depends on being started at the top level directory of your package to find the information it needs to set up its environment properly.

### Add packages to your project
In VSCode use the folder view to show the files in your project. Right click on the `src` directory and select `New File` from the menu. Call this file `FirstProgram.jl`. Use the folder view to open `FirstProgram.jl` and type this:

```
module Package1

using Plots

function first_plot()
    x = 1:10; y = rand(10); # These are the plotting data
    plot(x, y)
end
export first_plot

end #module
```

Now open a Julia REPL in VSCode. Open the command pallette (ctrl-shift-p) and type `Julia: Start REPL`. In the REPL type `]` to enter package manager mode. 

Verify that the prompt has changed to `Package1:`. If it hasn't then type `activate Package1`. This will tell the REPL where to look for the source code of your package. You can also type `activate .` to activate the package in the current directory, which is this case will be `Package1`.

Type `add Plots` and hit enter. The package manager will download the `Plots` package from the Julia registry so your program will be able to find it when you execute the first_plot function.

Exit package manager mode by typing backspace and then type `using Package1`. After your package loads type `first_plot()`. This should display a simple plot on a plot window in VSCode.

### Set `Julia:use an existing custom sysimage when starting the REPL`
Some packages, such as Plots, take a long time to load. If your project uses several such packages the startup time for a new REPL session can easily be tens of seconds. You can precompile these packages into what is called a sysimage which loads more quickly. A custom sysimage can reduce startup time from tens of seconds to less than a second. If startup time is an issue for you then you should use a precompiled sysimage.

By default VSCode is not configured to use compiled sysimages. Turn this feature on by opening the command palette and typing `Preferences:Open Settings (UI)`. Then type `julia:use custom sysimage` in the settings search bar. Select the box `use an existing custom sysimage when starting the REPL`. This will make VSCode use a custom sysimage if one is available.

Now you need to compile the sysimage. Open the command palette and type `Tasks:Run Build Task`. Select the option `Julia: Build sysimage for current environment (experimental)`. This will compile the files necessary to start your project into a single large file which will be loaded at startup time.

If you add a new package in the package editor or update your packages the sysimage will not be automatically updated; it will still contain the code from the package versions used when the sysimage file was created. Every time you add or update packages you should rerun `Tasks:Run Build Task`, `Julia: build custom sysimage for current (experimental)`.

Sometimes compiling a sysimage doesn't work. A fix that frequently works is to update all your packages. To update your packages type `]` at the `julia` command prompt to enter the package manager and type `update`. Then try running the sysimage task again.

If updating your packages didn't fix the problem you may have a package that won't compile into a sysimage. You'll have to look at the stack trace in the error message to find the packages which appear to be causing precompilation to fail. You can tell VSCode which packages to exclude by creating a `JuliaSysimage.toml` file. See this [link](https://www.julia-vscode.org/docs/stable/userguide/compilesysimage/) for detailed instructions. 

The contents of `JuliaSysimage.toml` will look like this:
```
[sysimage]
exclude=["FailingPackage1","FailingPackage2"]   # Additional packages to be excluded in the system image
statements_files=[]  # Precompile statements files to be used, relative to the project folder
execution_files=[] # Precompile execution files to be used, relative to the project folder
```
Notice that the names of the packages to be excluded must be in double quotes.

### Starting your project efficiently
Before you can begin using or debugging the code in your project you may have to load packages, define test functions and variables, etc., at the REPL. During the development phase of your code you may be restarting the REPL many times per day as you make changes that Revise can't track, or if your REPL state becomes corrupted. When this happens typing setup commands repeatedly can be tiresome. 

You can save time by creating a file, let's call it `setup.jl`, that contains all these commands and loading the file at the beginning of each programming session using the Julia `include` function. Place this file in the root directory of your project. Typically this setup file is only used during development. It is not meant to be included in a released package. 

Here's an example `setup.jl` file:

```
# This script is for setting up the REPL environment for development of the Differentation package. No use in production.

using Differentiation
using Differentiation.Experiments
using Differentiation.SphericalHarmonics

using Symbolics

@variables x y
```

 which you execute by typing the following command at the REPL

```
 include("setup.jl")
```

## Code formatting suggestions
### Autoformat on Save
If you are not using git and github this is less important. But if you are, especially if several people are working on the same project, then formatting your files every time they are saved will prevent small formatting changes from showing up as diffs that will create many trivial git commits. 

First set up the Julia formatter: in VSCode type `ctrl-shift-p`, select `Preferences:open settings (UI)`, then type `julia format` in the search box. In the dropdown menu for `Editor:default formatter` select `Julia:julialang.language-julia`. 

Now set up format on auto save: type `ctrl-shift-p`, select `Preferences:open settings (UI)`, then type `Editor:format on save` in the search box. Click the check box for the option to format on save. 

Now your files will automatically be formatted consistently before they are committed.
### Type function arguments as generically as possible
It is not necessary to type function arguments in Julia. The compiler will figure out the types automatically making it easy to write very generic code. However not all code is generic; sometimes only a few types make sense as arguments. In these cases adding types to the function declaration makes your code easier to read and use. For example, if this function is part of the official API of a package you are using:
```
transform(obj,mat)
```
what is obj, or mat? You could document this or you could add types which will automatically document it for you:
```
transform(obj::Geometry,transform::SMatrix{3,4}).
```
In VSCode these types will show up when you hover your mouse over the function call.

 If you decide to type function arguments do so as generically as possible. Example:
```
multiply(a::Number,b::AbstractMatrix)
```

Here `Number` is used instead of `Real` because it is more generic:
```
julia> subtypes(Number)  #subtypes, and supertype, are defined in the InteractiveUtils package. This is loaded by default in the Main module of the REPL. If you want to call it from your code you will need a using InteractiveUtils statement at the top level of your module.
4-element Vector{Any}:
 Complex
 DualNumbers.Dual
 Real
 Static.StaticInteger
 ```
 
 If you know that `a` will always be a `Real` then you could type it like this:
 ```
multiply(a::Real,b::AbstractMatrix)
```
You might think that even `Real` is too generic since perhaps you expect `a` to always be `AbstractFloat` or even `Float64`. However, if you intend to use automatic differentiation on this function then you should stick with the more generic `Real`. This is because `ForwardDiff.Dual` inherits from `Real`. The dual numbers defined in `ForwardDiff` are used in automatic differentiation:

```
julia> subtypes(Real)
12-element Vector{Any}:
 AbstractFloat
 AbstractIrrational
 FixedPointNumbers.FixedPoint
 ForwardDiff.Dual
 Integer
 Rational
 Static.StaticFloat64
 StatsBase.PValue
 StatsBase.TestStat
 SymbolicUtils.LiteralReal
 SymbolicUtils.SafeReal
 Symbolics.Num

```
### Put export statements immediately after function definitions

By convention the user accessible API of a module is assumed to consist of only those names which are exported from the module. All other names are assumed to be unsupported internal functions. You export names from a module using the `export` statement. This can be placed anywhere in your module code:
```
julia> module ExportExample
       export f1
       
       f1(x) = println("f1")
       
       f2(x) = println("f2")
       export f2
       end
```
You can see which functions, variables, etc., are exported from a module with the `names` function:
```
julia> names(ExportExample)
3-element Vector{Symbol}:
 :ExportExample
 :f1
 :f2
```
Most Julia style guides recommend you put all exported names at the top of the file. I recommend you don't, at least not until you become a more experienced Julia programmer and maybe not even then. Here's why. Suppose you delete function `f1`. If `f1` is defined at the bottom of a long file you won't see the `export f1` statement at the top. And it is likely you'll forget to remove `f1` from the export list (ask me how I know). What happens then?
```
julia> module ExportExample
       export f1
       
       f2(x) = println("f2")
       export f2
       end
WARNING: replacing module ExportExample.
```
Notice that `f1` is still exported even though it is no longer defined in `ExportExample`:
```
julia> names(ExportExample)
3-element Vector{Symbol}:
 :ExportExample
 :f1
 :f2
 ```
 The compiler won't complain if you export an undefined name. Why is this a problem? Lets suppose we have another module `M2` that uses `ExportExample`:
 ```
 julia> module M2
       using Main.ExportExample
       f3(x) = f1(x)
       export f3
       end

julia> using Main.M2

julia> f3(1)
ERROR: UndefVarError: f1 not defined
```
The compiler blithely compiled module `M2` without even a warning about the undefined `f1`. The error doesn't occur until you *execute* `f3` which then calls the undefined function `f1`. 

This can be a big problem in a larger project with several modules. You delete a function or change its name in one module and assume everything is fine because everything compiles. Then a month later you execute some rarely used function in a different module and you see this `not defined` error, which you will waste considerable time tracking down (again, ask me how I know).

If you put the `export` statement immediately after a definition then you are much less likely to forget to update it because the `export` statement will be right in front of your eyes when you edit the code.

These export errors are usually caught by a test suite as part of a continuous integeration methodology, *if* you are vigilant about writing tests and have good test coverage. But when you are just starting Julia you probably don't have this machinery set up. Until you do put export statements immediately after definitions.
## Tips and tricks
This last section is a grab bag of commands that you may find useful.
### For the REPL
Starting from the `julia` prompt you can enter the different REPL modes by typing:
* `?` help mode: get documentation on functions and find out what tab sequence generates a character.
* `]` package mode: add, remove, list, update, etc. packages for your project.
* `;` shell mode: execute command line functions. This mode works in Linux but not in Windows.
To exit any of these modes use backspace.



You see a cool symbol ∢ in another person's Julia code and want to use it. How do you type it with tab completion? One way is to find the symbol on this web [page](https://docs.julialang.org/en/v1/manual/unicode-input/) which lists all the Unicode symbols you can use tab completion to type. Another is to copy the symbol then enter help mode by typing ? at the REPL prompt. Paste the symbol into the REPL and hit enter:
```
help?> ∢
"∢" can be typed by \sphericalangle<tab>
```
Suppose you have already typed this function and want to add a line after `y=sqrt(x)`
```
julia> function f(x)
       y = sqrt(x)
       end
```
Type `↑` to display the function again in the REPL. Then type `←` to move the cursor into the function text. Use `↑` to move the cursor to where you want to enter a new line and type alt-enter.
```
function f(x)
       y = sqrt(x)
       return (y,x) #use alt-enter to create a new line here
       end
 ```

 Need to see a compiler error message again? Type Revise.retry(). This will trigger Revise to reevaluate your functions. Doesn't always give as full an error message as killing the REPL and starting again but it is faster than reloading your enviroment from scratch.
### For VSCode
Type F12 to go to a function definition.

Type ctrl-shift-o to open a menu to search for a function definition by name.

Refactoring code. Place the cursor in a function or variable name you want to change and type F2. Enter the new name. This mostly works in Julia, but sometimes changes parts of the code it shouldn't. 


# Appendix

## Gotchas
```
julia> f(a::Vector{Integer}) = sum(a)
f (generic function with 2 methods)

julia> f([1,2,3])
ERROR: MethodError: no method matching (::Colon)(::Int64, ::Vector{Int64})
Closest candidates are:
  (::Colon)(::T, ::Any, ::T) where T<:Real at range.jl:41
  (::Colon)(::A, ::Any, ::C) where {A<:Real, C<:Real} at range.jl:10
  (::Colon)(::T, ::Any, ::T) where T at range.jl:40
  ...
Stacktrace:
 [1] f(n::Vector{Int64})
   @ Main .\REPL[2]:1
 [2] top-level scope
   @ REPL[6]:1

julia> f(a::Vector{T}) where{T<:Integer} = sum(a)
f (generic function with 3 methods)

julia> f([1,2,3])
6
```