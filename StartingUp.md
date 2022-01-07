# Quick setup

**Disclaimer:**

>This is a very opinionated guide on how to get started fast with O3DE on Windows.
Most of the things should work on any other system, but some build-specific commands are Windows-only (like adding the /m for parallel compilations, etc.).

## Requirements

Make sure that you have **git lfs** installed by running

```
git lfs --version 
```

We will assume that you have the latest **Visual Studio 2022** installed.

You will need **CMake**.

For other requirements check the official [readme](https://github.com/o3de/o3de#build-requirements-and-redistributables).

## Folder structure

```
C:\o3de
C:\o3de\engines
C:\o3de\projects
C:\o3de\gems
```

- Make `o3de` a root project for everything O3DE related. Obviously you can have it anywhere in your system, I used `C:\` for simplification.
- Inside, use `engines` subfolder to store all (or just one) versions of the engine. For instance, you could have a development version, a couple of release versions, your forked version, etc.
- `projects` subfolder will contain all of your projects
- `gems` subfolder will contain all of your gems

## Getting the latest version of the engine

```

git clone https://github.com/o3de/o3de.git C:\o3de\engines\dev
```

This will get the latest version of the engine from the development branch. It can take some time for the project to download because it is 5GB+ big.

> Because it is a development version, it could have some issues. If you want to use a stable version, check the [release versions](https://github.com/o3de/o3de/releases). On the left side of the list of released versions, you will find a commit id for every release. For instance, version *2111.1* has an id *1da5ef1*. Copy this id and from the folder where you cloned the development version of the engine (C:\o3de\engines\dev) type the following in the console:
> ```
> git checkout 1da5ef1
> ```

If you want to update the repository with the latest code:

```
git checkout development
git pull
```

## Project centric installation

There are a couple of ways how to install the engine and create the projects, and the recommended way is to use project-centric / engine-prebuilt installation. For this, we need to build the engine first.

If you plan to use multiple engine versions, modify the `engine.json` in the `source` folder of the engine (i.e. `C:\o3de\engines\dev`) by setting the name of the engine, and if you want, you can also set the version.

Now register the engine:

```
C:\o3de\engines\dev\scripts\o3de.bat register --this-engine
```

### Configure and compile the engine

Now we need to configure the engine using `C:\o3de\engines\dev` as source and `C:\o3de\engines\dev-sdk` as destination.

```
C:\o3de\engines\cmake -S C:\o3de\engines\dev -B C:\o3de\engines\dev-sdk -G "Visual Studio 17"
```

Once that's done, let's build (compile) the `dev-sdk` version of the engine in a profile configuration (for development):

```
cmake --build C:\o3de\engines\dev-sdk\ --config profile --target INSTALL -- /m
```

### Create, configure and build the project

Now that the engine is prebuilt, we will create a project. Let's call it **DemoPlatformer**.

```
C:\o3de\engines\dev\scripts\o3de.bat create-project -pp C:\o3de\projects\DemoPlatformer
```

For more options on creating the project, run:

```
C:\o3de\engines\dev\scripts\o3de.bat create-project -h
```

Next, we need to configure the project:

```
cmake -S C:\o3de\projects\DemoPlatformer -B C:\o3de\projects\DemoPlatformer\build\win -G "Visual Studio 17" -DCMAKE_MODULE_PATH=C:\o3de\engines\dev\cmake
```

Now we will build it

```
cmake --build C:\o3de\projects\DemoPlatformer\build\win\ --config profile -- /m
```

After the build is done, you can go to `C:\o3de\projects\DemoPlatformer\build\win\bin\profile` and run the `Editor.exe` and you are ready to go. Also, you can open the solution file with Visual Studio from `C:\o3de\projects\DemoPlatformer\build\win\` folder and start coding.

However, if you plan to add some functionality with the code, the preferred way is to do it with gems.

### Create Gem

> Gem is like a plugin that can be added to one or multiple projects. With gem you can modify the engine, or add new functionalities to your projects, or it can just have some assets that you want to reuse on multiple projects.

To create a gem:

```
C:\o3de\engines\dev\scripts\o3de.bat create-gem -gp C:\o3de\gems\Platformer
```

Gem needs to be added to the project:

```
C:\o3de\engines\dev\scripts\o3de.bat enable-gem -pn DemoPlatformer -gn Platformer
```

Now build the project once more

```
cmake --build C:\o3de\projects\DemoPlatformer\build\win\ --config profile -- /m
```

This will update the project's cmake files and build the gem.

Once this is done, you can open the project solution file from `C:\o3de\projects\DemoPlatformer\build\win\` and you will find the gem files in `o3de\gems\Platformer\Code\Platformer.Static` project in solution explorer in Visual Studio.

> When you make any change to the gem, just rebuild the whole project. Only that gem will be rebuilt, so the compile times are really quick.

## Working with gems

### Adding new files to gem

The easiest way to work on gems is with Visual Studio. Once you have the gem project opened in Visual Studio, you can add new files to it through the Solution Explorer > Right click on Source folder > Add > New Item. When the dialog pops up, make sure that you use the correct path, because by default Visual Studio will try to save the file inside the projects folder, not the gem folder.

![Add new file](/assets/vs-add-new.png)

The correct path should be `C:\o3de\gems\Platformer\Code\Source`.

Once you have added a new file to the source folder, it needs to be added to `platformer_files.cmake` file aswell. You can find it in Solution Explorer right under the Source folder where you have added this new file.

Now that this is done, and you are happy with the code that you have added, simply rebuild the whole project - only the gem will be built, but it will be added to the project as well:

```
cmake --build C:\o3de\projects\DemoPlatformer\build\win\ --config profile -- /m
```

### Debugging the gem from Visual Studio

> Before starting with debugging, make sure that you use the **profile** solution configuration from the top menu in Visual Studio (by default it is set to debug).

Debugging is quite easy if you have done everything from above. In Visual Studio, open the Debug main menu, and then *ALL BUILD Debug Options*. This will open up a dialog where you need to select a *Configuration options* > *Debugging* from the left side list. Once you do that, on the right side you will see an option *Command* and in there type the path to the **Editor.exe** in your project:

```
Command = C:\o3de\projects\DemoPlatformer\build\win\bin\profile\Editor.exe
```

Now set the breakpoint somewhere in the code (it doesn't even have to be your code).

Once you do this, compile the project (if you have any changes since the last build), then click on the green arrow in the top menu in Visual Studio that says *Local Windows Debugger*. This will start the Editor and Visual Studio should stop on the breakpoint if the project reaches that part of the code.

# Final thoughts and ideas

Even though you can compile the whole project from Visual Studio, I find it easier to do it from the console, simply because each time you compile it with Visual Studio, the solution file will be changed and the Visual Studio will ask you to reload the whole project. You don't get this extra step if you compile it from the onsole.

For this reason, it makes sense to create some simple batch file where you could just pass the project name and it will resolve the correct path and build the project for you, something like:

```
C:\o3de\scripts\build-project.ps1 DemoPlatformer
```

# Additional resources

For more information on building and configuring the engine and projects, check out the following videos:

- [Workshop: Build the Engine and Project - Esteban Papp, Amazon](https://www.youtube.com/watch?v=p7zNjS6-87o&list=PLCQwFpnHSZQgkw7MdNQwagKKKvywt6b80&index=2)
- [Workshop: Core Libs & O3DE SDK - Derric McGarrah, Amazon](https://www.youtube.com/watch?v=67EymkpF_Ow&list=PLCQwFpnHSZQgkw7MdNQwagKKKvywt6b80&index=7)

