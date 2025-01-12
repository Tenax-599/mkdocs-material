---
date:
  created: 2024-12-31
tags:
  - Rust Course
authors: [Tenax]
description: >
  Rust Course Chapter 1: Learning the Basics of the Rust Language Section 1 Notes
---

# Rust 语言圣经(Rust Course) 第一章 Rust 语言基础学习 I

_<!-- more -->_

```rust
fn main() {
   let penguin_data = "\
   common name,length (cm)
   Little penguin,33
   Yellow-eyed penguin,65
   Fiordland penguin,60
   Invalid,data
   ";

   let records = penguin_data.lines();

   for (i, record) in records.enumerate() {
     if i == 0 || record.trim().len() == 0 {
       continue;
     }

     // 声明一个 fields 变量，类型是 Vec
     // Vec 是 vector 的缩写，是一个可伸缩的集合类型，可以认为是一个动态数组
     // <_>表示 Vec 中的元素类型由编译器自行推断，在很多场景下，都会帮我们省却不少功夫
     let fields: Vec<_> = record
       .split(',')
       .map(|field| field.trim())
       .collect();
     if cfg!(debug_assertions) {
         // 输出到标准错误输出
       eprintln!("debug: {:?} -> {:?}",
              record, fields);
     }

     let name = fields[0];
     // 1. 尝试把 fields[1] 的值转换为 f32 类型的浮点数，如果成功，则把 f32 值赋给 length 变量
     //
     // 2. if let 是一个匹配表达式，用来从=右边的结果中，匹配出 length 的值：
     //   1）当=右边的表达式执行成功，则会返回一个 Ok(f32) 的类型，若失败，则会返回一个 Err(e) 类型，if let 的作用就是仅匹配 Ok 也就是成功的情况，如果是错误，就直接忽略
     //   2）同时 if let 还会做一次解构匹配，通过 Ok(length) 去匹配右边的 Ok(f32)，最终把相应的 f32 值赋给 length
     //
     // 3. 当然你也可以忽略成功的情况，用 if let Err(e) = fields[1].parse::<f32>() {...}匹配出错误，然后打印出来，但是没啥卵用
     if let Ok(length) = fields[1].parse::<f32>() {
         // 输出到标准输出
         println!("{}, {}cm", name, length);
     }
   }
 }
```

## **为什么会有`penguin_data`中会有反斜杠\?**

这里的反斜杠 `\` 是用来告诉 Rust 编译器这个字符串是一个跨多行的字符串字面量。反斜杠本身会“消除”换行符，因此字符串字面量的内容在内存中是一个连续的文本行，不会存在实际的换行符。

没有反斜杠

```rust
Lines(Map { iter: SplitInclusive { 0: SplitInternal { start: 0, end: 124, matcher: CharSearcher { haystack: "\n    common name,length (cm)\n    Little penguin,33\n    Yellow-eyed penguin,65\n    Fiordland penguin,60\n    Invalid,data\n    ", finger: 0, finger_back: 124, needle: '\n', utf8_size: 1, utf8_encoded: [10, 0, 0, 0] }, allow_trailing_empty: false, finished: false } } })
```

有反斜杠

```rust
Lines(Map { iter: SplitInclusive { 0: SplitInternal { start: 0, end: 119, matcher: CharSearcher { haystack: "common name,length (cm)\n    Little penguin,33\n    Yellow-eyed penguin,65\n    Fiordland penguin,60\n    Invalid,data\n    ", finger: 0, finger_back: 119, needle: '\n', utf8_size: 1, utf8_encoded: [10, 0, 0, 0] }, allow_trailing_empty: false, finished: false } } })
```

可以看出没有反斜杠`haystack`开头会出现一个\n

## **lines()方法？做了什么处理？**

打印 penguin_data

```rust
"common name,length (cm)\n    Little penguin,33\n    Yellow-eyed penguin,65\n    Fiordland penguin,60\n    Invalid,data\n    "
```

打印 records

```rust
Lines(Map { iter: SplitInclusive { 0: SplitInternal { start: 0, end: 119, matcher: CharSearcher { haystack: "common name,length (cm)\n    Little penguin,33\n    Yellow-eyed penguin,65\n    Fiordland penguin,60\n    Invalid,data\n    ", finger: 0, finger_back: 119, needle: '\n', utf8_size: 1, utf8_encoded: [10, 0, 0, 0] }, allow_trailing_empty: false, finished: false } } })
```

lines()在文档中的解释：

以字符串切片的形式返回字符串各行的迭代器。

在换行符（\n）或回车符后换行符（\r\n）序列的行结束符处分行。

迭代器返回的行中不包括行结束符。

请注意，任何回车（\r）后没有立即换行（\n），都不会分行。因此，这些回车符也包含在生成的行中。

最后一行的结尾是可选的。以最后行结束符结束的字符串将返回与没有最后行结束符的相同字符串相同的行。

（看着就是套了一圈结构体）

## **enumerate()枚举？是进行了怎样的处理？有什么改变？**

```rust
Enumerate { iter: Lines(Map { iter: SplitInclusive { 0: SplitInternal { start: 0, end: 119, matcher: CharSearcher { haystack: "common name,length (cm)\n    Little penguin,33\n    Yellow-eyed penguin,65\n    Fiordland penguin,60\n    Invalid,data\n    ", finger: 0, finger_back: 119, needle: '\n', utf8_size: 1, utf8_encoded: [10, 0, 0, 0] }, allow_trailing_empty: false, finished: false } } }), count: 0 }
```

可以看出变化为外部套了个`Enumerate{, count:0}`

```rust
pub struct Enumerate<I> {
    iter: I,
    count: usize,
}
```

## **split()方法？看看做了什么处理**

```rust
println!("{:#?}", record.split(","));
```

```rust
Split(
    SplitInternal {
        start: 0,
        end: 21,
        matcher: StrSearcher {
            haystack: "    Little penguin,33",
            needle: ",",
            searcher: TwoWay(
                TwoWaySearcher {
                    crit_pos: 0,
                    crit_pos_back: 1,
                    period: 1,
                    byteset: 17592186044416,
                    position: 0,
                    end: 21,
                    memory: 0,
                    memory_back: 1,
                },
            ),
        },
        allow_trailing_empty: true,
        finished: false,
    },
)
Split(
    SplitInternal {
        start: 0,
        end: 26,
        matcher: StrSearcher {
            haystack: "    Yellow-eyed penguin,65",
            needle: ",",
            searcher: TwoWay(
                TwoWaySearcher {
                    crit_pos: 0,
                    crit_pos_back: 1,
                    period: 1,
                    byteset: 17592186044416,
                    position: 0,
                    end: 26,
                    memory: 0,
                    memory_back: 1,
                },
            ),
        },
        allow_trailing_empty: true,
        finished: false,
    },
)
Split(
    SplitInternal {
        start: 0,
        end: 24,
        matcher: StrSearcher {
            haystack: "    Fiordland penguin,60",
            needle: ",",
            searcher: TwoWay(
                TwoWaySearcher {
                    crit_pos: 0,
                    crit_pos_back: 1,
                    period: 1,
                    byteset: 17592186044416,
                    position: 0,
                    end: 24,
                    memory: 0,
                    memory_back: 1,
                },
            ),
        },
        allow_trailing_empty: true,
        finished: false,
    },
)
Split(
    SplitInternal {
        start: 0,
        end: 16,
        matcher: StrSearcher {
            haystack: "    Invalid,data",
            needle: ",",
            searcher: TwoWay(
                TwoWaySearcher {
                    crit_pos: 0,
                    crit_pos_back: 1,
                    period: 1,
                    byteset: 17592186044416,
                    position: 0,
                    end: 16,
                    memory: 0,
                    memory_back: 1,
                },
            ),
        },
        allow_trailing_empty: true,
        finished: false,
    },
)
```

对比之前是把`SplitInternal`取了出来增加了`matcher` `allow_trailing_empty` `finished`

当我尝试

```rust
let fields: Vec<_> = record.split(',');
```

会报错

```rust
mismatched types
expected struct `Vec<_>`
   found struct `std::str::Split<'_, char>`
```

显然`Vec<_>`并不接受这种形式的结构

## **map()方法？看看做了什么处理**

```rust
Map {
    iter: Split(
        SplitInternal {
            start: 0,
            end: 21,
            matcher: StrSearcher {
                haystack: "    Little penguin,33",
                needle: ",",
                searcher: TwoWay(
                    TwoWaySearcher {
                        crit_pos: 0,
                        crit_pos_back: 1,
                        period: 1,
                        byteset: 17592186044416,
                        position: 0,
                        end: 21,
                        memory: 0,
                        memory_back: 1,
                    },
                ),
            },
            allow_trailing_empty: true,
            finished: false,
        },
    ),
}
Map {
    iter: Split(
        SplitInternal {
            start: 0,
            end: 26,
            matcher: StrSearcher {
                haystack: "    Yellow-eyed penguin,65",
                needle: ",",
                searcher: TwoWay(
                    TwoWaySearcher {
                        crit_pos: 0,
                        crit_pos_back: 1,
                        period: 1,
                        byteset: 17592186044416,
                        position: 0,
                        end: 26,
                        memory: 0,
                        memory_back: 1,
                    },
                ),
            },
            allow_trailing_empty: true,
            finished: false,
        },
    ),
}
Map {
    iter: Split(
        SplitInternal {
            start: 0,
            end: 24,
            matcher: StrSearcher {
                haystack: "    Fiordland penguin,60",
                needle: ",",
                searcher: TwoWay(
                    TwoWaySearcher {
                        crit_pos: 0,
                        crit_pos_back: 1,
                        period: 1,
                        byteset: 17592186044416,
                        position: 0,
                        end: 24,
                        memory: 0,
                        memory_back: 1,
                    },
                ),
            },
            allow_trailing_empty: true,
            finished: false,
        },
    ),
}
Map {
    iter: Split(
        SplitInternal {
            start: 0,
            end: 16,
            matcher: StrSearcher {
                haystack: "    Invalid,data",
                needle: ",",
                searcher: TwoWay(
                    TwoWaySearcher {
                        crit_pos: 0,
                        crit_pos_back: 1,
                        period: 1,
                        byteset: 17592186044416,
                        position: 0,
                        end: 16,
                        memory: 0,
                        memory_back: 1,
                    },
                ),
            },
            allow_trailing_empty: true,
            finished: false,
        },
    ),
}
```

map()在文档中的解释：

较之前，外面套了一层 Map {iter:}

获取一个闭包并创建一个迭代器，在每个元素上调用该闭包。

map() 通过其参数将一个迭代器转换成另一个迭代器：实现 FnMut 的东西。它会产生一个新的迭代器，在原始迭代器的每个元素上调用这个闭包。

如果你擅长用类型来思考，你可以这样来理解 map()： 如果你有一个迭代器，它提供了某种类型 A 的元素，而你想要一个其他类型 B 的迭代器，你可以使用 map()，传递一个接收 A 并返回 B 的闭包。

map() 在概念上类似于 for 循环。不过，由于 map() 是懒惰的，所以最好在已经使用其他迭代器的情况下使用。如果你正在为某种副作用进行某种循环，那么使用 for 比使用 map() 更习惯。

## **|field| field.trim()为什么这样写?||又是什么意思?**

闭包在 Rust 中可以通过多种方式定义，包括使用 `move`、`||` 和显式的函数签名等方式。这里的 `|field| field.trim()` 是最常见的简写方式。

- `||` 是闭包参数的分隔符。
- `|field|` 中的 `field` 是闭包的单个输入参数。你也可以有多个参数，如 `|a, b|`。

## **collect()方法**

map()在文档中的解释：

将迭代器转换为集合。

collect() 可以接收任何可迭代的数据，并将其转化为相关的集合。这是标准库中功能更强大的方法之一，可用于多种情况。

使用 collect() 的最基本模式是将一个集合转化为另一个集合。你可以获取一个集合，调用 [iter] 对其进行一系列转换，然后在最后使用 collect()。

collect() 还可以创建非典型集合类型的实例。例如，可以用 [char]s 创建一个字符串，也可以将一个 Result<T, E> 项的迭代器收集到 Result<Collection<T>, E> 中。更多信息，请参阅下面的示例。

由于 collect() 非常通用，可能会导致类型推断问题。因此，collect() 是你能看到的为数不多的被亲切地称为 '涡轮鱼'的语法之一: ::<>。这可以帮助推理算法具体理解你要收集到哪个集合中。

## 总结

Rust 编译器非常强大，在函数处理的过程中能明显感觉到需要很多算法的处理，编译器内置 linter 也是十分的强大
