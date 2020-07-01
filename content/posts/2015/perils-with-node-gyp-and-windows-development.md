+++
author = "James D Hughes"
categories = ["npm", "Node.js", "Windows-sdk", "node-gyp", "javaonnode"]
date = 2015-04-10T05:04:49Z
description = ""
draft = false
slug = "perils-with-node-gyp-and-windows-development"
tags = ["npm", "Node.js", "Windows-sdk", "node-gyp", "javaonnode"]
title = "Windows, NodeJS, and node-gyp. The Battle and the Conquest"

+++


**Update!**

It has been brought to my attention that there is a another way to fix this solution that seems a bit simpler than my original approach.
See the comments from Cees Timmerman and Mark Pinder in the comments.

Step 1: Install Visual Studio community edition (2015 seems to be tried, tested and true). Ensure that you do a *custom* install and include the **C++** components.

Step 2: rebuild node-gyp using the following:
``` 
npm install -g --msvs_version=2015 node-gyp rebuild
```
The installations should complete now without issue.

The original blog entry is below. It still reflects some of my disdain for development in windows if you want to rah-rah around it :-D

-------------------------------------------------
Here's a depressing fact.
Lots of jobs that you're going to go to are going to strictly regimen Windows Development. Corporations love windows. If you're not in a start-up, you're probably going to be expected to work with this disastrous developers experience.
To us this translates into a lot of grumbling and then eventual compliance. It will be a valiant fight, but we soon realise that there are hills to die on and this probably isn't the one.

It was a glorious afternoon. I had recently deployed a neato MEAN application (mongo/express/angular/node) and it was running smoothly as could be.  Soon, as with any non-contract software development, came the demands. What else can we make this thing do!?
So before I knew it, I was generating Jasper Reports and, instead of hosting it on a Jasper Report server and using there Rest_v2, I decided to play with some neato Node libraries and set up an API to get reports instead!

This required a neat library that allows you to run Java in Node.js. It's called .... java.
Let's do it! Pop open my command prompt, navigate to my source code in the development environment
```
npm install java --save
```
Uh Oh!
```
gyp ERR! configure error
gyp ERR! stack Error: Can't find Python executable "python", you can set the PYTHON env variable.
```
Looks like I need python for in order for node-gyp to configure itself.
//installs python<br/>
//tries to install java again<br/>
WOAH!

```
ERROR: The system was unable to find the specified registry key or value.

Error: Command failed: C:\\WINDOWS\\system32\\cmd.exe /s /c \"reg query \"hklm\\softwa
re\\wow6432node\\javasoft\\java development kit""
ERROR: The system was unable to find the specified registry key or value.

gyp: Call to \'node findJavaHome.js\' returned exit status 1. while trying to load
 binding.gyp
gyp ERR! configure error
gyp ERR! stack Error: \`gyp\` failed with exit code: 1
```

Looks like I forgot I never actually use the JDK on Windows because.. It's the JDK on windows.
Let's try this again, shall we?
//installs JDK

```
npm install java --save
```
GRAR!
```
> node-gyp rebuild

C:\path\to\projectnode_modules\java>node "C:\Pro
gram Files\nodejs\node_modules\npm\bin\node-gyp-bin\\..\..\node_modules\node-gyp
\bin\node-gyp.js" rebuild
Building the projects in this solution one at a time. To enable parallel build,
please add the "/m" switch.
C:\Program Files (x86)\MSBuild\Microsoft.Cpp\v4.0\Microsoft.CppBuild.targets(29
7,5): warning MSB8003: Could not find WindowsSDKDir variable from the registry.
  TargetFrameworkVersion or PlatformToolset may be set to an invalid version nu
mber. [C:\path\to\projectnode_modules\java\buil
d\nodejavabridge_bindings.vcxproj]
C:\Program Files (x86)\MSBuild\Microsoft.Cpp\v4.0\Platforms\x64\Microsoft.Cpp.x
64.Targets(146,5): error MSB6006: "CL.exe" exited with code -1073741515. [C:\Us
ers\admin_jhughes\Documents\project\node_modules\java\build\nodejavabr
idge_bindings.vcxproj]
gyp ERR! build error
gyp ERR! stack Error: `C:\Windows\Microsoft.NET\Framework\v4.0.30319\msbuild.exe
` failed with exit code: 1
gyp ERR! stack     at ChildProcess.onExit (C:\Program Files\nodejs\node_modules\
npm\node_modules\node-gyp\lib\build.js:267:23)
gyp ERR! stack     at ChildProcess.emit (events.js:98:17)
gyp ERR! stack     at Process.ChildProcess._handle.onexit (child_process.js:820:
12)
gyp ERR! System Windows_NT 6.2.9200
gyp ERR! command "node" "C:\\Program Files\\nodejs\\node_modules\\npm\\node_modu
les\\node-gyp\\bin\\node-gyp.js" "rebuild"
gyp ERR! cwd C:\path\to\projectnode_modules\java

gyp ERR! node -v v0.10.36
gyp ERR! node-gyp -v v1.0.1
gyp ERR! not ok

npm ERR! java@0.4.6 install: `node-gyp rebuild`
npm ERR! Exit status 1
npm ERR!
npm ERR! Failed at the java@0.4.6 install script.
npm ERR! This is most likely a problem with the java package,
npm ERR! not with npm itself.
npm ERR! Tell the author that this fails on your system:
npm ERR!     node-gyp rebuild
npm ERR! You can get their info via:
npm ERR!     npm owner ls java
npm ERR! There is likely additional logging output above.
npm ERR! System Windows_NT 6.2.9200
npm ERR! command "C:\\Program Files\\nodejs\\\\node.exe" "C:\\Program Files\\nod
ejs\\node_modules\\npm\\bin\\npm-cli.js" "install" "java" "--save"
npm ERR! cwd C:\Users\admin_jhughes\Documents\project
npm ERR! node -v v0.10.36
npm ERR! npm -v 1.4.28
npm ERR! code ELIFECYCLE
npm ERR! not ok code 0
```
Here's where we get the suck. It caused me plenty of grief and here was my step for finding a solution.

######1. Compilers.
I've become so used to my ```sudo apt-get install build-essential``` that when I first experienced the requirements for Microsofts Visual Studio and C++ redistributables being required to compile C code that I felt my heart sink immensely.
I don't WANT to install visual studio.
I don't WANT to fight over which version of C++ redistributable packages I'm supposed to use.
I don't WANT to program on Windows.
But I did anyways. I tried installing Visual Studio 2013... to no avail. So I tried Visual Studio 2010 ... to no avail.
I decided to just try the C++ Redistributable library... to no avail.

Here's where I started getting annoyed and had to dig into the depths of some random forums.  It turns out that It wasn't the compiler and the error messages that were generated by npm really just lead me on a wild goose chase.

Realizing that this is already getting too wordy, I'll cut right to it.
My issue was: Windows x64, Node x86.
Bazing.
I had to download the [windows sdk](http://www.microsoft.com/en-ca/download/details.aspx?id=3138) in order to get node-gyp to run. 
Install it, then execute the following code to get your console into a compatibility mode.
```
"C:\Program Files\Microsoft SDKs\Windows\v7.1\bin\SetEnv.CMD" /Release /x64
```
All my text went green then I could execute
```
npm install java --save
```
and after a bunch of scary looking text, hazah I've got Java installed on NPM. After the libraries were installed I could start my Node server with no issues.

