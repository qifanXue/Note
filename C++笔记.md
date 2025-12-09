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

# 指针

指针是一个数字，一个存储内存地址的数字

1. 空指针

   空指针不是一个有效的地址，我们不能从空指针中读取或写入

   ```
   void* ptr = 0;
   void* ptr = NULL;
   void* ptr = nullptr;
   ```

2. 指针的类型与解引用

   在debug-x64的编译环境中，不论指针是什么类型，都是一个64bit的数字，但是类型在解引用（逆向引用到地址对应的变量）时，有以下几种情况：

   1. 无类型逆向引用时，我们只知道一个地址，不知道这个变量的类型没无法读写

      ```
      int var = 8;
      void *ptr = &var;
      //*ptr = 10;会报错
      ```

   2. 类型转换

      ```
      double* ptr = (double*)&var;
      //*ptr解引用得到的是值-9.2559592117432085e+61，是无意义的，因为ptr实际指向的是int类型变量var的内存 而不是double，代码中将 &var（类型是int\*）强制转换为double\*导致未定义行为 
      ```

3. 指针的内存分配

   1. `new`关键字分配

      ```
      char* buffer = new char[8];
      //未分配的堆内存，表现为 cd cd cd cd cd cd cd cd
      //未分配的堆内存，表现为 cc cc cc cc cc cc cc cc
      ```

      `delete`关键字删除数据

      ```
      delete[] buffer;
      ```

   2. `memset`函数分配

      ```
      /*memset接收一个指针，分配_Size字节的内存，将每个字节填入_Val*/
      void *__cdecl memset(void *_Dst, int _Val, size_t _Size) 
      memset(buffer,'a',8);//61 61 61 61 61 61 61 61
      ```

4. 二级指针

   指针本身也是变量 也存储在内存中 所以我们可以做指向指针的指针 二级指针或者三级指针

   ```c++
   char** ptr = &buffer;
   ```

   |  名称   |                    值                     |  类型  |
   | :-----: | :---------------------------------------: | :----: |
   | buffer  |           0x000002ac05d55070 ""           | char*  |
   | &buffer | 0x000000c1deeff728{0x000002ac05d55070 ""} | char** |
   |   ptr   | 0x000000c1deeff728{0x000002ac05d55070 ""} | char** |
   |  *ptr   |           0x000002ac05d55070 ""           | char*  |




# 引用

引用是指针的语法方法，相当于变量的别名，对引用所做的操作相当于对原变量做了同样的操作

```c++
int a = 8;
int& ref = a;//ref 是 a 的引用
ref = 10;// a = 10
```

在函数中，变量复制到参数中，离开函数后，变量的改变不会被保存

```c++
void ChangeVal(int value){
    value++;
}

int main(){
    int a = 8;
    ChangeVal(a);
    std::cout << a << std::endl;//8
}
```

如何保留函数内的改变呢？有两种方式：

1. 使用指针作为传入参数

   ```
   void ChangeVal(int* value){
       (*value)++;
   }
   
   int main(){
       int a = 8;
       int* ptr = &a;
       ChangeVal(ptr);
       std::cout << a << std::endl;//9
   }
   ```

   > [!NOTE]
   >
   > 记录一下指针在这里的解引用
   >
   > ```c++
   > int a = 8; // a是一个整型变量，占据sizeof(int)的空间，值为8
   > int* ptr = a; //ptr是一个指针，存储的是a的地址空间
   > int b = *(ptr ++)/; //b是地址+1的地址所存储的值
   > int c = (*ptr) ++; //(*ptr)指的是a，a++后a为9，c也为9
   > ```
   >
   > 

2. 使用引用作为传入参数

   ```c++
   void ChangeVal(int& value){
       value++;
   }
   
   int main(){
       int a = 8;
       int& ref = a;
       ChangeVal(ref);
       std::cout << a << std::endl;//9
       ChangeVal(a);
       std::cout << a << std::endl;//10
   }
   ```

当指定了引用的对象后，不可以转而引用别的对象；指针可以

```
int a = 5;
int b = 8;

int& ref = a;
ref = b;
std::cout<<a<<std::endl;//8
ref ++;
std::cout<<a<<std::endl;//9

int *ptr = &a;
std::cout<<(*ptr)<<std::endl;//9
ptr = &b;
std::cout<<(*ptr)<<std::endl;//8
```



# 类(Class)/结构(Struct)

类和结构都是一种语法方法，将一些数据与处理这些数据的函数集成。

类/结构的区别在于，类默认是`private`,结构默认是`public`

> [!Note]
>
> 结构是C++为了兼容C遗留的，因为C中只有`struct`
>
> 对于只表示变量的结构，不包含大量功能的，倾向于使用`struct`
>
> 我们不会倾向于在`struct`中使用继承

```c++
class Player
{
public:
    int id;
    int x,y;
    int speed;
private:
	int m_scores;
public:
    void Move(int xa,int xb)
    {
        x += xa*speed;
        y += xb*speed;
    }
    
};
    
Player player;
```

>[!Note]
>
>`m_`前缀，约定这是一个私有的类成员变量

```c++
struct Vec2
{
    float x, y;
    
    void Add(const Vec2& other)
    {
        x += other.x;
        y += other.y;
    }
};
```



# `static`关键字

static可以修饰类/结构/函数/变量

static修饰的对象的生存期几乎是整个程序运行的过程，作用域是所在的函数/类内。

1. 在类/结构外使用

   若在两个不同文件中同时定义一个相同名称的对象，会出现报错

   ```c++
   //static.cpp
   int variable = 5;
   };
   
   //main.cpp
   int variable = 5;
   ```

   有两种解决方法:

   - 使用`static`,被`static`修饰的对象无法被作用域外识别到

     ```c++
     //static.cpp
     static int variable = 5;
     //main.cpp
     int variable = 10;
     ```

   - 使用`extern`,会在外部寻找

     ```	
     //static.cpp
     int variable = 5;
     //main.cpp
     extern int variable;//5
     ```

     

2. 在类/结构内使用

   在类/结构中用`static`修饰的对象是所有类/结构的实例所共享的数据,这个对象必须在类外面声明和初始化

   ```c++
   #include <iostream>
   
   class Player {
   public:
   	static int areaID;
   	static void area() {
   		std::cout << areaID << std::endl;
   	}
   };
   int Player::areaID;
   
   int main() {
   	Player::areaID = 10;
   	Player::area();//10
   	Player p;
   	p.areaID = 20;
   	p.area();//20
   }
   ```

   

# 枚举(Enum)

`enum`就是将数值放到一个集合里面，创建如下:

```c++
enum Example
{
    A,B,C
};
/*几个注意点
1.enum内变量后面没有分号
*/
```

`enum`可以指定数值的数据类型，但是必须是整型，不能是浮点数

`enum`内初始化可以全部初始化，也可以全不初始化（默认是0，1，2），或初始化第一个，后面的递增

```c++
#include <iostream>

class Player {
public:
	/*static const int LogLevelError = 0;
	static const int LogLevelWarning = 1;
	static const int LogLevelInfo = 2;*/
	enum Example3 {
		LogLevelError = 0, LogLevelWarning, LogLevelInfo
	};
private:
	int m_LogLevel = LogLevelInfo;
};

enum Example1 {
	A,B,C
};

enum Example2 : unsigned char {
	D=1,E=2,F=3
};

enum {
	G = 1, H = 2, I = 3
};

int main() {
	std::cout << A << " " << B << " " << C << " " << std::endl;//0,1,2
	std::cout << sizeof(A)  << std::endl;//4
	std::cout << sizeof(D) << std::endl;//1
	//Example1::A = 7;报错是需要可修改的左值
	int A = 7;
	std::cout << A << std::endl;//7
	std::cout << Example1::A << std::endl;//0
	std::cout << Player::LogLevelError << " " << Player::LogLevelWarning << " " << Player::LogLevelInfo << " " << std::endl;//0,1,2
}
```

# 构造函数/析构函数

使用类的几个步骤：创建类、实例化类、初始化类、调用类的成员变量与函数、销毁类

其中构造函数同时完成了实例化类和初始化类，我们先看看不适用构造函数的例子：

```c++
#include <iostream>

class Player {
public:
	int x, y;
	void Init(int sx,int sy) {
		x = sx;
		y = sy;
	}
	void printPosition() {
		std::cout << "Player position: (" << x << ", " << y << ")\n";
	}
};

int main() {
	Player p;
	p.Init(10, 20);
	p.printPosition();
	return 0;	
}
```

## 构造函数

构造函数同时完成了实例化和初始化

> [!NOTE]
>
> 构造函数的语法注意以下几点：
>
> 1. 构造函数的名字必须和类同名
> 2. 构造函数前面没有数据类型和修饰词
> 3. 构造函数可以没有参数，还可以传递不同参数,同样的函数名不同的参数叫做函数**重载**，初始化列表的顺序必须按照类成员变量声明的顺序写

```c++
#include <iostream>

class Player {
public:
    int x, y;

    // 默认构造函数（只保留一个）
    Player() : x(0), y(0) {
        std::cout << "Default constructor called\n";
    }

    // 两个参数的构造函数
    Player(int sx, int sy) : x(sx), y(sy) {
        std::cout << "Two-parameter constructor called\n";
    }

    // 一个参数的构造函数
    Player(int sx) : x(sx), y(0) {
        std::cout << "One-parameter constructor called\n";
    }

    // 拷贝构造函数（应该用const引用，而不是指针）
    Player(const Player& p) : x(p.x), y(p.y) {
        std::cout << "Copy constructor called\n";
    }
	
    void printPosition() {
        std::cout << "Player position: (" << x << ", " << y << ")\n";
    }
};

int main() {
    Player p1;           
    Player p2(10, 20);   
    Player p3(30);      
    Player p4(p2);      
    Player p5(p2);       

    p1.printPosition();
    p2.printPosition();
    p3.printPosition();
    p4.printPosition();
    p5.printPosition();

    return 0;
}
```

## 析构函数

析构函数是用来销毁类实例对象的

> [!NOTE]
>
> 析构函数的语法：
>
> 构造函数前面加上`~`

```
class Player {
public:
    int x, y;

    Player() : x(0), y(0) {
        std::cout << "Default constructor called\n";
    }
    
    ~Player(){
    	std::cout << "destroy the instance" << std::endl;
    }
};
```

## 栈内存和堆内存

上面使用的构造方式都是栈内存，栈内存的实例的析构函数会在实例作用域失效后自动调用

而堆内存，构造和销毁分别需要使用`new`、`delete`关键字

```c++
#include <iostream>

class Player {
public:
    int x, y;
    Player(int sx,int sy){
		x = sx;
		y = sy;
        std::cout << "Default constructor called\n";
    }
    ~Player(){
        std::cout << "Destructor called\n";
	}
    void printPosition() {
        std::cout << "Player position: (" << x << ", " << y << ")\n";
    }
};

void func() {
    Player p1(1,1);
	p1.printPosition();
}

int main() {
    func();
    Player* p2 = new Player(2,6);      
    p2->printPosition();
	delete p2;

    return 0;
}

/*
Default constructor called
Player position: (1, 1)
Destructor called
Default constructor called
Player position: (2, 6)
Destructor called
*/
```































