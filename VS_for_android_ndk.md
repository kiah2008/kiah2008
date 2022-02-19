Using Visual Studio Code as an Android C++ editor

Personally I find Android Studio Clion based C++ editing somewhat irritating as it does not seem to integrate with even slightly complex CMakeLists.txt configurations properly leading to correctly specified include files being flagged as missing. This normally results in particularly annoying popup error windows which don’t go away and get in the way when you are editing.

Visual Studio Code combined with the default C++ plugin and the CMake plugin is much more configurable (see also). The following steps allow use of VS Code as an editor for Android C++ NDK files (I still do the build using Android Studio, ie VSCode is running in another desktop workspace, although you could probably configure VSCode to compile the library files too). In the following the NDK is assumed to reside at the default location for Linux eg /opt/android-sdk/ndk-bundle/, and VSCode is assumed to be started in the NDK source directory (eg the gradle default is {Project-Dir}/{module}/src/main/cpp, although personally I prefer changing it in the module build.gradle file using sourceSets).

Open the command palette and select ‘Edit user local cmake kits’, then add below:
take for example.
Users\name\AppData\Local\CMakeTools\cmake-tools-kits.json

```
{
 "name": "Android clang",
 "compilers": {
   "C": "/opt/android-sdk/ndk-bundle/toolchains/llvm/prebuilt/linux-x86_64/bin/clang",
   "CXX": "/opt/android-sdk/ndk-bundle/toolchains/llvm/prebuilt/linux-x86_64/bin/clang++"
 }
}
```

to the file (~/.local/share/CMakeTools/cmake-tools-kits.json on Linux).

In the NDK source directory create a **VSCode settings directory (.vscode/ in Linux)** if one does not already exist.

In the setting directory create a local setting file **settings.json** eg vscode/settings.json and add the following:

```
{
"cmake.configureSettings": {
   "CMAKE_TOOLCHAIN_FILE": "/opt/android-sdk/ndk-bundle/build/cmake/android.toolchain.cmake",
   "ANDROID_NDK": "/opt/android-sdk/ndk-bundle",
   "CMAKE_BUILD_TYPE": "Debug",
   "ANDROID_ABI": "x86_64",
   // "ANDROID_SYSROOT": "/opt/android-sdk/ndk-bundle/sysroot/",
   "ANDROID_NATIVE_API_LEVEL": "28",
   "ANDROID_HOST_TAG":  "linux-x86_64",
   "ANDROID_SYSROOT_ABI": "x86_64",
   "ANDROID_NDK_ABI_NAME": "x86_64",
   "ANDROID_ARCH_NAME": "x86_64"
}
}
```

The **ANDROID_HOST_TAG** tag should be **windows-x86_64 for Windows** (speaking under correction).

In the same setting directory create a c_cpp_properties.json file:
```
{
 "configurations": [
     {
         "name": "Android",
         "includePath": [
             "${workspaceFolder}/**",
             "/media/opt/android-sdk/ndk-bundle/sysroot/usr/include",
             "/media/opt/android-sdk/ndk-bundle/sources/cxx-stl/llvm-libc++/include"
         ],
         "defines": [],
         "compilerPath": "/opt/android-sdk/ndk-bundle/toolchains/llvm/prebuilt/linux-x86_64/bin/x86_64-linux-android28-clang",
         "cStandard": "c11",
         "cppStandard": "c++14",
         "intelliSenseMode": "clang-x64",
         "configurationProvider": "vector-of-bool.cmake-tools",
         "compileCommands": "${workspaceFolder}/build/compile_commands.json"
     }
 ],
 "version": 4
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

# reference
## ninja
Ninja
Ninja is a small build system with a focus on speed. It differs from other build systems in two major respects: it is designed to have its input files generated by a higher-level build system, and it is designed to run builds as fast as possible.

Why yet another build system?
Where other build systems are high-level languages Ninja aims to be an assembler.

Ninja build files are human-readable but not especially convenient to write by hand. (See the generated build file used to build Ninja itself.) These constrained build files allow Ninja to evaluate incremental builds quickly.

Should you use Ninja?
Ninja's low-level approach makes it perfect for embedding into more featureful build systems; see a list of existing tools. Ninja is used to build Google Chrome, parts of Android, LLVM, and can be used in many other projects due to CMake's Ninja backend.

## [Cross Compiling for Android with the NDK](https://cmake.org/cmake/help/latest/manual/cmake-toolchains.7.html#id22)

## [Android Cmake](https://developer.android.com/ndk/guides/cmake#command-line)
