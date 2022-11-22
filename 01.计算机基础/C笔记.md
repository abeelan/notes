> 苦肝 40 小时 C 语言基础。

# 基础

## 计算机发展

- 第一代：采用**真空电子管**制作计算机
- 第二代：以**晶体管**为主要元件的计算机
- 第三代：IBM 推出的系列计算机
- 第四代：**大规模集成电路和微处理器**

1946 年，第一台电子数字积分计算机 -- **ENIAC**，它的主要元件是**电子管**。

**冯诺伊曼**提出了**存储程序**的概念。



## 计算机语言发展

1. 机器语言

用**二进制代码**表示的能被计算机识别和执行的指令集合。

2. 汇编语言

**汇编程序**对**汇编语言源程序**进行汇编，生成一个可重定位的目标文件。优点是利用**助记符**代替机器语言。

3. 高级语言

使用**高级语言**编写的程序称为**源程序**，源程序翻译为二进制程序后执行。

翻译方式分为两种：

- 编译方式：将源程序全部翻译为二进制程序后再执行，完成翻译工作的程序称为**编译程序**，编译后的二进制文件称为**目标程序**。
- 解释方式：翻译一句执行一句，边解释边执行，完成翻译工作的程序称为**解释程序**。

世界上第一个高级语言是**FORTRAN**。



## 算法概念

定义：**解决问题的步骤序列就是算法。**

特性：**可执行性、确定性、有穷性、有输入信息的说明、有输出信息的步骤。**

算法的描述方法：自然语言、传统流程图、N-S流程图、伪代码（介于**自然语言**和**计算机语言**之间）、计算机语言。



## 程序和程序设计方法

计算机程序是指根据**算法描述**，用**计算机语言**表示的能被计算机识别和执行的**指令集合**。

```c
# include <stdio.h>
void main() {
    printf("This is a c program!\n");
}
```

程序 = 数据结构 + 算法 + 程序设计方法 + 程序设计语言 + 开发环境



程序设计方法分为两种：
- **面向过程（结构化程序）设计方法**（C 语言）
- 面向对象程序设计方法（C++、Java）。



##### 结构化程序设计方法

以 **模块功能和处理过程设计** 为主的详细设计的基本原则。 

优点

- 采用 **自顶向下，逐步求精** 的设计方法
- **先易后难，先抽象后具体**
- 程序由 **相互独立的模块** 构成

缺点

- **难以**在需求分析阶段被准确确定
- 开发周期长，需求若发生变化，需要推倒重来



##### 面向对象程序设计方法

吸取结构化程序设计方法的一切优点。

- 采用**数据抽象和信息隐藏技术**使组成类的**数据和操作不可分割**，避免数据和过程分离引起的错误。
- 由**类、对象（类的实例）和对象之间的动态联系**组成的。



# C 语言基础

### 简介

C 语言源于 **ALGOL60** 语言，1963 年剑桥大学发展为 **CPL**，1970 年剑桥大学进行简化为 **BCPL**，1970 年美国贝尔实验室以此为基础设计出 **B语言**，1972 年贝尔实验室在此基础上最终设计出 **C语言**。1978 年贝尔实验室正式推出。



C 语言的特点：

- **结构化语言**
- 运算能力强大
- 数据类型丰富
- 具有预处理能力
- 可移植性好
- 程序执行效率高
- 程序设计自由度大



##### 字符集

字符是组成语言最基本的元素，由**字母、数字、空格、标点和特殊字符**组成，C 语言的字符集就是 ASCII 字符集。

![](https://gitee.com/abeelan/image-hosting-service/raw/master/img/image-20220315181924002.png)



### 词汇分类

C 语言的词汇分为六类：常量、标识符、关键字、运算符、注释符和分隔符。



##### 1. 常量

又称为常数，是在程序运行过程中其值**不能改变**的数据。

![](https://gitee.com/abeelan/image-hosting-service/raw/master/img/image-20220316142854551.png)



字符常量和字符串常量是不同的，区别在于：

- 定界符不同，字符常量使用**单引号**，字符串常量使用**双引号**
- 长度不同，字符常量长度固定为 1，字符串常量长度大于等于 0
- 存储要求不同，字符常量只占用 1 字节，字符串常量为 n+1 字节（包含 \0）



##### 2. 标识符

用户对变量、符号常量、自定义函数等进行命名，形成用户标识符。

- 只能由 字母、数字、下划线组成，第一个字符不能是数字
- 大小写字母含义不同，一般小写
- 不能使用关键字，不能与库函数重名
- 命名应**见名知意、不宜混淆**
- 必须**先定义，后使用**



##### 3. 关键字

又称为“保留字”，主要用于构成语句，共 32 个，均由小写字母组成。

![](https://gitee.com/abeelan/image-hosting-service/raw/master/img/image-20220315184519595.png)



##### 4. 运算符

除单目运算符、赋值运算符和条件运算符（三目）是右结合之外，其他的运算符都是左结合。换言之，双目运算符中除赋值运算符外，剩下所有的运算符都是左结合。

![](https://gitee.com/abeelan/image-hosting-service/raw/master/img/%E8%BF%90%E7%AE%97%E7%AC%A6.jpg)

表达式中的运算符都是算术运算符的称为**算数表达式**，算数表达式由**运算对象**（常量、变量和函数等）、**圆括号**和**算术运算符**组成。

 

##### 5. 注释符

- 单行注释：`//`
- 多行注释：`/* ... */`



##### 6. 分隔符

- 逗号：用于类型说明和函数参数表中，分隔各个变量
- 空格：用于语句各单词之间，作为间隔符



### 变量

##### 变量的定义

定义：其值可以改变的量。

```c
# include <stdio.h>

int main() {
    int a;  // 变量名
    a = 3;  // 变量值
    printf("a=%d", a);
    
   	// 变量的初始化有多种方法
    int b, c;
    int d = 5, e, f = 6;
    // int x = y = 1;  // 错误写法
    
    // b=11 c=5 e=6
    b = (c = 5) + (e = 6);

    return 0;
}
```

任何变量必须**先定义后使用**，变量定义后，系统自动为其分配连续的内存单元，所占用的字节数取决于变量的数据类型。



##### 变量的类型

![](https://gitee.com/abeelan/image-hosting-service/raw/master/img/image-20220316142125384.png)



##### 有名常量

如果定义了变量并赋予其初值，但不希望程序中对其值进行修改，则可以将该变量定义为有名常量。

```c
int main() {
    const int i = 1, j = 2;
    const char c1 = 'C', c2 = 'c';
    return 0;
}
```



##### 变量的存储类型

动态存储方式：

- 自动型（auto）
- 寄存器型（register）

静态存储方式：

- 外部型（extern）
- 静态型（static）



##### 变量的作用域

局部变量：在函数内定义，只在本函数内有效。不同函数中同名变量，占不同内存单元。可用的存储类型：auto、register、static，默认为 auto。

全局变量：外部变量，在函数外定义，可为本文件所有的函数共用。从定义变量的位置开始到本源文件结束，及有 extern 说明的其他源文件。

```c
// static 变量具有继承性
int main() {
    void increment();
    increment();
    increment();
    increment();
    return 0;
}

void increment() {
    static int x;
    x++;
    printf("%d\t", x);
}

// 调用了三次，用 static 声明 x 变量后，输出为：1 2 3
// 如果未用 static 声明的话，每次 x 的值都是 1
```



### 数组

数组是具有相同数据类型的一组有序数据的集合。数组中的数据称为数组元素，通过数组名和下标引用。

- 数组名表示内存首地址
- 数组下标从 0 开始



##### 一维数组

```c
// 先给数组赋值，再倒序打印出来
int main() {
    int a[5], i;  // 定义长度为 5 的整型数组
    for (i = 0; i < 5; i++) {
        a[i] = i;
    }
    for (i = 4; i < 0; i--) {
        printf("%d", a[i]);  // 4 3 2 1 0
    }
    return 0;
}

// 打印数组元素
int main() {
    int a[5] = {1, 2, 3, 4, 5};  // 可以直接赋值
    int i;
    for (i = 0; i < 5; i++) {
        printf("%d", a[i]); 
    }
}
```



##### 二维数组

```c
/*
二维数组的初始化：分行初始化；按元素排列顺序初始化
int a[3][4];  // 三行四列

int a[][3] = {1, 2, 3, 4, 5};  // 自动填充
a[0] => 1, 2, 3
a[1] => 4, 5, 0（没有值就补 0）
*/


// 将二维数组行列元素互换，存到另一数组中
int main() {
    int a[2][3] = {{1, 2, 3}, {4, 5, 6}};
    int b[3][2];  // 存放互换后的元素
    int i, j;
    
    printf("原数组：\n");
    for (i = 0; i < 2; i++) {
        for (j = 0; j < 3; j++) {
            printf("%d\t", a[i][j]);
            b[j][i] = a[i][j];
        }
        printf("\n");
    }
    
    printf("互换后：\n");
    for (i = 0; i < 3; i++) {
        for (j = 0; j < 2; j++) {
            printf("%d\t", b[i][j]);
        }
        printf("\n");
    }
    
    /*
    原数组：
    1	2	3	
    4	5	6	
    互换后：
    1	4	
    2	5	
    3	6	
    */
    return 0;
}
```



##### 字符数组和字符串

```c
// 一维数组初始化
char a[6] = {'c', 'h', 'i', 'n', 'a', '\0'};
char b[6] = {99, 104, 105, 110, 97, 0};  // china 用 ascii 码初始化
char c[6] = {"china"};  // 字符串末尾会自动加 \0
char d[6] = "china";
char e[8] = {"abc"};  // 未提供值的部分自动赋值 \0

// 二维数组初始化,同上
char x[2][10] = {{'c', 'h', 'i', 'n', 'a'}, {'n', 'i', 'c', 'e'}};
char y[2][10] = {{"china"}, {"nice"}};

// 输出字符串遇到 \0 停止
int main() {
    char a[] = {'h', 'e', 'l', '\0', 'l', 'o'};
    printf("%s", a);  // hel
}

// 二维字符数组输出
int main() {
    char x[2][10] = {{'c', 'h', 'i', 'n', 'a'}, {'n', 'i', 'c', 'e'}}; 
    int i, j;

    // 方法一
    for (i = 0, i < 2; i++) {
        for (j = 0; x[i][j] != 0; j++) {
            printf("%c", x[i][j]);
        }
        printf("\n");
    }
    
    // 方法二
    for (i = 0; i < 2; i++) {
        printf("%s\n", x[i]);
    }
    
    // 方法三
    for (i = 0; i < 2; i++) {
        puts(x[i]);
    }
} 
```



### 指针

##### 指针变量

```c
// 指针变量的初始化
int main() {
    /*
	定义时：指针变量必须带 * 
	使用时：不带 * 代表地址，带 * 代表实际值
	*/
    int i, 
    int *p, *q;  // 定义指针变量
   	p = &i;  // 指针变量赋值，需要为地址符，也可以是数组名
    q = p;  // 也可以是指针变量名
    
	*p = 5;  // *指针变量名 代表里面具体的值
    
    return 0;
}
```

```c
void example(){
    // 通过指针变量访问整型变量
    int a, b, *p1, *p2;
    a = 100;
    b = 10;
    p1 = &a;
    p2 = &b;

    // %x 代表 16 进制的占位符
    printf("example ==> \n");
    printf("a=%d, b=%d\n", a, b);  // a=100, b=10
    printf("*p1=%d, *p2=%d\n", *p1, *p2);  // p1=100, p2=10
    printf("&a=%x, &b=%x\n", p1, p2);  // &a=e32750fc, &b=e32750f8
    printf("p1=%x, p2=%x\n", p1, p2);  // p1=e32750fc, p2=e32750f8
    printf("&p1=%x, &p2=%x\n", &p1, &p2);  // &p1=e32750f0, &p2=e32750e8；
    // 指针变量也是变量，加上 & 地址符也是取地址
}
```



##### 指针一维数组

数组的指针是指向数组在内存的起始地址（数组名），数组元素的指针是指向数组元素在内存的起始地址。

```c
// 指针数组及元素初始化
int main() {
    /* 指针数组 定义*/
    int a[10], *p = array;  // 定义时指针变量需要加 * 号
    
    int a1[10], *p;
    p = a1;  // 程序中赋值不需要加 * 号
    
    /* 指针元素 定义*/
    int a2[10], *p = &a2[0];  // 数组元素赋值，需要带地址符
    
    int a3[10], *p;
    p = &a3[0];
    
    /* 指针变量引用一维数组的方法
    1、*(指针变量 + i)
    	*(p + 0) == *p == p[0]
    	*(p + 2) == p[2]
    2、*(数组名 + i)
    	*(a + 2) == a[2]
    3、指针变量[i]
    	p[2]
    4、数组名[i]
    	a[2]
    */
    return 0;
}
```



##### 指针字符串

```c
// 指针字符串初始化
int main() {
    char *s = "abcd";  // 直接定义
    
    char *s1;  // 在程序中赋值
    s1 = "abcd";
    
    // 两种方法都是将存放字符串连续内存单元的首地址赋值到指针变量
    
    printf("%s", s);  // 输入整个字符串
    printf("%s", s1);
    
    printf("%c", *s);  // 输出首字符
    
    return 0;
}
```

```c
int main() {
    char string[] = "I Love China.";
    printf("%s", string);  // I Love China.
    printf("%s", string + 7);  // China.
    
    return 0;
}
```



##### 指针数组

```c
# include <stdio.h>

// 输入三个国家的名称，按字母顺序排序后输出
int main() {
    char *s[] = {"China", "America", "Russia"}, *p;
    int i, j, k = 3;

    // 控制总循环轮次
    for (i = 0; i < k - 1; i++) {
        // 两两交换，3 个元素需要 两次，也就是 j < 3 - 1 得 0 1 次循环
        for (j = 0; j < k - 1; j++) {
            if (strcmp(s[j], s[j + 1]) > 0) {
                // 把小得字符串往前面放，进行互换
                p = s[j];
                s[j] = s[j + 1];
                s[j + 1] = p;
            }
        }
    }
	
    // 打印数组
    for (i = 0; i < k; i++) {
        printf("%s\t", s[i]);
    }

    return 0;
}
```



### 基本语句分类

C 语言基本语句分类为：

- 数据定义语句
- 赋值语句
- 函数调用语句：函数名(参数1， 参数2); 参数可以为空
- 表达式语句：计算表达式的值
- 流程控制语句：共 9 种，看下图
- 复合语句：大括号内可以写多条语句，顺序执行
- 空语句：void

C 语言的每个语句都以**分号**结束。

![](https://gitee.com/abeelan/image-hosting-service/raw/master/img/image-20220316172636256.png)



### 程序基本组成

- 每个 C 程序由一个或多个函数组成
- 每个 C 程序有且只有一个主函数（main 函数）
- **函数**是 C 程序的基本单位，每个函数是由**函数首部**和**函数体**两部分组成
- 每一语句后面都以分号结束，但预处理命令、函数首部和右花括号不写分号
- C 程序的执行**总是从主函数开始，并在主函数结束**
- C 程序可以有预处理命令，通常写在程序的最前面
- 主函数可以调用任何其他函数，任何其他函数之间可以相互调用，但**不能调用主函数**



##### C 语言格式特点

- 习惯小写字母，大小写敏感
- 不使用行号，**无程序行**概念
- 可使用空行和空格
- 常用锯齿形书写格式



##### 程序的运行步骤

- 编辑：新建程序文件，(`*.c`或`*.cpp`)
- 编译：**编译程序**对源程序进行编译得到`*.obj`文件
- 连接：经过编译的目标程序还无法直接运行，因为源程序中可能包含库函数，因此需要把库函数的处理过程连接到目标程序中，生成`*.exe`可执行文件
- 运行：在操作系统中直接运行



##### 编译预处理命令

编译预处理命令是指对源程序进行编译之前，先对源程序中的各种预处理命令进行处理，再将处理结果和源程序一起进行编译，以获得目标代码。包括以下三个部分：

- 宏定义命令
- 文件包含命令
- 条件编译命令



1、宏定义命令

宏定义用于程序中的符号常量、类型别名、运算式和语句代换等，分为两种宏定义命令：

- 有参宏定义：`# define 宏名(参数表) 宏体`
- 无参宏定义：`# define 宏名 字符序列`

```c
# include <stdio.h>
# define PI 3.1415926
# define S(r) PI*r*r
# define X 10-4

int main() {
    float x = 3.6, area;
    area = S(x);
    printf("r=%f\narea=%f\n", x, area);
    /*
    	r=3.600000
     area=40.715038  
    */
    
    // 宏只是原样替换
    printf("%d", 6 * x);  // 6 * 10 - 4 = 56
}
```

优点：

- 提高源程序的可读性
- 提高源程序的可修改性，一改全改
- 避免源程序中重复书写字符串



2、文件包含命令

```c
# include <stdio.h>  
// 直接去系统指定的“包含文件目录”查找

# include "stdio.h"
// 先在当前目录下查找源文件
// 如果没找到再去系统指定的“包含文件目录”查找
```



3、条件编译命令

用户可以选择对源程序的一部分内容进行编译，即不同的编译条件产生不同的目标程序。

```c
// 标识符必须是被 #define 命令定义

// 格式一: 当标识符被定义
# ifdef 标识符
	程序段1
# else
	程序段2
# endif
        
// 格式二：当标识符没被定义
# ifndef 标识符
	程序段1
# else
	程序段2
# endif
        
// 格式三：为真执行程序 1，否则程序 2
// 表达式是在编译阶段计算值的，必须是常量或 define 定义的值       
# if 表达式
	程序段1
# else
	程序段2
# endif
        
// 以上 else 部分都可以省略       
```



##### 函数

函数的实参在**数量、类型和顺序**上与形参必须一一对应和匹配。

```c
# include <stdio.h>

//比较两数并输出最大值
int main() {
    int max(int x, int y);  // 函数调用前先声明
    int a, b, c;
    
    a = 10;
    b = 8;
    c = max(a, b);  // a,b 是实参
    
    printf("Max num is %d", c);
    return 0;
}

int max(int x, int y) {  
    // x,y 是形参
    int z;
    z = (x > y) ? x : y;
    return z;
}
```



调用其他文件内的函数，使用 `extern` 声明。

```c
// func.c
int max(int x, int y) {
    return x > y ? x : y;
}
```

```c
// call.c
# include "func.c"

int main() {
    extern int max(int x, int y);
    int x, y, z;
    z = max(x, y);
    printf("Max num is %d", z);
}
```



函数的参数和数据传递方式，分为三种：**值传递、地址传递、返回值和全局变量传递** 四种方式。

```c
# include <stdio.h>

// 数组选择排序实现
int main() {
    void sort(int array[], int n);
    int a[10], i;
    
    printf("Enter the array:\n");
    for (i = 0; i < 10; i++) {
        scanf("%d", &a[i]);
    } 
    
    sort(a, 10);  // 地址传递
    
    printf("The sorted array:\n");
    for (i = 0; i < 10; i++) {
        printf("%d", a[i]);
    }
    printf("\n");
}

void sort(int a[], int len) {
    // 找到最小值，并与当前的数进行替换
    int i, j, pos, temp;

    for (i = 0; i < len - 1; i++) {
        pos = i;

        // i 与后面所有的数进行比较，找到最小值就覆盖掉，然后与 i 进行替换
        for (j = i + 1; j < len; j++) {
            if (a[j] < a[pos]) {
                pos = j;  // 重新赋值最小值
            }
        }
        // pos 为最小值的下标
        temp = a[i];
        a[i] = a[pos];
        a[pos] = temp;
    }
}
```



##### EXAMPLE

```c
# include <stdio.h>
# include <math.h>

int main() {
    void triangle(void);
    void output_char();

    triangle();
    output_char();

    return 0;
}

void triangle() {
    // 输入三边长，求三角形的面积
    float a, b, c, s, area;
    scanf("%f%f%f", &a, &b, &c);

    s = (a + b + c) / 2;
    area = sqrt(s * (s - a) * (s - b) * (s - c));

    printf("a=%.2f b=%.2f c=%.2f \ns = %.2f\narea = %.2f\n", a, b, c, s, area);

    /*
        a=3.00 b=4.00 c=6.00
        s = 6.50
        area = 5.33
    */
}

void output_char() {
    // 输入大写字母，输出小写字母及 ascii 码
    int c1, c2;
    c1 = getchar();
    printf("[ INPUT  ]%c -> %d\n", c1, c1);

    c2 = c1 + 32;
    printf("[ OUTPUT ]%c -> %d\n", c2, c2);
    
    /*
        [ INPUT  ]Z -> 90
        [ OUTPUT ]z -> 122
    */
}
```



##### 常用库函数

![](https://gitee.com/abeelan/image-hosting-service/raw/master/img/%E5%B8%B8%E7%94%A8%E5%87%BD%E6%95%B0.jpg)



### 程序设计

##### if 条件语句

```c
// if-else 双分支选择结构
int main() {
    int x, y;
    x = 20;
    y = 10;
    if (x > y) {
        printf("%d", x);
    } else {
        printf("%d", y);
    }
    return 0;
}

// if-else-if
int main() {
    int num, cost;
    if (num > 500) {
        cost = 0.15;
    } else if (num > 300) {
        cost = 0.1;
    } else if (num > 100) {
        cost = 0.075;
    } else if (num > 50) {
        cost = 0.05;
    } else {
        cost = 0;
    }
    printf("cost = %d", cost);
    return 0;
}

// 例：输入一个字母，如果是大写需要转为小写输出
int main() {
    char ch;
    scanf("%c", &ch);
    // 条件表达式
    ch = (ch >= 'A' && ch <= 'Z') ? ch + 32 : ch;
    printf("%c", ch);
    return 0;
}
```



##### switch 多分支选择语句

```c
# include <stdio.h>

// 根据字符打印对应的分数
int main() {
    char grade;
    grade = getchar();
    switch (grade) {
        case 'A': {printf("85~100\n"); break;}
        case 'B': {printf("70~84\n"); break;}
        case 'C': {printf("60~69\n"); break;}
        case 'D': {printf("<60\n"); break;}
        default: printf("Error\n");
    }
}
```



##### while 当型循环语句

```c
# include <stdio.h>

// 求 1-100 的和
int main() {
    int i = 1, sum = 0;
    while (i <= 100) {
        sum += i;
        i++;
    }
    printf("sum = %d", sum);
    return 0;
}
```



##### do-while 直到型循环语句

```c
# include <stdio.h>

// 求 1-100 的和
int main() {
    int i = i, sum = 0;
    do {
        sum += i;
        i++;
    } while (i <= 100);
    printf("sum = %d", sum);
    return 0;
}
```



##### for 次数型循环语句

```c
# include <stdio.h>

// 求 1-100 的和
int main() {
    int i; sum = 0;
    for (i = 1; i <= 100; i++) {
        sum += i;
    }
    printf("sum = %d", sum);
    return 0;
}
```



### 结构体类型

##### 定义及引用

```c
// 定义
struct 结构体类型名 {
    int a;
    char b;
    数据类型符 成员名;
};

// 定义结构体的同时，定义结构体类型变量
struct student {
    int no;
    char name[10];
    char sex;
} zhangsan = {1, "zhangsan", 'M'};

// 定义结构体类型变量；顺序数据类型都不能错
struct student lisi = {2, "lisi", "W"}; 
```

结构体类型名是用户命名的标识符，结构体内的数据类型符可以是基本的数据类型符，也可以是用户自定义的某种结构体类型。末尾**分号**不能省略。

结构体变量占用内存的字节数是**各成员占用内存字节数之和**。

```c
// 引用；注意不能整体引用
// 只能通过结构体变量名.成员名进行引用
zhangsan.no = 1;
```

结构体类型变量成员地址的引用方法：

- &结构体类型变量名.成员名：成员的地址
- 结构体类型变量名.成员名：成员是数组，就是首地址，否则就是值
- &结构体类型变量名.成员数组[下标]：成员是数组，引用数组元素



##### 结构体数组

```c
// 定义
struct student {
    int no;
    int name[10];
}

struct student s[3] = {
    {1, "zhangsan"},
    {2, "lisi"},
    {3, "wangwu"}
};

// 也可以在定义结构体类型的同时赋初值

// 定义无名称结构体并赋初值
struct {
    int no;
    int name[10];
} s[2] = {{1, "zhangsan"}, {2, "lisi"}};


// 引用
s[0].no = 0;
```



##### 结构体类型指针

指向结构体类型变量的指针变量。

```c
struct student stu1;
struct student *p=&stu1;
stu1.no = 101;  // ==> (*p).no = 101

/*
指针变量 p 已指向结构体类型，以下三种方式等价：
	stu1.no = 101;
	(*p).no = 101;  // 注意括号不能省略
	p -> no = 101;
*/
```



### 自定义类型

typedef 将原类型符定义为用户自定义的新类型符。

```c
int main() {
    // 自定义基本类型
    typedef int abee;
    abee 10;  // 等价于 int 10;
    
    // 自定义数组类型
    typedef int array[10];
    array a, b, c;  // 等价于 int a[10], b[10], c[10];
    
    // 自定义结构体类型
    typedef struct date {
        int year, month, day;
    } DATE;
    DATE s1, s2;
    
    // 自定义指针
    typedef int *P;
    typedef char *PC[5];
    P p1, p2;  // 等价于 int *p1, *p2;
    PC pc1, pc2;  // 等价于 char *pc1[5], *pc2[5];
}
```



### 文件

##### 概述

文件是由一个一个的 **字符（ASCII码文件）或字节（二进制文件）** 组成的，称为流式文件。

存储在磁盘上的文件称为“磁盘文件”。

![](https://gitee.com/abeelan/image-hosting-service/raw/master/img/image-20220318143501586.png)



##### 文件的打开和关闭函数

文件使用方式中各字符的含义：
- r：读
- w：写
- a：追加
- b：二进制文件（rb、wb、ab）
- +：读写

```c
// 文件打开函数
FILE *fp;
fp = fopen("a.c", "w");
if (fp == NULL) {
    // 文件打开失败，返回 NULL
    printf("File open error!\n");
    exit(0);  // 关闭所有文件调用
}

fclose(fp);  // 文件关闭函数，正常关闭返回 0，否则为 EOF（-1）
```

标准设备文件的打开和关闭，程序开始运行时，系统自动打开三种标准设备文件，并分别定义了响应的文件型指针。其中：

1. stdin 指向标准输入（通常为键盘）
2. stdout 指向标准输出（通常为显示器）
3. stderr 指向标准错误输出（通常为显示器）

三种保准设备文件使用后，不必关闭，因为系统退出时将自动关闭。



##### 文件的读写函数

```c
char c = "A";
fputc(c, fp);  // 把一字节代码写入文件中；正常返回 c，出错返回EOF（-1）
fputs(str, fp);

fgetc(fp);  // 从文件中读取一字节代码，正常返回读到的代码值，读到文件尾或出错返回 EOF（-1）
fgets(fp);

feof(fp);  // 判断读取文件是否结束; 结束返回非 0，否则 0

// 从 buffer 开始的 count 个数据，一次性全部写入 fp；正常返回 count，否则返回 0
fwrite(void *buffer, int size, int count, FILE *fp);

// 正常返回 count，否则返回 0
fread(void *buffer, int size, int count, FILE *fp);

// 格式化写函数; 正常返回输出字符个数，否则返回负数
fprintf(fp, char *format[,arg...]);

// 格式化读函数；正常返回读入数据个数，否则返回 EOF
fscanf(fp, char *format[,arg...]);
```


##### 文件的定位函数

```c
// 将 fp 指向文件的指针重置到文件头；正常返回 0，否则返回非 0
int rewind(FILE *fp)

// 将文件的位置指针随机移动，从 origin 开始移动 offset 位
// 正常返回 0，否则返回非 0
int fseek(FILE *fp, long offset, int origin)
```
<br>

完，没油了。