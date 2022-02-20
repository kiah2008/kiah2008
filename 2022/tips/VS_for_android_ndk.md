Using Visual Studio Code as an Android C++ editor

Visual Studio Code combined with the default C++ plugin and the CMake plugin is much more configurable (see also). The following steps allow usingof VS Code as an editor for Android C++ NDK files (I still do the build using Android Studio, ie VSCode is running in another desktop workspace, although you could probably configure VSCode to compile the library files too). 

# Pre-requires

- install cmake

- install ninja

- install ndk

- install vs code


# configure for cmake toolkits

make sure you had install softwares required, especially for ninja when using windows.

Open the command palette and select ‘Edit user local cmake kits’, then add below:

## windows:

***\path\to\Users\AppData\Local\CMakeTools\cmake-tools-kits.json***

```
  {
    "name": "Android Clang",
    "compilers": {
      "C": "/path/to/ndk/21.4.7075529/toolchains/llvm/prebuilt/windows-x86_64/bin/clang.exe",
      "CXX": "/path/to/ndk/21.4.7075529/toolchains/llvm/prebuilt/windows-x86_64/bin/clang++.exe"
    },
    "environmentVariables": {
      "ANDROID_NDK": "/path/to/ndk/21.4.7075529/"
    }
  },
```

to the file (~/.local/share/CMakeTools/cmake-tools-kits.json on Linux).

In the following the NDK is assumed to reside at the default location for Linux eg /opt/android-sdk/ndk-bundle/, and VSCode is assumed to be started in the NDK source directory (eg the gradle default is {Project-Dir}/{module}/src/main/cpp, although personally I prefer changing it in the module build.gradle file using sourceSets).

create settings.json. open command palette and input "preference: open workspace settings".

add the following:

```
{
    "cmake.additionalKits": [
        "Android-Clang"
    ],
    "cmake.configureArgs": [
        "-DCMAKE_TOOLCHAIN_FILE=${env:ANDROID_NDK}/build/cmake/android.toolchain.cmake",
        "-DCMAKE_ANDROID_ARCH_ABI=arm64-v8a",
        "-DANDROID_ABI=arm64-v8a",
        "-DCMAKE_SYSTEM_VERSION=28",
        "-DANDROID_PLATFORM=android-28"
    ]
}
```

Open VSCode in the NDK source directory and select the ‘Android clang’ toolchain.

#  [Cross Compiling for Android with the NDK](https://cmake.org/cmake/help/latest/manual/cmake-toolchains.7.html#id22)

For example, a toolchain file might contain:
```
set(CMAKE_SYSTEM_NAME Android)
set(CMAKE_SYSTEM_VERSION 21) # API level
set(CMAKE_ANDROID_ARCH_ABI arm64-v8a)
set(CMAKE_ANDROID_NDK /path/to/android-ndk)
set(CMAKE_ANDROID_STL_TYPE gnustl_static)
```
Alternatively one may specify the values without a toolchain file:
```
$ cmake ../src \
  -DCMAKE_SYSTEM_NAME=Android \
  -DCMAKE_SYSTEM_VERSION=21 \
  -DCMAKE_ANDROID_ARCH_ABI=arm64-v8a \
  -DCMAKE_ANDROID_NDK=/path/to/android-ndk \
  -DCMAKE_ANDROID_STL_TYPE=gnustl_static
```

build hello-jni 范例
```
Executable : ${HOME}/Android/Sdk/cmake/3.10.2.4988404/bin/cmake
arguments :
-H${HOME}/Dev/github-projects/googlesamples/ndk-samples/hello-jni/app/src/main/cpp
-DCMAKE_FIND_ROOT_PATH=${HOME}/Dev/github-projects/googlesamples/ndk-samples/hello-jni/app/.cxx/cmake/universalDebug/prefab/armeabi-v7a/prefab
-DCMAKE_BUILD_TYPE=Debug
-DCMAKE_TOOLCHAIN_FILE=${HOME}/Android/Sdk/ndk/22.1.7171670/build/cmake/android.toolchain.cmake
-DANDROID_ABI=armeabi-v7a
-DANDROID_NDK=${HOME}/Android/Sdk/ndk/22.1.7171670
-DANDROID_PLATFORM=android-23
-DCMAKE_ANDROID_ARCH_ABI=armeabi-v7a
-DCMAKE_ANDROID_NDK=${HOME}/Android/Sdk/ndk/22.1.7171670
-DCMAKE_EXPORT_COMPILE_COMMANDS=ON
-DCMAKE_LIBRARY_OUTPUT_DIRECTORY=${HOME}/Dev/github-projects/googlesamples/ndk-samples/hello-jni/app/build/intermediates/cmake/universalDebug/obj/armeabi-v7a
-DCMAKE_RUNTIME_OUTPUT_DIRECTORY=${HOME}/Dev/github-projects/googlesamples/ndk-samples/hello-jni/app/build/intermediates/cmake/universalDebug/obj/armeabi-v7a
-DCMAKE_MAKE_PROGRAM=${HOME}/Android/Sdk/cmake/3.10.2.4988404/bin/ninja
-DCMAKE_SYSTEM_NAME=Android
-DCMAKE_SYSTEM_VERSION=23
-B${HOME}/Dev/github-projects/googlesamples/ndk-samples/hello-jni/app/.cxx/cmake/universalDebug/armeabi-v7a
-GNinja
jvmArgs :

Build command args: []
Version: 1
```



# 定义Tasks

| Task属性     | 语义                                                |
| ------------ | --------------------------------------------------- |
| label        | 在用户界面上展示的Task标签                          |
| type         | Task类型：分为shell和process                        |
| command      | 真正执行的命令                                      |
| windows      | Windows中的特定属性，在Windows中覆盖默认定义的属性  |
| group        | 定义Task属于哪一组                                  |
| presentation | 定义用户界面如何处理Task的输出                      |
| options      | 定义cwd（当前工作目录）、env（环境变量）和shell的值 |
| runOptions   | 定义Task何时运行及如何运行。                        |



```
{
    // See https://go.microsoft.com/fwlink/?LinkId=733558
    // for the documentation about the tasks.json format
    "version": "2.0.0",
    "tasks": [
        {
            "label": "run",
            "type": "shell",
            "command": "adb push ./build/test_ndk /data/local/tmp/; adb shell 'cd /data/local/tmp/&& chmod +x test_ndk && ./test_ndk'",
            "problemMatcher": [],
            "group": {
                "kind": "test",
                "isDefault": true
            },
            "presentation": {
                "echo": false,
                "reveal": "always",
                "focus": false,
                "panel": "shared",
                "showReuseMessage": true,
                "clear": true
            }
        }
    ]
}
```

## 预定义变量

1.  支持以下预定义变量：
   `${workspaceRoot}` -VSCode中打开文件夹的路径 the path of the folder opened in VS Code
   `$ {workspaceFolder}` -在VS Code中打开的文件夹的路径
   `$ {workspaceFolderBasename}`-在VS Code中打开的文件夹名称，不带任何斜杠（/）
   `$ {file}` -当前打开的文件
   `$ {relativeFile}` -相对于当前打开的文件workspaceFolder
   `$ {relativeFileDirname}` -相对于当前打开的文件的目录名workspaceFolder
   `$ {fileBasename}` -当前打开的文件的基本名称
   `$ {fileBasenameNoExtension}`-当前打开的文件的基本名称，没有文件扩展名
   `$ {fileDirname}`-当前打开的文件的目录名
   `$ {fileExtname}` -当前打开的文件的扩展名
   `$ {cwd}` -启动时任务运行器的当前工作目录
   `$ {lineNumber}` -活动文件中当前选择的行号
   `$ {selectedText}` -活动文件中的当前选定文本 

    `$ {execPath}`-正在运行的VS Code可执行文件的路径
   `$ {defaultBuildTask}` -默认构建任务的名称 

2. Demo
    ```
    $ {workspaceFolder} -/home/your-username/your-project
    $ {workspaceFolderBasename} -your-project
    $ {file} -/home/your-username/your-project/folder/file.ext
    $ {relativeFile} -folder/file.ext
    $ {relativeFileDirname} -folder
    $ {fileBasename} -file.ext
    $ {fileBasenameNoExtension} -file
    $ {fileDirname} -/home/your-username/your-project/folder
    $ {fileExtname} -.ext
    $ {lineNumber} -光标的行号
    $ {selectedText} -在代码编辑器中选择的文本
    $ {execPath} -Code.exe的位置
    ```

1. 每个工作空间文件夹范围内的变量
   通过将根文件夹的名称附加到变量（用冒号分隔），可以进入工作空间的同级根文件夹。如果没有根文件夹名称，则该变量的作用域为使用该文件夹的相同文件夹。

   例如，在具有文件夹`Server`和`Client`的多根工作区中，`${workspaceFolder:Client}`表示Client根的路径。

## 环境变量

您也可以通过`$ {env：Name}`语法引用环境变量（例如`${env:USERNAME}`）。

```
{
  "type": "node",
  "request": "launch",
  "name": "Launch Program",
  "program": "${workspaceFolder}/app.js",
  "cwd": "${workspaceFolder}",
  "args": ["${env:USERNAME}"]
}
```

## 配置变量

您可以通过`$ {config：Name}`语法（例如，`${config:editor.fontSize}`）来引用VS Code设置（也称为“`configurations`” ）。

## 命令变量

如果上面的预定义变量不足，则可以通过`$ {command：commandID}`语法将任何VS Code命令用作变量。

插入命令变量后，将运行命令，并用命令的（字符串）结果替换该变量。命令的实现范围从无UI的简单计算到基于VS Code扩展API可用的UI功能的一些复杂功能。

## 输入变量

命令变量已经很强大，但是它们缺少一种机制来配置针对特定用例运行的命令。例如，不可能将提示消息或默认值传递给通用的“用户输入提示”。

此限制的解决，是输入变量具有的语法：`${input:variableID}`。所述variableID指`launch.json`和`tasks.json`中inputs的条目部分，其中指定了额外的配置属性。

以下示例显示了task.json使用输入变量的的总体结构：

```
{
  "version": "2.0.0",
  "tasks": [
    {
      "label": "task name",
      "command": "${input:variableID}"
      // ...
    }
  ],
  "inputs": [
    {
      "id": "variableID",
      "type": "type of input variable"
      // type specific configuration attributes
    }
  ]
}

```

当前，VS Code支持三种类型的输入变量：

- promptString：显示一个输入框，用于从用户那里获取字符串。
- pickString：显示一个“快速选择”下拉列表，使用户可以从多个选项中进行选择。
- command：运行任意命令。
  每种类型都需要其他配置属性：

1. promptString：
   - 描述：快速输入中显示的内容为输入提供了上下文。
   - default：如果用户未输入其他内容，将使用的默认值。
2. pickString：

- 描述：快速选择中显示的内容为输入提供了上下文。
- options：供用户选择的选项数组。
- default：如果用户未输入其他内容，将使用的默认值。它必须是选项值之一。

1. Command：

- command：命令正在变量插补上运行。
- args：传递到命令的实现的可选选项包。
  以下是的示例tasks.json，说明了使用inputsAngular CLI 的用法：

```
{
  "version": "2.0.0",
  "tasks": [
    {
      "label": "ng g",
      "type": "shell",
      "command": "ng",
      "args": ["g", "${input:componentType}", "${input:componentName}"]
    }
  ],
  "inputs": [
    {
      "type": "pickString",
      "id": "componentType",
      "description": "What type of component do you want to create?",
      "options": [
        "component",
        "directive",
        "pipe",
        "service",
        "class",
        "guard",
        "interface",
        "enum",
        "enum"
      ],
      "default": "component"
    },
    {
      "type": "promptString",
      "id": "componentName",
      "description": "Name your component.",
      "default": "my-new-component"
    }
  ]
}
```

运行示例：

 ![在这里插入图片描述](./VS_for_android_ndk.gif) 

# 输入示例

下面的示例演示如何在调试配置中使用command类型的用户输入变量，该变量使用户可以从特定文件夹中找到的所有测试用例的列表中选择一个测试用例。假定某个扩展提供了一个`extension.mochaSupport.testPicker``命令，该命令可将所有测试用例放置在可配置的位置，并显示一个选择器UI来选择其中的一个。命令输入的参数由命令本身定义。

```
{
  "configurations": [
    {
      "type": "node",
      "request": "launch",
      "name": "Run specific test",
      "program": "${workspaceFolder}/${input:pickTest}"
    }
  ],
  "inputs": [
    {
      "id": "pickTest",
      "type": "command",
      "command": "extension.mochaSupport.testPicker",
      "args": {
        "testFolder": "/out/tests"
      }
    }
  ]
}
1234567891011121314151617181920
```

命令输入也可以与任务一起使用。在此示例中，使用了内置的Terminate Task命令。它可以接受参数以终止所有任务。

```
{
  "version": "2.0.0",
  "tasks": [
    {
      "label": "Terminate All Tasks",
      "command": "echo ${input:terminate}",
      "type": "shell",
      "problemMatcher": []
    }
  ],
  "inputs": [
    {
      "id": "terminate",
      "type": "command",
      "command": "workbench.action.tasks.terminate",
      "args": "terminateAll"
    }
  ]
}
12345678910111213141516171819
```

# 常见问题

1. 用户和工作区设置中是否支持变量替换？
   预定义变量在选定数量的设定键的被支持`settings.json的文件，如终端cwd，env，shell和shellArgs的值。一些[settings](https://code.visualstudio.com/docs/getstarted/settings)像`window.title`一样有自己的变量：

```
  "window.title": "${dirty}${activeEditorShort}${separator}${rootName}${separator}${appName}"
1
```

请参阅“设置”编辑器（`Ctrl +，`）中的注释，以了解有关设置特定变量的信息。

1. 为什么没有记录`$ {workspaceRoot}？`
   `${workspaceRoot}`不建议使用该变量，`${workspaceFolder}`以便更好地与`[Multi-root Workspace](https://code.visualstudio.com/docs/editor/multi-root-workspaces)`支持保持一致。
2. 我如何知道变量的实际值？
   检查变量运行时值的一种简单方法是创建VS Code 任务，以将变量值输出到控制台。例如，要查看的解析值`${workspaceFolder}`，您可以在以下目录中创建并运行以下简单的“ echo”任务（“ 终端” > “运行任务”）`tasks.json`：

```
{
  "version": "2.0.0",
  "tasks": [
    {
      "label": "echo",
      "type": "shell",
      "command": "echo ${workspaceFolder}"
    }
  ]
}
```



# Reference

## Ninja
Ninja is a small build system with a focus on speed. It differs from other build systems in two major respects: it is designed to have its input files generated by a higher-level build system, and it is designed to run builds as fast as possible.

Why yet another build system?
Where other build systems are high-level languages Ninja aims to be an assembler.

Ninja build files are human-readable but not especially convenient to write by hand. (See the generated build file used to build Ninja itself.) These constrained build files allow Ninja to evaluate incremental builds quickly.

Should you use Ninja?
Ninja's low-level approach makes it perfect for embedding into more featureful build systems; see a list of existing tools. Ninja is used to build Google Chrome, parts of Android, LLVM, and can be used in many other projects due to CMake's Ninja backend.

[download binary](https://github.com/ninja-build/ninja/releases)

## [Cross Compiling for Android with the NDK](https://cmake.org/cmake/help/latest/manual/cmake-toolchains.7.html#id22)

## [Android offical Cmake Guide](https://developer.android.com/ndk/guides/cmake#command-line)
