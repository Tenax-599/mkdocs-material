---
date:
  created: 2025-02-08
tags:
  - C#
authors: [Tenax]
description: >
  Liu Tiemeng - C# Introductory Learning Diary.
---

# 刘铁猛 C#语言入门详解

<!-- more -->

## 类和命名空间

### 类的初识-程序主体和树形结构

```c#
namespace HelloWorld
{
  class Program
  {
     static void Main(string[] args)
     {
        System.Console.WriteLine("Hello, World!");
        System.Console.WriteLine("Good morning!");
     }
  }
}
```

如果没有引用命名空间`using System;`的时候，想用`Console`这个类必须在前面加带命名空间`System`，`System.Console`这叫权限命名

??? info "如何知道类是那个命名空间呢？"

     有两种方法：
    
     1.通过文档，visual studio中上方有help选项，View Help就是文档
    
     2.通过智能提示，liner中可以选择是通过using加入命名空间，还是在类前面加上命名空间。（加入命名空间快捷键为ctrl + .）

### 类库引用-黑盒引用和白盒引用

<u>不同的项目模板引用的只是不同的技术所需要的类库</u>

### 依赖关系-高内聚和低耦合

### 错误排除

## 类和对象

### 类的实例-引用变量

- 对象也叫实例，是类经过"实例化"后得到的内存中的实体

      Formally"instance”is synonymous with"object"一一对象和实例是一回事
        
      "飞机"与"一架飞机"有何区别？天上有（一架）飞机一一必需是实例飞，概念是不能飞的
        
      有些类是不能实例化的，比如"数学”（Math class），我们不能说“一个数学

- 依照类，我们门可以创建对象，这就是“实例化

      现实世界中常称"对象”，程序世界中常称"实例”
        
      二者并无太大区别，常常混用，初学者不必迷惑

- 使用 new 操作符创建类的实例

- 引用变量与实例的关系

      孩子与气球
      
      气球不一定有孩子牵着
      
      多个孩子可以使用各自的绳子牵着同一个气球，也可以都通过一根绳子牵着气球

```c#
namespace ClassAndInstance
{
    class Program
    {
        static void Main(string[] args)
        {
            Form myForm1;
            Form myForm2;
            myForm1 = new Form();
            myForm2 = myForm1;
            myForm1.Text = "My Form";
            myForm2.Text = "I changed it!";
            myForm2.ShowDialog();
        }
    }
}
```

`new Form()`这对圆括号表示，这个表单的实例在内存中诞生之后你用什么样的方法进行初始化，这个叫做构造器

### 类的实例-引用变量

- 属性（Property）

      存储数据，组合起来表示类或对象当前的状态

- 方法（Method）

      由 C 语言中的函数（function）进化而来，表示类或对象“能做什么”
        
      工作中 90%的时间是在与方法打交道，因为它是“真正做事”、“构成逻辑“的成员

- 事件（Event）

      类或对象通知其它类或对象的机制，为 C#所特有（Java 通过其它办法实现这个机制）
        
      善用事件机制非常重要

- 使用 MSDN 文档（鼠标移到某个类上，按下 F1）

- 某些特殊类或对象在成员方面侧重点不同

      模型类或对象重在属性，如 Entity Framework
      
      工具类或对象重在方法，如 Math，Console
      
      通知类或对象重在事件，如各种 Timer

### 类的成员-属性类方法类事件类

```c# title="侧重属性的类"
namespace PropertySample
{
  class Program
  {
    static void Main(string[] args)
    {
      AdventureWorksLT2012Entities proxy = new AdventureWorksLT2012Entities();
      foreach(Product p in proxy.Products)
      {
         Console.WriteLine(p.Name);
      }

      Console.WriteLine("==================");
      Console.WriteLine(proxy.Product.Count());
    }
  }
}
```

```c# title="侧重方法的类"
namespace MethodSample
{
  class Program
  {
    static void Main(string[] args)
    {
      double x = Math.Pow(2, 3);
      Console.WriteLine(x);
    }
  }
}
```

`Math`这个类中没有属性

```c# title="侧重事件的类"
namespace EventSample
{
  public partial class MainWindow:Window
  {
    public MainWindow()
    {
      InitializeComponent();
      DispatcherTimer timer = new DispatcherTimer();
      time.Interval = TimeSpan.FromSeconds(1);
      timer.Tick += timer_Tick;
      timer.start();
    }

    void timer_Tick(object sender, EventArgs e)
    {
      this.timeTextBox.Text = DateTime.Now.Tostring();
    }
  }
}
```

Timer 如果想使用，首先实例化对象`DispatcherTimer`，然后给属性`Interval`赋值，然后挂接事件处理器`timer_Tick`，最后`start()`方法调用

### 类的成员-静态成员和实例成员

- 静态（Static）成员在语义上表示它是类的成员
- 实例（非静争态）成员在语义表示它是“对象的成员
- 绑定（Binding）指的是编译器如何把一个成员与类或对象关联起来

  不可小靓的”“操作符——成员访问

早绑定，指的就是编译在编译这个类的时候，就已经知道这个成员是到底隶属于这个类呢还是隶属于这个类的对象

而晚绑定，是编译器不管这个事情，直到这个程序运行起来的之后，才由程序去确定，一个方法到底是属于某个类呢，还是属于那个对象（有晚绑定功能的这种语言一般叫做动态语言，比如说 JavaScript）

```c#
namespace StaticSample
{
    class Program
    {
        static void Main(string[] args)
        {
            Console.WriteLine("Hello!");
            Form form = new Form();
            form.Text = "Hello";
            form.ShowDialog();
        }
    }
}
```

`WriteLine`这个方法直接访问这个类，说明这个方法是隶属于这个类的，说明这个方法就是静态方法。通过引用变量`from`访问它的实例方法 `ShowDialog` 和属性 `Text`。

<u>MSDN 文档中，如果发现属性、方法或者事件有个红色大写的 S，说明它是个静态的。</u>

## 程序和语言的基本元素

### 五类元素

- 构成 C#语言的基本元素

```yaml title="标记（Token）"
关键字（Keyword）
操作符（Operator）
标识符（ldentifier）
什么是合法的标识符
怎么样阅读语言定义文档
大小写规范
命名规范
标点符号
文本（字面值）
整数
多种后缀
实数
多种后缀
字符
字符串
布尔
空（null）
```

注释与空白  
 单行  
 多行（块注释）

- 简要介绍类型、变量与方法
- 算法简介

标记就是对编译器来说有意义的这样的记号，编译器能识别出来这些东西

对类命名的时候一定要是名词或者名词的复数，对类的成员命名的时候，属性一定要是名词或者名词的复数，方法一定要是个动词，或者动词短语；在 c#中变量名用驼峰法，方法名、类名和命名空间用 pascal 法，第一个单字首字母采用大写字母；

什么样的标识符是合法的？首先这个标识符不能跟关键字冲突，因为关键字又叫保留字，保留字就是说这门编程语言已经把这个单词给保留了，不能够再拿这个单词去作为标识符来使用。其次可以用字母，大写字母小写字母、数字和下划线来组成你的标识符，但是不能拿数字开头，不能拿数字开头可以拿字母或下划线开头，那符合这样规范的标识符它就是合法的标识符。如果硬要拿一个关键字的单词想拿它当标识符，前面加上@符号就可以了。

```c#
namespace IdentifiersExample
{
    class Program
    {
        static str = null;
        From f = null;
        f.ShowDialog();
    }
}
```

没有引用一个实例，还要去访问这个实例的方法会报错`System.NullReferenceExecption`

## 程序数据算法-D01-类型-变量和方法

- 初识类型（Type）

      亦称数据类型（DataType）

- 变量是存放数据的地方，简称“数据

      变量的声明
        
      变量的使用

- 方法（旧称函数）是处理数据的逻辑，又称"算法

      方法白声明
        
      方法的调用

- 程序=数据+算法

      有了变量和方法就可以写有意义的程序了

```c#
namespace MyExample
{
    Class Program
    {
        static void Main(string[] args)
        {
            var x = 3;
            Console.WriteLine(x.GetType().Name);//int 32(32个比特位来存储的整数)
            var x = 3L;
            Console.WriteLine(x.GetType().Name);//加上个L说明是个长整型，int 64（64个比特位来存储的整数）
            var x = 3.0;
            Console.WriteLine(x.GetType().Name);//double类型，双精度浮点型
            var x = 3.0F;
            Console.WriteLine(x.GetType().Name);//single类型，单精度浮点型
        }
    }
}
```

像`var`这种叫自动识别变量的类型，一般会直接使用数据类型来声明变量比如`float x = 3.0F`

```C#
namespace MyExample
{
    Class Program
    {
        static void Main(string[] args)
        {
            int x;
            x = 100;//一个普通的整型字面值装在一个整型变量里
            x = 100L;//一个长整型的字面量装在一个整型变量里就不行，64位转不进32位里
            float x;
            x = 3.0;//float是单精度浮点型，3.0是双精度类型的字面值
            x = 3.0F;
            double x;
            x = 3.0;
        }
    }
}
```

`double x`变量的数据类型+变量名，变量名必须是一个合法的标识符；`x = 3.0`能够与这个变量类型匹配的字面值或者一个运算结果，通过`=`赋值符号，赋值给变量

```c#
namespace MyExample
{
    class Program
    {
        static void Main(string[] args)
        {
            Calculator c = new Calculator();
            // int x = c.Add(2, 3);
            // Console.WriteLine(x);
            // string str = c.Today();
            // Console.WriteLine(str);
            c.PrintSum(4, 6);
        }
    }

    class Calculator
    {
        public int Add(int a, int b)
        {
            int result = a + b;
            return result;
        }

        public string GetToday
        {
            int day = DataTime.Now.Day;
            return day.ToString();
        }

        public void PrintSum(int a, int b)
        {
            int result = a + b;
            Console.WriteLine(result);
        }
    }
}
```

Add 函数中接收类型为`int`整型 a 和 b 这两个参数，函数内部`int`整型 result 等于 a 加 b，最后用 return 返回回去，`int x = c.Add(2, 3);`，用 x 接收这个返回值，为了在`Calculator`这个类外面也能也能够访问 Add 这个函数在前面添加`public`；GetToday 这个方法没有要求接收参数，返回的是`string`类型，`string str`就用 string 类型 str 来接收；PrintSum 这个函数没有返回值就在前面加`void`，由于不采取返回值，PrintSum 这个函数选择直接打印了结果；

## 程序数据算法-D02-算法-循环和递归

- 循环初体验
- 递归初体验
- 计算 1 到 100 的和

```c# title="循环"
namespace MyExample
{
    class Program
    {
        static void Main(string[] args)
        {
            Calculator c = new Calculator();
            c.PrintXto1(10);
        }
    }

    class Calculator
    {
        public void PrintXto1(int x)
        {
            for(int i = x; i>0; i--)
            {
                Console.WriteLine(i);
            }
        }
    }
}
```

```c# title="递归"
namespace MyExample
{
    class Program
    {
        static void Main(string[] args)
        {
            Calculator c = new Calculator();
            c.PrintXto1(10);
        }
    }

    class Calculator
    {
        public void PrintXto1(int x)
        {
            if(x == 1)
            {
                Console.WriteLine(x);
            }
            else
            {
                Console.WriteLine(x);
                PrintXto1(x-1);
            }
        }
    }
}
```

```c# title="计算1到100的和"
namespace MyExample
{
    class Program
    {
        static void Main(string[] args)
        {
            Calculator c = new Calculator();
            int result = c.SumFrom1ToX(100);
            Console.WriteLine(result);
        }
    }

    class Calculator
    {
        public int SumForm1ToX(int x)
        {
            int result = 0;
            for(int i = 1; i < x + 1; i++)
            {
                result = result + i;
            }
            return result;
        }

        public int SumFrom1ToX(int x)
        {
            if(x==1)
            {
                return 1;
            }
            else
            {
                int result = x + SumFrom1ToX(x-1);
                return result;
            }
        }

        public int SumFrom1ToX(int x)
        {
            return(1 + x) * x / 2;
        }
    }
}
```

![1.avif](../assets/Csharp/1.avif)

## 类型

### 本质-值的集合和值的操作

- 什么是类型（Type）
- 类型在 C 井语言中的作用
- C#语言的类型系统变量
- 对象与内存

**什么是类型（Type）**

- 又名数据类型（Data Type）

  A data type is a homogeneous collection of values, effectively presented, egquipped with a set of operations which manipulate these values

  是数据在内存中存储时的“型号”

  小内存容纳大尺寸数据会丢失精确度、发生错误

  大内存容纳小尺寸数据会导致浪费

  编程语言的数据类型与数学中的数据类型不完全相同

- 强类型语言与弱类型语言的比较

  C 语言示例：if 条件

  JavaScript 示例：动态类型

  C#语言对弱类型/动态类型的模仿

### 强类型和弱类型语言

`dynamic`和 javascript 中的`var`比较类似，主要用在底层数据打交道

### 作用-D01-六种作用

- 反射

  我在程序运行的过程中，拿到一个对象拿到一个类型，我可以立刻知道它里面有哪些成员，根据需要操作这些成员

```c#
namespace TypeSample
{
    class Program
    {
        static void Main(string[] args)
        {
            Type myType = typeof(Form);
            PropertyInfo[]pInfos = myType.GetProperties();
            PropertyInfo[]mInfos = myType.GetMethods();
            foreach(var m in mInfos)
            {
                Console.WriteLine(m.Name);
            }
        }
    }
}
```

**类型在 C 语言中的作用**

```yaml title="一个C#类型中所包含的信息有："
存储此类型变量所需的内存空间大小
此类型值可表示的最大，最小值范围
此类型所包含的成员（如方法、属性、事件等）
此类型由何基类派生而来
程序运行的时候，此类型的变量在分配在内存的什么位置
Stack简介
Stackoverflow
Heap简介
使用Performance Monitor查看进程的堆内存使用量
关于内存泄漏
此类型所允许的操作（运算)
```

### 作用- D02-内存分配-栈和堆

```c#
namespace StackOverflow
{
    class Program
    {
        static unsafe void Main(string[] args)
        {
            int* p = stackalloc int[9999999];
        }
    }
}
```

```c#
namespace StackOverflow
{
    class Program
    {
        static void Main(string[] args)
        {
            unsafe
            {
             int* p = stackalloc int[9999999];
            }
        }
    }
}
```

C#严格的对不安全的代码有限制功能，需要再项目设置 Build 过程中 Allow unsafe code

### 类型系统-两大类型和五大类型

![2.avif](../assets/Csharp/2.avif)

C#语言都有哪些数据类型？

C#它的类型系统包括引用类型和值类型两种，引用类型包括类、接口和委托，值类型包括结构体和枚举，所有类型都以 Object 都为自己的基类型

## 变量

### 变量本质和七种变量

![3.avif](../assets/Csharp/3.avif)

```c#
namespace TypeInCSharp
{
    class Program
    {
        static void Main(string[] args)
        {
            Student.Amount //静态成员变量
            Student stu = new Student(); //实例变量
            int[] array = new int[100]; //arry[0]到array[99]为数组元素
            int x; //局部变量
        }
    }

    class Student
    {
        public static int Amount;
        public int Age;
        public string Name;

        public double Add(double a, double b) //double a和double b为值参数变量；在前面加上ref修饰符为引用参数变量，out为输出参数变量
        {
            double result = a + b; //局部变量
            return a + b;
        }
    }
}
```

```c#
namespace TypeInCsharp
{
    class Program
    {
        static void Main(string[] args)
        {
            int a;// 声明变量
            a = 100;
            int b;
            b = 200;
            int c = a + b;
            Console.WriteLine(c);
        }
    }

    class Student
    {
        public static int Amount = 0; //有效的修饰符组合public static;类型int；变量名Amount;初始化器=0
    }
}
```

### 值类型变量和引用类型变量-内存分配

```c#
namespace TypeInCSharp
{
    class Program
    {
        static void Main(string[] args)
        {
            Student stu;
            stu = new Student();
            Student stu2;
            stu2 = stu;
        }
    }

    class Student
    {
        uint ID;
        ushort Score;
    }
}
```

![4.avif](../assets/Csharp/4.avif)

如果是实例变量，引用变量存储的是实例的地址，在堆内存上内存地址

### 变量默认值和常量

```c#
namespace TypeInCSharp
{
    class Program
    {
        static void Main（string[] args)
        {
            // Student stu;
            // Student stu2;
            // stu = new Student();
            // stu2 = stu;

            // int x;
            // const int x = 100；
        }
    }

    class Student
    {
        public uint ID;
        public ushort name;
    }
}
```

`Student stu`是声明在 Main 方法体内的，是一个局部变量，局部变量一定会在栈里面分配内存

成员变量在声明了之后，不给它赋值的话，它是能够获得一个默认值的。一旦这个变量在内存中分配好了之后，就把他的内存块统统刷成 0，这就是它的默认值。

如果生成了一个本地变量，比如`int x`，在 C 或者 C++中会有默认值，但是 C#为了避免这种不安全的代码出现，不给它初始值的话，是不能编译过去的（常量是不能再次赋值的，一旦给了一个值之后，就不能再赋值），初始化器对于常量来说必须同时带着，比如说`int x`必须带着`= 100`

### 装箱与拆箱

```c#
namespace TypeInCSharp
{
    class Program
    {
        static void Main(string[] args)
        {
            int x = 100;
            object obj;
            obj = x;
            int y = (int)obj;
        }
    }
}
```

![5.avif](../assets/Csharp/5.avif)

装箱，指的是把栈上的值类型的值，把它封装成一个`object`类型的实例放在堆上；拆箱就是把堆上的`object`类型的实例里面的值，再按照我的要求拆成我的目标数据类型存储到我的栈上去。装箱和拆箱会损失程序的性能

## 方法

### 历史和本质-算法和指令

- 方法的由来
- 方法的定义与调用
- 构造器（一种特殊的方法）方法的重载（Overload）
- 如何对方法进行 debug
- 方法的调用与栈\*

**方法的由来**

- 方法（method）的前身是 C/C++语言的函数（function）  
   方法是面向对象范寿的概念，在非面向对象语言中仍然称为函数  
   使用 C/C++语言做对比

- 永远者都是类（或结构体）的成员  
  C#语言中函数不可能独立于类（或结构体）之外  
  只有作为类（结构体）的成员时才被称为方法  
  C++中是可以的，称为"全局函数"

- 是类（或结构体）最基本的成员之一  
  最基本的成员只有两个一一字段与方法（成员变量与成员方法），本质还是数据+算法  
  方法表示类（或结构体）“能做什么事情“

- 为什么需要方法和函数  
  目的 1：隐藏复杂的逻辑  
  目的 2：复用（reuse，重用）  
  示例：计算圆面积、圆柱体积、圆锥体积

```C title="Source.c"
#include <stdio.h>

//Function fun
double Add(double a, double b)
{
    return a + b;
}

int main()
{
    double x = 3.0;
    double y = 5.0;
    double result = Add(x, y);
    printf("%f+%f=%f",x,y,result);
    return 0;
}
```

```c++ title="Source.cpp"
#include <iostream>

double Add(double a, double b)
{
    return a + b;
}

int main()
{
    double x = 3.0;
    double y = 5.0;
    double result = Add(x, y);
    std::count<< x << "+" << y << "=" << result;
    return 0;
}
```

`Student.h`文件是类的声明，`Student.cpp`是对类的定义，c#对类的声明和定义是放在一起的，而 c++是分开的

```c++ title="Student.h"
#pragma once
class Student
{
public:
    Student();
    ~Student();
    void SayHello();
    double Add(double a, double b);
}
```

```c++ title="Student.cpp"
#include "Student.h"
#include <iostream>

Student::Student()
{
}

Student::~Student()
{
}

void Student::SayHello()
{
    std::count<<"Hello! i'm a student!";
}

double Student::Add(double a, double b)
{
    return a + b;
}
```

```c++ title="Souce.cpp"
#include <iostream>
#include "Student.h"

double Add(double a, double b)
{
    return a + b;
}

int main()
{
    Student *pStu = new Student();
    double x = 3.0;
    double y = 5.0;
    double result = pStu->Add(x, y); //当一个函数以类的成员的身份出现的时候他就不叫函数了，而叫做方法
    std::cout<<x<<"+"<<y<<"="<<result;
    // pStu->SayHello();
    return 0;
}
```

```c#
namespace CSharpFun
{
    class Program
    {
        //        double Add(double a, double b)
        //{
        //    return a + b;
        //}
        static void Main(string[] args)
        {
        //        double Add(double a, double b)
        //{
        //    return a + b;
        //}
        }
        double Add(double a, double b)
        {
            return a + b;
        }
    }
}
```

函数在成为类的成员之后就叫做方法，方法永远是类和结构体的成员

### 作用-封装复用和算法分解

```c#
namespace CSharpMethodExample
{
    class Program
    {
        static void Main(string[] args)
        {
            Calculator c = new Calculator();

        }
    }

    class Calculator
    {
        // public double GetCircleArea(double r)
        // {
        //     return 3.14 * r * r;
        // }

        // public double GetCylinderVolume(double r, double h)
        // {
        //     return 3.14 * r * r * h;
        // }

        // public double GetConeVolume(double r,double h)
        // {
        //     return 3.14 * r * r * h / 3;
        // }

        public double GetCircleArea(double r)
        {
            return Math.PI * r * r;
        }

        public double GetCylinderVolume(double r, double h)
        {
            return GetCircleArea(r) * h;
        }

        public double GetConeVolume(double r, double h)
        {
            return GetCylinderVolume(r, h) / 3;
        }
    }
}
```

自顶向下，逐步求精，标准的面向过程的算法

### 声明-形参变量和访问级别

![6.avif](../assets/Csharp/6.avif)

![7.avif](../assets/Csharp/7.avif)

### 调用-形参和实参

### 构造器

- 构造器（constructor）是类型的成员之一
- 狭义的构造器指的是“实例构造器”（instance constructor）
- 如何调用构造器
- 声明构造器
- 构造器的内存原理

```c#
namespace ConstructorExample
{
    class Program
    {
        static void Main(string[] args)
        {
            Student stu = new Student(2, "Mr.Okay");
            Console.WriteLine(stu.ID);
            Console.WriteLine(stu.Name);
            Console.WriteLine("===============");
            Student stu2 = new Student();
        }
    }

    class Student
    {
        public Student(int initId, string initName)
        {
            this.ID = initId;
            this.Name = initName;
        }

        public Student()
        {
            this.ID = 1;
            this.Name = "No name";
        }

        public int ID;
        public string Name;
    }
}
```

ctor code snippet 快速生成构造器

```c#
namespace constructorExample
{
    class Program
    {
        static void Main(string[] args)
        {
            Student stu = new Student();
        }
    }

    class Student
    {
        public int ID;
        public string Name;
    }
}
```

![8.avif](../assets/Csharp/8.avif)

```c#
namespace constructorExample
{
    class Program
    {
        static void Main(string[] args)
        {
            Student stu = new Student(1，"Mr.Okay");
            Console.WriteLine(stu.Name == null);
        }
    }

    class Student
    {
        public int ID;
        public string Name;
    }
}
```

![9.avif](../assets/Csharp/9.avif)

### 重载-方法签名

```c#
namespace OverloadExample
{
    class Program
    {
        static void Main(string[] args)
        {
            Calculator c = new Calculator();
        }
    }

    class Calculator
    {
        public int Add(int a,int b)
        {
            return a + b;
        }

        public int Add(int a, int b, int c)
        {
            return a + b + c;
        }

        public double Add(double a, double b)
        {
            return a + b;
        }
    }
}
```

### 调试-StepIn-StepOut-StepOver-CallStack

StepInto 进入正在被调用的函数里面看看具体怎么回事

StepOver 不走进这个方法

StepOut 不走进这个方法，直接跳出到调用自身方法的方法中

### 调用栈

stack frame 表示方法被调用后在栈内存中的布局

```c#
namespace CSharpMethodExample
{
    class Program
    {
        static void Main(string[] args)
        {
            double result = Calculator.GetConeVolume(100, 100);
        }
    }

    class Calculator
    {
        public double GetCircleArea(double r)
        {
            return Math.PI * r * r;
        }

        public double GetCylinderVolume(double r, double h)
        {
            double a = GetCircleArea(r);
            return a * h;
        }

        public double GetConeVolume(double r, double h)
        {
            return GetCylinderVolume(r, h);
            return cv / 3;
        }
    }
}
```

![10.avif](../assets/Csharp/10.avif)

**函数的返回值是存在寄存器里面的，特殊情况寄存器存不下这个值的时候会在栈上开辟空间**

## 操作符

### 本质-函数的简记法

- 操作符概览
- 操作符的本质
- 操作符的优先级
- 司级操作符的运算顺序
- 各类操作符的示例

![11.avif](../assets/Csharp/11.avif)

**操作符的本质是函数（即算法）的"简记法**

- 假如没有发明"+"、只有 Add 函数，算式 3+4+5 将可以写成 Add(Add(3,4),5)
- 假如没有发明"×"、只有 Mul 函数，那么算式 3+4×5 将只能写成 Add(3,Mu(4,5))，注意优先级

**操作符不能脱离与它关联的数据类型**

- 可以说操作符就是与固定数据类型相关联的一套基本算法的简记法
- 示例：为自定义数据类型创建操作符

```c#
namespace CreateOperator
{
    static void Main(string[] args)
    {
        Person person1 = new Person();
        Person person2 = new Person();
        person1.Name = "Deer";
        person2.Name = "Deer's wife";
        List<Person>nation = person1 + person2;// Person.GetMarry(person1, person2);
        foreach(var p in nation)
        {
            Console.WriteLine(p.Name);
        }
    }

    class Person
    {
        public string Name;

        public static List<Person> operator +(Person p1, Person p2)// GetMarry(Person p1, Person p2)
        {
            List<Person>people = new List<Person>();
            people.Add(p1);
            people.Add(p2);
            for(int i = 0; i < 11; i++)
            {
                Person child = new Person();
                child.Name = p1.Name + "&" + p2.Name + "s child";
                people.Add(child);
            }

            return people;
        }
    }
}
```

这里的代码实现了一个 `+` 运算符的重载，在 `Person` 类中。重载 `+` 运算符后，`person1 + person2` 并不是直接调用 `GetMarry(person1, person2)` 方法，而是执行了你在 `operator +` 里面定义的行为。

### 优先级\

```c#
namespace OperatorPriority
{
    class Program
    {
        static void Main(string[] args)
        {
            int x = 100;
            int y = 200;
            int z = 300;
            x += y += z;
            Console.WriteLine(x); // 600
            Console.WriteLine(y); // 500
            Console.WriteLine(z); // 300
        }
    }
}
```

操作符的优先级

- 可以使用圆括号提高被括起来表达式的优先级
- 圆括号可以嵌套
- 不像数学里有方括号和花括号，在 C#语言里”[]“与“{}“有专门的用途

同优先级操作符的运算顺序

- 除了带有赋值功能的操作符，同优先级操作符都是由左向右进行运算
- 带有赋值功能的操作符的运算顺序是由右向左
- 与数学运算不同，计算机语言的同优先级运算没有“结合率
  - 3+4+5 只能理解为 Add(Add(3,4),5)不能理解为 Add(3,Add(4,5)

### 基本运算操作符

`x.y`**成员访问操作符**

可以用来访问外层空间的子集空间，其次它可以访问名称空间当中的类型，它还可以用来访问类型的静态成员，最后它还可以访问对象的成员

```c#
namespace OperatorsExample
{
    class Program
    {
        static void Main(string[] args)
        {
            System.IO.File.Create("D:\\HelloWorld.txt");

            Form myForm = new Form();
            myForm.Text = "Hello, World!"; // 访问实例成员（属性）
            myForm.ShowDialog(); // 访问方法
        }
    }
}
```

`System`是一个命名空间，`IO`是它的一个子集命名空间，`File`是一个类类型，`Create`是一个静态成员`

`f(x)`**方法调用操作符**

```c#
namespace OperatorsExample
{
    class Program
    {
        static void Main(string[] args)
        {
            Calculator c = new Calculator();
            double x = c.Add(3.0, 5.0);

            c.PrintHello();

            Action myAction = new Action(c.PrintHello); //访问c.PrintHello这个成员，把这个成员的名字交给myAciton
            myAction();
        }
    }

    class Calculator
    {
        public double Add(double a, double b)
        {
            return a + b;
        }

        public void PrintHello()
        {
           Console.WriteLine("Hello")
        }
    }
}
```

委托，不用直接去调用一个方法，而通过委托间接的调用一个方法

`a[x]`**元素访问操作符 常见的访问元素的操作，就是访问数组的元素和字典的元素**

```c#
namespace OpratorsExample
{
    class Program
    {
        static void Main(string[] args)
        {
            // int[] myIntArray = new int[10];

            Dicitionary<string, Student> stuDic = new Dictionary<string, Student>();
            for(int i = 1; i <= 100; i++)
            {
                Student stu = new Student();
                stu.Name = "s_" + i.ToString();
                stu.Score = 100 + i;
                stuDic.Add(stu.Name, stu);
            }

            Student number6 = stuDic["s_6"];
            Console.WriteLine(number6.Score);
        }
    }
}
```

元素访问操作符的方括号里面是集合的索引，这个索引不一定都是整数

`x++ x--`**后置的自增和后置的自减**

```c#
namespace OpreatorsExample
{
    class Program
    {
        static void Main(string[] args)
        {
            int x = 100;
            int y = x++;
            // int y = x;
            // x = x + 1;
            int y = x--;
            // int y = x;
            // x = x - 1;
            int y = x;
            x = x - 1;
        }
    }
}
```

`typeof`**操作符**

```c#
namespace OperatorsExample
{
    class Program
    {
        static void Main(string[] args)
        {
            Type t = typefo(int);
            Console.WriteLine(t.Namespace); //System
            Console.WriteLine(t.FullName); //System.Int32
            Console.Writeline(t.Name); //Int32
            int c = t.GetMethods().Length;
            foreach(var mi in t.GetMethods())
            {
                Console.WriteLine(mi.Name);
            }
            Console.WriteLine(c);
        }
    }
}
```

`Default`操作符，它的功能是帮助我们去获取一个类型它的默认值，下面着重举例了三个类型，结构体类型、引用类型和枚举类型（结构体类型和枚举类型都是值类型，而引用类型是跟值相对的的另外一种类型）

```c#
namespace OperatorsExample
{
    class Program
    {
        static void Main(string[] args)
        {
            int x = default(int);
            Console.WriteLine(x); // 0
            double x = default(double);
            Console.WriteLine(x); // 0
            Form myForm = default(From);
            Console.WriteLine(myFrom==null);// true
            Level level = default(Level);
            Console.WriteLine(level); // Mid,赋了值就是Low，没有0返回0
        }
    }

    enum Level
    {
        // Mid = 2,
        // Low = 1,
        // High = 3

        // Mid = 0,
        // Low = 1,
        // High = 2

        Mid,
        Low,
        High
    }
}
```

`new`操作符

```C#
namespace OperatorsExample
{
    class Program
    {
        static void Main(string[] args)
        {
            // Form myForm = new Form();
            // myForm.Text = "Hello";

            // Form myForm = new Form(){Text = "Hello", FormBorderStyle = FormBorderStyle.SizableToolWindow};
            // myForm.ShowDialog();

            // new Form(){Text = "Hello"}.ShowDialog();

            // int x = 100;
            // string name = "Tim";
            // string name = new string();

            //int[] myArray = new int[10];
            //int[] myArray = {1,2,3,4};

            Form myForm = new Form(){Text = "Hello"};
            var person = new {Name = "Mr.Okay", Age = 34}; //是为匿名类型创建对象并且用隐式类型变量来引用这个实例
            Console.WriteLine(person.Name); // Mr.Okay
            Console.WriteLine(person.Age); // 34
            Console.WriteLine(person.GetType().Name);  //<>f_AnonymousType0`2
        }
    }
}
```

`new`操作符的主要功能是帮助我们在内存当中创建类型的实例，立刻调用它的实例构造器，如果在`new`操作符的左边有赋值符号的话，`new`操作符会把自己拿到的实例的内存地址通过赋值操作符交给负责访问这个实例的变量；`new`操作符除了能调用这个实例的构造器之外，还能去调用这个实例的初始化器

只要我想在内存当中创建类型的实例，就要去调用`new`操作符？并不是这样比如说`string`，它是一个类类型，类类型在创建实例的时候要使用`new`操作符，平时使用比如`string name = "Tim";`就不用调用`new`操作符，为什么呢，这个叫做 C#语言的语法糖衣，string 类型是一个非常基本的类型，微软为了统一基本类型使用时候的基本体验，它把`new`操作符隐藏起来了；`int[] myArray = {1,2,3,4};`也没用到`new`操作符，这也是一个语法糖衣，在创建某些类型的实例的时候我们可以选择使用`new`操作符或者不使用`new`操作符

<u><>f_AnonymousType02</u> ，<>f_AnonymousType 是约定的前缀，0 指的是我在程序当中创建的第一个，`2就是这个类型呢，是一个泛型类，构成这个类型的时候呢，需要两个类型来构成他，一个是`string 类型一个是 int 类型

**`new`操作符类型强大，不能够乱使，那么在写大型程序的时候，为了避免`new`操作符造成的紧耦合我们有一种依赖注入的设计模式**

```c#
namespace OperatorsExample
{
    class Program
    {
        static void Main(string[] args)
        {
            Student student = new Student();
            stu.Report(); // I'm a student
            CsStudent csStu = new CsStudent();
            csStu.Report(); // I'm CS student
        }
    }

    class Student
    {
        public void Report()
        {
            Console.WriteLine("Im a student.");
        }
    }

    class CsStudent: Student
    {
        new public void Report() //子类对父类的方法进行隐藏
        {
            Console.WriteLine("I'm CS student");
        }
    }
}
```

`checked`操作符

```c#
namespace OperatorsExample
{
    class Program
    {
        static void Main(string[] args)
        {
            uint x = uint.MaxValue;
            Console.WriteLine(x);
            string binStr = Convert.ToString(x, 2);
            Console.WriteLine(binStr);
            //try
            //{
            //    uint y = checked(x + 1);
            //    // uint y = unchecked(x + 1); //会打印出来0，不再检验溢出
            //    Console.WriteLine(y);
            //}
            //catch(OverflowException ex)
            //{
            //    Console.WriteLine("There's overflow!");
            //}

            checked //unchecked
            {
                try
                {
                    uint y = x + 1;
                    Console.WriteLine(y);
                }
                catch(OverflowException ex)
                {
                    Console.WriteLine("There's overflow!");
                }
            }
        }
    }
}
```

`delegate`操作符

```c#
namespace Example
{
    /// <summary>
    /// Interaction logic for MainWindow.xaml
    /// </summary>
    public partial class MainWindow : Window
    {
        public MainWindow()
        {
            InitializeComponent();
            // this.myButton.Click += myButton_Click;
            // this.myButton.Click += delegate(object sender, RoutedEventArgs e)
            // {
            //    this.myTextBox.Text = "Hello, World!"
            // }

            this.myButton.Click += (sender, e) =>
            {
                this.myTextBox.Text = "Hello, World!";
            }
        }

        //void myButton_Click(object sender,RoutedEventArgs e)
        //{
        //    this.myTextBox.Text = "Hello, World!";
        //}
    }
}
```

`sizeof`操作符

```c#
namespace OperatorsExample
{
    class Program
    {
        static void Main(string[] args)
        {
            unsafe
            {
                int x = sizeof(Student);
                Console.WriteLine(x); // 16
            }
        }
    }

    struct Student
    {
        int ID;
        long Score;
    }
}
```

`->`操作符

```c#
namespace OperatorsExample
{
    class Program
    {
        static void Main(string[] args)
        {
            unsafe
            {
            Student stu;
            stu.ID = 1;
            stu.Score = 99;
            Student* pStu = &stu;
            pStu->Score = 100;
            Console.WriteLine(Stu.Score); //100
            }
        }
    }

    struct Student
    {
        public int ID;
        public long Score;
    }
}
```

在 c#中有严格的规定，像指针操作，取地址操作，以及用指针访问去访问成员的操作，只能用来操作结构体类型，不能用来操作引用类型

### 壹元操作符

`&`取地址操作符,`*`取引用符号

```c#
namespace OperatorsExample
{
    class Program
    {
        static void Main(string[] args)
        {
            unsafe
            {
                Student stu;
                stu.ID = 1;
                stu.Score = 99;
                Student* pStu = &stu;
                pStu->Score = 100;
                (*pStu).Score = 1000;
                Console.WriteLine(stu.Score);
            }
        }
    }
}
```

`+ - ! ~`正 负 非 反操作符

```c#
namespace OperatorsExample
{
    class Program
    {
        static void Main(string[] args)
        {
            int x = 100;
            int y = -x; // -100
            int y = -(-x); // 100

            int x = int.MinValue; //
            int y = checked(-x); //System.OverflowException 栈溢出
        }
    }
}
```

```c#
namespace OperatorsExample
{
    class Program
    {
        static void Main(string[] args)
        {
            int x = 12345678;
            int y = ~x;
            string xStr = Covert.Tostring(x, 2).PadLeft(32, '0');// 00000000101111000110000101001110
            string yStr = Covert.Tostring(y, 2).PadLeft(32, '0');// 11111111010000111001111010110001

            int x = int.MinValue;
            int y = -x; // -2147483648
            string xStr = Convert.ToString(x, 2).PadLeft(32, '0'); // 10000000000000000000000000000000
        }
    }
}
```

```c#
namespace OperatorsExample
{
    class Program
    {
        static void Main(string[] args)
        {
            int x = 100;
            int y = ++x;
            // x = x + 1;
            // int y = x;
            int y = --x;
            // x = x - 1;
            // int y = x;
        }
    }
}
```

**类型转换**

- 隐式(implicit)类型转换
  - 不丢失精度的转换
  - 子类向父类的转换
  - 装箱（性能上的损失，不会在精度上进行损失）
- 显式(explicit)类型转换
  - 有可能丢失精度（甚至发生错误）的转换，即 cast（铸造）
  - 拆箱
  - 使用 Convert 类
  - ToString 方法与各数据类型的 Parse/TryParse 方法
- 自定义类型转换操作符
  - 示例

```c#
namespace ConversionExample
{
    class Program
    {
        static void Main(string[] args)
        {
            string str1 = Console.ReadLine();
            string str2 = Console.ReadLine();
            Console.WriteLine(str1 + str2); //字符串拼接
            int x = Convert.ToInt32(str1);
            int y = Convert.ToInt32(str2);
            Console.WriteLine(x + y);
        }
    }
}
```

![12.avif](../assets/Csharp/12.avif)

[_csharp-language-specification-5.0_](../assets/Csharp/csharp-language-specification-5.0.pdf)

```c#
namespace ConversionExample
{
    class Program
    {
        static void Main(string[] args)
        {
            Teacher t = new Teacher();
            Human h = t;
            Animal a = h;
            h.; //C#语言规定，当你试图拿一个引用变量去访问它所引用着的实例成员的时候，你这个时候只能访问到这个变量的类型，它所具有的成员，是你这个变量的类型，而不是你这个变量所引用的实例的类型;h只能访问Eat()和Think()
            a.;//只能访问到Eat();
        }
    }

    class Animal
    {
        public void Eat()
        {
            Console.WriteLine("Eating...");
        }
    }

    class Human : Animal
    {
        public void Think()
        {
            Conosle.WriteLine("Who i am?");
        }
    }

    class Teacher : Human
    {
        public void Teach()
        {
            Console.WriteLine("I teach programming.");
        }
    }
}
```

`(T)x`显示类型操作符(cast)

```c#
namespace ConversionExample
{
    class Program
    {
        static void Main(string[] args)
        {
            Console.WriteLine(ushort.MaxValue); // 65535
            uint x = 65536;
            ushort y = (ushort)x;
            Console.WriteLine(y);  // 0
        }
    }
}
```

```c#
namespace ConversionExample
{
    class Program
    {
        static void Main()
        {
            int negativeValue = -1;
            uint unsignedValue = (uint)negativeValue;

            Console.WriteLine($"Original int (negative): {negativeValue}"); // Original int (negative): -1
            Console.WriteLine($"Converted uint: {unsignedValue}"); // Converted uint: 4294967295
        }
    }
}
```

- **负数在 `int` 中的存储**：`-1` 在 `int` 类型中，存储为补码，二进制表示为 `11111111 11111111 11111111 11111111`（32 位）。其中，最高位 `1` 表示负号。
- **转换为 `uint` 类型时**：当我们将 `-1` 转换为 `uint` 类型时，`-1` 的二进制表示中的所有 `1` 被直接当作数值使用，而不考虑符号。因此，`-1`（`11111111 11111111 11111111 11111111`）被视作一个无符号整数，其对应的十进制值为 `4294967295`。

一个有符号类型的负数它的负数向一个无符号类型的值去转化的时候，肯定会发生值上的变化，有符号数据类型它的最高位那个 1 是来表示负数的，你现在把它转化成一个无符号的数之后，一个无符号的数就直接拿最高位的 1 当作真实的数值了

`Convert`类

```c#
namespace Convert
{
    /// <summary>
    /// Interaction logic for MainWindow.xaml
    /// </summary>
    public partial class MainWindow : Window
    {
        public MainWindow()
        {
            InitializeComponent();
        }

        private void btn_Click(object Sender, RoutedEventArgs e)
        {
            double x = System.Convert.ToDouble(tb1.Text);
            double y = System.Convert.ToDouble(tb2.Text);
            double result = x + y;
            this.tb3.Text = result.ToString();
            object o;
            o. ; //object类型带有Equals、GethashCode、GetType、ToString四个方法
        }
    }
}
```

**c#语言它的类型体系是以 object 为根的，那么任何一种数据类型都是由 object 类型直接或间接派生出来的**

```c#
namespace Convert
{
    /// <summary>
    /// Interaction logic for MainWindow.xaml
    /// </summary>
    public partial class MainWindow : Window
    {
        public MainWindow()
        {
            InitializeComponent();
        }

        private void btn_Click(object Sender, RoutedEventArgs e)
        {
            double x = double.Parse(this.tb1.Text);
            double y = double.Parse(this.tb2.Text);
            double result = x + y;
            this.tb3.Text = result.ToString(); // x为12，y为12警告System.FormatException；x为1e2，y为12输出112
        }
    }
}
```

自定义类型转换操作符

```c#
namespace ConversionExample
{
    class Program
    {
        static void Main(string[] args)
        {
            Stone stone = new Stone();
            stone
            Monkey m = (Monkey)Stone;
            Console.WriteLine(wukongSun.Age);
        }
    }

    class stone
    {
        public int Age;

        public static explicit operator Monkey(Stone stone)
        {
            Monkey m = new Monkey();
            m.Age = stone.Age / 500;
            return m;
        }
    }

    class Monkey
    {
        public int Age;
    }
}
```

`explicit`显示类型转换，`implicit`隐式类型转换

### 算术运算操作符

`*`乘法运算符

```c#
namespace ConversionExample
{
    class Program
    {
        static void Main(string[] args)
        {
            var x = 3 * 4; //System.Int32
            var x = 3.0 * 4.0;//System.Double
            var x = 3.0 * 4;//System.Double 数据类型提升（为了不产生精度丢失）

            int x = 100;
            double y = x; //int类型向double类型转换的时候是隐式转换，不会丢失精度；double类型的精度要比int类型是要高的
        }
    }
}
```

```c#
namespace ConversionExample
{
    class Program
    {
        static void Main(string[] args)
        {
            int x = 5;
            int y = 4;
            int z = x / y; //第一个特点，因为是整除，小数点后面的数据都省去了；第二个特点，被除数不能是0

            double x = 5;
            double y = 4;
            double z = x / y; //1.25

            double x = 5.0;
            double y = 0;
            double z = x / y; //Infinity;如果是-5.0就是-Infinity
            double a = double.PostitiveInfinity;
            double b = double.NegativeInfinity;
            double c = a / b; //NaN

            double x = (double)5 / 4; //1.25（数据类型提升）
            double x = (double)(5 / 4); //1（1.0）
        }
    }
}
```

`%`余数操作符

```c#
namespace ConversionExample
{
    class Program
    {
        static void Main(string[] args)
        {
            for(int i = 0; i < 100 ; i++)
            {
                Console.WriteLine(i % 10);//一直取的是最后一位
            }

            double x = 3.5;
            double y = 3;
            Console.WriteLine(x % y); // 0.5(取余也会涉及到数据类型提升)
        }
    }
}
```

`+`加法运算符

```c#
namespace ConversionExample
{
    class Program
    {
        static void Main(string[] args)
        {
            var x = 3.0 + 4;
            Console.WriteLine(x.GetType().Name); //Double
            Console.WriteLine(x); //7(7.0)

            string s1 = "123";
            string s2 = "abc";
            string s3 = s1 + s2;
            Console.WriteLine(s3); //123abc
        }
    }
}
```

`-`减法运算符

+∞ - +∞ = NaN

### 位移运算操作符

```c#
namespace ConversionExample
{
    class Program
    {
        static void Main(string[] args)
        {
            int x = 7;
            int y = x >> 2;
            string strX = Convert.ToString(x, 2).PadLeft(32, '0'); // 00000000000000000000000000000111
            string strY = Convert.Tostring(y, 2).PadLeft(32, '0'); // 00000000000000000000000000000001
            Console.WriteLine(strX);
            Console.WriteLine(strY);
            Console.WriteLine(y); // 1，溢出了，但是是在unchecked上下文当中，所以不会产生溢出，在checked上下文当中会产生溢出
        }
    }
}
```

当在没有溢出的情况下，左移就是乘二，右移就是除二；左移和右移补位的规定，左移不论正数还是负数补进来的都是 0，右移的时候如果你现在操作的是一个正数的话最高补进来的是 0，如果你操作的是一个负数的话，那么最高位补进来的是 1

### 关系运算操作符

所有关系操作符运算结果一定都是布尔类型的，要么都是真，要么都是假

```c#
namespace ConversionExample
{
    class Program
    {
        static void Main(string[] args)
        {
            int x = 5;
            double y = 4.0;
            var result = x > y;
            Console.WriteLine(result.GetType().FullName); //System.Double 布尔类型
            Console.WriteLine(result); //True

            int x = 5;
            double y = 4.0;
            var result = x < y;
            Console.WriteLine(result.GetType().FullName); //System.Double 布尔类型
            Console.WriteLine(result); //False

            int x = 5;
            double y = 4.0;
            var result = x <= y;
            Console.WriteLine(result.GetType().FullName); //System.Double 布尔类型
            Console.WriteLine(result); //False

            int x = 5;
            double y = 4.0;
            var result = x == y;
            Console.WriteLine(result.GetType().FullName); //System.Double 布尔类型
            Console.WriteLine(result); //False

            int x = 5;
            double y = 4.0;
            var result = x!= y;
            Console.WriteLine(result.GetType().FullName); //System.Double 布尔类型
            Console.WriteLine(result); //True

            char char1 = 'a';
            char char2 = 'A';
            var result = char1 > char2;
            Console.WriteLine(result); //True

            char char1 = 'a';
            char char2 = 'A';
            ushort u1 = (ushort)char1;
            ushort u2 = (ushort)char2;
            Console.WriteLine(u1); //97
            Console.WriteLine(u2); //65

            string str1 = "abc";
            string str2 = "Abc";
            Console.WriteLine(str1==str2); //false，str1改为Abc就为True

            string str1 = "abc";
            string str2 = "Abc";
            Console.WriteLine(str1.ToLower() == str2.ToLower()); //第一个字母小写，True
            string.Compare();
        }
    }
}
```

### 类型检测操作符

`is as`类型检验操作符

```c#
namespace ConversionExample
{
    class Program
    {
        static void Main(string[] args)
        {
            Teacher t = new Teacher();
            var result = t is Teacher;
            Console.WriteLine(result.GetType().FullName); //System.Boolean
            Console.WriteLine(result); //True,t = null为False

            var resslt = t is Human; // True

            var result = t is Animal; // True

            Car car = new Car();
            Console.WriteLine(car is Animal);// False
            Console.WrtieLine(car is object);// True

            var result = h is Teacher;

            Humman h = new Humman();
            var result = h is Teacher;
            Console.WriteLine(result); // False

            object o = new Teacher();
            if (o is Teacher)
            {
                Teacher t = (Teacher)o;
                t.Teach();
            }

            object o = new Teacher();
            Teacher t = o as Teacher; //如果这个o像Teacher一样那么就把这个对象的地址交给t这个变量，否则的话就把一个null值交给t
            if(t! = null)
            {
                t.Teach();
            }
        }
    }

    class Animal
    {
        public void Eat()
        {
            Console.WriteLine("Eating...");
        }
    }

    class Human : Animal
    {
        public void Think()
        {
            Console.WriteLine("Who I am?");
        }
    }

    class Teacher : Human
    {
        public void Teache()
        {
            Console.WriteLine("I teach programming.")
        }
    }

    class Car : Object //不论有没有显式的写出来Object，C#编译器都会让我们的类从Object派生出来
    {
        public void Run()
        {
            Console.WriteLine("Running...");
        }
    }
}

```

### 逻辑运算操作符

`& ^ |`逻辑”与“、逻辑异或和逻辑或

```c#
namespace ConversionExample
{
    class Program
    {
        static void Main(string[] args)
        {
            int x = 7;
            int y = 28;
            int z = x & y;
            string strX = Convert.Tostring(x, 2).PadLeft(32, '0'); // 00000000000000000000000000000111
            string strY = Convert.Tostring(y, 2).PadLeft(32, '0'); // 00000000000000000000000000011100
            string strZ = Convert.Tostring(z, 2).PadLeft(32, '0'); // 00000000000000000000000000000100
            Console.WriteLine(strX);
            Console.WriteLine(strY);
            Console.WriteLine(strZ);
        }
    }
}
```

```c#
namespace ConversionExample
{
    class Program
    {
        static void Main(string[] args)
        {
            int x = 7;
            int y = 28;
            int z = x & y;
            string strX = Convert.Tostring(x, 2).PadLeft(32, '0'); // 00000000000000000000000000000111
            string strY = Convert.Tostring(y, 2).PadLeft(32, '0'); // 00000000000000000000000000011100
            string strZ = Convert.Tostring(z, 2).PadLeft(32, '0'); // 00000000000000000000000000011111
            Console.WriteLine(strX);
            Console.WriteLine(strY);
            Console.WriteLine(strZ);
        }
    }
}
```

```c#
namespace ConversionExample
{
    class Program
    {
        static void Main(string[] args)
        {
            int x = 7;
            int y = 28;
            int z = x & y;
            string strX = Convert.Tostring(x, 2).PadLeft(32, '0'); // 00000000000000000000000000000111
            string strY = Convert.Tostring(y, 2).PadLeft(32, '0'); // 00000000000000000000000000011100
            string strZ = Convert.Tostring(z, 2).PadLeft(32, '0'); // 00000000000000000000000000011011
            Console.WriteLine(strX);
            Console.WriteLine(strY);
            Console.WriteLine(strZ);
        }
    }
}
```

### 条件运算操作符

`&& ||`条件运算操作符

```c#
namespace CoversionExample
{
    class Program
    {
        static void Main(string[] args)
        {
            int x = 3;
            int y = 4;
            int a = 3;
            if(x>y && a++>3)
            {
                Console.WriteLine("Hello");
            }
            Console.WriteLine(a); // 不打印Hello，a打印出3

            int x = 5;
            int y = 4;
            int a = 3;
            if(x>y && a++>3)
            {
                Console.WriteLine("Hello");
            }
            Console.WriteLine(a); // 不打印Hello，a打印出4

            int x = 5;
            int y = 4;
            int a = 3;
            if(x > y || a++ > 3)
            {
                Console.WriteLine("Hello");
            }
            Console.WriteLine(a); // 打印Hello，a不执行，a打印3

            int x = 3;
            int y = 4;
            int a = 3;
            if(x > y || ++a > 3) //避开这种短路效应
            {
                Console.WriteLine("Hello");
            }
            Console.WriteLine(a); // 打印Hello，a打印4
        }
    }
}
```

### null 值合并操作符

`??`null 值合并操作符

```c#
namespace ConversionExample
{
    class Program
    {
        static void Main(string[] args)
        {
            Nullable<int> x = null;
            x = 100;
            Console.WriteLine(x); //100
            Console.WriteLine(x.HasValue);//ture

            Nullable<int> x = null;
            Console.WriteLine(x.HasValue);//False

            int? x = null;
            x = 100;
            Console.WriteLine(x); //100
            Console.WriteLine(x.value);//100

            int? x = null; // Nullable<int>的简写，可空整型
            int y = x ?? 1; //x如果是null值就赋1
            Console.WriteLine(y);
        }
    }
}
```

### 条件三元操作符

`?:`条件操作符

```c#
namespace ConversionExample
{
    class Program
    {
        static void Main(string[] args)
        {
            int x = 80;
            string str = strting.Empty;
            if(x >= 60)
            {
                str = "Pass";
            }else
            {
                str = "Failed";
            }
            Console.WriteLine(str); //pass

            // string str = (x>= 60) ? "Pass" : "Failed"; // if else简写
        }
    }
}
```

### 赋值运算操作符

`= *= /= %= += -= <<= >>= &= ^= |= =>`复制运算操作符

```c#
namespace ConversionExample
{
    class Program
    {
        static void Main(string[] args)
        {
            int x = 5;
            x = x + 1; // x += 1;

            int x = 7;
            x<<=2; // x = x<<2;

            int x = 5;
            int y = 6;
            int z = 7;
            int a = x + y * = z; // 47
        }
    }
}
```

## 表达式

### 定义

- 表达式的定义

- 各类表达式概览

- 语句的定义

- 语句详解

- 什么是表达式

  - Expressions, together with commands and declarations, are one of the basic components of every programming language. We can say that expressions are the essential component of every language.
  - An expressions is a syntactic entity whose evaluation either produces a value or fails to terminate,in which case the expression is undefined
  - 各种编程语言对表达式的实现不尽相同，但大体上都符合这人定义

- C#语言对表达式的定义
  - An expression is a seguence of one or more operands and zero or more operators that can be evaluated to a single value, object, method, or namespace.Expressions can consist of a literal value, a method invocation, an operator and its operands, or a simple name. Simple names can be the name of a variable, type member, method parameter, namespace or type.

  - 算法逻辑的最基本（最小）单元，表达一定的算法意图

  - 因为有操作符有优先级，所以表达式也就有了优先级

```c#
  namespace ExperssionExample
  {
      class Program
      {
          static void Main(string[] args)
          {
              int x;
              x = 100;
              x++;
              ++x; //single value

              (new Form()).ShowDialog(); //object

               Action myAction = new Action(Console.WriteLine); //method 成员访问表达式，操作数有两个，左边是类型，右边是类型的成员。这个表达式最后得到的是一个方法，方法本身也是一个语法实体，我们并没有调用它，只是得到它
              System.Windows.Forms.Form myForm = new Form();

              string name;
              name = "Mr.Okay"; //字面值

              double x = Math.Pow(2, 3);//函数调用构成表达式

              int X = 2 + 3;//操作数和操作符构成的表达式

              int x = 100;
              int y;
              y = x; //simple name

              Type myType = typeof(Int64);//class name(simple name)
              Console.WriteLine(myType.FullName) //System.Int64
          }
      }
  }
```

`Action myAction = new Action(Console.WriteLine); `method 成员访问表达式，操作数有两个，左边是类型，右边是类型的成员。这个表达式最后得到的是一个方法，方法本身也是一个语法实体，我们并没有调用它，只是得到它

`System.Windows.Forms.Form myForm = new Form(); `中`System`是主名称空间，`Windows`是子名称空间，`System.Windows`经过求值之后得到的是名称空间

### 分类

- C#语言中表达式的分类
  - A value.Every value has an associated type.任何能得到值的运算（回顾操作符和结果类型）

  - A variable.Every variable has an associated type

    ```c#
    int x = 100;
    int y;
    y = x;
    ```

  - A namespace

    ```c#
    System.Windows.Forms.Form myForm; //System是名称空间、Windows是子名称空间，Forms是类型，每个名称空间本身就是独立的表达式
    ```

  - A type

    ```c#
    var t = typeof(Int32);
    ```

  - A method group 例如：Console.WriteLine，这是一组方法，重载决策决定具体调用哪一个

    ```c#
    Console.WriteLine("Hello,World!"); //Console.WriteLine拿到是方法组，而在后面调用的时候，才是从这个方法组里利用重载决策来选出最合适的方法进行调用
    ```

  - A null literal

    ```c#
    Form myForm = null; //null不属于任何数据类型，因为它是空的
    int x = 100; //数值类型
    ```

  - An anonymouse function

    ```c#
    Acton a = delegate(){ Console.WriteLine("Hello, World!")}; //匿名方法表达式，这个表达式返回值是一个委托
    a(); //"Hello, World!"
    ```

  - A property access

    ```c#
    Form myForm = new Form();
    myForm.Text = "Hello"; //成员访问表达式，访问到属性了
    myForm.ShowDialog();
    ```

  - An event access

    ```c#
    Form myForm = new Form();
    myForm.Text = "Hello";
    myForm.Load += myForm_Load; //访问事件表达式
    myForm.ShowDialog();

    static void myForm_Load(object sender, EventArgs e)
    {
        Form form = sender as Form;
        if(form == null)
        {
            return;
        }

        form.Text = "New Title";
    }
    ```

  - An indexer access

    ```c#
    List<int> intList = new List<int>(){1,2,3};
    int x = intList[2]; //访问索引器
    ```

  - Nothing.对返回值为 void 的方法的调用

    ```c#
    Console.WriteLine("Hello!") //无返回值，Nothing
    ```
- 复合表达式的求值
  - 注意操作符的优先级和同优先级操作符的运算方向
- 参考 C#语言定义文档
  - 仅作参考，不必深究 ———— 毕竟我们是在学习语言、不是去实现这门语言

```c# title="x.y成员访问表达式"
namespace ExperssionExample
{
    class Program
    {
        static void Main(string[] args)
        {
            Student stu = new Student();
            var x = stu.Name;
            Console.WriteLine(x.GetType());
        }
    }

    class Student
    {
        public int ID;
        public string Name;
    }
}
```

```c# title="f(x)方法调用表达式"
namespace ExperssionExample
{
    class Program
    {
        static void Main(string[] args)
        {
            var x = Math.Pow(2, 3);
            Console.WriteLine(x.GetTpye().Name); //Double
        }
    }
}
```

在定义`Math.Pow()`的时候，它的返回值类型就是 double

```c# title="元素访问表达式"
namespace ExpressionExample
{
    class Program
    {
        static void Main(string[] args)
        {
            List<int> intList = new List<int>() {1, 2, 3};
            double[] doubleArray = new double[] {1.0, 2.0, 3.0};
            var x = intList[1];
            Console.WriteLine(x.GetType()); //System.Int32

            var x = doubleArray[1];
            Console.WriteLine(x.GetType()); //System.Double
        }
    }
}
```

由元素访问操作所组成的表达式，它的类型是跟着你访问的这个集合的元素类型来决定的

```c# title="x++ x--表达式"
namespace ExpressionExample
{
    class Program
    {
        static void Main(string[] args)
        {
            int x = 100;
            Console.WriteLine((x++).GetType().FullName); //System.Int32
        }
    }
}
```

```c# title="new 创建实例表达式"
namespace ExpressionExample
{
    class Program
    {
        static void Main(string[] args)
        {
            Console.WriteLine((new From()).GetType().FullName); //System.Windows.Forms.Form
        }
    }
}
```

```c# title="Default操作符"
namespace ExprssionExample
{
    class Program
    {
        static void Main(string[] args)
        {
            var x = default(Int32);
            Console.WriteLine(x); // 0
            Console.WriteLine(x.GetType().FullName); // System.Int32
        }
    }
}
```

```c# title="checked unchecked操作符"
namespace ExpressionExample
{
    class Program
    {
        static void Main(string[] args)
        {
            var x = checked(111 + 222);
            Console.WriteLine(x.GetType().FullName); //System.Int32
        }
    }
}
```

`+ -`不会影响操作类型；`!`操作数必须是布尔类型的，而所得到的结果也是布尔类型的，只是把 true 变成 false，把 false 变成 true；`(T)x`数据类型是一定会转变的，而数据类型就是你要转变的数据类型；

```c# title="算术运算符"
namespace ExprssionExample
{
    class Program
    {
        static void Main(string[] args)
        {
            var x = 4 / 3;// System.Int32

            var x = 4.0 / 3;// 数据提升，System.Double
        }
    }
}
```

```c# title="移位操作符"
namespace ExpressionExample
{
    class Program
    {
        static void Main(string[] args)
        {
            long x = 100;
            Console.WriteLine((x<<2).GetType().FullName); //System.Int64，数据类型个操作符左边的数据类型一致
        }
    }
}
```

`is`操作符组成的也是布尔类型表达式，`as`操作符所组成的表达式，如果`as`成功了那就和右边的数据类型一致，如果不成功就返回一个 null 值；`& ^ |`这三个操作符所组成的表达式，它的数据类型和操作数的数据类型是一致的；`&& ||`这两个操作符所组成的表达式也都是布尔类型表达式，由它们所组成的表达式，在经过求值之后得到的是布尔类型的值，而且这两个操作符它们的操作数也必须是布尔类型的操作数（所以这两个操作符，经常连接一长串的布尔类型的表达式或者布尔类型的值）；

```c# title="null值合并操作符"
namespace ExpressionExample
{
    class Program
    {
        static void Main(string[] args)
        {
            int? x = null;
            var y = x ?? 100;
        }
    }
}
```

`??`它的数据类型是由 null 值合并操作符左边的操作数的数据类型的类型参数来决定的

```c# title="条件运算符?:"
namespace ExpressionExample
{
    class Program
    {
        static void Main(string[] args)
        {
            var x = 5 > 3 ? 2 : 3.0; //System.Double
        }
    }
}
```

`?:`所组成的表达式，它的数据类型是由：左右两边的操作数数据类型决定的，而且这两个操作数，谁的精度高就用谁的数据类型

```c# title="赋值表达式"
namespace ExpressionExample
{
        static void Main(string[] args)
        {
            int x = 100;
            int y;
            Console.WriteLine(y=x); //100
            Console.WriteLine((y=x).GetType().FullName); //System.Int32

            bool b = false;
            if(b == 5 > 3) // false 不等于 true，换成b = 5 > 3就执行
            {
                Console.WriteLine("Hello");
            }
        }
    }
}
```

## 语句

### 定义

- Wikipeida 对语句的定义

  - Incomputer programming a statement is the smallest standalone element of an imperative programming language which expresses some action to be carried out.A program written in such a language is formed by a seguence of one or more statements.A statement will have internal components (e.gexpressions).
  - 语句是高级语言的语法——编译语言和机器语言只有指令（高级语言中的表达式对应低级语言中的指令），语句等价于一人或一组有明显逻辑关联的指令。举例：求圆柱体积

- C#语言对语句的定义

  - The actions that a program takes are expressed in statements. Common actions include declaring variables, assigning values, calling methods, looping through collections, and branching to one or another block of code, depending on a given condition. The order in
    which statements are executed in a program is called the flow of control or flow of execution
    The flow of control may vary every time that a program is run, depending on how the program reacts to input that it receives at run time
  - C 语言的语句除了能够让程序员"顺序地"(sequentially)表达算法思想，还能通过条件判断、跳转和循环等方法控制程序逻辑的走向
  - 简言之就是：陈述算法思想，控制逻辑走向，完成有意义的动作(action）
  - C#语言的语句由分号（；）结尾，但由分号结尾的不一定都是语句
  - 语句一定是出现在方法体里

### 分类-D01-概要

![13.avif](../assets/Csharp/13.avif)

```C#
namespace StatementsExample
{
    class Program
    {
        static void Main(stinrg[] args)
        {
            int score = 55;
            if(score>=60)
                if(score>=85)
                    Console.WriteLine("Best!");
                else
                    Console.WriteLine("Good!");
            else
                Console.WriteLine("Failed!")
        }
    }
}
```

### 分类-D02-声明语句

```c#
namespace StatementsExample
{
    class Program
    {
        static void Main(string[] args)
        {
            int x = 100;  //声明变量的时候追加了变量的初始化器

            int x;
            x = 100; //声明的时候没有初始化，而在后面的时候对变量赋值

            int[] myArray = {1, 2, 3};
            Console.WriteLine(myArray[1]); //数组初始化器

            const int x = 100;
            x = 200; //能接受赋值一定是个变量，常量的值是不能够再改变的

            const int x; //常量必须要被初始化
        }
    }
}
```

### 分类-D03-表达式语句

```c#
namespace StatementsExample
{
    class Program
    {
        static void Main(string[] args)
        {
            Add(3.0, 4.0);
        }

        static double Add(double a, double b)
        {
            double result = a + b;
            Console.WriteLine("Result is {0}", result);
            return result;
        }
    }
}
```

上面这个方法实际只做了两件事情，当一个方法做两件事情的时候，有一件事情一定是它的主功能，而另一件事情是它的副功能，这样在未来的时候，无论那个功能需求上发生了变化我们都要回来改这个方法，而如果有别的方法还调用这个方法的话呢，我的这个改动就有可能引入意想不到的 bug，这个时候我们就可以说我们的代码设计实际是有问题的

```c#
namespace StatementsExample
{
    class Program
    {
        static void Main(string[] args)
        {
            int x = 100;
            int y = 200;
            x + y; //在C#不能当作语句使用，毫无意义
        }
    }
}
```

### 分类-D04-块语句

```c#
namespace StatementsExample
{
    class Program
    {
        static void Main(string[] args)
        {
            {
                int x = 100; //声明语句
                if(x > 80)Console.WriteLine(x);//嵌入式语句
                hello: Console.WriteLine("Hello, World!"); //标签语句
                goto hello;
            }
        }
    }
}
```

编译器永远把块语句当成一条语句来看待，无论块语句容纳了多少子语句

ctrl+}会在花括号开始和结束之间跳动

```c#
namespace StatementsExample
{
    class Program
    {
        static void Main(string[] args)
        {
            int x = 100; //作用域在方法体内
            {
                Console.WriteLine(x);
                int y = 200; //作用域在块语句内
                Console.WriteLine(y);
            }
            Console.WriteLine(y);
        }
    }
}
```

### 分类-D05-选择语句 - if 语句

```c#
namespace StatementsExample
{
    class Program
    {
        static void Main(string[] args)
        {
            if(5 > 3)Console.WriteLine("Hello");
            if(5 + 3)Console.WriteLine("Hello");//编译不过去，原括号里需要布尔类型表达式

            int y = 100;
            int x = 200;
            if(x < y)
                Console.WriteLine("Hello");
            Console.WriteLine("World");

            if(x < y)
            {
                Console.WriteLine("Hello");
                Console.WriteLine("World");
            }
            else
            {
                Console.WriteLine("World!");
                Console.WriteLine("No");
            }
        }
    }
}
```

```c#
namespace StatementsExample
{
    class Program
    {
        static void Main(string[] args)
        {
            int score = 100;
            if (score >= 0 && score <= 100)
            {
                if (score >= 60)
                {
                    if (score >= 80)
                    {
                        Console.WriteLine("A");
                    }
                    else
                    {
                        Console.WriteLine("B");
                    }
                }
                else
                {
                    if (score >= 40)
                    {
                        Console.WriteLine("C");
                    }
                    else
                    {
                        Console.WriteLine("D");
                    }
                }
            }
            else
            {
                Console.WriteLine("InputError");
            }
        }
    }
}
```

```c#
namespace StatementsExample
{
    class Program
    {
        static void Main(string[] args)
        {
            int score = 100;
            if(score>=80 && score<=100)
            {
                Console.WriteLine("A");
            }
            else
            {
                if(score>=60)
                {
                    Console.WriteLine("B");
                }
                else
                {
                    if(score>=40)
                    {
                        Console.WriteLine("C");
                    }
                    else
                    {
                        if(score>=0)
                        {
                            Console.WriteLine("D");
                        }
                        else
                        {
                            Console.WriteLine("Input Error!");
                        }
                    }
                }
            }
        }
    }
}
```

```c#
namespace StatementsExample
{
    int score = 100;
    if(score > 80 && score <= 100)
    {
        Console.WriteLine("A");
    }
    else if(score >= 60)
    {
        Console.WriteLine("B");
    }
    else if(score >= 40)
    {
        Console.WriteLine("C");
    }
    else if(score >= 0)
    {
        Console.WriteLine("D");
    }
    else
    {
        Console.WriteLine("Input Error!");
    }
}
```

### 分类-D06-选择语句-switch 语句

```c#
namespace StatementsExample
{
    class Program
    {
        static void Main(string[] args)
        {
            int score = 101;
            // int y = 10;
            switch(score/10)
            {
                // casy y: //不能是变量
                case 10:
                    if(score == 100)
                    {
                        goto case 8:
                    }
                    else
                    {
                        goto default;
                    }
                case 8:
                    // Console.WriteLine("A"); //一旦标签后面跟着语句，他就变成了section而不是单独的标签，既然是section就必须显式的写一个break
                case 9:
                    Console.WriteLine("A");
                    break;
                case 6:
                case 7:
                    Console.WriteLine("B");
                    break;
                case 4:
                case 5:
                    Console.WriteLine("C");
                    break;
            }

        }
    }
}
```

如果 switch 表达式的类型为 sbyte、byte、short、ushort、int、uint、long、ulong、bool、char、string 或 enum-type，或者是对应于以上某种类型的可以为 null 的类型，则该类型就是 switch 语句的主导类型。

```c#
namespace StatementsExample
{
    internal class Program
    {
        private static void Main(string[] args)
        {
            Level myLevel = Level.High;
            switch (myLevel)
            {
                case Level.High:
                    Console.WriteLine("High level!");
                    break;

                case Level.Mid:
                    Console.WriteLine("Mid level!");
                    break;

                case Level.Low:
                    Console.WriteLine("Low level!");
                    break;

                default:
                    break;
            }
        }
    }

    internal enum Level
    {
        High,
        Mid,
        Low
    }
}
```

### 分类-D07-异常捕捉语句-try 语句

```c#
namespace StatementsExample
{
    class Program
    {
        static void Main(string[] args)
        {
            Calculator c = new Calculator();
            // int r = c.Add(null,"200"); //FormatException，没有容错能力
            Console.WriteLine(r);
        }
    }

    class Calculator
    {
        public int Add(string arg1, string arg2)
        {
            int a = int.Parse(arg1);
            int b = int.Parse(arg2);
            int result = a + b;
            return result;
        }
    }
}
```

```C#
namespace StatementsExample
{
    class Program
    {
        static void Main(string[] args)
        {
            Calculator c = new Calculator();
            int r = c.Add(null, "200"); //Your arguments are null
            int r = c.Add("abc", "200"); //Your argument<s> are not number
            int r = c.Add("99999999999", "200"); //Out of range
            Console.WriteLine(r);
        }
        catch（ArgumentNullExcpetion)
        {
            Console.WriteLine("Your argument(s) are null");
        }
        catch(FormatException)
        {
            Console.WriteLine("Your argument(s) are not number");
        }
        catch(OverflowException)
        {
            Console.WriteLine("Out of range!");
        }

        int result = a + b;
        return result;
    }
}
```

```c#
namespace StatementsExample
{
    class Program
    {
        private static void Main(string[] args)
        {
            Calculator c = new Calculator();
            int r = c.Add("99999999999999999", "200"); //Value was either too large or too small for an Int32.
            // int r = c.Add("abc", "200"); //The input string 'abc' was not in a correct format.
            //int r = c.Add("null", "200"); //The input string 'null' was not in a correct format.
            Console.WriteLine(r);
        }
    }

    class Calculator
    {
        public int Add(string arg1, string arg2)
        {
            int a = 0;
            int b = 0;
            try
            {
                a = int.Parse(arg1);
                b = int.Parse(arg2);
            }
            catch (ArgumentNullException ane)
            {
                Console.WriteLine(ane.Message);
            }
            catch (FormatException fe)
            {
                Console.WriteLine(fe.Message);
            }
            catch (OverflowException oe)
            {
                Console.WriteLine(oe.Message);
            }

            int result = a + b;
            return result;
        }
    }
}
```

```c#
namespace StatementsExample
{
    class Program
    {
        private static void Main(string[] args)
        {
            Calculator c = new Calculator();
            int r = c.Add("0", "0"); //Done! 0
            // int r = c.Add("abc", "100"); //Input string was not in a correct format;Execution has error!;0
            Console.WriteLine(r);
        }
    }
    class Calculator
    {
        public int Add(string arg1, string arg2)
        {
            int a = 0;
            int b = 0;
            bool hasError = false;
            try
            {
                a = int.Parse(arg1);
                b = int.Parse(arg2);
            }
            catch (ArgumentNullException ane)
            {
                Console.WriteLine(ane.Message);
                hasError = true;
            }
            catch (FormatException fe)
            {
                Console.WriteLine(fe.Message);
                hasError = true;
            }
            catch (OverflowException oe)
            {
                Console.WriteLine(oe.Message);
                hasError = true;
            }
            finally
            {
                if(hasError)
                {
                    Console.WriteLine("Execution has error!");
                }
                else
                {
                    Console.WriteLine("Done!")
                }
            }
            int result = a + b;
            return result;
        }
    }
}
```

```c#
namespace StatementsExample
{
    class Program
    {
        private static void Main(string[] args)
        {
            Calculator c = new Calculator();
            int r = 0;
            try
            {
                r = c.Add("999999999999999999","100"); //Value was either too large or too small for an Int32.;0
            }
            catch(OverflowException oe)
            {
                Console.WriteLine(oe.Message);
            }
            Console.WriteLine(r);
        }
    }
    class Calculator
    {
        public int Add(string arg1, string arg2)
        {
            int a = 0;
            int b = 0;
            try
            {
                a = int.Parse(arg1);
                b = int.Parse(arg2);
            }
            catch (ArgumentNullException ane)
            {
                Console.WriteLine(ane.Message);
            }
            catch (FormatException fe)
            {
                Console.WriteLine(fe.Message);
            }
            catch (OverflowException oe)
            {
                // Console.WriteLine(oe.Message);
                throw oe;
            }
            int result = a + b;
            return result;
        }
    }
}
```

`thorw`谁调用谁就抓住异常处理

### 分类-D08-循环语句-while 语句

```c#
namespace StatementsExample
{
    class Progran
    {
        static void Main(string[] args)
        {
            int score = 0;
            bool canContinue = true;

            {
                Console.WriteLine("Please input first number:");
                string str1 = Console.ReadLine();
                int x = int.Parse(str1);

                Console.WriteLine("Please intput second number:");
                string str2 = Console.ReadLine();
                int y = int.Parse(str2);

                int sum = x + y;
                if(sum == 100)
                {
                    score++;
                    Console.WriteLine("Correct!{0}+{1}={2}", x, y, sum);
                }
                else
                {
                    Console.WriteLine("Error! {0}+{1}={2}", x, y, sum);
                    canContinue = false;
                }while(canContinue)
            }

            Console.WriteLine("Your score is {0}", score);
            Console.WriteLine("GAME OVER!");
        }
    }
}
```

### 分类-D09-循环语句-do 语句

```c#
namespace StatementsExample
{
    class Progran
    {
        static void Main(string[] args)
        {
            int score = 0;
            int sum = 0;
            do
            {
                Console.WriteLine("Please input first number:");
                string str1 = Console.ReadLine();
                int x = int.Parse(str1);

                Console.WriteLine("Please intput second number:");
                string str2 = Console.ReadLine();
                int y = int.Parse(str2);

                sum = x + y;
                if(sum == 100)
                {
                    score++;
                    Console.WriteLine("Correct!{0}+{1}={2}", x, y, sum);
                }
                else
                {
                    Console.WriteLine("Error! {0}+{1}={2}", x, y, sum);
                }
            }while(sum == 100);

            Console.WriteLine("Your score is {0}", score);
            Console.WriteLine("GAME OVER!");
        }
    }
}
```

### 分类-D10-跳转语句-continue 语句

```C#
namespace StatementsExample
{
    class Program
    {
        static void Main(string[] args)
        {
            int score = 0;
            int sum = 0;
            do
            {
                Console.WriteLine("Please input first,number:");
                string str1 = Console.ReadLine();

                if(str1.ToLower() == "end")
                {
                    break；
                }

                int x = 0;
                try
                {
                    x = int.Parse(str1);
                }
                catch
                {
                    Console.WriteLine("First number has problem!Restart.");
                    continue;
                }

                Console.WriteLine("Plase input second number:");
                string str2 = Console.ReadLine();
                if(str2.ToLower() == "end")
                {
                    break;
                }

                int y = 0;
                try
                {
                    y = int.Parse(str2);
                }
                catch
                {
                    Console.WriteLine("Second number has problem!Restart.")
                    continue;
                }

                sum = x + y;
                if(sum == 100)
                {
                    score++;
                    Console.WriteLine("Correct!{0}+{1}={2}", x, y, sum);
                }
                else
                {
                    Console.WriteLine("Error!{0}+{1}={2}", x, y, sum);
                }
            }while(sum == 100);

            Console.WriteLine("Your score is {0}.", score);
            Console.WriteLine("GAME OVER!");
        }
    }
}
```

### 分类-D11-循环语句-for 语句

```c# title="九九乘法表"
namespace StatementsExample
{
    class Program
    {
        static void Main(string[] args)
        {
            for (int a = 1; a <= 9; a++)
            {
                for (int b = 1; b <= a; b++)
                {
                    Console.Write("{0}x{1}={2}\t", a, b, a * b);
                }
                Console.WriteLine();
            }
        }
    }
}
```

### 分类-D12-循环语句-foreach 语句

```c#
namespace StatementsExample
{
    class Program
    {
        static void Main(string[] args)
        {
            int[] intArray = new int[] {1, 2, 3, 4, 5, 6, 7, 8};
            IEnumerator enumerator = intArray.GetEnumerator();
            while(enumerator.MoveNext())
            {
                Console.WriteLine(enumerator.Current);
            }

            enumerator.Reset();
            while(enumerator.MoveNext())
            {
                Console.WriteLine(enumerator.Current);
            }

            List<int> intList = new List<int>() {1, 2, 3, 4, 5, 6};
            IEnumerator enumerator = intList.GetEnumerator();
            while(enumerator.MoveNext())
            {
                Console.WriteLine(enumerator.Current);
            }

            enumerator.Reset();
            while(enumerator.MoveNext())
            {
                Console.WriteLine(enumerator.Current);
            }
        }
    }
}
```

```C
namespace StatementsExample
{
    class Program
    {
        static void Main(string[] args)
        {
            int[] intArray = new int[] {1, 2, 3, 4, 5, 6, 7, 8};

            List<int> intList = new List<int>() {1, 2, 3, 4, 5, 6};
            foreach(var current in intList)
            {
                Console.WriteLine(current);
            }
        }
    }
}
```

### 分类-D13-跳转语句-return 语句

```c#
namespace StatementsExample
{
    class Program
    {
        static void Main(string[] args)
        {
            var result = WholsWho("Mr.Hello");
            Console.WriteLine(result);
        }

        static string WholsWho(string alias)
        {
            if(alias == "Mr.Okay")
            {
                retrun "Tim";
            }
            else
            {
                return "I don't know";
            }
        }

        static void Greeting(string name)
        {
            if(string.IsNullOrEmpty(name))
            {
                retrun;
            }

            Console.WriteLine("Hello, {0}!", name);
        }
    }
}
```

## 成员

### 字段

- 什么是字段
  - 字段（field）是一种表示与对象或类型（类与结构体）关联的变量
  - 字段是类型的成员，旧称“成员变量”
  - 与对象关联的字段亦称“实例字段”
  - 与类型关联的字段称为“静态字段”，由 static 修饰
- 字段的声明
  - 参见 C#语言定义文档
  - 尽管字段声明带有分号，但它不是语句
  - 字段的名字一定是名词
- 字段的初始值
  - 无显示初始化时，字段获得其类型的默认值，所以字段"永远都不会未被初始化"
  - 实例字段初始化的时机————对象创建时
  - 静态字段初始化的时机————类型被加载（load）时
- 只读字段
  - 实例只读字段
  - 静态只读字段

```c
struct Student
{
    int ID,
    char* Name,
};

void Main()
{
    struct Student stu;
    stu.ID = 1;
    stu.Name = "Mr.Okay";
    printf("Student, #%d is %s.", stu.ID, stu.Name);
}
```

```c#
namespace DataMemberExample
{
    class Program
    {
        static void Main(string[] args)
        {
            Student stu1 = new Student();
            stu1.Age = 40;
            stu1.Score = 90;

            Student stu2 = new Student();
            stu2.Age = 24;
            stu2.Score = 60;

            Student.ReportAmount(); // 2
        }
    }

    class Student
    {
        public int Age;
        public int Score;

        public static int AverageAge;
        public static int AverageScore;
        public static int Amount;

        public Student() //每次实例字段创建的时候执行
        {
            Student.Amount++；
        }

        public static void ReportAmount()
        {
            Console.WriteLine(Student.Amount);
        }
    }
}
```

![14.avif](../assets/Csharp/14.avif)

```c#
namespace DataMemberExample
{
    class Program
    {
        static void Main(string[] args)
        {
            List<Student>stuList = new List<Student>();
            for(int i = 0; i < 100; i++)
            {
                Student stu = new Student();
                stu.Age = 24;
                stu.Score = i;
                stuList.Add(stu);
            }

            int totalAge = 0;
            int totalScore = 0;
            foreach (var stu in stuList)
            {
                totalAge += stu.Age;
                totalScore += stu.Score;
            }

            Student.AverageAge = totalAge / Student.Amount;
            Student.AverageScore = totalScore / Student.Amount;

            Student.ReportAmount();
            Student.ReportAverageAge();
            Student.ReportAverageScore();
        }
    }

    class Student
    {
        public int Age;
        public int Score;

        public static int AverageAge;
        public static int AverageScore;
        public static int Amount;

        public Student()
        {
            Student.Amount++;
        }

        public static void ReportAmount()
        {
            Console.WriteLine(Student.Amount);
        }

        public static void ReportAverageAge()
        {
            Console.WriteLine(Student.AverageAge);
        }

        public static void ReportAverageScore()
        {
            Console.WriteLine(Student.AverageScore);
        }
    }
}
```

```c#
namespace DataMemberExample
{
    class Program
    {
        static void Main(string[] args)
        {
            Console.WriteLine(Student.Amount);
        }
    }

    class Student
    {
        public int Age = 20;
        public int Score;

        public static int AverageAge;
        Public static int AverageScore;
        public static int Amount; // = 100;

        static Student()
        {
            Student.Amount = 100
        }
    }
}
```

```c#
namespace DataMemberExample
{
    class Program
    {
        static void Main(string[] args)
        {
            Console.WriteLine(Brush.DefaultColor.Red);
            Console.WriteLine(Brush.DefaultColor.Green);
            Console.WriteLine(Brush.DefaultColor.Blue);

            Brush.Default = new Color() {Red = 255, Green = 255, Blue = 255}; // 静态只读字段 不能够被赋值
        }
    }

    struct Color
    {
        public int Red;
        public int Green;
        public int Blue;
    }

    class Brush
    {
        public static readonly Color DefaultColor // = new Colo() {Red = 0, Green = 0, Blue = 0}
        static Brush()
        {
            Brush.DefaultColor = new Colo() {Red = 0, Green = 0, Blue = 0};
        }
    }
}
```

### 属性

- 什么是属性

  - 属性(property）是一种用于访问对象或类型的特征的成员，特征反映了状态
  - 属性是字段的自然拓展
    - 从命名上看，field 更偏向于实例对象在内存中的布局，property 更偏向于反映现实世界对象的特征
    - 对外：暴露数据，数据可以是存储在字段里的，也可以是动态计算出来的
    - 对内：保护字段不被非法值污染
  - 属性由 Get/Set 方法对进化而来
  - 又一个“语法糖”————属性背后的秘密

- 属性的声明
  - 完整声明————后台（back）成员变量与访问器（注意使用 code snippet 和口 refactor 工具）
  - 简略声明————只有访问器（查看 IL 代码）
  - 动态计算值的属性
  - 注意实例属性和静态属性
  - 属性的名字一定是名词
  - 只读属性————只有 getter 没有 setter
    - 尽管语法上正确，几乎没有人使用“只写属性”，因为属性的主要目的是通过向外暴露数据而表示对象/类型的状态
- 属性与字段的关系
  - 一般情况下，它们都用于表示实体（对象或类型）的状态
  - 属性大多数情况下是字段的包装器（wrapper）
  - 建议：永远使用属性（而不是在字段）来暴露数据，即字段永远都是 private 或 protected 的

```c#
namespace PropertyExample
{
    class Program
    {
        static void Main(string[] args)
        {
            try
            {
                Student stu1 = new Student();
                stu1.SetAge(20);

                Student stu2 = new Student();
                stu2.SetAge(20);

                Student stu3 = new Student();
                stu3.SetAge(20);

                stu3.SetAge(200); //Age value has error.

                int avgAge = (stu1.GetAge() + stu2.GetAge() + stu3.GetAge()) / 3;

                Console.WriteLine(avgAge);
            }
            catch(Exception ex)
            {
                Console.WriteLine(ex.Message);
            }
        }

        class Student
        {
            private int age;

            public int GetAge()
            {
                return this.age;
            }

            public void SetAge(int value)
            {
                if(value >=0 && value <= 120)
                {
                    this.age = value;
                }
                else
                {
                    throw new Exception("Age value has error.");
                }
            }
        }
    }
}
```

```c#
namespace PropertyExample
{
    class Program
    {
        static void Main(string[] args)
        {
            try
            {
                Student stu1 = new Student();
                stu1.Age = 20;

                Student stu2 = new Student();
                stu2.Age = 20;

                Student stu3 = new Student();
                stu3.Age = 20;

                stu3.Age = 200; //Age value has error

                int avgAge = (stu1.Age + stu2.Age + stu3.Age) / 3;
                Console.WriteLine(avgAge);
            }
            catch(Exception ex)
            {
                Console.WriteLine(ex.Message);
            }
        }

        class Student
        {
            private int age;

            public int Age
            {
                get
                {
                    return this.age;
                }

                set
                {
                    if(value>=0 && value<=120)
                    {
                        this.age = value;
                    }
                    else
                    {
                        throw new Exception("Age value has error.")
                    }
                }
            }
        }
    }
}
```

`value`上下文关键字，在某个特定的代码上下文当中是关键字，关键字会被高亮为蓝色

```c#
namespace PropertyExample
{
    class Program
    {
        static void Main(string[] args)
        {
            try
            {
                Student stu1 = new Student();
                stu1.Age = 20;

                Student stu2 = new Student();
                stu2.Age = 20;

                Student stu3 = new Student();
                stu3.Age = 20;

                stu3.Age = 200; //Age value has error

                int avgAge = (stu1.Age + stu2.Age + stu3.Age) / 3;
                Console.WriteLine(avgAge);
            }
            catch(Exception ex)
            {
                Console.WriteLine(ex.Message);
            }
        }

        class Student
        {
            private int age;

            public int Age
            {
                get { return age; }
                set
                {
                    if(value>=0 && value<=120)
                    {
                        age = value;
                    }
                    else
                    {
                        throw new Exception("Age value has error.")
                    }
                }
            }
        }
    }
}
```

![15.avif](../assets/Csharp/15.avif)

![16.avif](../assets/Csharp/16.avif)

```c#
namespace ProrertyExample
{
    class Program
    {
        static void Main(string[] args)
        {
            try
            {
                Student.Amount = -100;
                Console.WriteLine(Student.Amount);
            }
            catch(Exception ex)
            {
                Console.WriteLine(ex.Message);
            }
        }
    }

    class Student
    {
        private int age;

        public int Age
        {
            get { return age; }
            set {
                if(value>=0 && value<=120)
                   {
                       age = value;
                   }
                   else
                   {
                       throw new Exception("Age value has error.");
                   }
            }
        }

        private static int amount;

        public static int Amount
        {
            get { return amount; }
            set {
                if(value>=0)
                {
                    Student.amount = value;
                }
                else
                {
                    throw new Exception("Amount must greater than 0.");
                }
            }
        }
    }
}
```

实例属性表示的是实例的某个数据，通过这个数据反映这个实例的某个状态，静态属性表示这个类型当前的某个数据

```c#
namespace ProrertyExample
{
    class Program
    {
        static void Main(string[] args)
        {
            try
            {
                Student.Amount = new Student();
                stu.Age = 1000;
            }
            catch(Exception ex)
            {
                Console.WriteLine(ex.Message);
            }
        }
    }

    class Student
    {
        public int Age { get;set; }
    }
}
```

![17.avif](../assets/Csharp/17.avif)

```c#
namespace ProrertyExample
{
    class Program
    {
        static void Main(string[] args)
        {
            try
            {
                Student.Amount = new Student();
                // stu.Age = 1000; // 外界访问不到
            }
            catch(Exception ex)
            {
                Console.WriteLine(ex.Message);
            }
        }
    }

    class Student
    {
        private int age;

        public int Age
        {
            get { return age; }
            private set { age = value; } //不是只读属性readonly
        }

        public void SomeMethod()
        {
            this.Age = 20; //
        }
    }
}
```

```c#
namespace ProrertyExample
{
    class Program
    {
        static void Main(string[] args)
        {
            try
            {
                Student.Amount = new Student();
                stu.Age = 12;
                Console.WriteLine(stu.Canwork); //False

                stu.Age = 18;
                Console.WriteLine(stu.Canwork); //True
            }
            catch(Exception ex)
            {
                Console.WriteLine(ex.Message);
            }
        }
    }

    class Student
    {
        private int age;

        public int Age
        {
            get { return age; }
            set
            {
               age = value;
               this.CalcumateCanWork();
            }
        }

        private bool canWork;

        public bool CanWork
        {
            get { return canWork; }
        }

        private void CalcumateCanWork()
        {
            if(this.age>=16)
            {
                this.canWork = true;
            }
            else
            {
                this.canWork = false;
            }
        }

        public bool CanWork {
            get
            {
                if(this.age>=16)
                {
                    return true;
                }
                else
                {
                    return false;
                }
            }
        }
    }
}
```

`CanWork`容易浪费性能，不如每次在设定值的时候就把`canWork`计算出来，那么相反如果学生 age 这个值经常被设定，而`canWork`这个值不经常被访问的时候，我们现在这个 CalcumateCanWork 设计就有点浪费计算机性能了

### 索引器

```c#
namespace IndexerExample
{
    class Program
    {
        static void Main(sting[] args)
        {
            Student stu = new Student();
            stu["Math"] = 90; // 90
            stu["Math"] = 100; // 100
            var mathScore = stu["Math"];
            Console.WriteLine(mathScore == null); //True
            Console.WriteLine(mathScore); //什么都没有，null
        }
    }

    class Student
    {
        private Dictionary<string, int> scoreDictionary = new Dictionary<string, int>();

        public int?[string subject]
        {
            get
            {
                if(this.socreDictionary.ContainsKey(subject))
                {
                    return this.scoreDictionary[subject];
                }
                else
                {
                    return null;
                }
            }
            set {
                if(value.HasValue == false)
                {
                    throw new Exception("Score cannot be null.");
                }

                if(this.scoreDictionary.ContainsKey(subject))
                {
                    this.scoreDictionary[subject] = value.Value;
                }
                else
                {
                    this.scoreDictionary.Add(subject, value.Value);
                }
            }
        }
    }
}
```

### 常量

- 什么是常量
  - 常量（constant）是表示常量值（即，可以在编译时计算的值）的类成员
  - 常量隶属于类型而不是对象，即没有“实例常量”
    - “实例常量”的角色由只读实例字段来担当
  - 注意区分成员常量与局部常量
- 常量的声明
- 各种”只读“的应用场景
  - 为了提高程序的可读性和执行效率————常量
  - 为了防止对象的值被改变————只读字段
  - 向外暴露不允许修改的数据————只读属性（静态或非静态），功能与常量有一些重叠
  - 当希望成为常量的值其类型不能被常量声明接受时（类/自定义结构体）————静态只读字段

```c#
namespace ConstantExample
{
    class Program
    {
        static void Main(string[] args)
        {
            const int x = 100;
            x = 200;
        }
        static double GetArea(double r)
        {
            double a = Math.PI * r * r;
            return a;
        }
    }
}
```

```c#
namespace ConstantExample
{
    class Program
    {
        static void Main(string[] args)
        {
            Consoel.WriteLine(WASPEC.WebsiteURL)
        }
        static double GetArea(double r)
        {
            public const string WebsiteURL = "http://www.waspec.org";
            // public const Building Location = new Building("Some Address"); //不能够用类类型或者自定义的结构体类型，来作为常量的类型
            public static readonly Building Location = new Building("Some Address"); //静态只读字段
        }

        class Building
        {
            public Building(string address)
            {
                this.Address = address;
            }
            public string Address { get; set; }
        }
    }
}
```

## 参数

### 传值参数

- 传值参数（声明时不带修饰符的形参是值形参。一个形参对应于一个局部变量，只是它的初始值来自该方法调用所提供的相应实参）
- 输出参数
- 引用参数
- 数组参数
- 具名参数
- 可选参数
- 拓展方法（this 参数）

![18.avif](../assets/Csharp/18.avif)

```c#
namespace ParametersExample
{
    class Program
    {
        static void Main(string[] args)
        {
            Student stu = new Student();
            int y = 100;
            stu.AddOne(y);
            Console.WriteLine(y); // 100
        }
    }

    class Student
    {
        public void AddOne(int x)
        {
            x = x + 1;
            Console.WriteLine(x); // 101
        }
    }
}
```

![19.avif](../assets/Csharp/19.avif)

```C#
namespace ParametersExample
{
    class Program
    {
        static void Main(string[] args)
        {
            Student stu = new Student(){Name = "Tim"};
            SomeMethod(stu);
            Console.WriteLine("{0}, {1}", stu.GetHashCode(), stu.Name);
        }

        static void SomeMethod(Student stu)
        {
            stu = new Student(){Name = "Tim"};
            Console.WriteLine("{0}, {1}", stu.GetHashCode(), stu.Name);
        }

        class Student
        {
            public string Name { get; set }
        }
    }
}
```

`Student stu = new Student(){Name = "Tim"};`中的 stu 作用域是 Main 函数方法体，而作为`static void SomeMethod(Student stu)`中的 stu 作用域是 SomeMethod 方法体

![20.avif](../assets/Csharp/20.avif)

```c#
namespace ParametersExample
{
    class Program
    {
        static void Main(string[] args)
        {
            Student stu = new Student(){ Name = "Tim" };
            UpdateObject(stu);
            Console.WriteLine("HashCode={0}, Name={1}", stu.GetHashCode(), stu.Name); //一样
        }

        static void SomeMethod(Student stu)
        {
            stu.Name = "Tom"; //副作用，side-effect
            Console.WriteLine("HashCode={0}, Name={1}", stu.GetHashCode(), stu.Name); //一样
        }
    }
}
```

### 输出参数

![21.avif](../assets/Csharp/21.avif)

```c#
namespace ParametersExample
{
    class Program
    {
        static void Main(string[] args)
        {
            Console.WriteLine("Please input first number:");
            string arg1 = Console.ReadLine();
            double x = 0;
            bool b1 = double.TryParse(arg1, out x);
            if(b1 == false)
            {
                Console.WriteLine("Input error!");
                return;
            }

            Console.WriteLine("Please input second number");
            string arg2 = Console.ReadLine();
            double y = 0;
            bool b2 = double.TryParse(arg2, out y);
            if(b2==false)
            {
                Console.WriteLine("Input error!");
                return;
            }

            double z = x + y;
            Console.WriteLine("{0}+{1}={2}", x, y, z);
        }
    }
}
```

```c#
namespace ParametersExample
{
    class Program
    {
        static void Main(string[] args)
        {
            double x = 100;
            bool b = Double.TryParse("ABC", out x);
            if(b == true)
            {
                Console.WriteLine(x + 1);
            }
            else
            {
                Console.WriteLine(x);
            }
        }

        class DoubleParser
        {
            public static bool TryParse(string input, out double result)
            {
                try
                {
                    result = double.Parse(input);
                    return true;
                }
                catch
                {
                    result = 0;
                    return false;
                }
            }
        }
    }
}
```

![22.avif](../assets/Csharp/22.avif)

```c#
namespace ParametersExample
{
    class Program
    {
        static void Main(sting[] args)
        {
            Student stu = null;
            bool b = StudentFactory.Create("Tim", 34, out stu);
            if(b == true)
            {
                Console.WriteLine("Student{0}, age is {1}.", stu.Name, stu.Age);
            }
        }
    }

    class Student
    {
        public int Age { get; set; }
        public string Name { get; set; }
    }

    class StudentFactory
    {
        public static bool Create(string stuName, int stuAge, out Student result)
        {
            result = null;
            if(string.IsNullOrEmpty(stuName))
            {
                return false;
            }

            if(stuAge<20 || stuAge > 80)
            {
                return false;
            }

            result = new Student(){Name = stuName, Age = stuAge};
            return true;
        }
    }
}
```

### 数组参数

- 必需是形参列表中的最后一个，由 params 修饰
- 举例：String.Format 方法和 String.Split 方法

```c#
namespace ParametersExample
{
    class Program
    {
        static void Main(string[] args)
        {
            // int[] myIntArray = new int[] {1, 2, 3};
            // int result = CalculateSum(myIntArray);
            int result = CalculateSum(1, 2, 3);
            Console.WriteLine(result);
        }

        static int CalculateSum(params int[] intArray)
        {
            int sum = 0;
            foreach(var item in intArray)
            {
                sum += item;
            }
            return sum;
        }
    }
}
```

```c#
namespace ParametersExample
{
    class Program
    {
        static void Main(string[] args)
        {
            string str = "Tim;Tom,Amy.Lisa";
            string[] result = str.Split(';','.',',');
            foreach (var name in result)
            {
                Console.WriteLine(name);
            }
        }

        static int CalculateSum(params int[] intArray)
        {
            int sum = 0;
            foreach(var item in intArray)
            {
                sum += item;
            }
        }
    }
}
```

### 具名参数

- 参数的位置不再受约束

```C#
namespace ParametersExample
{
    class Program
    {
        static void Main(string[] args)
        {
            PrintInfo(name: "Tim", age: 34);
        }

        static void PrintInfo(string name, int age)
        {
            Console.WriteLine("Hello {0}, you are {1}.", name, age);
        }
    }
}
```

### 可选参数

- 参数因为具有默认值而变得"可选"
- 不推荐使用可选参数

```c#
namespace ParametersExample
{
    class Program
    {
        static void Main(string[] args)
        {
            PrintInfo();
        }

        static void PrintInfo(string name = "Tim", int age = 34)
        {
            Console.WriteLine("Hello {0}, you are {1}.", name, age);
        }
    }
}
```

### 扩展方法（this 参数）

- 方法必需是公有、静态的，即被 public static 所修饰
- 必需是形参列表中的第一个，由 this 修饰
- 必需由一个静态类（一般类名为 SomeTypeExtension）来统一收纳对 SomeType 类型的扩展方法
- 举例：LINQ 方法

```c#
namespace ParamatersExample
{
    class Program
    {
        static void Main(string[] args)
        {
            double x = 3.14159;
            double y = x.Round(x, 4);
            Console.WriteLine(y);
        }
    }

    static class DoubleExtension
    {
        public static double Round(this double input, int digits)
        {
            double result = Math.Round(input, digits);
            return result;
        }
    }
}
```

```c#
namespace ParamterssExample
{
    class Program
    {
        static void Main(string[] args)
        {
            List<int>myList = new List<int>(){11, 12, 9, 14, 15};
            bool result = myList.All(i => i > 10); //All扩展方法，存在于Enumerable静态类当中的
            Console.WriteLine(result);
        }

        static bool AllGreaterThanTen(List<int> intList)
        {
            foreach(var item in intList)
            {
                if(item <= 10)
                {
                    return false;
                }
            }
            return ture;
        }
    }
}
```

**各种参数的使用场景**

- 传值参数：参数的默认传递方式
- 输出参数：用于除返回值外还需要输出的场景
- 引用参数：用于需要修改实际参数值的场景
- 数组参数：用于简化方法的调用
- 具名参数：提高可读性
- 可选参数：参数拥有默认值
- 扩展方法（this 参数）：为目标数据类型“追加"方法

## 委托

### 历史和本质-函数指针的升级版

什么是委托

- 委托（delegate）是函数指针的“升级版”

  - 实例：C/C++中的函数指针

- 一切皆地址

  - 变量（数据）是以某个地址为起点的一段内存中所存储的值

  - 函数（算法）是以某个地址为起点的一段内存中所储存的一组机器语言指令

- 直接调用与间接调用

  - 直接调用：通过函数名来调用函数，CPU 通过函数名直接获得函数所在地址并开始执行->返回

  - 间接调用：通过函数指针来调用函数，CPU 通过读取函数指针存储的值获得函数所在地址并开始执行->返回

- Java 中没有与委托相对应的功能实体

- 委托的简单使用

  - Action 委托

  - Func 委托

```c#
typedef int(* Calc)(int a, int b);

int Add(int a, int b)
{
    int result = a + b;
    return result;
}

int Sub(int a, int b)
{
    int result = a - b;
    return result;
}

int main()
{
    int x = 100;
    int y = 200;
    int z = 0;

    Calc = funcPoint1 = &Add;
    Calc = funcPoint2 = &Sub;

    z = funcPoint1(x, y);
    printf("%d+%d=%d\n", x, y, z);

    z = funcPoint2(x, y);
    printf("%d-%d=%d\n", x, y, z);

    system("pause");
    return 0;
}
```

```c#
namespace DelegateExample
{
    class Program
    {
        static void Main(string[] args)
        {
            Calculator calculator = new Calculator();
            Action action = new Action(calculator.Report);
            calculator.Report();
            action.Invoke();
            action();

            Func<int, int, int> func1 = new Func<int, int, int>(calculator.Add);
            Func<int, int, int> func2 = new Func<int, int, int>(calculator.Sub);
            
            int x = 100;
            int y = 200;
            int z = 0;
            
            z = func1.Invoke(x, y);// z = func1(x, y);
            Console.WriteLine(z);// 300
            z = func2,Invoke(x, y);// z = func2(x, y);
            Console.WriteLine(z);// -100
        }
    }

    class Calculator
    {
        public void Report()
        {
            Console.WriteLine("I have 3 methods.")
        }

        public int Add(int a, int b)
        {
            int result = a + b;
            return result;
        }

        public int Sub(int a, int b)
        {
            int result = a - b;
            return result;
        }
    }
}
```

### 自定义委托

- 委托是一种类（class），类是数据类型所以委托也是一种数据类型
- 它的声名方式与一般的类不同，主要是为了照顾可读性和C/C++传统
- 注意声名委托的位置
  - 避免写错地方结果声名成嵌套类型
- 委托与所封装的方法必须"类型兼容"  

![image-20250322104957705](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20250322104957705.png)


```c#
namespace DelegateExample
{
    public delegate double Calc(double x, double y);
    
    class Program
    {
        static void Main(string[] args)
        {
            Calculator calculator = new Calculator();
            Calc calc1 = new Calc(calculator.Add);
            Calc calc2 = new Calc(calculator.Sub);
            Calc calc3 = new Calc(calculator.Mul);
            Calc calc4 = new Calc(calculator.Div);
            
            double a = 100;
            double b = 200;
            double c = 0;
            
            c = calc1.Invoke(a, b);
            Console.WriteLine(c); //300
            c = calc2.Invoke(a, b);
            Console.WriteLine(c); //-100
            c = calc3.Invoke(a, b);
            Console.WriteLine(c); //20000
            c = calc4.Invoke(a, b);
            Console.WriteLine(c); //0.5            
        }
    }
    
    class Calculator
    {
        public double Add(double x, double y)
        {
            return x + y;
        }
        
        public double Sub(double x, double y)
        {
            return x - y;
        }
        
        public double Mul(double x, double y)
        {
            return x * y;
        }
        
        public double Div(double x, double y)
        {
            return x / y;
        }
    }
}
```

