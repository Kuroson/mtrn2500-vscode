# 1. Info before reading

**For linux/macos users only.**. All of them can be configured for windows however, you just need to change some of the command names etc

I recommend using WSL2 Ubuntu if you are using Windows anyway.

A decent guide made for CSE students can be found here: https://abiram.me/wsl-github. There are also many guides that can be found on Google.

It is meant for COMP1511 students studying C. Some of the steps should be skipped/changed (e.g., install g++ instead of dcc).

# 2. Using code runner

Code runner is a great way of compile **single** file applications and running them automatically. Minor configuration is needed

NOTE: You can configure it to compile and run multi-file applications, it may get confusing.

You can install it here for vscode: https://marketplace.visualstudio.com/items?itemName=formulahendry.code-runner

After installation, you should see a 'play' button on the top RHS of your editor.

You will want to include the following values in your `.vscode/settings.json` (either global or per root directory of each project file)

```json
{
  "code-runner.runInTerminal": true,
  "code-runner.executorMap": {
    "cpp": "clear && echo \"$(tput setaf 2)$(tput bold)Running$(tput setaf 1) $fileName$(tput setaf 2). Compiling using g++ -std=c++14$(tput sgr0)\" && cd $dir && g++ -std=c++14 $fileName -o $fileNameWithoutExt.out && $dir$fileNameWithoutExt.out && rm $fileNameWithoutExt.out"
  }
}
```

The first line ensures that code is run inside your integrated terminal (rather then some vscode terminal where you cant type anything to STDIN). I have also formatted it to be a bit colourful and descriptive. It also removes the compiled file as well.

An example of it running:

![example](/images/fancier-version.PNG)

The simple version:

```json
{
  "cpp": "cd $dir && g++ -std=c++14 $fileName -o $fileNameWithoutExt.out && $dir$fileNameWithoutExt.out"
}
```

An example of it being run:

![example](/images/simple-demo.PNG)

# 3. Using vscode's inbuilt debugger and auto-building

VSCode's F5 runs a compile command (located in `.vscode/task.json`) then runs the compiled program using gdb (config located in `.vscode/settings.json`). This is not ideal for running applications normally, however, we can setup the build command (`control+shift+b`).

## 3.1. `task.json`

You first need to configure the `task.json` file that is located in your `.vscode` folder directory.

I am using these settings:

```json
{
  "tasks": [
    {
      "type": "shell",
      "label": "g++ compile using C++14",
      "command": "g++",
      "args": [
        "-std=c++14",
        "-g",
        "-o",
        "${fileDirname}/${fileBasenameNoExtension}.out",
        "${fileDirname}/${fileBasename}"
      ],
      "group": {
        "kind": "build",
        "isDefault": true
      }
    }
  ],
  "version": "2.0.0"
}
```

The important stuff to understand is:

- `command` is what vscode uses to compile it
- `args`: args are arguments after the command, delimited by spaces by vscode
- `isDefault`: when you use `control+shift+b`, it will automatically run this task since its default. You can turn this off if you have multiple builds

NOTE: I changed the compiled output to have a `*.out`, so i can easily .gitignore it all

What it should look like when you run the build (`control+shift+b`):

![](/images/task-build.PNG)

## 3.2. `launch.json`

For the debug program (`.vscode/launch.json`), I use

```json
{
  // Use IntelliSense to learn about possible attributes.
  // Hover to view descriptions of existing attributes.
  // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
  "version": "0.2.0",
  "configurations": [
    {
      "name": "g++ - Build and debug active file",
      "type": "cppdbg",
      "request": "launch",
      "program": "${fileDirname}/${fileBasenameNoExtension}.out",
      "args": [],
      "stopAtEntry": false,
      "cwd": "${fileDirname}",
      "environment": [],
      "externalConsole": false,
      "MIMode": "gdb",
      "setupCommands": [
        {
          "description": "Enable pretty-printing for gdb",
          "text": "-enable-pretty-printing",
          "ignoreFailures": true
        }
      ],
      "preLaunchTask": "g++ compile using C++14",
      "miDebuggerPath": "/usr/bin/gdb"
    }
  ]
}
```

The most important values to note are:

- `program`: I have included an ".out" to match my output generated by my task-build file
- `args`: If your program uses args to run instead of STDIN, you input them here
- `preLaunchTask`: name has to match your task.json `label` value

By using F5, it will automatically compile your file and run it using gdb. You can add breakpoints on vscode (click the LHS of line numbers), and it will stop at those places.

What it should look like when run using `F5`,

![example](/images/debug-example.PNG)
