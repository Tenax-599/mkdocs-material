---
draft: true
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
