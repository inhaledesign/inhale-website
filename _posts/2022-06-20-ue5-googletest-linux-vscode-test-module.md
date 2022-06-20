---
layout: post
title: "Creating a Test Module"
series: "Unreal Engine 5: Getting Started with C++, GoogleTest, and VS Code"
series_order: 3
---

## {{ page.subtitle }}
Now that we've got some tools and an empty project, we're going to set up our testing framework.

For now, we're going to use VS Code to create and edit code files, but actually compile from the command line. In the next section, we'll talk about how to use all of this through the IDE.

I'd be remiss to not emphasize how indebted I am to [Eric Lemes' "Unit Tests in Unreal" guide.][eric-lemes] I never would have figured this setup out if he hadn't given me a place to start.

### Add GoogleTest
The first thing we're going to do is add GoogleTest to our project.

1. Make a source directory for the module: `MyProject/Source/ThirdParty/GoogleTestModule`
2. [Download the latest GoogleTest release][github-googletest] (1.11.0 at the time of this writing)
3. Unzip the source to create `MyProject/Source/ThirdParty/GoogleTestModule/googletest-release-1.11.0`
4. Create a build file for the module:
`MyProject/Source/ThirdParty/GoogleTestModule/GoogleTestModule.Build.cs`
	``` csharp
	using UnrealBuildTool;
	using System.IO;

	namespace UnrealBuildTool.Rules
	{

	    public class GoogleTestModule : ModuleRules
	    {
	        public GoogleTestModule(ReadOnlyTargetRules Target) : base(Target)
	        {
	            Type = ModuleType.External;
	            PCHUsage = ModuleRules.PCHUsageMode.UseExplicitOrSharedPCHs;

	            string googleTestBasePath = Path.Combine(ModuleDirectory, "googletest-release-1.11.0");
	            PublicSystemIncludePaths.Add(Path.Combine(googleTestBasePath, "googlemock"));
	            PublicSystemIncludePaths.Add(Path.Combine(googleTestBasePath, "googlemock", "src"));
	            PublicSystemIncludePaths.Add(Path.Combine(googleTestBasePath, "googlemock", "include"));
	            PublicSystemIncludePaths.Add(Path.Combine(googleTestBasePath, "googletest"));
	            PublicSystemIncludePaths.Add(Path.Combine(googleTestBasePath, "googletest", "src"));
	            PublicSystemIncludePaths.Add(Path.Combine(googleTestBasePath, "googletest", "include"));

	 
	            if (Target.Platform == UnrealTargetPlatform.Win64) {
	                PublicDefinitions.Add("GTEST_OS_WINDOWS=1");
	            }
	            else if (Target.Platform == UnrealTargetPlatform.Mac) {
	                PublicDefinitions.Add("GTEST_OS_MAC=1");
	            }
	            else if (Target.IsInPlatformGroup(UnrealPlatformGroup.Unix)) {
	                PublicDefinitions.Add("GTEST_OS_LINUX=1");
	            }

	            PublicDefinitions.Add("__GXX_EXPERIMENTAL_CXX0X__=0");
	            PublicDefinitions.Add("WIN32_LEAN_AND_MEAN=1");

	            // There are more configurations you may want to use. For brevity, I've included just the bare minimum.
	        }
	    }
	}
	```

Some notes:
* We've added includepaths for GoogleTest in this file. As previously noted, this is the proper place to add to IncludePath. Doing so within VS Code will cause headches, no matter what the little light bulb suggests.
* Internally, GoogleTestuses macro definitions for configuration. This will come up when we configure the Tests target.
* I've chosen to name the module `GoogleTestModule` instead of `GoogleTest` because the latter is the same name as [UE's included GoogleTest module.][ue-googletest] Calling them both `GoogleTest` doesn't cause any namespace collisions, but I'd rather err on the side of clarity.

### Create a Module
We're trying to separate some of our code from the main project module so we can unit test it. So let's create a module so we can test it.
1. Make source directories for your module:
	* `MyProject/Source/MyModule/private`
	* `MyProject/Source/MyMoudule/public`
2. Create a build file for the module:
`MyProject/Source/MyModule/MyModule.Build.cs`
	``` csharp
	using UnrealBuildTool;
	using System.IO;

	namespace UnrealBuildTool.Rules
	{

	public class MyModule : ModuleRules
	{
		public MyModule(ReadOnlyTargetRules Target) : base(Target)
		{
			bAddDefaultIncludePaths = true;
			PCHUsage = PCHUsageMode.UseExplicitOrSharedPCHs;
			PublicDependencyModuleNames.AddRange(new string[] { "Core" });
		}
	}
	}
	```

We're only including UE's Core module as a dependency. Core will allow us to utilize the fundamental types of UE, like `TArray` and `FVector`. Depending on any other UE module will likely break things. Not only will our GoogleTests not compile, but our module will no longer be nice and decoupled.

3. Add a C++ class to the module, so we have a subject to test:
`MyModule/public/MyPolygon.h`
	``` c++
	#pragma once
	#include "CoreMinimal.h"

	class MyModule_API MyPolygon : {
	    int Size;
	    TArray<FVector> Vertices;

	public:
	    MyPolygon();

	    MyPolygon(int VertexCount) {
	    	Size = VertexCount;
	    	Vertices.Init(FVector(0.0), VertexCount);
	    }

	    int GetSize() { return Size; }
	    
	    FVector GetVertex(int Index) { return Vertices[Index]; }
	    
	    void SetVertex(int Index, double X, double Y) {
	        FVector& Vertex = Vertices[Index];
	        Vertex.X = X;
	        Vertex.Y = Y;
	    }
	};
	```

### Create a Tests Module
Now we're going to create a module named Tests that contains tests of MyModule. It will compile to a standalone executable that runs the tests.

1. Create a directory for the Tests module: 
	* Root of the Tests module: `MyProject/Source/Tests/`
	* Unit tests for MyModule will live in: `MyProject/Source/Tests/MyModule`
2. Create a build file for the Tests module:
`MyProject/Source/Tests/Tests.Build.cs`
	``` cs
	using UnrealBuildTool;
	using System.IO;

	namespace UnrealBuildTool.Rules
	{

	public class Tests : ModuleRules
	{
		public Tests(ReadOnlyTargetRules Target) : base(Target)
		{
			bAddDefaultIncludePaths = true;
			
			PCHUsage = PCHUsageMode.UseExplicitOrSharedPCHs;
			PublicDependencyModuleNames.AddRange(new string[] { "Core", "GoogleTestModule" });
			PrivateDependencyModuleNames.AddRange(new string[] { "MyModule" });
		}
	}
	}
	```

3. Create an entry point for the program:
`MyProject/Source/Tests/Tests.cpp`
	``` c++
	// 	These two includes are provided by GoogleTest. They contain a boiletplate `void main()` for our test application.
	#include "gtest_main.cc"
	#include "gtest-all.cc"
	```

4. Create a test file:
`MyProject/Source/Tests/MyModule/MyPolygonTest.cpp`
	``` c++
	#include "MyPolygon.h"
	#include "gtest/gtest.h"

	class MyPolygonTest : public ::testing::Test {};

	TEST(MyPolygonTest, "Size") {
		MyPolygon Triangle(3);
		EXPECT_EQ(Triangle.GetSize(), 3);
	}
	```

Note that we don't have do anything special to tell GoogleTest where our tests live. The `TEST` macro does this for us.

### Create a Tests Target
Create a build target for our test executable:
`MyProject/Source/Tests.Target.cs`
``` csharp
using UnrealBuildTool;

 
public class TestsTarget : TargetRules
{
    public TestsTarget(TargetInfo Target) : base(Target)
    {
    	DefaultBuildSettings = BuildSettingsVersion.V2;

    	// Make this target a standalone console executable
        Type = TargetType.Program;
        bIsBuildingConsoleApplication = true;
        LinkType = TargetLinkType.Modular;

        // Program will look for main() in our Tests module
        LaunchModuleName = "Tests";

        // Reduce the build time for this target
        bCompileAgainstEngine = false;
        bCompileAgainstCoreUObject = false;

        // Keeps macro definitions in Tests.Build.cs from throwing errors.
        // Note: The space at the beginning of the string is necessary, otherwise the first argument is ignored.
        if(Platform == UnrealTargetPlatform.Linux) {
            AdditionalCompilerArguments=" -Wno-error=macro-redefined -Wno-error=undef";
        }
    }
}
```

The `AdditionalCompilerArguments` for Linux are worth unpacking. Recall that in our `GoogleTestModule.Build.cs`, we configure GoogleTest using a bunch of macro definitions. UBT treats a lot of warnings as errors. On Linux, this includes macro redefinitions and macro undefineds, which causes the GooleTestModule build to fail. We need to pass in compiler arguments to not treat those warnings as errors for this target.

### UProject
Add your modules to your UProject file:
`MyProject.uproject`
``` json
"Modules:" [
	{
		"Name": "MyModule",
		"Type": "Runtime",
		"LoadingPhase": "Default"
	},
	{
		"Name": "Tests",
		"Type": "Program",
		"TargetAllowList": [ 
			"Program"
		]
	},
	{
		"Name": "GoogleTestModule",
		"Type": "Program",
		"TargetAllowList": [
			"Program"
		]
	}
]
```

### Build/Run Scripts
We're finally ready to run something.  I like to keep BuildTests and RunTests scripts in the root of my project directory:
* `BuildTests.sh`
	``` bash
	#! /bin/bash

	UEDir=../../UE5
	buildScript=$UEDir/Engine/Build/BatchFiles/Linux/Build.sh
	projectFile=ShapeArt.uproject
	testTarget=Tests
	build="$buildScript $testTarget Development Linux $PWD/$projectFile -waitmutex"

	echo $build
	$build
	```

* `RunTests.sh`
	``` bash
	#! /bin/bash

	./Binaries/Linux/Tests
	```

* `BuildTests.bat`
	``` batchfile
	@echo off

	setlocal enableDelayedExpansion

	set UEDir=..\..\UE5
	set buildScript=!UEDir!\Engine\Build\BatchFiles\Build.bat
	set projectDir=%~dp0
	set projectFile=ShapeArt.uproject
	set testTarget=Tests
	set build="!buildScript!" "!testTarget!" Development Win64 "!projectDir!!projectFile!" -waitmutex

	call !build!
	```
* `RunTests.bat`
	``` batchfile
	@echo off
	setlocal enableDelayedExpansion

	set testExecutable=".\Binaries\Win64\Tests.exe"

	call !testExecutable!
	```

Building will create an executable in the `MyProject/Binaries` folder. Running that executable from the command line will output the results of our TestsModule tests.

Congratulations! It's been a bit of a slog, but you've made it.

### Addendums
#### Other Methods of Adding GoogleTest
It's worth noting that there are alternative ways to add GoogleTest to your project.

1. [Compile GoogleTest as a shared library][googletest-shared-library].
2. Use UE's [included GoogleTest module][ue-googletest]

I personally ruled out (1) because I found it difficult to execute. I ruled (2) out because if a new version of GoogleTest comes out, I couldn't use it until the UE module gets updated.

But reasonable people disagree, and you may find reasons to prefer these options.

#### Git Submodule
Instead of downloading the GoogleTest source, you can add it as a [git submodule.][git-submodules]

[eric-lemes]: https://ericlemes.com/2018/12/12/unit-tests-in-unreal-pt-1/
[git-submodules]: https://git-scm.com/book/en/v2/Git-Tools-Submodules
[github-googletest]: https://github.com/google/googletest/releases
[googletest-shared-library]: https://blog.zuru.tech/coding/2022/03/30/integration-of-googletest-in-ue4-as-a-shared-library
[ue-googletest]: https://github.com/EpicGames/UnrealEngine/tree/release/Engine/Source/ThirdParty/GoogleTest