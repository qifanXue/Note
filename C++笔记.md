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



# 类(`class`)/结构(`struct`)

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

```c++
#include<iostream>
int* returnX() {
    static int x = 10;
    return &x;
}
int main() {
    int* a1 = returnX();
    std::cout << a1 << " " << (*a1) << std::endl;
    int* a2 = returnX();
    std::cout << a2 << " " << (*a2) << std::endl;

    return 0;
}
/*
00007FF6F9ECF004 10
00007FF6F9ECF004 10
*/
```

c

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

   

# 枚举(`enum`)

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



# 继承

子类可以继承父类中所有`public`的变量与函数

```c++
class People{
public:
    const char* name;
    People(const char* name):name(name){
        std::cout<<"New name is created!"<<std::endl;
    }
};

class Student : public People{
public:
    int ID;
    Student(int ID):ID(ID){}
};
```



# 虚函数

先看一种向上转型的例子

```c++
#include <iostream>

class People {
public:
    /*const char* name;
    People(const char* name) :name(name) {
        std::cout << "New name is created!" << std::endl;
    }*/
    void Getname() {
        std::cout << "People!" << std::endl;
    }
};

class Student : public People {
public:
    int ID;
    //Student(int ID) :ID(ID) {};
    void Getname() {
        std::cout << "Student" << std::endl;
    }

};


int main() {
    People* p = new People();
    Student* s = new Student();
    p->Getname();//People!
    s->Getname();//Student
    People* n = s;//People!
    n->Getname();

    return 0;
}
```

子类可以对父类里的一些同名的方法进行覆盖，在父类中的函数前加`virtual`关键字变成虚函数，在子类的函数中加入`override`

```c++
#include <iostream>

class People {
public:
    /*const char* name;
    People(const char* name) :name(name) {
        std::cout << "New name is created!" << std::endl;
    }*/
    virtual void Getname() {
        std::cout << "People!" << std::endl;
    }
};

class Student : public People {
public:
    int ID;
    //Student(int ID) :ID(ID) {};
    void Getname() override {
        std::cout << "Student" << std::endl;
    }

};


int main() {
    People* p = new People();
    Student* s = new Student();
    p->Getname();//People!
    s->Getname();//Student
    People* n = s;
    n->Getname();//Student

    return 0;
}
```

虚函数是如何运作的呢？当基类中存在虚函数时，类就会创建一个对应的虚表，虚表中记载着该类所有虚函数的地址，同时在类中留下虚表的指针

虚函数（virtual）是C++实现运行时多态的关键机制 它的核心原理是

- 虚表（vtable）：每个包含虚函数的类都有一个虚表 本质是一个函数指针数组 存储该类所有虚函数的实际地址
- 虚表指针（vptr）：每个对象内部隐含一个指针（vptr） 指向其所属类的虚表 

在运行时 通过对象的vptr找到虚表 再通过虚表索引调用正确的函数实现

内存布局：

- People对象：

  ```plaintext
  | vptr (指向 People 的虚表) | People 其他成员... |
  ```

- Student对象：

  ```plaintext
  | vptr (指向 Student 的虚表) | Student 基类成员... | Student 成员（如ID）... |
  ```

虚表内容：

- Entity的虚表：

  ```plaintext
  [0] People::GetName 的地址
  ```

- Player的虚表：

  ```plaintext
  [0] Student::GetName 的地址  // 覆盖了基类的函数地址
  ```

当执行`entity->GetName()`时：  

1. 获取vptr：通过People指针找到对象的vptr（位于对象内存起始位置）
2. 查找虚表：通过vptr找到所属类的虚表 而entity也就是p的这个地址的起始位置 存储的其实仍然是Player的虚表 所以会调用到Student的GetName
3. 调用函数：从虚表中按索引（例如索引0对应GetName）取出函数地址 调用 `Student::GetName()`

# 纯虚函数

纯虚函数在基类中没有实现这个函数，强制要求子类实现，否则无法实例化

```c++
#include <iostream>

class People {
public:
    virtual void Getname() = 0;
};

class Student : public People {
public:
    int ID;
    //Student(int ID) :ID(ID) {};
    /*void Getname() override {
        std::cout << "Student" << std::endl;
    }*/

};


int main() {
    //People* p = new People();
    Student* s = new Student();//报错
 
    return 0;
}
```

# `public`/``protected`/`private`

| 访问控制符    | 当前类 | 派生类 | 类外部 | 友元 |
| :------------ | :----- | :----- | :----- | :--- |
| **public**    | ✓      | ✓      | ✓      | ✓    |
| **protected** | ✓      | ✓      | ✗      | ✓    |
| **private**   | ✓      | ✗      | ✗      | ✓    |

# 数组

创建的数组传递的其实是一个指针，在指针的基础上加上偏移量来改变给定位置的值

```c++
#include <iostream>

int main() {
	// 栈创建
	int example1[5];
	example1[0] = 2;
	std::cout << example1 << std::endl;
	std::cout << example1[0] << std::endl;

	// 堆创建
	int* example2 = new int[5];
	example2[0] = 6;
	std::cout << example2 << std::endl;
	std::cout << example2[0] << std::endl;
	delete[] example2;

	//观察指针
	int* ptr = example1;
	example1[2] = 7;
	std::cout << example1[2] << std::endl;
	*(ptr + 2) = 9;
	std::cout << example1[2] << std::endl;

	return 0;
}
/*
000000BEFB7EFC08
2
0000027C12585BB0
6
7
9
*/
```



# 字符串

## c风格字符串

```c++
#include<iostream>

int main() {
	const char* name1 = "123";
	const char name2[3] = { '1','2','3'};
	const char name3[4] = { '1','2','3','\0'};

	std::cout << name1 << std::endl;
	std::cout << name2 << std::endl;
	std::cout << name3 << std::endl;

	return 0;
}
/*
123
123烫烫烫烫烫烫烫烫烫烫烫烫烫烫?23
123
*/
```

## string

包括字符串的拼接、多行文本、查找

```c++
#include<iostream>
using namespace std::string_literals;
int main() {
	/*字符串拼接*/ 
	//后缀
	std::string s1 = "hello"s + " " + "world";
	std::cout << s1 << std::endl;
	//显示转换
	std::string s2 = (std::string)"hello" + " " + "China";
	std::cout << s2 << std::endl;
	//+=
	std::string s3 = "Hello";
	s3 += " Shanghai";
	std::cout << s3 << std::endl;

	/*处理多行文本*/
	//使用R
	std::string l1= R"(Line1
Line2
Line3
)"s;  
	std::cout << l1 << std::endl;
	//使用+拼接
	std::string l2 = "Line1\n"s +
		"Line2\n" +
		"Line3\n" ;
	std::cout << l2 << std::endl;
	//方法三
	std::string l3 = "Line1\n"s
		"Line2\n"s
		"Line3\n"s;
	std::cout << l3 << std::endl;

	/*查找*/
	bool findch = s1.find("lo") != std::string::npos;
	if (findch) std::cout << "Get!" << std::endl;

	return 0;
}
/*
hello world
hello China
Hello Shanghai
Line1
Line2
Line3

Line1
Line2
Line3

Line1
Line2
Line3

Get!
*/
```



# `const`关键字

`const`修饰表示不能更改

## `const`修饰指针

> [!NOTE]
>
> 有一个方便理解的角度：
>
> 1. `const int *p`：`const`修饰的是`*p`，是指针的解引用，即指针所指向的值是不可变的
> 2. `int* const p`：`const`修饰的是`p`,是指针本身，即指针存储的地址是不可变的
> 3. `const int* const p`：指针以及指针所指向的值都是不可变的

1. `const`在指针前

   ```c++
   #include<iostream>
   
   int main() {
   	int a = 10;
   	const int* p = &a; // pointer to const int
   	std::cout << "Value of a: " << a << std::endl;
   	std::cout << "Value of *p: " << *p << std::endl;
   	// *p = 20; // This line would cause a compilation error
   	/*
   	Value of a: 10
   	Value of *p: 10	
   	*/
   	return 0;
   }
   ```

   

2. `const`在指针后

   ```c++
   #include<iostream>
   
   int main() {
   	int a = 10;
   	int* const p = &a; // pointer to const int
   	std::cout << "Value of a: " << a << std::endl;
   	std::cout << "Value of *p: " << *p << std::endl;
   	// p = nullptr; // Error: cannot change the address stored in a const pointer
   	/*
   	Value of a: 10
   	Value of *p: 10	
   	*/
   	return 0;
   }
   ```

   

## `const`在类内使用

- 在类的方法名后面添加`const`，表示这个方法**不会修改类的任何成员变量**

```c++
class Entity {
private:
    int m_X, m_Y;
public:
    int GetX() const  // 这是一个const方法
    {
        // m_X = 2;  // 不合法！不能修改成员变量
        return m_X;
    }
};
```

- 指针和`const`的结合

```c++
class Entity {
private:
    int* m_X, *m_Y;     // ✅ 正确写法：两个都是指针
    
public:
    const int* const GetX() const
    {
        // 含义：
        // 1. 返回类型：指向常量int的常量指针
        // 2. 方法本身：不会修改类成员
        return m_X;
    }
};
```

- 为什么需要`const`方法

  1. 支持常量引用/指针参数

     ```c++
     void PrintEntity(const Entity& e)  // 常量引用参数
     {
         // 只能调用e的const方法
         std::cout << e.GetX() << std::endl;
     }
     ```

     **重要原则**：如果`GetX()`不是`const`方法，在`PrintEntity`中就不能调用它，因为不能保证它不会修改`e`

  2. 可以重载`const`和非`const`版本

     ```c++
     class Entity {
     private:
         int m_X, m_Y;
     public:
         int GetX() const       // const版本
         {
             return m_X;
         }
         
         int& GetX()            // 非const版本（返回引用）
         {
             return m_X;
         }
     };
     ```

     **编译器规则**：

     - 当对象是`const`时，调用`const`版本
     - 当对象不是`const`时，调用非`const`版本

## `mutable`关键字

1. 在类中的使用

   允许在`const`方法中修改特定的成员变量：

   ```c++
   class Entity {
   private:
       int m_X, m_Y;
       mutable int var;  // 标记为mutable
       
   public:
       int GetX() const
       {
           var = 2;  // ✅ 合法！因为var是mutable
           return m_X;
       }
   };
   ```

   

2. 在Lambda表达式中的使用

   - Lambda表达式捕获规则

     ```
     auto f = [捕获列表](参数列表) { 函数体 };
     ```

     | 捕获方式 | 含义                 | 示例               |
     | -------- | -------------------- | ------------------ |
     | `[x]`    | 值捕获x的副本        | 内部修改不影响外部 |
     | `[&x]`   | 引用捕获x            | 内部修改会影响外部 |
     | `[=]`    | 默认值捕获所有变量   | 所有变量都有副本   |
     | `[&]`    | 默认引用捕获所有变量 | 所有变量都是引用   |
     | `[]`     | 不捕获任何变量       |                    |

   - 使用`mutable`

     ```c++
     int main() {
         int x = 8;
         
         // 错误示例：默认值捕获的变量是const
         auto f1 = [=]() {
             // x++;  // ❌ 不能修改，x的副本是const
         };
         
         // 正确示例：使用mutable
         auto f2 = [=]() mutable {
             x++;  // ✅ 可以修改，但只修改副本
             std::cout << x << std::endl;  // 输出9
         };
         
         f2();
         std::cout << x << std::endl;  // 仍然是8
     }
     ```



# 智能指针

|      | 智能指针类型         | 核心机制                                                | 所有权                | 可否复制/赋值                               | 正确初始化方式 (推荐)                                       | 需避免的错误写法                                             | 主要用途与说明                                               |
| :--- | -------------------- | ------------------------------------------------------- | --------------------- | ------------------------------------------- | ----------------------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
|      | `std::unique_ptr<T>` | 独享对象所有权，离开作用域自动释放内存                  | **独占**              | **不可复制** (可通过 `std::move`转移所有权) | `std::unique_ptr<T> ptr = std::make_unique<T>();`           | 1. `std::unique_ptr<T> ptr = new T();`(构造函数为 explicit，禁止隐式转换)  2. `unique_ptr<T> a = b;`(拷贝赋值被禁止) | 1. 取代 new/delete，管理单一对象的生命周期。 2. 作用域结束时自动调用 delete，避免内存泄漏。 3. 优先使用 `std::make_unique`以获得异常安全。 |
|      | `std::shared_ptr<T>` | 引用计数，当计数为0时自动释放内存                       | **共享**              | **可以复制**                                | `std::shared_ptr<T> ptr = std::make_shared<T>();`           | `std::shared_ptr<T> ptr(new T());`(不推荐，会导致两次独立内存分配，效率较低) | 1. 多个指针需要共管同一对象时使用。 2. **强烈推荐 `std::make_shared`**：它将对象内存和控制块（存储引用计数）合并为单次分配，更高效。 3. 即使使用智能指针，底层仍未完全“取代” new/delete（内存分配仍发生）。 |
|      | `std::weak_ptr<T>`   | 不控制对象生命周期，从 `shared_ptr`创建，不影响引用计数 | **无所有权** (观察者) | **可以复制**                                | `std::weak_ptr<T> weakPtr = sharedPtr;`(从 shared_ptr 构造) | 直接用于访问对象（需先转换为 `shared_ptr`）                  | 1. 配合 `shared_ptr`使用，解决循环引用问题。 2. 用于临时需要访问对象但不要求其必须存在的场景（如缓存、观察者列表）。 3. 使用前需调用 `lock()`方法获取一个临时的 `shared_ptr`以安全访问对象。 |

# 拷贝与拷贝构造函数

| 概念                       | 说明                                 | 示例/代码                                                    | 重要注意事项                                                 |
| -------------------------- | ------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **基本数据类型拷贝**       | 复制值，a和b是不同的内存             | `int a = 2;` `int b = a;`                                    | 修改b不会影响a，因为值被复制                                 |
| **指针拷贝**               | 复制的是内存地址（一个数字）         | `int* p1 = new int(5);` `int* p2 = p1;`                      | 两个指针指向同一内存，修改任一都会影响另一处                 |
| **浅拷贝(Shallow Copy)**   | 只复制指针本身，不复制指针指向的数据 | 默认拷贝构造函数的行为                                       | 1. 两个对象的指针成员指向同一内存 2. 修改一个会影响另一个 3. 可能导致**双重释放**错误（程序崩溃） |
| **深拷贝(Deep Copy)**      | 复制指针指向的完整数据               | 自定义拷贝构造函数中分配新内存                               | 1. 为每个对象分配独立的内存 2. 复制原始数据到新内存 3. 对象完全独立，互不影响 |
| **拷贝构造函数**           | 用同类型对象初始化新对象             | `String(const String& other)`                                | 1. 必须接收**const引用** 2. 用于实现深拷贝 3. 如果不定义，C++提供默认浅拷贝版本 |
| **默认拷贝构造函数**       | C++自动生成，执行成员逐一浅拷贝      | 等价于： `String(const String& other)` `: m_Buffer(other.m_Buffer),` `m_Size(other.m_Size) {}` | 1. 对指针成员是危险的 2. 可能导致双重释放 3. 需要资源管理的类应自定义 |
| **禁止拷贝**               | 显式禁用拷贝构造函数                 | `String(const String& other) = delete;`                      | 1. 类似`unique_ptr`的做法 2. 防止意外的浅拷贝 3. 编译时会报错 |
| **深拷贝实现**             | 在拷贝构造函数中分配新内存           | `cpp<br>String(const String& other)<br>    : m_Size(other.m_Size)<br>{<br>    m_Buffer = new char[m_Size+1];<br>    memcpy(m_Buffer, other.m_Buffer, m_Size+1);<br>}<br>` | 1. 使用`other.m_Size`而非重新计算 2. 分配新内存（`new[]`） 3. 复制数据（`memcpy`） |
| **构造函数vs拷贝构造函数** | 不同场景下的初始化                   | 构造函数：`String(const char* string)` 拷贝构造函数：`String(const String& other)` | 1. 构造函数：从原始数据创建 2. 拷贝构造函数：从同类对象复制 3. 拷贝构造函数可使用已知的`m_Size` |
| **参数传递优化**           | 避免不必要的拷贝                     | `void PrintString(const String& string)`                     | 1. 传递对象时应用**const引用** 2. 避免调用拷贝构造函数 3. 提高性能，特别是对于大对象 |



# C++ 箭头操作符（->）

## 一、箭头操作符的基本概念

### 1.1 两种使用场景

- **对原始指针使用**：C++内置支持，直接访问指针指向对象的成员
- **对类对象使用**：需要重载`operator->()`，让对象表现得像指针

### 1.2 语法形式

```
// 在类内重载
ReturnType* operator->();           // 非const版本
const ReturnType* operator->() const; // const版本
```

## 二、箭头操作符重载详解

### 2.1 为什么需要重载？

- 让自定义类型（智能指针、迭代器等）提供与原始指针一致的接口
- 增强代码可读性和一致性
- 在访问时执行额外逻辑（如日志、安全检查）

### 2.2 工作原理

```
obj->member
// 等价于：
(obj.operator->())->member
```

1. 编译器调用`obj.operator->()`获取返回值
2. 对返回值再次应用`->`规则
3. 可以链式调用，直到返回原始指针

### 2.3 必须满足的条件

- 必须是类的成员函数
- 无参数（除了隐含的this）
- 必须返回可应用`->`的类型（指针或重载了`->`的对象）

## 三、内存偏移量计算技巧

### 3.1 结构体内存布局

```
struct Vector3 {
    float x, y, z;  // 假设4字节对齐
};
// 内存偏移：x=0, y=4, z=8
```

### 3.2 计算偏移量的方法

```
// 技巧性方法（理解原理，不推荐实际使用）
int offset = (int)&(((Vector3*)0)->x);

// 标准方法
#include <cstddef>
size_t offset = offsetof(Vector3, x);
```

### 3.3 原理解析

```
&((Vector3*)0)->x
1. (Vector3*)0     将0转换为Vector3*类型指针
2. ->x            访问成员x（不实际解引用）
3. &(...)         获取x的地址
4. 由于基地址为0，结果为x的偏移量
```

## 四、实际编程建议

### 4.1 需要重载的情况

1. **智能指针类**：`unique_ptr`、`shared_ptr`等
2. **迭代器实现**：让迭代器支持指针语法
3. **代理/包装类**：在访问前后执行额外操作
4. **资源管理类**：RAII模式中的资源句柄

### 4.2 不需要重载的情况

1. 简单数据包装，通过`get()`方法足够
2. 值语义对象，不应模仿指针行为
3. 接口需要明确区分对象和指针

### 4.3 最佳实践

```
// 智能指针示例
class SmartPtr {
private:
    T* ptr;
public:
    // 提供完整接口
    T* operator->() { return ptr; }
    const T* operator->() const { return ptr; }
    T& operator*() { return *ptr; }
    const T& operator*() const { return *ptr; }
    
    // 可选：提供原始指针访问
    T* get() { return ptr; }
    const T* get() const { return ptr; }
};
```

## 五、代码示例模板

```
// 基本重载模板
class Wrapper {
private:
    T* ptr;
public:
    // 必须的构造函数/析构函数
    Wrapper(T* p) : ptr(p) {}
    ~Wrapper() { /* 资源释放 */ }
    
    // 箭头操作符重载
    T* operator->() { 
        // 可在此添加额外逻辑
        return ptr; 
    }
    
    const T* operator->() const { 
        return ptr; 
    }
    
    // 通常同时重载解引用操作符
    T& operator*() { return *ptr; }
    const T& operator*() const { return *ptr; }
};
```





# C++ `std::vector`详解与优化

## 一、`std::vector`基本概念

### 1.1 为什么需要 `std::vector`？

- **传统数组的局限性**： `Vertex vertices_stack[5];        // 栈数组，大小固定 Vertex* vertices_heap = new Vertex[5];  // 堆数组，大小固定`都需要指定固定大小 无法动态调整容量
- **`std::vector`的优势**： `#include <vector> std::vector<Vertex> vertices;  // 动态数组，自动调整大小`

### 1.2 对象存储 vs 指针存储

| 存储方式     | 优点                       | 缺点                     |
| ------------ | -------------------------- | ------------------------ |
| **存储对象** | 内存连续，缓存友好         | 调整大小时需复制所有数据 |
| **存储指针** | 调整大小开销小，只复制指针 | 内存不连续，缓存不友好   |

```
// 推荐：优先存储对象
std::vector<Vertex> vertices;  // ✅ 内存连续，访问快

// 特定场景：存储指针
std::vector<Vertex*> vertices_ptr;  // 扩容时只复制指针
```

## 二、`std::vector`基本操作

### 2.1 创建与添加元素

```
#include <vector>

// 创建空vector
std::vector<Vertex> vertices;

// 添加元素
vertices.push_back({1, 2, 3});  // 使用初始化列表
vertices.push_back(Vertex(4, 5, 6));  // 显式构造
```

### 2.2 访问元素

```
// 使用下标运算符（已重载）
for (int i = 0; i < vertices.size(); i++) {
    std::cout << vertices[i] << std::endl;
}

// 范围for循环（C++11）
for (const Vertex& v : vertices) {  // 使用引用避免复制
    std::cout << v << std::endl;
}
```

### 2.3 删除元素

```
// 清空所有元素
vertices.clear();

// 删除特定元素（删除第3个元素，索引2）
vertices.erase(vertices.begin() + 2);

// 删除最后一个元素
vertices.pop_back();
```

## 三、性能优化技巧

### 3.1 问题：不必要的拷贝

```
struct Vertex {
    float x, y, z;
    
    // 添加拷贝构造函数以跟踪拷贝
    Vertex(const Vertex& other) 
        : x(other.x), y(other.y), z(other.z) {
        std::cout << "Copied!" << std::endl;
    }
};
```

**问题分析**：

```
std::vector<Vertex> vertices;
vertices.push_back({1, 2, 3});  // 输出1次Copied!
vertices.push_back({4, 5, 6});  // 输出2次Copied!（1次扩容+1次添加）
vertices.push_back({7, 8, 9});  // 输出3次Copied!（2次扩容+1次添加）
// 总计：6次拷贝！
```

### 3.2 优化方案1：预分配内存

```
// 方法1：reserve() - 只分配内存，不创建对象
std::vector<Vertex> vertices;
vertices.reserve(3);  // 分配容量为3，size仍为0

// 方法2：构造函数 - 创建指定数量的默认对象
std::vector<Vertex> vertices(3);  // 需要Vertex有默认构造函数
```

**区别**：

| 方法         | 作用        | 是否调用构造函数 |
| ------------ | ----------- | ---------------- |
| `reserve(n)` | 只分配容量  | 不调用           |
| `vector(n)`  | 创建n个元素 | 调用默认构造函数 |

### 3.3 优化方案2：使用 `emplace_back()`

```
std::vector<Vertex> vertices;
vertices.reserve(3);

// 使用emplace_back：直接在vector内存中构造对象
vertices.emplace_back(1, 2, 3);  // ✅ 无拷贝
vertices.emplace_back(4, 5, 6);
vertices.emplace_back(7, 8, 9);

// 对比push_back：
vertices.push_back(Vertex(1, 2, 3));  // ❌ 有拷贝
```

**`emplace_back`vs `push_back`**：

- `push_back`：先构造临时对象，再拷贝到vector
- `emplace_back`：直接在vector内存中构造，无临时对象

## 四、传递 `std::vector`的最佳实践

### 4.1 传引用，避免拷贝

```
// ❌ 不好：会复制整个vector
void processVector(std::vector<Vertex> vertices) { /* ... */ }

// ✅ 好：不复制，可修改原vector
void processVector(std::vector<Vertex>& vertices) { /* ... */ }

// ✅ 更好：只读访问
void processVector(const std::vector<Vertex>& vertices) { /* ... */ }
```

### 4.2 示例

```
// 函数声明
void processVertices(const std::vector<Vertex>& vertices);

int main() {
    std::vector<Vertex> vertices = {{1,2,3}, {4,5,6}, {7,8,9}};
    processVertices(vertices);  // 无拷贝
}
```

## 五、`std::vector`内部原理

### 5.1 扩容机制

```
// vector扩容策略（通常）：
// 1. 初始容量：0
// 2. 第一次push_back：容量=1
// 3. 后续扩容：新容量 = 2 * 旧容量
// 4. 扩容过程：
//    a. 分配新内存
//    b. 复制旧元素到新内存
//    c. 释放旧内存
```

### 5.2 容量 vs 大小

```
std::vector<int> vec;
vec.push_back(1);
vec.push_back(2);
vec.push_back(3);

std::cout << vec.size();      // 3 - 实际元素数量
std::cout << vec.capacity();  // 4 - 当前容量（可能大于size）

vec.reserve(100);             // 预分配容量
vec.shrink_to_fit();          // 减少容量到刚好容纳当前元素
```

## 六、调试技巧

### 6.1 查看vector状态

```
// 在Visual Studio等IDE中：
std::vector<Vertex> vertices = {{1,2,3}, {4,5,6}};

// 调试时悬停查看：
// vertices
//   size: 2
//   capacity: 2
//   [0]: {1, 2, 3}
//   [1]: {4, 5, 6}
```

## 七、完整示例

```
#include <iostream>
#include <vector>

struct Vertex {
    float x, y, z;
    
    Vertex(float x, float y, float z) : x(x), y(y), z(z) {}
    
    // 输出运算符重载
    friend std::ostream& operator<<(std::ostream& os, const Vertex& v) {
        return os << v.x << ", " << v.y << ", " << v.z;
    }
};

int main() {
    // 1. 创建并预分配
    std::vector<Vertex> vertices;
    vertices.reserve(3);
    
    // 2. 高效添加元素
    vertices.emplace_back(1, 2, 3);
    vertices.emplace_back(4, 5, 6);
    vertices.emplace_back(7, 8, 9);
    
    // 3. 遍历
    for (const Vertex& v : vertices) {
        std::cout << v << std::endl;
    }
    
    // 4. 删除元素
    vertices.erase(vertices.begin() + 1);  // 删除第二个
    
    // 5. 传递到函数
    processVertices(vertices);
    
    return 0;
}
```

## 八、关键要点总结

1. **预分配内存**：使用`reserve()`减少扩容带来的拷贝
2. **原位构造**：使用`emplace_back()`避免临时对象拷贝
3. **引用传递**：函数参数使用`const std::vector<T>&`避免拷贝
4. **范围循环**：使用`for(const auto& element : container)`遍历
5. **了解容量**：区分`size()`和`capacity()`，理解扩容机制
6. **选择存储方式**：优先存储对象（内存连续），特定场景存储指针



# C++ 库管理

## 一、库的基本概念与结构

### 1.1 库的获取方式

| 方式             | 优点                     | 缺点               |
| ---------------- | ------------------------ | ------------------ |
| **从源码构建**   | 可调试、可修改、版本控制 | 构建复杂、耗时     |
| **预构建二进制** | 快速使用、开箱即用       | 不可调试、不可修改 |

### 1.2 库的文件结构

```
GLFW/
├── docs/                    # 文档
├── include/                # 头文件
│   └── GLFW/
│       ├── glfw3.h
│       └── glfw3native.h
├── lib-mingw-w64/          # MinGW-w64 编译的库
├── lib-vc2013/             # VS 2013
├── lib-vc2015/             # VS 2015
├── lib-vc2017/             # VS 2017
├── lib-vc2019/             # VS 2019
├── lib-vc2022/             # VS 2022（当前使用）
└── lib-static-ucrt/        # 静态链接运行时库的动态库
```

## 二、静态链接 vs 动态链接

### 2.1 基本区别

| 特性         | 静态链接               | 动态链接                 |
| ------------ | ---------------------- | ------------------------ |
| **库文件**   | `.lib`（静态库）       | `.dll`+ `.lib`（导入库） |
| **发布方式** | 只需 exe               | 需 exe + dll             |
| **运行依赖** | 无外部依赖             | 需对应的 dll             |
| **性能**     | 链接时可优化           | 运行时加载，有开销       |
| **内存占用** | 库代码被复制到每个程序 | 多个程序可共享同一 dll   |
| **更新**     | 需重新编译程序         | 替换 dll 即可更新库      |

### 2.2 运行时库的链接

```
// ucrt（通用 C 运行时库）的链接方式：
// 1. 动态链接（默认）：程序依赖 ucrtbase.dll
// 2. 静态链接：将运行时库代码嵌入 exe，兼容旧系统
```

**`lib-static-ucrt`目录的含义**：

- 包含静态链接了 ucrt 的 glfw3.dll
- 解决旧系统（如 Win7）无 ucrtbase.dll 的问题
- 本质是动态库，但内嵌了运行时库

## 三、项目配置详细步骤

### 3.1 设置包含目录（头文件路径）

```
// 项目属性 → C/C++ → 常规 → 附加包含目录
$(SolutionDir)dependencies\GLFW\include

// 在代码中包含：
#include <GLFW/glfw3.h>  // 推荐：明确表示是外部库
```

### 3.2 设置库目录和依赖项

**静态链接配置**：

```
链接器 → 常规 → 附加库目录：
    $(SolutionDir)dependencies\GLFW\lib-vc2022

链接器 → 输入 → 附加依赖项：
    glfw3.lib
```

**动态链接配置**：

```
链接器 → 常规 → 附加库目录：
    $(SolutionDir)dependencies\GLFW\lib-vc2022

链接器 → 输入 → 附加依赖项：
    glfw3dll.lib  // 导入库，非 glfw3.lib

额外步骤：将 glfw3.dll 复制到 exe 同级目录
```

### 3.3 Visual Studio 宏变量

| 宏                 | 含义                        | 示例值                                    |
| ------------------ | --------------------------- | ----------------------------------------- |
| `$(SolutionDir)`   | 解决方案文件 `.sln`所在目录 | `D:\coding\C++\Project_test`              |
| `$(ProjectDir)`    | 项目文件 `.vcxproj`所在目录 | `D:\coding\C++\Project_test\Project_test` |
| `$(Configuration)` | 当前配置（Debug/Release）   | `Debug`                                   |
| `$(Platform)`      | 当前平台（Win32/x64）       | `x64`                                     |
| `$(OutDir)`        | 输出目录                    | `bin\x64\Debug`                           |

## 四、头文件包含的两种方式

### 4.1 `#include <>`vs `#include ""`

```
// 方式1：尖括号（系统/外部库）
#include <GLFW/glfw3.h>  // 编译器搜索顺序：
                         // 1. 系统包含目录
                         // 2. 附加包含目录
                         // 3. 不搜索当前文件所在目录

// 方式2：双引号（项目内文件）
#include "Engine.h"      // 编译器搜索顺序：
                         // 1. 当前文件所在目录
                         // 2. 项目包含目录
                         // 3. 系统包含目录
```

### 4.2 最佳实践

| 场景                  | 推荐写法                        | 原因                 |
| --------------------- | ------------------------------- | -------------------- |
| **标准库/系统头文件** | `#include <iostream>`           | 明确表示是系统文件   |
| **第三方库**          | `#include <GLFW/glfw3.h>`       | 明确表示是外部依赖   |
| **项目内头文件**      | `#include "Engine.h"`           | 项目内文件，易于定位 |
| **相对路径引用**      | 避免使用 `#include "../../..."` | 不便于项目重构       |

## 五、动态链接的优化与宏定义

### 5.1 `__declspec(dllimport)`的作用

```
// 在构建 DLL 时：
#define GLFWAPI __declspec(dllexport)  // 标记函数为导出

// 在使用 DLL 时：
#define GLFWAPI __declspec(dllimport)  // 标记函数为导入，优化调用

// 不定义 GLFW_DLL 时：
#define GLFWAPI  // 空定义，无优化
```

### 5.2 如何启用导入优化

```
// 在使用 GLFW DLL 前定义 GLFW_DLL
#define GLFW_DLL
#include <GLFW/glfw3.h>

// 效果：编译器生成更高效的调用代码
// 注意：必须在包含头文件前定义！
```

**查找是否需要定义宏的方法**：

1. 阅读库的官方文档
2. 查看头文件中的条件编译指令
3. 查看库提供的示例代码

## 六、创建和使用自己的库

### 6.1 创建静态库项目

```
步骤：
1. 新建空项目
2. 项目属性 → 常规 → 配置类型 → 静态库(.lib)
3. 设置输出目录（如：$(SolutionDir)bin\$(Platform)\$(Configuration)\）
4. 设置中间目录（如：$(SolutionDir)bin\intermediates\$(Platform)\$(Configuration)\）
```

### 6.2 库项目结构示例

```
Engine/                          # 库项目
├── src/
│   ├── Engine.h                # 公共头文件
│   └── Engine.cpp              # 实现文件
└── (生成 Engine.lib)

Game/                           # 应用程序项目
├── src/
│   └── Application.cpp         # 主程序
└── (生成 Game.exe，链接 Engine.lib)
```

### 6.3 链接自定义库的两种方式

**方式1：项目引用（推荐）**

```
// 1. 右键应用程序项目 → 添加 → 引用
// 2. 选择库项目
// 优点：自动处理依赖关系，修改库后自动重新编译
```

**方式2：手动配置**

```
// 1. 设置包含目录
附加包含目录：$(SolutionDir)Engine\src

// 2. 设置库目录
附加库目录：$(SolutionDir)bin\x64\Debug

// 3. 添加依赖项
附加依赖项：Engine.lib
```

## 七、路径与跨平台兼容性

### 7.1 路径分隔符

| 系统        | 原生分隔符    | 推荐写法      |
| ----------- | ------------- | ------------- |
| Windows     | ``（反斜杠）  | `/`（正斜杠） |
| Linux/macOS | `/`（正斜杠） | `/`（正斜杠） |

```
// 不推荐（Windows 特定）：
#include "..\..\dependencies\GLFW\include\GLFW\glfw3.h"

// 推荐（跨平台）：
#include <GLFW/glfw3.h>
// 配合附加包含目录：$(SolutionDir)dependencies/GLFW/include
```

### 7.2 相对路径 vs 绝对路径

```
// 不推荐：硬编码绝对路径
#include "D:/coding/C++/Project_test/dependencies/GLFW/include/GLFW/glfw3.h"

// 推荐1：使用宏变量
#include <GLFW/glfw3.h>
// 配合：附加包含目录 = $(SolutionDir)dependencies/GLFW/include

// 推荐2：使用环境变量
#include <GLFW/glfw3.h>
// 配合：附加包含目录 = $(GLFW_ROOT)/include
// 需先设置环境变量 GLFW_ROOT
```

## 八、调试与问题排查

### 8.1 常见链接错误

```
// 错误：LNK2019 - 无法解析的外部符号
// 原因1：未指定库文件
// 解决：检查附加依赖项和库目录

// 原因2：库版本不匹配（x86 vs x64）
// 解决：确保库平台与项目平台一致

// 原因3：缺少 DLL
// 解决：将 DLL 复制到 exe 同级目录
```

### 8.2 检查链接的库

```
// 方法1：查看生成输出
// 在输出窗口中搜索 ".lib" 或 "LINK"

// 方法2：使用 DUMPBIN 工具
// 打开开发者命令提示符：
dumpbin /DEPENDENTS Game.exe  // 查看依赖的 DLL
dumpbin /EXPORTS Engine.lib  // 查看库导出的函数
```

## 九、高级主题：库的依赖传递

### 9.1 依赖关系分析

```
// 场景：你的库依赖第三方库
// 选择1：静态链接第三方库
//   - 优点：用户无需处理第三方库
//   - 缺点：你的库体积增大

// 选择2：动态链接第三方库
//   - 优点：你的库体积小
//   - 缺点：用户需处理所有依赖的 DLL

// 选择3：头文件库（如 GLM）
//   - 无链接步骤，直接包含头文件
//   - 最易用，但编译时间可能增加
```

### 9.2 版本管理建议

```
依赖管理结构：
Project/
├── dependencies/           # 所有第三方依赖
│   ├── GLFW/
│   │   ├── include/
│   │   └── lib-vc2022/
│   └── glm/               # 头文件库
├── src/                   # 项目源代码
└── docs/                  # 文档

优点：
1. 版本可控
2. 便于团队协作
3. 离线开发
```

## 十、最佳实践总结

### 10.1 库使用原则

1. **优先使用静态链接**：简化分发，避免 DLL Hell
2. **包含目录使用相对路径**：`$(SolutionDir)`或 `$(ProjectDir)`
3. **头文件包含规范化**：第三方库用 `<>`，项目文件用 `""`
4. **版本一致性**：确保所有依赖使用相同编译器版本构建

### 10.2 创建库的建议

1. **提供清晰的头文件**：使用 Doxygen 风格注释
2. **区分 Debug/Release 版本**：提供不同优化的库
3. **考虑 ABI 兼容性**：保持二进制接口稳定
4. **提供使用示例**：包含简单的示例代码

### 10.3 跨平台注意事项

1. **使用正斜杠**：`/`在所有平台都有效
2. **条件编译**：使用预处理器处理平台差异
3. **动态库扩展名**：Windows `.dll`，Linux `.so`，macOS `.dylib`
4. **导出标记**：Windows 用 `__declspec`，Linux/macOS 用 `__attribute__`





# C++ 多返回值实现方法详解

## 一、C++ 多返回值问题背景

C++ 函数默认只能返回一个值，但实际开发中经常需要返回多个相关数据。以下是 6 种实现多返回值的常用方法。

## 二、6 种实现方法对比

| 方法         | 返回类型            | 优点                                 | 缺点                               | 适用场景                     |
| ------------ | ------------------- | ------------------------------------ | ---------------------------------- | ---------------------------- |
| 引用参数     | 无返回值            | 直观，无额外开销                     | 修改了参数，调用方需先创建对象     | 简单场景，修改现有对象       |
| 指针数组     | `T*`                | 简单，可返回多个同类型值             | 需手动管理内存，调用方不知数组大小 | 不推荐，有内存泄漏风险       |
| `std::array` | `std::array<T,N>`   | 栈上分配，无动态内存开销             | 只能返回相同类型                   | 固定数量、类型相同的返回值   |
| `std::tuple` | `std::tuple<Ts...>` | 可返回不同类型，标准库支持           | 访问时用索引，代码不直观           | 临时性、简单组合的返回值     |
| `std::pair`  | `std::pair<T1,T2>`  | 可返回两个不同类型值，有first/second | 只能返回两个值，命名不明确         | 恰好返回两个相关值的场景     |
| 结构体       | 自定义结构体        | 类型安全，可读性高，可扩展           | 需额外定义结构体类型               | 大多数场景，特别是复杂返回值 |

## 三、详细方法与代码示例

### 3.1 方法1：引用参数（传引用修改）

```
// 函数通过引用参数"返回"多个值
void ParseShader(const std::string& filepath, 
                 std::string& vertexSource, 
                 std::string& fragmentSource) {
    // ... 处理逻辑
    vertexSource = vs;    // 通过引用修改调用方的变量
    fragmentSource = fs;
}

// 调用方
int main() {
    std::string vs, fs;                     // 1. 先创建变量
    ParseShader("res/shaders/Basic/shader", // 2. 传入引用
                vs, fs);                    // 3. 函数修改它们
    // 现在 vs 和 fs 包含解析后的着色器代码
}
```

**注意事项**：

- 调用方必须事先创建对象
- 参数传递是传递地址，函数内修改直接影响原对象
- 适合需要修改现有对象或避免复制的场景

### 3.2 方法2：返回指针数组（不推荐）

```
// 返回动态分配的数组
std::string* ParseShader(const std::string& filepath) {
    // ... 处理逻辑
    std::string* result = new std::string[2];  // 堆分配
    result[0] = vs;
    result[1] = fs;
    return result;  // 调用方需记得 delete[]
}

// 调用方
int main() {
    std::string* sources = ParseShader("path");
    // 使用 sources[0], sources[1]
    delete[] sources;  // 必须手动释放
}
```

**问题**：

- 调用方不知道数组大小
- 容易内存泄漏
- 不推荐在现代 C++ 中使用

### 3.3 方法3：返回 `std::array`（同类型）

```
#include <array>

// 返回固定大小的数组
std::array<std::string, 2> ParseShader(const std::string& filepath) {
    // ... 处理逻辑
    std::array<std::string, 2> result;
    result[0] = vs;
    result[1] = fs;
    return result;
    
    // 或使用列表初始化（C++17及以上）：
    // return std::array<std::string, 2>{vs, fs};
}

// 调用方
int main() {
    auto sources = ParseShader("path");
    std::string vs = sources[0];  // 通过索引访问
    std::string fs = sources[1];
}
```

**特点**：

- 栈上分配，无动态内存开销
- 编译时确定大小，效率高
- 只能返回相同类型

### 3.4 方法4：返回 `std::tuple`（不同类型）

```
#include <tuple>

// 返回包含不同类型值的元组
std::tuple<std::string, std::string, int> ParseShader(const std::string& filepath) {
    // ... 处理逻辑
    std::string vs = "...";
    std::string fs = "...";
    int version = 330;
    
    return std::make_tuple(vs, fs, version);  // 创建元组
    // 或（C++17及以上）：
    // return {vs, fs, version};
}

// 调用方
int main() {
    auto sources = ParseShader("path");
    
    // 方法1：std::get 通过索引访问（不直观）
    std::string vs = std::get<0>(sources);
    std::string fs = std::get<1>(sources);
    int version = std::get<2>(sources);
    
    // 方法2：结构化绑定（C++17，推荐）
    auto [vertexSource, fragmentSource, ver] = ParseShader("path");
}
```

**结构化绑定（C++17 推荐）**：

```
// 自动解包元组，代码清晰
auto [vs, fs, version] = ParseShader("path");
// 现在可以直接使用 vs, fs, version
```

### 3.5 方法5：返回 `std::pair`（两个值）

```
#include <utility>  // 或 <tuple>，pair 在 utility 中

// 返回两个值
std::pair<std::string, std::string> ParseShader(const std::string& filepath) {
    // ... 处理逻辑
    return std::make_pair(vs, fs);
    // 或（C++17及以上）：
    // return {vs, fs};
}

// 调用方
int main() {
    auto sources = ParseShader("path");
    
    // 方法1：first/second
    std::string vs = sources.first;
    std::string fs = sources.second;
    
    // 方法2：结构化绑定
    auto [vertexSource, fragmentSource] = ParseShader("path");
}
```

**适用场景**：

- 恰好返回两个相关值
- 如键值对、坐标等

### 3.6 方法6：返回结构体（最推荐）

```
// 定义明确的结构体类型
struct ShaderProgramSource {
    std::string VertexSource;
    std::string FragmentSource;
    int Version = 330;  // 可提供默认值
    bool IsValid = true;  // 可添加额外状态信息
};

// 返回结构体
ShaderProgramSource ParseShader(const std::string& filepath) {
    // ... 处理逻辑
    return {vs, fs, 330, true};  // 聚合初始化
    // 或指定成员初始化（C++20起）：
    // return {.VertexSource = vs, .FragmentSource = fs, .Version = 330};
}

// 调用方
int main() {
    ShaderProgramSource sources = ParseShader("path");
    // 通过有意义的名称访问
    std::string vs = sources.VertexSource;
    std::string fs = sources.FragmentSource;
    int version = sources.Version;
    bool valid = sources.IsValid;
    
    // 或使用结构化绑定（C++17）
    auto [vertexSrc, fragmentSrc, ver, isValid] = ParseShader("path");
}
```

## 四、性能与可读性分析

### 4.1 性能考虑

```
// 1. 引用参数：无额外开销，但调用方需先创建对象
// 2. 返回值优化（RVO/NRVO）：现代编译器优化
ShaderProgramSource ParseShader(...) {
    ShaderProgramSource result;  // 在函数栈创建
    // ... 填充 result
    return result;  // 编译器可能直接构造到调用方内存
}

// 3. 移动语义（C++11+）
ShaderProgramSource ParseShader(...) {
    std::string vs = GetVertexSource();
    std::string fs = GetFragmentSource();
    return {std::move(vs), std::move(fs)};  // 移动而非复制
}
```

### 4.2 可读性与维护性

```
// 差：通过索引访问，不清楚含义
std::tuple<int, int, std::string> GetUserInfo();
auto info = GetUserInfo();
int age = std::get<0>(info);      // 这是什么？年龄？ID？
int score = std::get<1>(info);    // 分数？等级？
std::string name = std::get<2>(info);

// 好：通过结构体明确含义
struct UserInfo {
    int Age;
    int Score;
    std::string Name;
};
UserInfo GetUserInfo();
auto info = GetUserInfo();
int age = info.Age;    // 清晰！
int score = info.Score;
std::string name = info.Name;
```

## 五、现代 C++ 最佳实践

### 5.1 根据场景选择方法

```
// 场景1：修改现有对象 → 引用参数
void UpdatePosition(Vector3& pos, float deltaTime);

// 场景2：简单临时组合 → tuple + 结构化绑定
auto [min, max] = FindMinMax(values);

// 场景3：固定数量同类型 → array
std::array<float, 3> GetRGBColor();

// 场景4：两个相关值 → pair
std::pair<bool, std::string> TryParse(const std::string& input);

// 场景5：复杂、可扩展、需文档化 → 结构体
struct ParseResult {
    std::string Content;
    bool Success;
    std::string ErrorMessage;
    int LineCount;
};
ParseResult ParseFile(const std::string& path);
```

### 5.2 使用结构化绑定（C++17+）

```
// 结构化绑定适用于所有方法
#include <tuple>
#include <array>

// 1. tuple
auto [x, y, z] = std::make_tuple(1, 2.5, "hello");

// 2. pair
auto [key, value] = std::make_pair("name", "Alice");

// 3. array
std::array<int, 3> arr = {1, 2, 3};
auto [a, b, c] = arr;  // 注意：必须完全匹配元素数量

// 4. 结构体
struct Point { int x; int y; };
Point p{10, 20};
auto [px, py] = p;
```

### 5.3 错误处理模式

```
// 模式1：返回结果+错误码的结构体
struct OperationResult {
    std::string Result;
    int ErrorCode = 0;
    std::string ErrorMessage;
};

// 模式2：使用 std::optional（C++17）
#include <optional>
std::optional<std::string> TryLoadFile(const std::string& path) {
    if (file_exists) return file_content;
    return std::nullopt;  // 表示失败
}

// 模式3：使用 std::variant 或 std::expected（C++23）
#include <variant>
std::variant<std::string, std::error_code> SafeParse(...);
```

## 六、完整示例：着色器解析

```
#include <iostream>
#include <string>
#include <fstream>
#include <sstream>

// 方法6：结构体（推荐）
struct ShaderProgramSource {
    std::string VertexSource;
    std::string FragmentSource;
    
    // 可添加辅助方法
    bool IsValid() const { 
        return !VertexSource.empty() && !FragmentSource.empty(); 
    }
    
    void Print() const {
        std::cout << "Vertex Shader:\n" << VertexSource 
                  << "\nFragment Shader:\n" << FragmentSource << std::endl;
    }
};

ShaderProgramSource ParseShader(const std::string& filepath) {
    std::ifstream stream(filepath);
    if (!stream.is_open()) {
        return {"", ""};  // 返回空结构体表示失败
    }
    
    std::string line;
    std::stringstream ss[2];
    enum class ShaderType { NONE = -1, VERTEX = 0, FRAGMENT = 1 };
    ShaderType type = ShaderType::NONE;
    
    while (getline(stream, line)) {
        if (line.find("#shader") != std::string::npos) {
            if (line.find("vertex") != std::string::npos) {
                type = ShaderType::VERTEX;
            } else if (line.find("fragment") != std::string::npos) {
                type = ShaderType::FRAGMENT;
            }
        } else {
            ss[(int)type] << line << '\n';
        }
    }
    
    return {ss[0].str(), ss[1].str()};
}

int main() {
    // 使用结构体
    ShaderProgramSource sources = ParseShader("res/shaders/Basic.shader");
    
    if (sources.IsValid()) {
        std::cout << "Vertex Source: " << sources.VertexSource << std::endl;
        std::cout << "Fragment Source: " << sources.FragmentSource << std::endl;
        // 或
        sources.Print();
    } else {
        std::cerr << "Failed to parse shader!" << std::endl;
    }
    
    // 使用结构化绑定（C++17）
    auto [vs, fs] = ParseShader("res/shaders/Basic.shader");
    
    return 0;
}
```

## 七、关键要点总结

1. **引用参数**：简单直接，但需调用方先创建对象
2. **返回容器**：`array`用于固定大小同类型，`vector`用于动态大小
3. **标准库工具**： `tuple`：通用但索引访问不直观 `pair`：只能两个值，命名不明确
4. **结构体**：**最推荐**，类型安全、可读性高、可扩展
5. **现代特性**： 结构化绑定（C++17）显著提高代码可读性 返回值优化（RVO）减少拷贝开销 移动语义避免不必要的深拷贝

------

**选择建议**：

- 简单临时组合 → `tuple`+ 结构化绑定
- 两个相关值 → `pair`
- 复杂、可扩展、需维护 → 结构体
- 避免使用裸指针数组，优先使用智能指针或容器



