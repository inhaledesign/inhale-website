---
layout: post
title: "Introduction"
series: "Unreal Engine 5: Getting Started with C++, GoogleTest, and VS Code"
series_order: 1
description: "Why this series is necessary and what to expect."
---

If you've found this, you fall into the obscure set of developers who want to both (A) develop using the Unreal Engine and (B) take testing seriously. Possibly you even do *Test Driven Development,* and the thought of writing code without running tests every fifteen keystrokes fills you with revulsion, if not mild nausea.

Welcome to a very dark section of the Venn diagram. It is filled with headaches and unspeakable horrors that should, *under no circumstances,* have their names spoken out loud. 

But if that were going to stop you, we wouldn't be talking right now, would we?

### The Landscape
Odds are you've been poking around the internet looking for help. You've found Unreal's article on how to setup QT Creator. Maybe you followed it and reached that abrupt dead end. You've probably found [Eric Lemes' guide to unit testing][eric-lemes] that *almost* got you there, but not quite and definitely not on Linux. 

If you've dug deep enough, you may have even found evidence of others who have [stared too long into the void.][compiling-unreal] You've come to the painful conclusion that supporting C++ development does not appear to be high-priority for UE. Development on Linux even less so.

I know this because I was you once. I've walked through that gate and along that path. I've picked up pieces of what others have kindly shared, tested them out, improved some, found others lacking. 

I've put this all together into a development workflow where you can run reasonably fast unit tests that include the core datatypes of Unreal. The tests integrate with Visual Studio Code, and I've ensured that the tests run on both Windows and Linux.

Now it's my turn to share the fruits of my journey.

### Intentions
I will be your Virgil: I will lead you through the Inferno, but I will not hold your hand.

My goal with this series of posts is to help you get a reasonable Unreal/C++ development environment setp. I'll give you direct steps to follow. I'll also try to give you a broad understanding of *why* the setup is the way it is.

I'm assuming that you're an experienced programmer. I will tell you to "clone repos" and talk about "builds" assuming that those words automatically make sense to you and do not require explanation.

I'm also assuming that you're starting from scratch with Unreal. I won't skip over any steps of getting your development environment setup. I also promise you that I've personally used this setup for a whole sprint to make sure they work.

I'm *not* promising that this is the best setup out there. I'll warn you right now, support for code analysis is pretty rough. There's some evidence that the situation will improve: [VisualStudio's IntelliSense has been improved for Unreal][vs-intellisense-unreal], but only for UE 4.27.1 with a promise to support UE5 "at a later date." It also remains to be seen if those improvements will apply to VS Code's IntelliSense or will strictly apply to Visual Studio.

If this is a deal breaker to you and you have money lying around, just save yourself some headache and go get [JetBrains' ReSharper.][resharper]

[eric-lemes]: https://ericlemes.com/2018/12/12/unit-tests-in-unreal-pt-1/
[compiling-unreal]: https://github.com/Allar/compiling-unreal
[resharper]: https://www.jetbrains.com/resharper/
[vs-intellisense-unreal]: https://devblogs.microsoft.com/cppblog/18x-faster-intellisense-for-unreal-engine-projects-in-visual-studio-2022/