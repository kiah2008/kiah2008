# 编译

使用cmake+ndk

cmake

```cmake
cmake_minimum_required(VERSION 3.0.0)
project(miniaudio VERSION 0.1.0)
#添加实现定义，否则不会参与编译
add_definitions("-DMINIAUDIO_IMPLEMENTATION")
#根据CMAKE_SYSTEM_NAME判断是否构建Andrid
if("${CMAKE_SYSTEM_NAME}" STREQUAL "Android") 
set(extra_libs log)
endif()
#见miniaudio.h拆分为miniaudio.h和miniaudio.c，并构建为静态库
add_library(${PROJECT_NAME}  STATIC
    miniaudio.c)

target_include_directories(${PROJECT_NAME} PUBLIC 
${PROJECT_SOURCE_DIR}/
)

target_link_libraries(${PROJECT_NAME} dl m ${extra_libs})

#添加编译控制项
option(SUPPORT_PLAYBACK "simple playback" ON)

add_subdirectory(examples)
```



## 适配

## 检测编译器指针长度

```c++
#if defined(__LP64__) || defined(_WIN64) || (defined(__x86_64__) && !defined(__ILP32__)) || defined(_M_X64) || defined(__ia64) || defined(_M_IA64) || defined(__aarch64__) || defined(_M_ARM64) || defined(__powerpc64__)
    #define MA_SIZEOF_PTR   8
#else
    #define MA_SIZEOF_PTR   4
#endif
```

## c文件版本号定义

```c
#define MA_STRINGIFY(x)     #x
#define MA_XSTRINGIFY(x)    MA_STRINGIFY(x)

#define MA_VERSION_MAJOR    0
#define MA_VERSION_MINOR    11
#define MA_VERSION_REVISION 8
#define MA_VERSION_STRING   MA_XSTRINGIFY(MA_VERSION_MAJOR) "." MA_XSTRINGIFY(MA_VERSION_MINOR) "." MA_XSTRINGIFY(MA_VERSION_REVISION)
```

## android log 适配

```c
#pragma once
#ifdef __cplusplus
extern "C" {
#endif

#ifdef ANDROID
#include <android/log.h>

#ifndef LOG_TAG
#define LOG_TAG "mini_audio"
#endif

#define LOGD(format, ...) __android_log_buf_print(LOG_ID_MAIN, ANDROID_LOG_DEBUG, LOG_TAG, format, ##__VA_ARGS__)
#else
#define LOGD(format, ...) printf(format, ##__VA_ARGS__)
#endif

#ifdef __cplusplus
}
#endif
```

## cmake脚本配置

```shell
if("${CMAKE_SYSTEM_NAME}" STREQUAL "Android") 
set(extra_libs log)
endif()

target_link_libraries(${PROJECT_NAME} dl m ${extra_libs})

```

## 编译平台判断

```c
    #ifdef __unix__
        #define MA_UNIX
        #if defined(__DragonFly__) || defined(__FreeBSD__) || defined(__NetBSD__) || defined(__OpenBSD__)
            #define MA_BSD
        #endif
    #endif
    #ifdef __linux__
        #define MA_LINUX
    #endif
    #ifdef __APPLE__
        #define MA_APPLE
    #endif
    #ifdef __ANDROID__
        #define MA_ANDROID
    #endif
    #ifdef __EMSCRIPTEN__
        #define MA_EMSCRIPTEN
    #endif
```

## visual code tasks 定义

```
{
    // See https://go.microsoft.com/fwlink/?LinkId=733558
    // for the documentation about the tasks.json format
    "version": "2.0.0",
    "tasks": [
        {
            "label": "push to devices",
            "type": "shell",
            "command": "adb push ./build/examples/${input:componentType} /data/local/tmp/; adb shell 'cd /data/local/tmp/&& chmod +x ${input:componentType} && ./${input:componentType} sine_s16_mono_48000.wav 2>&1'",
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
        },
        {
            "label": "run test 1",
            "type": "shell",
            "command": "adb shell 'cd /data/local/tmp/&& chmod +x ${input:componentType} && ./${input:componentType} sine_s16_mono_48000.wav 2>&1'",
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
        },
        {
            "label": "push res",
            "type": "shell",
            "command": "adb push ${workspaceFolder}/tests/_build/res/sine_s16_mono_48000.wav /data/local/tmp/",
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
    ],
    "inputs": [
        {
          "type": "pickString",
          "id": "componentType",
          "description": "What type of component do you want to create?",
          "options": [
            "simple_playback",
            "simple_playback_sine",
          ],
          "default": "component"
        }
    ]
}
```

