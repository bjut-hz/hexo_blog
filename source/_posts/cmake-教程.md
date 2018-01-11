---
title: cmake 教程
date: 2017-03-11 10:09:34
categories:
- tools
tags:
- cmake
- C
- C++
---


## 简介
摘自维基百科：

> CMake是个开源的跨平台自动化建构系统，它用配置文件控制建构过程（build process）的方式和Unix的Make相似，只是CMake的配置文件取名为CMakeLists.txt。Cmake并不直接建构出最终的软件，而是产生标准的建构档（如Unix的Makefile或Windows Visual C++的projects/workspaces），然后再依一般的建构方式使用。

> “CMake”这个名字是"cross platform make"的缩写。虽然名字中含有"make"，但是CMake和Unix上常见的“make”系统是分开的，而且更为高级。

<!-- more -->

最近尝试了**autoconf**，**automake**，**libtool**等自动化构建工具，使用这些工具相比以前的手写Makefile来构建项目要简单了许多。但是，它们依然过于繁琐，而且对于跨平台来说不够友好。

之后突然想到在编译[youcompleteme](https://github.com/Valloric/YouCompleteMe)和[neovim](https://github.com/neovim/neovim)时用到的cmake命令，经过简单的学习发现cmake是一个很强大而且也比较简单的工项目构建工具。

这篇帖子是基于官方[tutorial](https://cmake.org/cmake-tutorial/)所写，扩充了官方文档中不太详细的部分。


### Cmake的安装

Cmake的安装非常的简单，官方已经提供了许多常用平台下的二进制文件发行版。

下载位置[https://cmake.org/download/](https://cmake.org/download/)。

这里以当前最新的cmake3.5.2版本为例，如果你和我一样使用的是Linux那么只需要下载*cmake-3.5.2-Linux-x86_64.tar.gz*或*cmake-3.5.2-Linux-i386.tar.gz*，解压到任何一个你觉得合适的目录下，将解压后的**bin**文件夹添加到`$PATH`环境变量中即可。

### Hello World 示例

创建一个**project**文件夹作为项目的根文件夹，并在其中创建一个**main.c**文件，内容如下：

```C
#include <stdio.h>

int main(int argc, char *argv[]) {
	printf("Hello World!\n");

	return 0;
}
```
### 使用Cmake来构建项目

#### 创建CMakeLists.txt

Cmake使用**CMakeLists.txt**文件来进行配置。对于我们的Hello World文件，**CMakeLists.txt**极为简单：

```cmake
# 用来构建该项目所用cmake的最低版本。
cmake_minimum_required (VERSION 2.6)

# 项目名
project (Hello_World)

# 使用main.c来生成hello。
add_executable(hello main.c)
```

#### 创建build文件夹

Cmake的一个有点就是它会将所有的中间文件，例如.o文件等放在一个文件夹中，从而不会让项目的源码变得混乱。在这里我们使用**build**文件夹来保存所有的中间文件。

```bash
mkdir build
cd build
```

#### 开始构建项目

在**build**文件夹中执行:

```bash
cmake ..
```

这时会在**build**文件夹中生成一个Makefile文件（对于Linux用户来说，如果是其他用户可能会不同），下面只需要执行**make**命令就可以生成最终的二进制文件了。

```bash
make
```

执行该二进制文件可以看到：

```bash
./hello
Hello World!   # Output
```

这时的文件树结构如下：

```
.
├── build
│   ├── CMakeCache.txt
│   ├── CMakeFiles
│   ├── cmake_install.cmake
│   ├── hello
│   └── Makefile
├── CMakeLists.txt
└── main.c
```

---


## 多文件构建

在实际的项目中，往往会有许多不同的子目录，用来保存项目不同模块的源码。使用cmake可以方便的管理这些分布在不同文件夹中的源码。

本文会使用cmake对多个不同文件夹下的c文件进行编译，最后再链接成二进制执行文件。



### 项目文件结构

这次用到的文件目录结构如下：

```bash
tree -L 2
```

```
.
├── build
├── CMakeLists.txt
├── include
│   ├── echo.h
│   └── operations.h
├── lib
│   ├── CMakeLists.txt
│   ├── echo.c
│   └── operations.c
└── main.c
```

### 目录说明

在该项目中，`main()`函数保存在**main.c**文件中。**main.c**文件通过`#include`来引用**include/operations.h**头文件和**include/echo.h**。

在**include/operations.h**头文件中，声明了一个`int add(int, int)`函数，该函数打印一行字符并返回两个整数的和。`int add(int, int)`的具体实现在**lib/operations.c**中。

在**include/echo.h**头文件中，声明了一个`void echo(void)`函数，该函数用来代替之前`main`函数中打印**Hello World**的功能，具体的定义保存在**lib/echo.c**中。

在cmake中，每一个包含有源文件的子目录都可以被当做一个子项目，因此每一个子目录也需要包含**CMakeLists.txt**文件用来说明当前子目录的情况。

子目录中的**CMakeLists.txt**会共享其父目录中的变量和设置。

### 新加入的文件

- **include/echo.h**

   ```C
   #ifndef ECHO_H__
   #define ECHO_H__

   void echo(void);

   #endif
   ```

- **include/operations.h**

   ```C
   #ifndef OPERATIONS_H__
   #define OPERATIONS_H__

   int add(int, int);

   #endif
   ```

- **lib/echo.c**

   ```C
   #include <stdio.h>
   #include "../include/echo.h"

   void echo(void) {
           printf("Hello World!\n");
   }
   ```

- **lib/operations.c**

   ```C
   #include <stdio.h>
   #include "../include/operations.h"

   int add(int a, int b) {
           printf("Add opertion!\n");
           return a + b;
   }

   int add(int, int);
   ```


### CMakeLists.txt文件

- **lib**中的**CMakeLists.txt**

   **lib**下的**CMakeLists.txt**内容很简单，仅包含下面一行：

   ```cmake
   # 增加一个叫做**lib**的库目标（library target），并且这个库由**operations.c**和**echo.c**两个源文件生成。
   add_library (lib operations.c echo.c)
   ```

- 修改项目根目录下的**CMakeLists.txt**。

   在根目录下的**CMakeLists.txt**中需要说明哪些目录是包含头文件的，哪些目录包含源文件。

   添加如下内容：

   ```cmake
   # 用来构建该项目所用cmake的最低版本。
   cmake_minimum_required (VERSION 2.6)

   # 项目名
   project (Hello_World)

   # 编译器用来寻找include文件的目录
   include_directories ("${PROJECT_SOURCE_DIR}/include")

   # 添加包含源文件的子目录
   add_subdirectory ("${PROJECT_SOURCE_DIR}/lib")

   # 使用main.c来生成hello。
   add_executable (hello main.c)

   # 链接target所需要的库
   target_link_libraries (hello lib)
   ```

### 构建项目

构建方法可以前一样：

```bash
cd build
cmake ..
make
./hello
```

输出如下:

```
Hello World!
Add opertion!
c = 3
```

### 分开编译不同的源文件

之前使用`add_library (lib operations.c echo.c)`是将operations.c和echo.c一起编译成了**liblib.a**，然后在链接到最后的**hello**二进制文件中。

但是有时候，需要将不同的源文件编译成不同的库文件然后按需链接。这时需要修改**lib/CMakeLists.txt**为如下内容：

```cmake
add_library(echo echo.c)
add_library(operations operations.c)
```

并且修改顶层**CMakeLists.txt**中的`target_link_libraries()`为如下内容：

```cmake
target_link_libraries (hello echo operations)
```

从而告诉cmake`hello`需要`echo`和`operations`两个库。

之后删除**build**文件夹内的内容，重新构建就好了。

---


## 变量与宏

在cmake中内置了许多默认的变量，此外还可以根据自己的需要设置变量。cmake还可以根据模板文件自动替换其中的变量，从而实现自动添加版本号等功能。


### 变量

- 变量的定义与使用

   在CMakeLists.txt中定义变量使用`set()`命令。

   ```cmake
   set(变量名 值)
   ```

   如果要使用变量，需要用以下方式：

   ```cmake
   ${变量名}
   ```

- 内置变量

   cmake内置了很多变量，这些变量可以直接拿来使用，具体的可以使用**man**命令来查看。

   ```cmake
   man 7 cmake-variables
   ```

### 模板替换

在实际的项目中，我们一般需要定义一个宏来说明当前的版本号。

在camke中，我们可以通过定义模板，在替换其中的指定部分来针对不同的编译参数使用不同的变量：

首先我们创建模板，`cmake/config.h.in`：

```C
#define PROJECT_VERSION_MAJOR @VERSION_MAJOR@
#define PROJECT_VERSION_MINOR @VERSION_MINOR@
```

其中的`@...@`部分会被替换。

之后我们在**CMakeLists.txt**中添加如下内容：

```cmake
# 设置变量名
set (VERSION_MAJOR 2)
set (VERSION_MINOR 0)

# 复制config.h.in到config.h，并替换其中@中间的变量
configure_file (
	"${PROJECT_SOURCE_DIR}/cmake/config.h.in"
	"${PROJECT_SOURCE_DIR}/include/config.h"
	)
```

`configure_file()`会将**config.h.in**中的内容用之前设定的变量所替换，并将替换后的内容保存在**include/config.h**中。

### 宏变量

很多情况下，我们需要根据条件来编译不同的模块。这里假设我们需要根据实际情况来选择是否要输出`Hello World!`。

我们可以设置一个叫做`DUMP_HELLO`的宏变量，来控制是否需要打印。

修改**main.c**如下：

```C
#include <stdio.h>
#include "include/config.h"
#ifdef DUMP_HELLO
#include "include/echo.h"
#endif
#include "include/operations.h"

int main(int argc, char *argv[]) {
        printf("The current of hello is %d.%d\n",
               PROJECT_VERSION_MAJOR, PROJECT_VERSION_MINOR);

#ifdef DUMP_HELLO
        echo();
#endif

        int a = 1, b = 2;
        int c = add(a, b);
        printf("c = %d\n", c);

        return 0;
}
```

我们可以通过修改模板**config.h.in**来控制是否定义该宏，在**config.h.in**中添加如下内容：

```C
#cmakedefine DUMP_HELLO
```

并在**CMakeLists.txt**中添加：

```cmake
option (DUMP_HELLO
	"print Hello World when execute" OFF)
```

注意这里我们设置其为`OFF`也就是不打印`Hello World！`。

既然不打印了，那也就不需要在编译**echo.c**这个文件了，我们可以在**lib/CMakeLists.txt**中修改如下：

```cmake
if (DUMP_HELLO)
	add_library(echo echo.c)
endif (DUMP_HELLO)
```

这样就可以了。

---


## 文件的安装

项目构建好之后，需要进行安装，使用cmake可以很方便的将不同类型的文件安装到不同的文件夹下。在本文中，我们要将生成的二进制文件安装到目标目录下的**bin**中，**.h**文件安装到**include**文件夹下，而生成的两个**.a**文件安装到**lib**中。


### 项目文件结构

这次的项目文件结构和之前的相同：

```
.
├── build
├── cmake
│   └── config.h.in
├── CMakeLists.txt
├── include
│   ├── config.h
│   ├── echo.h
│   └── operations.h
├── lib
│   ├── CMakeLists.txt
│   ├── echo.c
│   └── operations.c
└── main.c
```

### `CMAKE_INSTALL_PREFIX`变量

`CMAKE_INSTALL_PREFIX`变量是cmake的一个内置变量，它的作用和使用传统`./configure`安装时的`--prefix=<path>`作用是相同的，即指定文件的安装位置，在默认的情况下，该变量的位置是`/usr/local/`。

如果在安装时，要制定到其他位置，需要使用

```bash
cmake -DCMAKE_INSTALL_PREFIX:PATH=<path>
```

### 操作过程

- 二进制文件和头文件的安装

   我们最终生成的二进制文件名为**hello**，我们期望将它安装在**bin**目录中，那么需要在项目根目录下的**CMakeList.txt**中添加如下一行：

   ```cmake
   install (TARGETS hello DESTINATION bin)
   ```

   其中`TARGETS`为固定，`TARGETS`后面跟多个文件，这些文件将被安装到`DESTINATION`指定的文件夹中，也就是**bin**文件夹。这里的`DESTINATION`也是固定用法。

   安装头文件与此类似：

   ```cmake
   install (FILES "${PROJECT_SOURCE_DIR}/include/config.h" DESTINATION include)
   ```

   不过这里使用`FILES`而不是<del>`TARGETS`</del>。这里我们将项目目录下`include/config.h`文件安装到**include**文件夹中。

- 库文件的安装

   由于我们的库文件在**lib**文件夹下，因此对于库文件的安装可以把`install()`命令写在**lib**文件夹下的**CMakeList.txt**中。

   在**lib/CMakeLists.txt**修改为如下内容：

   ```cmake
   if (DUMP_HELLO)
       add_library(echo echo.c)
       install (TARGETS echo DESTINATION lib)
   endif (DUMP_HELLO)

   add_library(operations operations.c)

   install (TARGETS operations DESTINATION lib)
   ```

### 重新编译安装

假如我们要安装到**/tmp/hello**，那么执行：

```bash
cd build
cmake -DCMAKE_INSTALL_PREFIX:PATH=/tmp/hello ..
make
make install
```

使用`tree`命令可以看到：

```
/tmp/hello
├── bin
│   └── hello
├── include
│   └── config.h
└── lib
    ├── libecho.a
    └── liboperations.a
```

---

### ¶ The end

转自：https://d0u9.win/posts/858347849.html