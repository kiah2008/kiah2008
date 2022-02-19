Using Visual Studio Code as an Android C++ editor

Personally I find Android Studio Clion based C++ editing somewhat irritating as it does not seem to integrate with even slightly complex CMakeLists.txt configurations properly leading to correctly specified include files being flagged as missing. This normally results in particularly annoying popup error windows which don’t go away and get in the way when you are editing.

Visual Studio Code combined with the default C++ plugin and the CMake plugin is much more configurable (see also). The following steps allow use of VS Code as an editor for Android C++ NDK files (I still do the build using Android Studio, ie VSCode is running in another desktop workspace, although you could probably configure VSCode to compile the library files too). In the following the NDK is assumed to reside at the default location for Linux eg /opt/android-sdk/ndk-bundle/, and VSCode is assumed to be started in the NDK source directory (eg the gradle default is {Project-Dir}/{module}/src/main/cpp, although personally I prefer changing it in the module build.gradle file using sourceSets).

Open the command palette and select ‘Edit user local cmake kits’, then add
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
