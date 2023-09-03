---
title: "Transforming Sublime Text into an C/C++ IDE"
date: 2023-07-29T17:15:56+00:00
toc: true
comments: true
toc_label: "Table of contents"
categories:
  - Technical
tags:
  - C/C++
  - editors
---

How to turn your Sublime text editor into an IDE for C/C++.

{: .text-justify}
Disclaimer: These steps are for C/C++ and CMake build system in Linux, if you successfully made a setup or tested for another set of language and operating system I would be more than happy to link your findings here.

{: .text-justify}
The idea of this article is to document my findings, for future me, and also to share with people that also want to setup some Integrated Development Environment in this very nice text editor. After all, an IDE is not more than a text editor that integrate other programs in itself, so... let's try to do this with sublime.

{: .text-justify}
For all of this to work, you will need to set the [sublime CLI](https://www.sublimetext.com/docs/command_line.html)

# The Packages

{: .text-justify}
Firstly we need a set of packages. I will put the link for the packages in each section so it is easy to search and install, and also, of course, for documentation on how it should work.

## CMakeBuilder

- [Link](https://github.com/rwols/CMakeBuilder)

{: .text-justify}
I don't use this package too much because until now it doesn't support CMakePresets, it checks for every target in the builder and it is not very good to separate the release/debug/etc builds. But for simpler cmake projects, this is the way to go, it is easier to setup and build.

## CMake

- [Link](https://github.com/zyxar/Sublime-CMakeLists)

{: .text-justify}
For some syntax highlights, snippets, and basic language support.

## Fmt

- [Link](https://github.com/mitranim/sublime-fmt)

{: .text-justify}
A simple, yet, powerful tool... It is able to invoke any 3rd party program to format your code, so with this you don't need to install a formatter package for each language you work with, for example, "clang formatter", "cmake formatter", "python formatter", etc etc.

{: .text-justify}
The downside is that you need to install 3rd party tools, but you would need anyway with the specific sublime package. Anyway, you have to configure this package, so here it is for you:

```json
{
  "rules" : [
    {
      "selector" : "source.json",
      "cmd" : ["jq"],
      "format_on_save" : false,
      "merge_type" : "diff"
    },
    {
      "selector" : "source.cmake",
      "cmd" : [ "cmake-format", "$file" ],
      "format_on_save" : false,
      "merge_type" : "diff"
    },
    {
      "selector" : "source.c | source.c++ | source.objc | source.objc++ | source.cuda-c++",
      "cmd" : [ "clang-format", "$file", "--style=LLVM" ],
      "format_on_save" : false,
      "merge_type" : "diff"
    }
  ]
}
```

For this to work you need the following packages installed in your system:

- jq: https://jqlang.github.io/jq/
- cmake-format: installed via pip: https://pypi.org/project/cmake-format/0.2.0/
- clang-format: https://clang.llvm.org/docs/ClangFormat.html

Obs.: clang-format is also able to format json files, but the jq is more nice.

## LSP

- [Link](https://lsp.sublimetext.io/)

{: .text-justify}
This packages is what makes the magic happen. It will provide support for the language server protocol, in another words, it will give you support for the C/C++ linter, also for another languages if you wish, check their documentation.

## LSP-clangd

- [Link](https://github.com/sublimelsp/LSP-clangd)

{: .text-justify}
You can enable globally or by putting in the project settings. You need to provide the build folder with exported compiler commands, anyway, here is my setting for the global configuration, so every time I compile something, it defaults to the build folder.

```json
"LSP": {
  "clangd": {
    "initializationOptions": {
      "clangd.compile-commands-dir": "build"
    }
  }
}
```

You also need this package:

- clangd: https://clangd.llvm.org/

## ProjectEnvironment

- [Link](https://bitbucket.org/daniele-niero/sublimeprojectenvironment)

This will enable "global" environment variables for your build system.

## SublimeDebugger

- [Link](https://github.com/daveleroy/SublimeDebugger)

{: .text-justify}
This will give you a general GUI for attaching an external debugger, such as GDB, LLDB, and so on. The package will install the necessary debugger extension, so it can work with the preferred debugger.

{: .text-justify}
To be able to work with GDB, you need to provide `--interpreter=mi` in the `args` of the debug configuration json, either in your project settings or globally.

# Sublime Project

{: .text-justify}
I have a CMake template project together with the sublime project, [link here](https://github.com/eHonnef/cpp-project-start), my project relies on CMakePresets, because it is more contained and I can set what the user, or me, can see from the build. Of course I will not put all the sublime project file here, I will just explain what it contains.

The idea of how it should work:

1.  I configure the project with CMake
    - By calling `cmake --preset Debug`
    - And it should generate some build folder, that is known when you execute the script
2.  The build folder is given to the next command that is to build the project itself
    - By calling `cmake --build path_to_folder`
    - Or calling `cmake --build --preset Debug`
3.  Automatically feed the build folder to the LSP server
4.  Also give the built binary to the debugger configuration

{: .text-justify}
Simple enough, right? Actually, no... it is not that simple to do all of this, and seamless, not without developing your own package.

{: .text-justify}
Anyway, until this moment I was able to to steps 1 and 2, maybe you just need to adapt some naming or commands to your case, but in general it works pretty well. Unfortunately I still couldn't make steps 3 and 4 automatically, but it is also not a big deal, since you are probably developing in debug mode and only uses the release mode to, well, make releases.

The process inside the sublime build system should be the following:

1.  Clear the cache file if exists
2.  Configure the build folder using cmake, and;
3.  Parse the cmake output to some blackmagic commands, so;
4.  It creates a cache json file where it contains the build folder, then;
5.  It calls the sublime CLI to force an update in the environment variables, finally;
6.  You can call the build system command "Cmake Build", where;
7.  It uses the cached variable to know which folder it is suppose to build.
8.  ... Here are going to be the steps 4 and 3 from the previous list. For now, I haven't found out how to do that.

## The configure command

{: .text-justify}
The first ever step in any cmake build system is to invoke the "configurator", so it creates the build environment for you, in general it is `cmake -S . -B build`

In this case I want to be more fancy:

```bash
truncate -s 0 $project_path/.cache.json && cmake --preset=$preset_name | tee >(grep "Build files have been written to:" | sed "s/^.*: //" | xargs -i jq -n --arg cached_build_folder {} '{env:{cached_build_folder: $cached_build_folder}}' > $project_path/.cache.json) && subl --command "set_project_environment"
```

So... this is the main reason it needs Linux, or maybe it should work with mingw on Windows.

### Now, step by step:

We clear the cache file:

```bash
truncate -s 0 $project_path/.cache.json
```

We invoke the cmake with some configuration preset:

```bash
cmake --preset=$preset_name
```

The preset variable is set in some of the variants of the build system.

{: .text-justify}
Then we pipe the output to `tee`, so we at the same time can output in console to the user and the parse the output:

```bash
tee >(grep "Build files have been written to:" | sed "s/^.*: //" | xargs -i jq -n --arg cached_build_folder {} '{env:{cached_build_folder: $cached_build_folder}}' > $project_path/.cache.json)
```

{: .text-justify}
First here we filter the line that will contain the output folder, unfortunately I could not find a way to do that by just invoking the cmake command line and having it in an "official" way. It is hacky? yes, does it work? also yes. So everything is fine.

```bash
grep "Build files have been written to:"
```

Then we parse the filtered line to get the second part, after the symbol `:` :

```bash
sed "s/^.*: //"
```

{: .text-justify}
Then we feed this output to xargs, that will put the argument in the right place for the jq utility and then write to file.
I used the jq utility because it will output the json with the symbols already escaped, quoted and formatted. Doing all of this via bash is not very nice to the eyes:

```bash
xargs -i jq -n --arg cached_build_folder {} '{env:{cached_build_folder: $cached_build_folder}}' > $project_path/.cache.json
```

{: .text-justify}
And, finally, we call the sublime CLI to invoke the "set\_project\_environment" command from the [ProjectEnvironment](#projectenvironment) package, so it will force an update by reading the cache file:

```bash
subl --command "set_project_environment"
```

## The build command

Now that the cached variable is set, we just need to pass it to cmake:

```bash
cmake --build $cached_build_folder
```

In my project there is only one special case that is the "Coverage" target build. But besides that one, it is pretty straightforward.

# TL;DR

- This workflow was tested and thought for Linux systems and for C/C++ development.
- Set the [Sublime CLI](https://www.sublimetext.com/docs/command_line.html)
- Check the section [The Packages](#the-packages) to download and install the necessary packages.
- [Here is my CMake project template repository](https://github.com/eHonnef/cpp-project-start), you can navigate to .sublime folder over there and check what is happening.
