---
layout: post
title: "The UE Build System"
series: "Unreal Engine 5: Getting Started with C++, GoogleTest, and VS Code"
series_order: 3
description: "An overview of the Unreal Engine build system, and why it has to cause headaches."
---

### (or, "Why Using C++ With UE Hurts So Bad")
Anyone who tells you that they know how the Unreal build system works is lying to you. Fullstop. It is a dark place that [drives people mad.][compiling-unreal]

I will refrain from trying to say too much about what each part "does", lest I mislead you. However, it is important to understand the broad picture of the build system in order to understand how to use and configure your IDE properly.

### UnrealBuildTool
UnrealBuildTool ("UBT") is Unreal's build program, written in CSharp. The key thing you need to understans is that it does not *replace* your C++ compiler or make. Rather, it *wraps around* them. That is to say, it will pre-process your C++ code, generating headers and substituting platform-specific macros, and then run them through your C++ compiler.

This is how Unreal achieves being cross platform. It's also an added level of complexity that you have to understand, and makes things like IDE integration difficult. That's the price you pay.

Here's a pretty common example. Let's imagine you're plugging along, developing in VS Code. You see a red squiggle indicating that a header file you've included can't be found. "That's weird," you think, "because that file is included in almost every C++ file." Clicking the little lightbulb icon, it helpfully suggests that you pick a directory to add to your C++ includepath. You make the perfectly reasonable choice add the appropriate directory, and the red squiggle goes away.

Congratulations, you just broke your build. While VS Code's C++ extension may not know where your header file is, *UBT does.* If you were to ignore the red squiggles and run a build, your build would run fine. Adding the header to the C++ extension's includepaath will only break things.

The upshot is that in order to setup our tools properly, we have to learn to play nice with UBT.

### Build Configuration
Unreal C++ projects are organized into [modules.][ue-modules] When you create an empty project, you start with one module for your project that has dependencies on Unreal modules, like `Core` and `Engine`. You can--and should--create your own project modules to keep your code organized.

There are three types of files that concern us with respect to building in Unreal:

* **Build** files are used to configure how a specific module is built. Every module has to have one, and they will have the same name as your module (e.g. `MyModule.Build.cs`). The main thing you do here is declare the module's dependencies.

* **Target** files declare a program to compile and its build configuration. Every C++ project comes with two preconfigured targets: one for your game (`MyProject.Target.cs`) and one for UnrealEditor (`MyProjectEditor.Target.cs`). If you want to create another application from your source, you add a new target file for it.

* **UProject** files are kindof ineffable, because it is used for general things by more than one tool. When you open your project in UnrealEditor, you ask it to open this file. When unreal generates IDE integration files, it uses this file. When you build your targets, how your modules are declared in the UProject will affect what gets built. It's worth thinking of this as something like a manfiest. It describes the overall project: what modules and plugins it's using, what platforms it's tageting, etc.

Here's an example that demonstrates why this can all be messy.

We're going to create a new target to create a module for our tests. This module will have no dependencies on our main module, and vice versa, declared in their Build files. And yet, were you to compile my UnrealEditor target, the tests module will be compiled with it.

One way to fix this is to just not include your test module in the UProject file. The test target will still build just fine, and it will also keep the test module from being built with the Editor target. 

Unfortunately it will also exclude the test module from some IDE integrations, like code analysis. It took quite a lot of beating my head against this, but I found a way to declare in the UProject file that the test module should only be built for program targets. Why this has to be done is anyone's guess.

The upshot is that when you need to make changes to your build, it's not always clear *where* you're going to need to make that change. While build and target files have a clear separation of concerns, UProject files muddy things up.


[compiling-unreal]: https://github.com/Allar/compiling-unreal
[ue-modules]: https://docs.unrealengine.com/5.0/en-US/unreal-engine-modules/