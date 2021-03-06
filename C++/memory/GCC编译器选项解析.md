﻿# GCC编译器选项解析

## 总览

总览(SYNOPSIS)

```
gcc [options] filename ...
g++ [options] filename ...       
```

### 总体选项(Overall Option)

| 选项 | 解释                                                         |
| ---- | ------------------------------------------------------------ |
| -c   | 编译或汇编源文件,但是不作连接.编译器输出对应于源文件的目标文件. |
| -o   | 指定输出文件，如果没有使用 `-o` 选项,默认的输出结果是:可执行文件为 `a.out` |

### 调试选项(DEBUGGING OPTION)

| 选项      | 解释                                                         |
| --------- | ------------------------------------------------------------ |
| -g        | 以操作系统的本地格式(stabs, COFF, XCOFF,或DWARF)，产生调试信息，GDB能够使用这些调试信息 |
| -g*level* | 请求生成调试信息,同时用level指出需要多少信息，默认的level值是2；  level1: 输出最少量的信息,仅够在不打算调试的程序段内backtrace.包括函数和外部变量的描述,但是 没有局部变量和行号信息； Level3: 包含更多的信息,如程序中出现的所有宏定义.当使用`-g3'选项的时候,某些调试器支持 宏扩展 |

#### 预处理器选项(Preprocessor Option)

| 选项         | 解释                                             |
| ------------ | ------------------------------------------------ |
| -Dmacro      | x                                                |
| -Dmacro=defn | 定义宏macro的内容为defn                          |
| -Umacro      | 取消宏macro，`-U` 选项在所有的 `-D` 选项之后处理 |

### 连接器选项(LINKER OPTION)

| 选项        | 解释                                                         |
| ----------- | ------------------------------------------------------------ |
| -l*library* | 连接名为library的库文件                                      |
| -shared     | 生成一个共享目标文件，常搭配 `-fPIC` 使用                    |
| -Wl,option  | 把选项option传递给连接器.如果option中含有逗号,就在逗号处分割成多个选项. |
| -symbolic   | 建立共享目标文件的时候,把引用绑定到全局符号上                |

### 目录选项(DIRECTORY OPTION)

| 选项    | 解释                                                |
| ------- | --------------------------------------------------- |
| -I*dir* | 在**头文件**搜索路径列表中添加 `dir` 目录           |
| -L*dir* | 在 `-l` 选项**库文件**搜索路径列表中添加 `dir` 目录 |

### 警告选项(WARNING OPTION)

| 选项    | 解释                                |
| ------- | ----------------------------------- |
| -w      | 禁止所有警告信息(不建议使用)        |
| -Wall   | 开启大部分警告提示(建议使用)        |
| -Werror | 视警告为错误;出现任何警告即放弃编译 |

### 优化选项(OPTIMIZATION OPTION)

| 选项          | 解释                                                         |
| ------------- | ------------------------------------------------------------ |
| -O0           | 不优化                                                       |
| -O1           | 优化.对于大函数,优化编译占用稍微多的时间和相当大的内存.      |
| -O2           | 多优化一些.除了涉及空间和速度交换的优化选项,执行几乎所有的优化工作.例如不进行循环展开(loop unrolling)和函数内嵌(inlining). 和-O选项比较,这个选项既增加了编译时间,也提高了生成代码的 运行效果. |
| -O3           | 优化的更多.除了打开-O2所做的一切,它还打开了-finline-functions选项 |
| -ffloat-store | 不要在寄存器中存放浮点变量.这样可以防止某些机器上不希望的过高精度, 如68000的浮点寄存器(来自 68881)保存的精度超过了double应该具有的精度.  对于大多数程序,过高精度只有好处.但是有些程序严格依赖于IEEE浮点数的定义.对这样的程序可以使用 `-ffloat-store`选项. |

### 机器相关选项(MACHINE DEPENDENT OPTION)

| 选项         | 解释                                                         |
| ------------ | ------------------------------------------------------------ |
| -mhard-float | 输出包含浮点指令的目标码.这是缺省选项                        |
| -msoft-float | 警告:没有为SPARC提供GNU浮点库.一般说来使用该机型本地C编译器的相应部件, 但是不能直接用于交叉编译.你必须自己安排,提供用于交叉编译的库函数.  `-msoft-float` 改变了输出文件中的调用约定;因此只有用这个选项编译整个程序才有意义. |

### 代码生成选项(CODE GENERATION OPTION)

| 选项  | 解释                                                         |
| ----- | ------------------------------------------------------------ |
| -fpic | 如果支持这种目标机,编译器就生成位置无关目标码.适用于共享库(shared library). |
| -fPIC | 如果支持这种目标机,编译器就输出位置无关目标码. 适用于动态连接(dynamic linking),即使分支需要大范围 转移. |

### 语言选项(LANGUAGE OPTIONS)

| 选项       | 解释                    |
| ---------- | ----------------------- |
| -ansi      | 支持符合ANSI标准的C程序 |
| -std=c99   | C99标准                 |
| -std=c++11 | 支持c++11               |

------

> **参考：**  
>  [GNU GCC 手册](https://link.jianshu.com?t=http://www.shanghai.ws/gnu/gcc_1.htm)  
>  [gcc常用编译选项汇总](https://link.jianshu.com?t=http://blog.csdn.net/lee244868149/article/details/38754153)   
>  [GCC编译详解](https://link.jianshu.com?t=http://www.cnblogs.com/azraelly/archive/2012/07/07/2580839.html)

作者：俩芦苇

链接：https://www.jianshu.com/p/223d8b6aa879

来源：简书

简书著作权归作者所有，任何形式的转载都请联系作者获得授权并注明出处。
