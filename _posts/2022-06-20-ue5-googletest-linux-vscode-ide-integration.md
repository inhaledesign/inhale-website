---
layout: post
title: "Integrating with VS Code"
series: "Unreal Engine 5: Getting Started with C++, GoogleTest, and VS Code"
series_order: 5
---

## {{ page.subtitle }}
The integration with UE still leaves a lot to be desired. For reasons we'll get into, IDE integration is a bit hard for UE and requires accepting that we can only ever do the best we can with what we've got (or can afford).

### Unreal Editor Settings
Now we can generate VS Code files from UnrealEditor.

1. In the main Editor window, go to `Tools > Refresh Visual Studio Code Project`

This generates a bunch of files in your project directory. The important ones to understand
* `MyProject.code-workspace` is a VS Code project file. It contains configuration information for your VS Code workspace.
* `.vscode` is a directory that contains configuration files for VS Code. You'll want to add this to `.gitignore`
	* `.vscode/launch.json` defines launch configurations. You'll see a whole slew of these listed in the Run and Debug panel dropdown. The main one you're going to concern yourself with is "Launch MyProject Editor (development)"
	* `.vscode/tasks.json` defines build tasks. You can manually run any of these tasks by going to `Terminal > Run Task`. This is handy for when you want to run a build clean.
	* `.vscode/c_cpp_properties.json` and the rest are used by IntelliSense to do code analysis.

You're going to want to refresh your project this way every time you add a new C++ file to your project. This ensures that your new files are included in the IntelliSense, avoiding errant red squiggles. Every time you do this, all of the above files get overwritten. We're going to need a way to configure our workspace without modifying these files.

### Workspace
What we're going to do is create our own workspace files to your workspace file. As you work on a project, VS Code ends up addings a lot of user specific settings to your workspace. To avoid polluting the repository, what I do is add `*.code-workspace` to gitignore. Then add two example workspace files to the repo:

* `MyProject.Linux.code-workspace.example`
	``` json
	{
		"folders": [
			{
				"name": "MyProject",
				"path": "."
			},
			{
				"name": "UE5",
				"path": "../../UE5"
			}
		],
		"extensions": {
			"recommendations": [
				"ms-vscode.cpptools",
				"ms-dotnettools.csharp"
			]
		},
		"launch": {
			"version": "0.2.0",
			"configurations": [
				{
					"name": "Run Tests",
					"request": "launch",
					"program": "${workspaceFolder:MyProject}/Binaries/Linux/Tests",
					"preLaunchTask": "Tests Build",
					"args": [],
					"type": "cppdbg",
					"cwd": "${workspaceFolder:UE5}"
				}
			]
		},
		"tasks": {
			"version": "2.0.0",
			"tasks": [
				{
					"label": "Tests Build",
					"group": "build",
					"command": "Engine/Build/BatchFiles/Linux/Build.sh",
					"args": [
						"Tests",
						"Development",
						"Linux",
						"${workspaceFolder:MyProject}/MyProject.uproject",
						"-waitmutex"
					]
					,
					"options": {
						"cwd": "${workspaceFolder:UE5}"
					},
					"problemMatcher": "$msCompile",
					"type": "shell"
				},
				{
					"label": "Tests Clean",
					"group": "build",
					"command": "Engine/Build/BatchFiles/Linux/Build.sh",
					"args": [
						"Tests",
						"Development",
						"Linux",
						"${workspaceFolder:MyProject}/MyProject.uproject",
						"-waitmutex",
						"-clean"
					],
					"problemMatcher": "$msCompile",
					"type": "shell",
					"options": {
						"cwd": "${workspaceFolder:UE5}"
					}
				}
			]
		}
	}
	```
* `MyProject.Win64.code-workspace.example`
	``` json
	{
		"folders": [
			{
				"name": "MyProject",
				"path": "."
			},
			{
				"name": "UE5",
				"path": "..\\..\\UE5"
			}
		],
		"launch": {
			"version": "0.2.0",
			"configurations": [
				{
					"name": "Run Tests",
					"request": "launch",
					"program": "${workspaceFolder:MyProject}\\Binaries\\Win64\\Tests.exe",
					"preLaunchTask": "Tests Build",
					"args": [],
					"type": "cppvsdbg",
					"cwd": "${workspaceFolder:UE5}"
				}
			]
		},
		"tasks": {
			"version": "2.0.0",
			"tasks": [
				{
					"label": "Tests Build",
					"group": "build",
					"command": "Engine\\Build\\BatchFiles\\Build.bat",
					"args": [
						"Tests",
						"Development",
						"Win64",
						"${workspaceFolder:MyProject}\\MyProject.uproject",
						"-waitmutex"
					]
					,
					"options": {
						"cwd": "${workspaceFolder:UE5}"
					},
					"problemMatcher": "$msCompile",
					"type": "shell"
				},
				{
					"label": "Tests Clean",
					"group": "build",
					"command": "Engine\\Build\\BatchFiles\\Clean.bat",
					"args": [
						"Tests",
						"Development",
						"Win64",
						"${workspaceFolder:MyProject}\\MyProject.uproject",
						"-waitmutex"
					],
					"problemMatcher": "$msCompile",
					"type": "shell",
					"options": {
						"cwd": "${workspaceFolder:UE5}"
					}
				}
			]
		}
	}
	```

Now, when someone clones the repo, they can copy these files without the `.example` extensions. The working copies will be ignored by git, so each person can add configurations as they see fit. 

Once you've added the appropriate configuration to your project workspace, you'll see a launch configuration named "Run Tests". This will build your Tests module and run the executable in the internal terminal.

### Test Adapters
In order to integrate the tests into the IDE, we're going to need to install a test adapter extension. I chose the [GoogleTest Adapter][gta-marketplace], as it's simple and reasonably well coded. It does have a couple quirks:

* When you tell it to run or refresh tests, it only reruns the executable and reparses the output. This means you'll only see test changes after you rebuild the Tests executable.
* There is an [open issue][gta-issue] that affects our configuration: GTA doesn't handle multiroot workspaces. It will only read the first folder listed in your workspace, and it cannot parse folder-scoped variables like `${workspaceFolder:UE5}`.

To get around these issues, make the following changes to your workspace file
* Make sure your project folder is always the first listed (i.e. before the UE5 folder).
* Replace all instances of `${workspaceFolder:UE5}` with your UE5 path.
* Replace all instances of`${workspaceFolder:MyProject}` with your project path.

### IntelliSense
If you're looking at your code in VS Code, you've undoubtedly noticed that VS Code is highlighting problems that are not problems. You've also noticed that code suggestions are both slow and often incomplete.

Dear Reader, I wish that I could tell you that I had five step--or even one hundred steps--to follow that would fix this. Unfortunately I've beat my head against this one for too many hours and I haven't found a good fix. All I can shed light on is why this darkness is so impenetrable.

* UnrealEngine's source code is *huge,* and IntelliSense does not handle large projects well. This causes simple things, like renaming a local variable, to trigger a search of the whole UE5 code base.
* When you build, Unreal Build Tool generates files that contain macros UE's macros. These generated files aren't available to IntelliSense. This will result in Unreal macros showing as problems, as those macros will be defined in a generated class. Running Generate Project Files can help with this, but doesn't always.

There's some hope that the situation will improve. As I mentioned, [Microsoft has released changes to IntelliSense][ms-unreal-improvements] that improve the situation, but it doesn't work with UE5 yet, and it remains to be seen if it will apply to VS Code.

### Summary
This is the army we have, and I think it's enough to go war with. I've put a lot of time into making sure that this works, and finding solutions worthy of a serious project.

If you want to make any suggestions or find any issues, please use this site's [issue tracker][inhale-issues].

[gta-marketplace]: https://marketplace.visualstudio.com/items?itemName=DavidSchuldenfrei.gtest-adapter
[gta-issue]: https://github.com/DavidSchuldenfrei/gtest-adapter/issues/55
[ms-unreal-improvements]: https://devblogs.microsoft.com/cppblog/18x-faster-intellisense-for-unreal-engine-projects-in-visual-studio-2022/
[rider]: https://www.jetbrains.com/lp/rider-unreal/
[inhale-issues]: contact.html