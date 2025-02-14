---
draft: true
date:
  created: 2024-01-05
tags:
  - Rust Course
authors: [Tenax]
description: >
  Rust Course Chapter 2: Learning the Basics of the Rust Language Section 2.1 ~ 2.2 Notes
---

# Rust 语言圣经(Rust Course) 第二章 Rust 语言基础学习 I

_<!-- more -->_

## 2.1 变量绑定与解构

什么是 mut？

[Keyword mut](https://doc.rust-lang.org/std/keyword.mut.html)

[只使用 `_` 和使用以下划线开头的名称有些微妙的不同：比如 **`_x` 仍会将值绑定到变量，而 `_` 则完全不会绑定**。](https://course.rs/basic/match-pattern/all-patterns.html?highlight=_#%E4%BD%BF%E7%94%A8%E4%B8%8B%E5%88%92%E7%BA%BF%E5%BC%80%E5%A4%B4%E5%BF%BD%E7%95%A5%E6%9C%AA%E4%BD%BF%E7%94%A8%E7%9A%84%E5%8F%98%E9%87%8F)

[使用 `+=` 的赋值语句还不支持解构式赋值。](https://course.rs/basic/variable.html#%E8%A7%A3%E6%9E%84%E5%BC%8F%E8%B5%8B%E5%80%BC)

[在实际使用中，最好将程序中用到的硬编码值都声明为常量，对于代码后续的维护有莫大的帮助。如果将来需要更改硬编码的值，你也只需要在代码中更改一处即可。](https://course.rs/basic/variable.html#%E5%8F%98%E9%87%8F%E5%92%8C%E5%B8%B8%E9%87%8F%E4%B9%8B%E9%97%B4%E7%9A%84%E5%B7%AE%E5%BC%82)

[变量遮蔽(shadowing)](https://course.rs/basic/variable.html#变量遮蔽shadowing)

这和 `mut` 变量的使用是不同的，第二个 `let` 生成了完全不同的新变量，两个变量只是恰好拥有同样的名称，涉及一次内存对象的再分配 ，而 `mut` 声明的变量，可以修改同一个内存地址上的值，并不会发生内存对象的再分配，性能要更好。

---

[variables 练习](https://practice-zh.course.rs/variables.html)

```rust
// 修复错误
fn main() {
    println!("{}, world", x);
}

fn define_x() {
    let x = "hello";
}
```

两段代码的主要区别在于**返回值类型**和**变量生命周期**，具体分析如下：

第一段代码

```rust
fn main() {
    let x = define_x();
    println!("{}, world", x);
}

fn define_x() -> String {
    let x = "hello".to_string();
    x
}
```

解释：

- **返回值类型**：`define_x` 返回一个 `String` 类型。
  - `let x = "hello".to_string();` 将一个字符串字面量 `hello` 转换为 `String` 类型。`to_string()` 是一种方法，它会分配堆内存，并将该字符串存储在堆上，所以它的生命周期由 Rust 的所有权系统管理。
  - 返回的 `String` 是拥有自己的内存空间，可以在 `define_x` 返回后继续存在。
- **输出格式**：在 `main` 中调用 `define_x` 并打印输出，`println!("{}, world", x);` 使用 `{}` 占位符输出 `x`，`x` 被当作 `String` 来处理，`String` 实现了 `Display` trait，因此可以直接使用 `{}` 来输出。

```rust
fn main() {
    let x = define_x();
    println!("{:?}, world", x);
}

fn define_x() -> &'static str {
    let x = "hello";
    x
}
```

- **返回值类型**：`define_x` 返回一个字符串切片 `&'static str`。
  - `let x = "hello";` 这里 `x` 是一个字符串字面量（`'static str`）。字符串字面量在编译时就被加载到程序的静态内存区域，并且它的生命周期是 `'static`，即整个程序运行期间都有效。
  - 返回的 `x` 只是一个指向这个静态内存区域的引用（`&'static str`），而不是拥有它的所有权。
- **输出格式**：在 `main` 中调用 `define_x` 并打印输出，`println!("{:?}, world", x);` 使用 `{:?}` 占位符来输出 `x`。由于 `x` 是一个切片（`&str`），在输出时会使用 `Debug` trait 来输出，所以这里输出的是 `&"hello"`。

[答案](https://github.com/sunface/rust-by-practice/blob/master/solutions/variables.md)

---

## 2.2.1 数值类型

[浮点数陷阱](https://course.rs/basic/base-type/numbers.html#浮点数陷阱)

Rust 的 `HashMap` 数据结构，是一个 KV 类型的 Hash Map 实现，它对于 `K` 没有特定类型的限制，但是要求能用作 `K` 的类型必须实现了 `std::cmp::Eq` 特征，因此这意味着你无法使用浮点数作为 `HashMap` 的 `Key`，来存储键值对，但是作为对比，Rust 的整数类型、字符串类型、布尔类型都实现了该特征，因此可以作为 `HashMap` 的 `Key`。

为了避免上面说的两个陷阱，你需要遵守以下准则：

- 避免在浮点数上测试相等性
- 当结果在数学上可能存在未定义时，需要格外的小心

社区开发出高质量的 Rust 数值库：[num](https://crates.io/crates/num)。

---

## 2.2.2 字符、布尔、单元类型

没错， `main` 函数就返回这个[单元类型 `()`]()，你不能说 `main` 函数无返回值，因为没有返回值的函数在 Rust 中是有单独的定义的：`发散函数( diverge function )`，顾名思义，无法收敛的函数。

例如常见的 `println!()` 的返回值也是单元类型 `()`。

再比如，你可以用 `()` 作为 `map` 的值，表示我们不关注具体的值，只关注 `key`。 这种用法和 Go 语言的 **_struct{}_** 类似，可以作为一个值用来占位，但是完全**不占用**任何内存。

---

## 2.2.3 语句与表达式

语句会执行一些操作但是不会返回一个值，而表达式会在求值后返回一个值

由于 `let` 是语句，因此不能将 `let` 语句赋值给其它值，如下形式是错误的：

```rust
let b = (let a = 8);
```

[表达式如果不返回任何值，会隐式地返回一个 `()`https://course.rs/basic/base-type/char-bool.html#单元类型 。](https://course.rs/basic/base-type/statement-expression.html#%E8%A1%A8%E8%BE%BE%E5%BC%8F)

```rust
// 使用两种方法让代码工作起来
fn main() {
   let v = {
       let mut x = 1;
       x += 2
   };

   assert_eq!(v, 3);
}
```

为啥 x+=2 就不行 x+2 就行

感觉是 x += 2 已经返回赋值给了 x 然后 v 就不知道给什么了，x+2 是出来了 3 然后就返回赋值给 v

---

## 2.2.4 函数

函数要点

- 函数名和变量名使用[蛇形命名法(snake case)](https://course.rs/practice/naming.html)，例如 `fn add_two() -> {}`
- 函数的位置可以随便放，Rust 不关心我们在哪里定义了函数，只要有定义即可
- 每个函数参数都需要标注类型

例如单元类型 `()`，是一个零长度的元组。它没啥作用，但是可以用来表达一个函数没有返回值：

- 函数没有返回值，那么返回一个 `()`
- 通过 `;` 结尾的语句返回一个 `()`

### [永不返回的发散函数 `!`](https://course.rs/basic/base-type/function.html#永不返回的发散函数-)

当用 `!` 作函数返回类型的时候，表示该函数永不返回( diverge function )，特别的，这种语法往往用做会导致程序崩溃的函数：

```rust
fn dead_end() -> ! {
  panic!("你已经到了穷途末路，崩溃吧！");
}
```

下面的函数创建了一个无限循环，该循环永不跳出，因此函数也永不返回：

[发散函数( Diverging function )不会返回任何值，因此它们可以用于替代需要返回任何值的地方](https://practice-zh.course.rs/basic-types/functions.html)

```rust
fn forever() -> ! {
  loop {
    //...
  };
}
```

```rust

fn main() {
    println!("Success!");
}

fn get_option(tp: u8) -> Option<i32> {
    match tp {
        1 => {
            // TODO
        }
        _ => {
            // TODO
        }
    };

    // 这里与其返回一个 None，不如使用发散函数替代
    never_return_fn()
}

// 使用三种方法实现以下发散函数
fn never_return_fn() -> ! {

}
```

```rust
fn never_return_fn() -> ! {
    unimplemented!()
}
```

```rust
// IMPLEMENT this function in THREE ways
fn never_return_fn() -> ! {
    panic!()
}
```

```rust
// IMPLEMENT this function in THREE ways
fn never_return_fn() -> ! {
    todo!();
}
```

```rust
// IMPLEMENT this function in THREE ways
fn never_return_fn() -> ! {
    loop {
        std::thread::sleep(std::time::Duration::from_secs(1))
    }
}
```

unimplement!()：未实现（not implemented）
todo!()：尚未被实现（not yet implemented）
两个宏函数都可以导致 panic，但是这两个方法的语义不同。

```rust
fn main() {
    // 填空
    let b = __;

    let _v = match b {
        true => 1,
        // 发散函数也可以用于 `match` 表达式，用于替代任何类型的值
        false => {
            println!("Success!");
            panic!("we have no value for `false`, but we can panic")
        }
    };

    println!("Exercise Failed if printing out this line!");
}
```

```rust
fn main() {
    // FILL in the blank
    let b = false;

    let _v = match b {
        true => 1,
        // Diverging functions can also be used in match expression
        false => {
            println!("Success!");
            panic!("we have no value for `false`, but we can panic")
        }
    };

    println!("Exercise Failed if printing out this line!");
}
```

这个分支选择有点意思

---

**参考:**

[基本类型](https://course.rs/basic/base-type/index.html#基本类型)

[全模式列表](https://course.rs/basic/match-pattern/all-patterns.html?highlight=_#全模式列表)

[数值类型](https://course.rs/basic/base-type/numbers.html#数值类型)

[附录 B：运算符与符号](https://course.rs/appendix/operators.html#附录-b运算符与符号)

[类型转换](https://course.rs/advance/into-types/converse.html#类型转换)
