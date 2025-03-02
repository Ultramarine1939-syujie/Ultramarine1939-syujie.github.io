---
layout: post
title: VsCode-C++语言环境配置
subtitle: Each post also has a subtitle
categories: Vscode
tags: Vscode
---
# VsCode C++语言配置

## windows版

1、在应用程序拓展商店中下载安装 C/C++ Extension Pack 它包含了 vscode 编写 C/C++ 工程需要的插件，和以前比不需要一个个找了

2、windows 端编译运行C/C++的程序需要一套集成开发环境，这里可以使用 MinGW https://nuwen.net/mingw.html ，选择自己需要的安装包安装即可

3、下载MinGW的安装包，安装并配置MinGW的bin目录到系统环境变量(Path)。

![2](https://github.com/Ultramarine1939-syujie/Ultramarine1939-syujie.github.io/blob/main/_posts/images/2.png)

4、命令行gcc -v、g++ -v 验证环境变量是否配置成功

![1](https://github.com/Ultramarine1939-syujie/Ultramarine1939-syujie.github.io/blob/main/_posts/images/1.png)

5、安装好插件之后，先写一个简单的 cpp 文件，直接运行debug，可以让编辑器自己去创建 lanch.json 和 task.json 配置文件

6、这里选择C++(GDB/LLDB)、不要用C++(windows），debug运行的是 windows 自带的 cmd（同时要注意不要用中文路径名）

7、程序一闪而过的解决方法

```c
#include <stdlib.h>
system("pause");
```

8、两份json文件

launch.json

```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "(Windows) Launch",
            "type": "cppvsdbg",
            "request": "launch",
            "program": "cmd",
            "preLaunchTask": "echo",
            "args": [
                "/C",
                "${fileDirname}\\${fileBasenameNoExtension}.exe",
                "&",
                "echo.",
                "&",
                "pause"
            ],
            "stopAtEntry": false,
            "cwd": "${workspaceFolder}",
            "environment": [],
            // 之前是 "externalConsole":true，该字段已弃用
            "console":"externalTerminal"
        },
        {
            "name": "(gdb) Launch",
            "type": "cppdbg",
            "request": "launch",
            "program": "${workspaceFolder}/${fileBasenameNoExtension}.exe",
            "args": [],
            "stopAtEntry": false,
            "cwd": "${workspaceFolder}",
            "environment": [],
            "externalConsole": true,
            "MIMode": "gdb",
            "miDebuggerPath": "F:\\mingw\\bin\\gdb.exe",// 自己电脑的gdb
            "preLaunchTask": "echo",//这里和task.json的label相对应
            "setupCommands": [
                {
                    "description": "Enable pretty-printing for gdb",
                    "text": "-enable-pretty-printing",
                    "ignoreFailures": true
                }
            ]

        }
    ]
}

```

tasks.json

```json
{
    // See https://go.microsoft.com/fwlink/?LinkId=733558
    // for the documentation about the tasks.json format
    "version": "2.0.0",
    "tasks": [
        {
            "label": "echo",
            "type": "shell",
            "command": "gcc",
            "args": [
                "-g", 
                "${file}", 
                "-o", 
                "${fileBasenameNoExtension}.exe",
                "-fexec-charset=GBK"//解决中文乱码
            ]
        }
    ],
    "presentation": {
        "echo": true,
        "reveal": "always",
        "focus": false,
        "panel": "shared", 
        "showReuseMessage": true,
        "clear": false
    }
}
```



## Ubuntu版

1、在应用程序拓展商店中下载安装 C/C++ Extension Pack ，设定默认编译器

2、创建文件目录，写个简单测试程序，点击debug 按钮，可以让vscode 自己创建 launch.js 和 tasks.json
选择 C++(GDB/LLDB)

launch.json

```json
{
    "configurations": [
        {
            "name": "C/C++: gcc-9 构建和调试活动文件",
            "type": "cppdbg",
            "request": "launch",
            "program": "${fileDirname}/${fileBasenameNoExtension}",
            "args": [],
            "stopAtEntry": false,
            "cwd": "${fileDirname}",
            "environment": [],
            "externalConsole": false,
            "MIMode": "gdb",
            "setupCommands": [
                {
                    "description": "为 gdb 启用整齐打印",
                    "text": "-enable-pretty-printing",
                    "ignoreFailures": true
                },
                {
                    "description": "将反汇编风格设置为 Intel",
                    "text": "-gdb-set disassembly-flavor intel",
                    "ignoreFailures": true
                }
            ],
            "preLaunchTask": "C/C++: gcc-9 生成活动文件",
            "miDebuggerPath": "gdb"
        }
    ],
    "version": "2.0.0"
}
```

tasks.json

```json
{
    "tasks": [
        {
            "type": "cppbuild",
            "label": "C/C++: gcc-9 生成活动文件",
            "command": "/usr/lib/ccache/gcc-9",
            "args": [
                "-fdiagnostics-color=always",
                "-g",
                "${file}",
                "-o",
                "${fileDirname}/${fileBasenameNoExtension}"
            ],
            "options": {
                "cwd": "${fileDirname}"
            },
            "problemMatcher": [
                "$gcc"
            ],
            "group": {
                "kind": "build",
                "isDefault": true
            },
            "detail": "调试器生成的任务。"
        }
    ],
    "version": "2.0.0"
}
```

