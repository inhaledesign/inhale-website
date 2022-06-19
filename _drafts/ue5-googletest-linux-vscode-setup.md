---
layout: post
title: "Setup"
series: "Unreal Engine 5: Getting Started with C++, GoogleTest, and VS Code"
series_order: 2
date: "22-06-19 2:00"
---

## {{ page.subtitle }}
You're going to need to get the development tools onto your computer. Make sure you have a book handy, or a nice hike to go on. You're going to be doing some time consuming download and builds, and you'll want to have something healthier to do than sitting and staring at progress bars.

It's worth mentinoning that you'll need some serious hard drive space available. On my computer, my UE5 directory takes up 105GB. This includes both the source code and intermediate build files.

### Install Visual Studio Code
* [Download VS Code][vscode-download] and install it.

### Download UE5
There's a solid [official guide][ue-download] on downloading the UE source code. Some notes:

* **You do to have a GitHub account** and go through the steps of joining Unreal's developer group to get access to the source.
* **You do not have to fork the repo.** This would be a good idea if you want to get your hands dirty making changes to the UE source, so you can commit those changes to your fork. Personally I'm not there, and will cross that bridge if it comes up. Cloning from the main repo is fine.
* **Clone the `Release` branch.** Unless you have a specific, well researched reason to do otherwise. The release branch is always the latest official release.
* **Have some serious hard drive space available.** On my computer, my UE5 directory takes up 105GB. This includes both the source code and intermediate build files.
* **You should not download the source through the Epic Game launcher.** This installs the "rocket build", which is a stripped down version of the source code. You'll be tempted to do this to save hard drive space. It will "work", until you go to build some of your code and get some error that some API isn't accessible through the rocket build. It's also being phased out.

### Build UE5
There's also an official guide for [building UE from source][ue-build] that you should follow. This is pretty straight forward, and where your book/nice walk will come in handy.

One small note: The Linux directions have yet to be updated for UE5. Where the guide says to run `UE4Editor`, this binary has been renamed to `UnrealEditor` in UE5.

### Create a C++ Project
1. Launch UnrealEditor. (In Linux, this is `UE5/Engine/Binaries/Linux/UnrealEditor`). 
2. Go to the Games tab. Select Empty project. 
3. Under Project Defaults, choose C++, no starter content.
4. Choose your directory and name your project. I'll be using `MyProject` as an example project name.
5. Press Create.

### Configure UnrealEditor to Use VS Code
1. Go to `Edit > Editor Preferences...`
2. In the Editor Preferences window, go to `General > Source Code`
3. Select `Visual Studio Code` as your source code editor
4. (Optional) Press the `Set as Default` button to make VS Code your default IDE for future projects


[vscode-download]: https://code.visualstudio.com/Download
[ue-download]: https://docs.unrealengine.com/5.0/en-US/downloading-unreal-engine-source-code/
[ue-build]: https://docs.unrealengine.com/5.0/en-US/building-unreal-engine-from-source/