---
layout: post
title: 'Compiling Stuffs'
data: 2023-03-13
author: 刘浩光
cover: 'assets/images/header/compiling.png'
tags: Programming
---

## GCC/G++ 编译、make 编译、CMake 编译

> 计算机只能识别二进制代码，并不能够直接识别诸如 C++ 等高级语言编写的代码（源代码），所以将编写的高级语言代码转换为二进制代码，这样计算机才可以识别运行。

根据源代码转换为二进制代码的时机不同，可以将语言分为 编译型 语言和 解释型 语言。

* 编译型语言：使用编译器一次性将代码编译为可执行文件，一次编译可重复执行，代表有 C、C++、Golang 等。其有着执行效率高，编译后程序不可修改，保密性好等特点，但其可移植性差。
* 解释型语言：使用解释器一边解释一边执行，甬道哪些源代码就解释哪些源代码，不会生成可执行文件，代表有 Python、JavaScript、PHP 等。其移植性较好，只要有解释环境即可，但运行速度慢，效率低。

***注：*** Java 是一种混合型语言，其先将 `.java` 文件编译为 `.class` 类型文件，再解释运行。

### 1. GCC/G++ 简单编译

> 下述整个过程围绕 C++ 语言展开

从 C++ 高级语言到计算机可执行的二进制语言的过程叫做编译（compiling），单文件一步到位的编译指令为 `g++ main.cpp -o main`，其中 `main.cpp` 为要编译的文件，`-o` 为 output，后面的 `main` 为编译后的可执行文件名，通过 `./main` 运行。此过程一共要经历以下四个步骤：

1. 预处理：将文件里的头文件、宏定义等做展开或替换。使用 `g++ -E main.cpp -o test.i` 命令，参数 `-E` 表示仅处理到预处理阶段
2. 编译：将预处理后的文件编译（翻译）成汇编语言。使用 `g++ -S main.cpp -o test.s` 命令，参数 `-S` 表示仅处理到编译阶段
3. 汇编：将编译后程序翻译成计算机可以直接理解的二进制机械码。使用 `g++ -c main.cpp -o test.o` 命令，参数 `-c` 仅表示处理到汇编阶段
4. 连接：基于 `.o` 的目标文件，链接库，生成可执行文件。使用 `g++ test.o -o test` 命令，生成可执行文件

下面将项目目录复杂化，其目录结构如下：

```shell
cppDir % tree .
.
├── include
│   └── calc.h
├── main
├── main.cpp
└── src
    └── calc.cpp
```

- 二级目录 include 下有一个头文件，src 目录下放了实现它的源文件，根目录下的源文件 `main.cpp` 用来调用它
- 应该采用 `g++ main.cpp src/calc.cpp -Iinclude -o main` 命令来进行编译。`-I` 后跟自定义的头文件目录位置

#### 静态库与动态库

某些经常复用的代码块，可以编写为库文件，方便部署、发布，还可以对模块代码进行保密（对外仅提供接口即可）。

- 什么是库？

  库是写好的、现有的、成熟的、可以复用的代码。本质上来说，库是一种可执行代码的二进制式，可以被操作系统载入内存执行。库有两种：静态库 `（.a .lib）` 和动态库 `（.so .dll）`。所谓静态、动态是指链接。库文件是事先编译好的方法的合集

- 静态库和动态库的区别

  1. 静态库的扩展名一般为 `.a` 或 `.lib`；动态库的扩展名一般为 `.so` 或 `.dll`
  2. 静态库在编译时会直接整合到目标程序中，编译成功的可执行文件可独立运行；动态库在编译时不会放到连接的目标程序中，即可执行文件无法单独运行
  3. 静态库和动态库最本质的区别就是：**该库是否被编译进目标（程序）内部。**

  **静态（函数）库：**一般扩展名为 `（.a 或 .lib）`，这类的函数库通常扩展名为 `libxxx.a` 或 `xxx.lib`。这类库在编译的时候会直接整合到目标程序中，所以利用静态函数库编译成的文件会比较大，这类函数库最大的优点就是编译成功的可执行文件可以独立运行，而不再需要向外部要求读取函数库的内容；但是从升级难易度来看明显没有优势，如果函数库更新，需要重新编译

  **动态（函数）库：**一般扩展名为 `（.so 或 .dll）`，这类函数库通常名为 `libxxx.so` 或 `xxx.dll`。与静态函数库被整个捕捉到程序中不同，动态函数库在编译的时候，在程序里只有一个“指向”的位置而已，也就是说当可执行文件需要使用到函数库的机制时，程序才会去读取函数库来使用；也就是说可执行文件无法单独运行。这样从产品功能升级角度方便升级，只要替换对应动态库即可，不必重新编译整个可执行文件

两者的优缺点

- 静态库
  - 优点：静态库被打包到应用程序中加载速度快；发布程序无需提供静态库，移植方便
  - 缺点：相同的库文件数据可能在内存中被加载多份，消耗系统资源，浪费内存；库文件更新需要重新编译项目文件，生成新的可执行程序，浪费时间
- 动态库
  - 优点：可实现不同进程间的资源共享；动态库升级简单，只需要替换库文件，无需重新编译应用程序；可以控制何时加载动态库，不调用库函数动态库不会被加载 
  - 缺点：加载速度比静态库慢；发布程序需要提供依赖的动态库

#### 静态库和动态库的制作

- **静态库的生成**（以上面实例为例）

  - 静态库本身为一个二进制目标文件，采用 `g++ -c src/calc.cpp -Iinclude -o calc.o` 命令生成 `.o` 目标文件。

  - 将目标文件归档为 `.a` 的静态库。在命名的时候，采用 `lib + 库名 + 后缀` 格式，例如上述文件命名为 `libCalc.a`，代表名叫 Calc 的静态链接库。所采用的命令为 `ar rs libCalc.a calc.o`。

  - 静态库做好后，编译 `main.cpp` 并链接上此静态库，注意链接时也需要头文件的支持。`-l`表示链接哪个库，直接 `-l库名` 即可，还需要告诉编译器从哪里找这个库，使用 `-L` 进行指定。此处采用的命令为 `g++ main.cpp -Iinclude -L. -lCalc -o main`。

    ***注：*** 参数和参数实体中间空格可有可无，一般不写空格

- **动态库的生成**

  - 动态库以 `.so` 结尾，需要加上 `-shared` 参数指定生成的是动态库，直接用 `g++ src/calc.cpp -Iinclude -shared -fPIC -o libcalc.so` 命令生成（此处也可以分开来做，先使用 `-fPIC` 生存 `.o` 文件，再使用  `-shared` 生存 `.so` 文件）。此处的 `-fPIC` 作用于编译阶段，告诉编译器产生与位置无关代码（Position-Independent Code），因此产生的代码中，没有绝对地址，全部使用相对地址，故代码可以被加载器加载到内存的任意位置，都可以正确的执行。这正是共享库所要求的，共享库被加载时，在内存的位置不是固定的。
  - 接下来编译 `main.cpp`，并告诉连接器你想要链接哪个库。采用命令为`g++ main.cpp -Iinclude -L. -lCalc -o main`

- `-g` 参数使得在编译的时候生成调试信息，生成为可执行文件体积会稍微大一些

### 2. make 编译

当我们的工程扩展到很多个文件、很多个目录（方便管理、维护）的时候，直接通过 g++ 进行工程的编译就显得捉襟见肘了，此时就需要通过写 makefile 脚本，使用 make 命令来进行项目的构建。

makefile 是包含了整个工程编译规则的文件。其定义了一些列的规则来指定，哪些文件需要先编译，哪些文件需要后编译，哪些文件需要重新编译，甚至更复杂的功能。因此 makefile 就像一个 shell 脚本一样，其中也可以执行操作系统的命令。

makefile 的好处是**自动化编译**，一旦写好，只需要一个 make 命令，整个工程就可以自动编译，极大提高了开发的效率。再此之前，先粗略看看 makefile 的规则：

```shell
target ... : prerequisites ...
	command
	...
```

- `target` 可以是一个目标文件（object file），也可以是一个执行文件，还可以是一个标签（label）
- `prerequisites` 为生成该 target 所依赖的文件或 target
- `command` 是生成该 target 所需要执行的文件（或 任意的 shell 命令）

makefile 中最核心的内容为依赖关系。所以如果 target 不存在或 prerequisites 中有比 target 更新的文件，command 命令才会被执行。（更具时间戳进行判断）

下面就一个实际例子来进行简单的 make 编译说明：

```shell
cppDir % tree .
.
├── include
│   ├── test1.h
│   └── test2.h
├── main.cpp
├── makefile
└── src
    ├── test1.cpp
    └── test2.cpp
```

现在有以上的一目录结构，为其写一个最初级的 makefile 如下：

```shell
# /bin/sh/

main: main.o test1.o test2.o
    g++ main.o test1.o test2.o -Iinclude-o main

main.o: main.cpp
    g++ -c main.cpp -g -Wall -Iinclude -o main.o
test1.o: src/test1.cpp
    g++ -c test1.cpp -g -Wall -Iinclude -o test1.o
test2.o: src/test2.cpp
    g++ -c test2.cpp -g -Wall -Iinclude -o test2.o

clean:
    rm *.o main -rf
```

可以看到本项目 **最终要生成的目标文件（第一个出现的 target）**为 `main`，而 `main` 目标文件依赖于 `main.o test1.o test2.o` 三个二进制文件，三个二进制文件分别依赖于与它们同名的 `.cpp` 文件。

`clean` 后面没有依赖项，因此它是一个 label，可以使用 `make clean` 来执行此 label 下的 command 命令。它的作用是清理产生的中间文件。

但上面的写法有点繁琐（文件名写好几次了），在文件很多的时候写起来很麻烦，也很容易出错。所以可以通过变量的方式进行简化，简化后版本如下：

```shell
# /bin/sh/

OBJS=main.o test1.o test2.o
CC=g++
CFLAGS+=-c -g -Wall

main:$(OBJS)
    $(CC) $^ -Iinclude -o main

main.o: main.cpp
    $(CC) $^ $(CFLAGS) -Iinclude -o main.o
test1.o: src/test1.cpp
    $(CC) $^ $(CFLAGS) -Iinclude -o test1.o
test2.o: src/test2.cpp
    $(CC) $^ $(CFLAGS) -Iinclude -o test2.o

clean:
    $(RM) *.o main -r                     
```

使用 `OBJS` 变量来代替三个依赖项，`CC` 代替 g++， `CFLAGS` 代替编译所需的参数。makefile 中变量采用 `$()` 来表示，`$^` 表示为此处的依赖项， `$(RM)` 为默认的变量，表示 `rm -f`。这样就可以减少文件名的书写。但当文件比较多时，写起来还是很麻烦，所以又有了以下的终极版本：

```shell
# /bin/sh/

OBJS=main.o test1.o test2.o
CC=g++
CFLAGS+=-c -g -Wall
INCL=-Iinclude

main:$(OBJS)
    $(CC) $^ -o $@

%.o: %.cpp
    $(CC) $^ $(CFLAGS) $(INCL) -o $@
%.o: src/%.cpp
    $(CC) $^ $(CFLAGS) $(INCL) -o $@

clean:
    $(RM) *.o main -r
```

采用 `$@` 来表示 target ；采用类似于递推的方式，使用 % 在 `OBJS` 中进行通配来实现多文件的编译，代码复用性变得更强了。

以上只是一个简单的 makefile 使用示例，更详尽的 makefile 编写技巧见程浩老师的《跟我一起写Makefile》，其2022重置版 pdf 下载地址：https://seisman.github.io/how-to-write-makefile/Makefile.pdf

### 3. CMake 编译

未完待续。。。