---
title: VSCode中launch.json/tasks.json/settings.json是干什么的?
date: 2023-04-19 15:56:25
tags: Linux C++
---

在Linux中进行C++开发时, VSCode是一个极其好用的工具, 不同于VS等完整的IDE, 我们可能要手动进行一些配置便于VSCode配合我们的编译和调试等工作. 在项目文件夹.vscode路径下中的三个文件launch.json、tasks.json、settings.json便是我们用来调教VSCode的工具.

本文从快速使用出发进行总结, 并不涉及过多内容.

# launch.json

用于配置调试的的文件, VSCode根据这个文件搭建调试环境.

## 示例

```json
{
    "configurations": [
        {
            "name": "C/C++: g++ 生成和调试活动文件",
            "type": "cppdbg",//C++
            "request": "launch",//还有个attach
            "program": "${workspaceFolder}/build/cmexe",//生成的可执行文件路径
            "args": [],
            "stopAtEntry": false,
            "cwd": "${workspaceFolder}",
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
            "preLaunchTask": "Build",//在tasks.json中定义的任务
            "miDebuggerPath": "/usr/bin/gdb"
        }
    ],
    "version": "2.0.0"
}
```

直接使用VSCode自己生成的简单改改就是最快速使用的方法...

# tasks.json

对应launch.json中的preLaunchTask项.

这个文件决定了如何编译文件(规定了一些任务进行批处理).

执行完tasks.json中的命令后编译完成, 而后可以根据launch.json中的配置进行调试.

在配置好这两个文件之后便可以在VSCode中按下F5进行调试, 不用敲cmake编译再进gdb辣!

## 示例

```json
{
    "options": {
        "cwd": "${workspaceFolder}/build"
    },
    "tasks": [
        {
            "type": "shell",
            "label": "cmake",
            "command":"cmake",
            "args": [
                ".."
            ]
        },
        {
            "label": "make",
            "group": {
                "kind": "build",
                "isDefault": true,
            },
            "command":"make",
            "args": [

            ]
        },
        {
            "label": "Build",
            "dependsOrder": "sequence",
            "dependsOn":[
                "cmake",
                "make"
            ]
        }
    ],
    "version": "2.0.0"
}
```

使用CMake的工程中tasks.json的示例.

# settings.json

该项目对于VSCode的独有配置可以写在这个文件里, (好像对VSCode本身的settings.json有继承关系?)

可以改变缩进等编辑器配置以及代码补全相关功能.

对于项目编译调试等好像没啥影响.
