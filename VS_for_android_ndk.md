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



# Tasks

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
