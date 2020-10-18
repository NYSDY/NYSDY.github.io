---
title: 在macOS Catalina中配置VS Code C++开发环境
copyright: true
top: false
cover: false
toc: true
mathjax: true
date: 2020-09-26 13:11:01
password:
english_title: Configure VS Code C++ development environment in macOS Catalina
tags:
- C++
- vscode
- mac
categories:
- 技术
---



# 一、安装VS Code及扩展

1. 在官网下载安装mac版本VS Code
2. 安装C/C++、C/C++ Clang Command Adapter及**CodeLLDB**扩展
3. 按⇧⌘P，输入shell，选择如图命令

![](http://image.nysdy.com/20200926121753.png)

# 二、搭建测试项目

在Terminal输入以下命令

```shell
mkdir projects
cd projects
mkdir hello
cd hello
code . 
```

上述输入`code .`后，会直接在vscode中打开hello文件夹。

上述步骤也可以在vscode中创建一个新的`hello`文件夹代替。

# 三、设置编译器路径

⇧⌘P，输入C/C++，选择Edit Configurations (UI)。Edit Configurations (JSON)应该也可以，没有测试过![](http://image.nysdy.com/20200926122158.png)

保持默认设置即可，中间设置的解释可参考开头官网链接![](http://image.nysdy.com/20200926122327.png)

# 四、创建build task

1. 按`⇧⌘P`，输入Task
2. 选择tasks: Configure Default Build Task
3. 选择Create tasks.json file from template
4. 选择Others
5. VS Code会创建一个tasks.json，在编辑器中打开
6. 按照如下设置task.json

```json
{
    // See https://go.microsoft.com/fwlink/?LinkId=733558
    // for the documentation about the tasks.json format
    
    "version": "2.0.0",
    "tasks": [
        {
            "label": "Build with Clang",//这个任务的名字在launch.json最后一项配置
            "type": "shell",
            "command": "clang++",
            "args": [
                "-std=c++17",
                "-stdlib=libc++",
                //"test.cpp",这里是官方写法，不具有普遍性，注意两个配置文件的统一性即可
                "${fileBasenameNoExtension}.cpp",
                "-o",
                //"test.out",
                "${fileBasenameNoExtension}",
                "--debug"
            ],
            "group": {
                "kind": "build",
                "isDefault": true
            }
        }
    ]
}
```



# 五、设置debug setting

1. 按`⇧⌘P`，输入launch，选择Debug：Open launch.json
2. 选择LLDB
3. 生成如下launch.json文件，在编辑器打开
4. 替换成如下内容

```json
{
    // 使用 IntelliSense 了解相关属性。 
    // 悬停以查看现有属性的描述。
    // 欲了解更多信息，请访问: https://go.microsoft.com/fwlink/?linkid=830387

    "version": "0.2.0",
    "configurations": [
        {
            "type": "lldb",
            "request": "launch",
            "name": "Debug",
            //"program": "${workspaceFolder}/test.out",
            //上一行是官方写法，但是不同的cpp调试都要改配置，非常麻烦
            "program": "${workspaceFolder}/${fileBasenameNoExtension}",
            "args": [],
            "cwd": "${workspaceFolder}",
            "preLaunchTask": "Build with Clang"
        }
    ]
}
```

# 六、添加源文件

在`hello`文件夹下创建`helloworld.cpp`，输入如下代码

```cpp
#include <iostream>
#include <vector>
#include <string>

using namespace std;

int main()
{

    vector<string> msg {"Hello", "C++", "World", "from", "VS Code!"};

    for (const string& word : msg)
    {
        cout << word << " ";
    }
    cout << endl;
}
```

# 七、Build

按`⇧⌘B`来编译

会出现如下提示![](http://image.nysdy.com/20200926123016.png)

# 八、运行&Debug

点击Debug  即可直接运行出结果

![](http://image.nysdy.com/20200926123236.png)

设置断点，点击Debug

![](http://image.nysdy.com/20200926130522.png)

如果改变`hello.cpp`的文件位置，则task.json和launch.json需要相应作出改变，如下所示：

lauch.json:

```json
{
    // 使用 IntelliSense 了解相关属性。 
    // 悬停以查看现有属性的描述。
    // 欲了解更多信息，请访问: https://go.microsoft.com/fwlink/?linkid=830387

    "version": "0.2.0",
    "configurations": [
        {
            "type": "lldb",
            "request": "launch",
            "name": "Debug",
            //"program": "${workspaceFolder}/test.out",
            //上一行是官方写法，但是不同的cpp调试都要改配置，非常麻烦
            "program": "${workspaceFolder}/src/${fileBasenameNoExtension}",
            "args": [],
            "cwd": "${workspaceFolder}",
            "preLaunchTask": "Build with Clang"
        }
    ]
}
```

task.json:

```json
{
    // See https://go.microsoft.com/fwlink/?LinkId=733558
    // for the documentation about the tasks.json format
    
    "version": "2.0.0",
    "tasks": [
        {
            "label": "Build with Clang",//这个任务的名字在launch.json最后一项配置
            "type": "shell",
            "command": "clang++",
            "args": [
                "-std=c++17",
                "-stdlib=libc++",
                //"test.cpp",这里是官方写法，不具有普遍性，注意两个配置文件的统一性即可
                "./src/${fileBasenameNoExtension}.cpp",
                "-o",
                //"test.out",
                "./src/${fileBasenameNoExtension}",
                "--debug"
            ],
            "group": {
                "kind": "build",
                "isDefault": true
            }
        }
    ]

```

运行结果和文件结构如下所示：![](http://image.nysdy.com/20200926130948.png)

# 参考链接

- [在macOS Catalina中配置VS Code C++开发环境](https://zhuanlan.zhihu.com/p/100896963)
- [官网链接](https://code.visualstudio.com/docs/cpp/config-clang-mac)