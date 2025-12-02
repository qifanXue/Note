# 目录

[TOC]

# 源代码到可执行文件过程

在C++中，源代码(.cpp)到可执行文件(.exe)，中间有三个步骤，分别是：**预处理**、**汇编**、**编译**、**链接**

<u>单个文件编译</u>，是头文件在预处理中粘贴到当前文件中，随后汇编编译到obj文件

<u>整个项目编译</u>，就是每个单个文件编译后，将每个编译后的obj文件链接link后得到exe文件

## 预处理(.cpp->.i)

预处理器会处理所有`#`开头的指令，包括展开头文件、宏替换、条件编译与删除注释

1. 展开头文件

   处理`#include<文件A>`指令，将``文件A`粘贴到`include`的位置

2. 宏替换

   处理`#define A B`指令，将文件中所有A替换成B

3. 条件编译

   | 指令                 | 作用说明                                                     |
   | :------------------- | :----------------------------------------------------------- |
   | `#ifdef MACRO`       | 如果宏 `MACRO`已**定义**，则编译后续代码。                   |
   | `#ifndef MACRO`      | 如果宏 `MACRO`**未定义**，则编译后续代码。                   |
   | `#else`              | 提供 `#ifdef`, `#ifndef`, `#if`的备用分支。                  |
   | `#endif`             | 结束一个条件编译块。                                         |
   | `#if defined(MACRO)` | 功能与 `#ifdef MACRO`相同，但可以组合多个条件，如 `#if defined(MACRO1) && defined(MACRO2)`。 |
   | `#undef MACRO`       | 取消一个已定义的宏                                           |

4. 删除注释

   删除注释，添加行号和文件名标识

> [!NOTE] 
>
> 如何在Visual Studio中获得.i文件：
>
> 1. 右击项目文件夹，选择`属性`
> 2. 点击`C/C++`,下拉中选择`预处理器`
> 3. 将`预处理到文件`置为`是`

## 编译(.i->.s)

编译器接收.i文件，生成汇编代码，保存为.s文件

> [!NOTE]
>
> 如何在Visual Studio中查看.asm文件（另一种查看汇编的方案）：
>
> 1. 右击项目文件夹，选择`属性`
> 2. 点击`C/C++`,下拉中选择`输出文件`
> 3. 将`汇编程序输出`置为`仅有程序集的列表FA`

## 汇编(.s->.obj)

汇编器将.s文件翻译成二进制的机器指令，最终输出为目标文件.obj

目标文件包含代码段（机器指令）、数据段（全局变量）、符号表（函数/变量引用）

## 链接（.obj->.exe）

连接器将所有obj文件以及他们依赖的静态库/动态库链接在一起：

1. 合并所有目标文件和库（解析符号引用）
2. 分配内存地址，生成最终可执行的二进制文件
3. 处理静态库（代码直接嵌入）和动态库（运行时加载）

> [!NOTE]
>
> 1. 如何设置函数的入口点：
>
>    函数的入口点默认是`main`函数
>
>    可以通过`属性-链接器-高级-入口点`进行配置
>
> 2. 显示链接错误
>
>    Error列表中，C开头的错误代码，代表编译错误；LNK开头的错误代码，代表链接错误。
>
>    - “未解决的外部符号”报错，就是连接器没有找到需要的东西
>    - 参数不对 返回类型不对 函数名不对；函数或者变量 有相同的名字和相同的签名；都会导致链接错误
>
> 3. 如何避免头文件中的函数重复定义导致链接失败
>
>    比如在头文件中
>
>    ```C++
>    //<log.h>
>    void log(const char* message){
>    	std::cout<<"Hello"<<std::endl;
>    }
>    ```
>
>    然后分别在另外两个cpp文件中include这个头文件并调用函数，就会出现函数重复定义的链接错误
>
>    有三种方式避免：
>
>    1. 在头文件中使用`static`修饰函数
>
>       > [!IMPORTANT]
>       >
>       > `static`修饰函数，函数就只在当前文件内部生效，对于其他obj文件不可见，不参与链接
>
>    2. 在头文件中使用`inline`修饰函数
>
>       > [!IMPORTANT]
>       >
>       > `inline`修饰函数，就会直接粘贴函数体到文件
>
>    3. 声明与定义分离
>
>       在头文件中只声明函数，在翻译单元(Log.cpp)中定义

## 头文件

头文件中存储函数的声明，避免头文件重复粘贴，有以下方式：

1. `#pragma once`

   `#pragma once`意思是只包括这个文件一次

2. ``#ifndef`

   ```c++
   #ifndef _LOG_H
   #define _LOG_H
   //log.h的内容
   void log(const char* message);
   #endif
   ```

   

# Visual Studio设置

Visual Studio默认安装的是MSVC编译器（微软的C++编译器） 而非g++

| **场景**             | **推荐工具链**        | **优点**                       |
| :------------------- | :-------------------- | :----------------------------- |
| Windows 原生开发     | Visual Studio MSVC    | 深度集成 IDE，调试方便         |
| 跨平台项目（需 GCC） | MSYS2 + MinGW         | 兼容 Linux 代码，方便移植      |
| 快速管理第三方库     | vcpkg + Visual Studio | 自动处理依赖，无需手动配置路径 |

我使用的是vcpkg，安装步骤如下：

1. 在github上下载vcpkg文件夹[[microsoft/vcpkg: C++ Library Manager for Windows, Linux, and MacOS](https://github.com/microsoft/vcpkg)]

2. 解压后，进入文件夹，输入命令：

   ```shell
   //启动vcpkg
   cd vcpkg
   .\bootstrap-vcpkg.bat
   //安装库命令（以 libyuv 为例）
   .\vcpkg install libyuv:x64-windows
   //集成 vcpkg 到 Visual Studio
   .\vcpkg integrate install
   ```

   之后在 Visual Studio 新建或已有的 C++ 项目中，直接 #include 你通过 vcpkg 安装的库头文件即可，无需额外配置

创建项目配置：

创建项目的时候 不要勾选将解决方案和项目放在同一个目录中 创建之后我们得到:

```
├──project_test
|  ├───project_test.sln
|  ├───project_test                 # 文件夹
|  |   ├───Project_test.vcxproj     # 实际上是XML文件
|  |   ├───Project_test.vcxproj.filters
|  |   ├───Project_test.vcxproj.user
```

我们可以看到那个解决方案(.sln)是和项目同名的,然后在解决方案里:

```
├──project_test
|  ├───引用
|  ├───外部依赖项
|  ├───头文件
|  ├───源文件
|  ├───资源文件
```

如果我们在源文件 右键 - 添加 - 新建项 创建一个cpp文件 会发现它就和那些`Project_test.vcxproj` `Project_test.vcxproj.filters` `Project_test.vcxproj.user` 在同一个文件夹里 这太混乱了 所以我们还是创建一个名为`source`或者`src`的文件夹 在其中存放所有源代码 头文件一类的东西

## 如何解决visual studio 无法打开源文件 “xxx.h“

项目 -> 属性 -> VC++目录 -> 包含目录 -> 编辑

![img](https://i-blog.csdnimg.cn/blog_migrate/4fa27e1beed6ecb1a5e2d41ebd208471.png)