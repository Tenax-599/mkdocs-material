---
date:
  created: 2025-03-26
tags:
  - T-SQL
authors: [Tenax]
description: >
  Learning Transact-SQL
---

# Transact-SQL 笔记

<!-- more -->

## 介绍

### [关系数据](https://learn.microsoft.com/zh-cn/training/modules/introduction-to-transact-sql/1-introduction)

SQL 最常（但并不总是）用来查询关系数据库中的数据。 关系数据库是一种已将数据组织到多个表中（技术上称为“关系”）的数据库，其中每个表代表一种特定类型的实体（例如客户、产品或销售订单）。 这些实体的属性（例如客户名称、产品价格或销售订单的订单日期）被定义为表的列或属性，而表中的每一行代表一个实体类型实例（例如特定客户、产品或销售订单）。

数据库中的表通过键列相互关联，键列唯一标识所代表的特定实体。 为每个表定义一个主键，并将任意相关表中对此键的引用定义为外键。 通过示例可以更轻松地了解这些信息：

![一个包含四个表的关系数据库](../assets/T-SQL/relational-database.avif)

该图显示了一个包含四个表的关系数据库：

- **客户**
- **SalesOrderHeader**
- **SalesOrderDetail**
- **产品**

每个客户由唯一的 CustomerID 字段标识 - 此字段是“客户”表的主键。 SalesOrderHeader 表中包含标识每个订单的主键，名为 OrderID，还包括引用“客户”表中主键的外键 CustomerID，它用于标识哪个客户与哪个订单相关联。 订单中各项的数据存储在 SalesOrderDetail 表中，它的主键是复合主键，即将 SalesOrderHeader 表中的 OrderID 与 LineItemNo 值结合在了一起。 这些值的组合用于唯一标识行项。 此外，“OrderID”字段用作外键来指示行项所属的订单，“ProductID”字段用作“产品”表中的 ProductID 主键的外键，指示订购的哪款产品。



## 探索 SQL 语句的结构

在任何 SQL 方言中，SQL 语句都会组合成几种不同类型的语句。 这些不同类型包括：

- 数据操作语言 (DML) 是侧重于查询和修改数据的 SQL 语句集。 DML 语句包括 SELECT（本训练的主要重点）以及 INSERT、UPDATE 和 DELETE 等修改语句。
- 数据定义语言 (DDL) 是处理数据库对象（例如表、视图和过程）定义和生命周期的 SQL 语句集。 DDL 包括 CREATE、ALTER 和 DROP 等语句。
- 数据控制语言 (DCL) 是用于管理用户和对象安全权限的 SQL 语句集。 DCL 包括 GRANT、REVOKE 和 DENY 等语句。

有时，可能还会看到 TCL 被列为语句类型，这是指事务控制语言。 此外，有些列表可能会将 DML 重新定义为“数据修改语言”，其中不包括 SELECT 语句，但随后会添加数据查询语言 (DQL) 用于 SELECT 语句。

在此模块中，我们将重点介绍 DML 语句。 数据分析师通常使用这些语句来检索用于报表和分析的数据。 应用程序开发人员也会使用 DML 语句来执行“CRUD”操作，以创建、读取、更新或删除应用程序数据。

**[什么是复合主键](https://en.wikipedia.org/wiki/Composite_key)**

In database design, a composite key is a candidate key that consists of two or more attributes, (table columns) that together uniquely identify an entity occurrence (table row).【在数据库设计中，复合键是一种候选键，由两个或多个属性（表列）组成，共同唯一标识一个实体（表行）。】

A compound key is a composite key for which each attribute that makes up the key is a foreign key in its own right.【复合键是一种复合键，其组成该键的每个属性本身就是一个外键。】

<u>唯一标识行？</u>



## [检查 SELECT 语句](https://learn.microsoft.com/zh-cn/sql/t-sql/queries/select-transact-sql?view=sql-server-ver16)

```sql
SELECT OrderDate, COUNT(OrderID) AS Orders
FROM Sales.SalesOrder
WHERE Status = 'Shipped'
GROUP BY OrderDate
HAVING COUNT(OrderID) > 1
ORDER BY OrderDate DESC;
```

现在，你已了解每个子句的作用，接着我们来看 SQL Server 对它们的实际计算顺序：

1. 首先计算 FROM 子句，为其余语句提供源行。 这里将<u>创建一个虚拟表</u>并将其传递到下一步。
2. 接着，计算 WHERE 子句后，从源表中筛选出与谓词匹配的行。 将筛选后的虚拟表传递到下一步。
3. 然后计算 GROUP BY，根据 GROUP BY 列表中的唯一值来组织虚拟表中的行。 这时将创建一个包含组列表的新虚拟表，并将其传递到下一步。 从操作流的此刻开始，其他元素只能引用 GROUP BY 列表中的列或聚合函数。
4. 接下来，计算 HAVING 子句，根据谓词筛选出整个组。 筛选在步骤 3 中创建的虚拟表，并将其传递到下一步。
5. 最后执行 SELECT 子句，确定查询结果中显示哪些列。 由于 SELECT 子句在其他步骤之后计算，因此这时创建的任何列别名（在本例中为 Orders）都无法用于 GROUP BY 或 HAVING 子句。
6. 最后一步执行 ORDER BY，按列列表对确定的行排序。

要将这种理解应用于示例查询，下面是上述 SELECT 语句运行时的逻辑顺序：

```sql
FROM Sales.SalesOrder
WHERE Status = 'Shipped'
GROUP BY OrderDate
HAVING COUNT(OrderID) > 1
SELECT OrderDate, COUNT(OrderID) AS Orders
ORDER BY OrderDate DESC;
```

每个写入的 SELECT 语句并非都需要所有可能的子句。 唯一必需的子句是 SELECT 子句，它们在某些情况下可以单独使用。 通常还会包括 FROM 子句，用于标识要查询的表。 此外，Transact-SQL 还包含其他可添加的子句。

**如你所见，写入 T-SQL 查询的顺序与其计算的逻辑顺序并不相同。 计算的运行时顺序决定哪些数据可用于哪些子句，因为子句仅有权访问已处理子句提供的信息。 因此，在写入查询时了解真正的逻辑处理顺序非常重要。**



### 选择所有列

SELECT 子句通常称为 SELECT 列表，因为它会列出要在查询结果中返回的值。

最简单的 SELECT 子句形式是使用星号字符 (_) 返回所有列。 在 T-SQL 查询中使用时，它被称为“星型”。 尽管 SELECT _ 适合用于快速测试，但应避免在生产工作中使用它们，原因如下：

- 对表进行的添加列或重新排列列等更改会反映在查询结果中，进而可能导致使用该查询的应用程序或报表出现输出意外。
- 如果源表包含大量行，则返回不需要的数据可能会减慢查询速度并导致性能问题。



### 格式化查询

通过此部分的示例，你可能会注意到查询代码的格式可以灵活设置。 例如，可以每行写入一个子句（或整个查询），也可以将其拆分成多行写入。 在大多数数据库系统中，代码不区分大小写，并且有些 T-SQL 语言元素是可选项（包括前面提到的 AS 关键字，甚至是语句末尾的分号）。

请考虑以下准则，让 T-SQL 代码容易阅读（从而更便于理解和调试！）：

- 将 T-SQL 关键字（如 SELECT、FROM、AS 等）大写。 大写关键字是一种常用约定，这样查找复杂语句的每个子句会变得更加轻松。
- 另起新的一行写入语句的每个主子句。
- 如果 SELECT 列表不止有几列、几个表达式或别名，请考虑在每行列出其各列。
- 缩进包含子子句或列的行，以明确每个代码属于哪个主子句。



## 使用数据类型

### 数据类型转换

兼容的数据类型值可按要求隐式转换。 例如，假设可以使用 **+** 运算符向十进制数添加整数，或连接固定长度的 char 值和可变长度的 varchar 值。 但是，在某些情况下，可能需要将值从一种数据类型显式转换为另一种数据类型 - 例如，尝试使用 **+** 连接 varchar 值和十进制值将会出错，除非先将数值转换为兼容的字符串数据类型。


??? info "备注"

    隐式和显式转换适用于某些数据类型，有些转换是不可能的。 有关详细信息，请使用 [Transact-SQL 参考文档](https://learn.microsoft.com/zh-cn/sql/t-sql/data-types/data-type-conversion-database-engine)中的图表。



## T-SQL 包含有助于显式转换数据类型的函数

### CAST 和 TRY_CAST

CAST 函数可将值转换为指定的数据类型，但该值必须与目标数据类型兼容。 如果不兼容，将返回错误。

例如，以下查询使用 CAST 将 ProductID 列中的整数值转换为 varchar 值（最多为 4 个字符），以便将它们与其他基于字符的值连接：

```sql
SELECT CAST(ProductID AS varchar(4)) + ': ' + Name AS ProductName
FROM Production.Product;

/*
该查询的结果可能如下所示：

ProductName

680：HL 公路车架 - 黑色，58

706：HL 公路车架 - 红色，58

707：Sport-100 头盔，红色

708：Sport-100 头盔，黑色

...
*/
```

但是，假设 Production.Product 表中的“大小”列是 nvarchar（可变长度，Unicode 文本数据）列，其中包含一些数值大小（例如 58）和一些基于文本的大小（例如“S”、“M”或“L”）。 以下查询尝试将此列中的值转换为“整数”数据类型：

```sql
SELECT CAST(Size AS integer) As NumericSize
FROM Production.Product;
```

该查询将导致以下错误消息：

> 错误：将 nvarchar 值“M”转换为 int 数据类型时转换失败。

假设该列中至少有一些值是数值，你可能希望转换这些值，并忽略其他值。 可以使用 TRY_CAST 函数来转换数据类型。

```sql
SELECT TRY_CAST(Size AS integer) As NumericSize
FROM Production.Product;

/*
这次的结果可能如下所示：

NumericSize

58

58

Null

Null

...
*/
```

对于可转换为数值数据类型的值，将返回十进制值；对于不兼容的值，将返回 NULL，用于指示该值未知。



### CONVERT 和 TRY_CONVERT

CAST 是 ANSI 标准 SQL 函数，用于转换数据类型，适用于许多数据库系统。 在 Transact-SQL 中还可以使用 CONVERT 函数，如下所示：

```sql
SELECT CONVERT(varchar(4), ProductID) + ': ' + Name AS ProductName
FROM Production.Product;

/*
同样，此查询将返回转换为指定数据类型的值，如下所示：

ProductName

680：HL 公路车架 - 黑色，58

706：HL 公路车架 - 红色，58

707：Sport-100 头盔，红色

708：Sport-100 头盔，黑色

...
*/
```

同 CAST 一样，CONVERT 包含 TRY_CONVERT 变量，对于不兼容的值将返回 NULL。

相比 CAST，使用 CONVERT 的另一点好处是 CONVERT 还包括一个参数，该参数支持在将数值和日期值转换为字符串时指定格式样式。 例如，考虑以下查询：

```sql
SELECT SellStartDate,
       CONVERT(varchar(20), SellStartDate) AS StartDate,
       CONVERT(varchar(10), SellStartDate, 101) AS FormattedStartDate 
FROM SalesLT.Product;

/*
该查询的结果可能如下所示：
SellStartDate          StartDate             FormattedStartDate
2002-06-01T00:00:00    Jun 1 2002 12:00AM    6/1/2002
2002-06-01T00:00:00    Jun 1 2002 12:00AM    6/1/2002
2005-07-01T00:00:00    Jul 1 2005 12:00AM    2005/7/1
2005-07-01T00:00:00    Jul 1 2005 12:00AM    2005/7/1
...                    ...                   ...
*/
```

??? info "备注"

    隐式和显式转换适用于某些数据类型，有些转换是不可能的。 有关详细信息，请使用 [Transact-SQL 参考文档](https://learn.microsoft.com/zh-cn/sql/t-sql/functions/cast-and-convert-transact-sql?view=sql-server-ver16)中的图表。



### PARSE 和 TRY_PARSE

PARSE 函数旨在转换代表数值或日期/时间值的格式化字符串。 例如，请考虑以下查询（使用文本值，而不是表中列的值）：

```sql
SELECT PARSE('01/01/2021' AS date) AS DateValue,
   PARSE('$199.99' AS money) AS MoneyValue;
   
/*
该查询的结果如下所示：

DateValue                MoneyValue

2021-01-01T00:00:00      199.99
*/
```

同 CAST 和 CONVERT 类似，PARSE 包含一个 TRY_PARSE 变量，对于不兼容的值将返回 NULL。



### [STR](https://learn.microsoft.com/zh-cn/sql/t-sql/functions/str-transact-sql?view=sql-server-ver16)

STR 函数可将数值转换为 varchar。

例如：

```sql
SELECT ProductID,  '$' + STR(ListPrice) AS Price
FROM Production.Product;

/*
结果将如下所示：

ProductID                价格

680                      $1432.00

706                      $1432.00

707                      $35.00

...                      ...
*/

/* 注意
STR(RevisionNumber, 1)
SO71895 (2)                   2008.06.01

STR(RevisionNumber)
SO71938 (         2)                   2008.06.01
```



## 处理 NULL

NULL 值表示“无值”或“未知”。 不是指零或空白，也不表示空字符串。 这些值并不是未知。 NULL 值可用于表示尚未提供的值，例如客户尚未提供电子邮件地址时。 如前文所述，当某个值与目标数据类型不兼容时，一些转换函数也可能返回 NULL 值。

要处理 NULL，通常需要采取特殊的步骤。 NULL 实际上是非值。 它是未知的。 不等于任何内容，也等于任何内容。 NULL 不大于或小于任何内容。 对于它是什么，我们一无所解，但有时我们需要处理 NULL 值。 令人欣慰的是，T-SQL 提供用于转换或替换 NULL 值的函数。



### ISNULL

COUNTX 函数有两个参数。 第一个是我们要测试的表达式。 如果第一个参数为 NULL，则该函数将返回第二个参数。 如果第一个表达式不是 NULL，则返回不变的值。

例如，假设数据库中的 Sales.Customer 表包含一个允许 NULL 值的 MiddleName 列。 查询此表时，可以选择返回特定值（如“无”），而不是在结果中返回 NULL。

```sql
SELECT FirstName,
      ISNULL(MiddleName, 'None') AS MiddleIfAny,
      LastName
FROM Sales.Customer;

/*
FirstName            MiddleIfAny          LastName
奥兰多                N.                   Gee
Keith                无                   Howard
...                  ...                  ...
*/
```

**替换 NULL 的值必须与要计算的表达式的数据类型相同。 在以上示例中，MiddleName 是 varchar 类型，因此替换值不能是数值。 此外，需要选择不会作为常规值在数据中显示的值。 有时，可能很难找到永远不会出现在数据中的值。**

前面示例处理了源表中的 NULL 值，但可以对可能返回 NULL 的任何表达式使用 ISNULL，包括在 ISNULL 函数中嵌套 TRY_CONVERT 函数。

```sql
SELECT OrderID,
       ISNULL(TRY_CONVERT(decimal(10, 2), Amount), 0) AS ValidAmount
FROM Sales.Order;

/*
示例：
假设 Sales.Order 表中的数据如下：

OrderID	Amount
1	100.50
2	NULL
3	'Invalid'
4	150.75
执行上述查询后，你会得到：

OrderID	ValidAmount
1	100.50
2	0.00
3	0.00
4	150.75
*/
```



### COALESCE

ISNULL 函数不是 ANSI 标准函数，因此你可能希望改为使用 COALESCE 函数。 COALESCE 稍微灵活一些，它可以采用可变数量的参数，每个参数都是一个表达式。 <u>它将返回列表中第一个不为 NULL 的表达式</u>。

如果只有两个参数，则 COALESCE 的行为与 ISNULL 类似。 但是，对于两个以上参数，COALESCE 可以用作使用 ISNULL 的多部分 CASE 表达式的替代项。

如果所有参数均为 NULL，则 COALESCE 返回 NULL。 所有表达式必须返回相同或兼容的数据类型。

语法如下：

```sql
SELECT COALESCE ( expression1, expression2, [ ,...n ] )
```

以下示例使用一个名为 HR.Wages 的虚构表，其中包括三列有关雇员每周收益的信息：每小时费率、每周工资和每单位销售的佣金。 但是，每个雇员只能接受一种付款方式。 对于每位雇员，三列中有一列会包含一个值，其他两列将为 NULL。 要确定支付给每位雇员的总金额，可以使用 COALESCE 仅返回在这三列中找到的非 null 值。

```sql
SELECT EmployeeID,
      COALESCE(HourlyRate * 40,
                WeeklySalary,
                Commission * SalesQty) AS WeeklyEarnings
FROM HR.Wages;

/*
结果可能如下所示：

EmployeeID             WeeklyEarnings

1                      899.76

2                      1001.00

3                      1298.77

...                    ...
*/
```



### NULLIF

NULLIF 函数允许在某些条件下返回 NULL。 在数据清理等领域想要将空白或占位符字符替换为 NULL 时，该函数十分有用。

NULLIF 采用两个参数，如果它们等效，则返回 NULL。 如果它们不相等，则 NULLIF 将返回第一个参数。

在此示例中，NULLIF 将折扣 0 替换成了 NULL。 如果折扣值不是 0，则返回该折扣值：

```sql
SELECT SalesOrderID,
      ProductID,
      UnitPrice,
      NULLIF(UnitPriceDiscount, 0) AS Discount
FROM Sales.SalesOrderDetail;

/*
结果可能如下所示：

销售订单 ID         ProductID          单价            折扣

71774              836                356.898        Null

71780              988                112.998        0.4               

71781              748                818.7          Null

71781              985                112.998        0.4

...                ...                ...            ...
```



## 插入数据

### INSERT 语句

INSERT 语句用于向表中添加一行或多行。 语句的形式有数种。

简单 INSERT 语句的基本语法如下所示：

```sql
INSERT [INTO] <Table> [(column_list)]
VALUES ([ColumnName or an expression or DEFAULT or NULL],…n)
```

利用这种形式的 INSERT 语句（称为“插入值”），可以指定将在其中放置值的列以及为插入表中的每一行显示数据的顺序。 column_list 是可选的，但建议使用。 在没有 column_list 的情况下，INSERT 语句对表中的每一列都期望一个值，顺序与定义列的顺序相同。 还可以作为逗号分隔列表来提供这些列的值。

列出值时，关键字 DEFAULT 表示将使用某个预定义值（表创建表时指定）。 可通过三种方式确定默认值：

- 如果列已定义为具有自动生成的值，则将使用该值。 本模块后续部分中将讨论自动生成值。
- 创建表时，可以为列提供默认值，如果指定了 DEFAULT，则使用该默认值。
- 如果列已定义为允许 NULL 值，并且该列不是自动生成的列，且没有定义默认值，则 NULL 将作为 DEFAULT 插入。

表创建的细节超出了本模块的范围。 但是，查看表中有哪些列通常很有用。 最简单的方法就是对表执行 SELECT 语句，而不返回任何行。 通过使用永远不会返回 TRUE 值的 WHERE 条件，不能返回任何行。

| PromotionName | StartDate | ProductModelID | 折扣 | 说明 |
| ------------- | --------- | -------------- | ---- | ---- |

```sql
INSERT INTO Sales.Promotion (PromotionName,StartDate,ProductModelID,Discount,Notes)
VALUES
('Clearance Sale', '01/01/2021', 23, 0.1, '10% discount');
-- 对于上面的示例，可以省略列清单，因为我们按正确的顺序为每列提供一个值：

-- 假设表定义为：当前日期的默认值应用于 StartDate 列，并且 Notes 列允许 NULL 值。 可以明确指示想要使用这些值，如下所示：

INSERT INTO Sales.Promotion
VALUES
('Pull your socks up', DEFAULT, 24, 0.25, NULL);

/*或者，可以省略 INSERT 语句中的值，如果省略，则使用默认值（如果已定义），如果没有默认值，但列允许 NULL 值，则将插入 NULL。 如果没有为所有列提供值，则必须包含列清单，指出为哪些列提供了值。*/

INSERT INTO Sales.Promotion (PromotionName, ProductModelID, Discount)
VALUES
('Caps Locked', 2, 0.2);

/* INSERT VALUES 语句除了能够一次插入一行之外，还可用于插入多行，方法是提供多个逗号分隔的值集。 值集也用逗号分隔，如下所示：
(col1_val,col2_val,col3_val),
(col1_val,col2_val,col3_val)
*/
-- 这一值列表称为表值构造函数。 下面的示例使用表值构造函数将另外两行插入表中：

INSERT INTO Sales.Promotion
VALUES
('The gloves are off!', DEFAULT, 3, 0.25, NULL),
('The gloves are off!', DEFAULT, 4, 0.25, NULL);
```



### INSERT ... SELECT

除了在 INSERT 语句中指定一组文本值以外，T-SQL 还支持使用其他运算的结果来提供 INSERT 的值。 可以使用 SELECT 语句的结果或存储过程的输出来提供 INSERT 语句的值。

要将 INSERT 与嵌套 SELECT 一起使用，请生成一个 SELECT 语句来替换 VALUES 子句。 通过这种格式（称为 INSERT SELECT），<u>可将 SELECT 查询返回的行集插入到目标表中</u>。 使用 INSERT SELECT 时，将显示与 INSERT VALUES 相同的注意事项：

- 可以选择在表名称后面指定列清单。
- 必须为每个列提供列值或默认值，或者提供 DEFAULT 或 NULL。

以下语法说明了 INSERT SELECT 的用法：

以下语法说明了 INSERT SELECT 的用法：

```sql
INSERT [INTO] <table or view> [(column_list)]
SELECT <column_list> FROM <table_list>...;
```

**也可以将来自存储过程（甚至动态批处理）的结果集用作 INSERT 语句的输入。 这种格式的 INSERT（称为 INSERT EXEC）在概念上类似于 INSERT SELECT，并且将显示相同的注意事项。 但是，存储过程可能会返回多个结果集，因此需要格外小心。**

```sql
-- 以下示例将为名为 Get Framed 的新促销插入多行，方法是：从 Production.ProductModel（这是一个表，列出了名称中包含“frame”的所有模型）中检索模型 ID 和模型名称。

INSERT INTO Sales.Promotion (PromotionName, ProductModelID, Discount, Notes)
SELECT DISTINCT 'Get Framed', m.ProductModelID, 0.1, '10% off ' + m.Name
FROM Production.ProductModel AS m
WHERE m.Name LIKE '%frame%';

-- 与子查询不同，与 INSERT 一起使用的嵌套 SELECT 不用括号括起。
```



### SELECT ... INTO

插入行的另一种选择是 SELECT INTO 语句，该语句与 INSERT SELECT 语句类似。 INSERT SELECT 和 SELECT INTO 的最大区别在于，<u>SELECT INTO 不能用于将行插入到现有表中，因为 SELECT INTO 始终会根据 SELECT 的结果创建一个新表。 新表中的每列的名称、数据类型、为 null 性均与 SELECT 列表中对应的列（或表达式）相同。</u>

要使用 SELECT INTO，请在查询的 SELECT 子句中添加 <>（就在 FROM 子句前面）。 下面的示例将 Sales.SalesOrderHeader 表中的数据提取到名为 Sales.Invoice 的新表中。

```sql
SELECT SalesOrderID, CustomerID, OrderDate, PurchaseOrderNumber, TotalDue
INTO Sales.Invoice
FROM Sales.SalesOrderHeader;
```

**如果已经存在一个表名与 INTO 后面指定的名称相同，则 SELECT INTO 将失败**。 创建表后，可以像处理任何其他表一样处理该表。 可以选择这个表，将其联接到其他表，或在表中插入更多行。



[SQL Server labs](https://microsoftlearning.github.io/dp-080-Transact-SQL/Instructions/Labs/00-setup.html)（Microsoft Azure Data Studio连接不上）

[adventureworkslt.sql](../assets/T-SQL/adventureworkslt.sql) （本地数据库跑一个）

[练习 - 使用 SELECT 语句](https://microsoftlearning.github.io/dp-080-Transact-SQL/Instructions/Labs/01-get-started-with-tsql.html)

```sql
 SELECT SalesOrderID, OrderDate,
     CASE
         WHEN ShipDate IS NULL THEN 'Awaiting Shipment'
         ELSE 'Shipped'
     END AS ShippingStatus
 FROM SalesLT.SalesOrderHeader;
```

[RDBMS 和安装程序（第 1 部分（共 7 部分） |使用 T-SQL 为初学者编程数据库](https://learn.microsoft.com/zh-cn/shows/programming-databases-with-t-sql-for-beginners/rdbms-and-setup-1-of-7-programming-databases-with-t-sql-for-beginners/)

[Transact-SQL 简介（第 2 部分（共 7 部分） |使用 T-SQL 为初学者编程数据库](https://learn.microsoft.com/zh-cn/shows/programming-databases-with-t-sql-for-beginners/introduction-to-transact-sql-2-of-7-programming-databases-with-t-sql-for-beginners/)

---

## 对结果进行排序

在查询处理的逻辑顺序中，ORDER BY 是要执行的 SELECT 语句的最后一个阶段。 ORDER BY 可用于控制从 SQL Server 返回到客户端应用程序的行的排序。 SQL Server 无法保证表中的行的物理顺序，要控制行返回到客户端的顺序，只能使用 ORDER BY 子句。 此行为与关系理论一致。



### 使用 ORDER BY 子句

要命令 SQL Server 以特定顺序返回查询结果，请以以下格式添加 ORDER BY 子句：

```sql
SELECT<select_list>
FROM <table_source>
ORDER BY <order_by_list> [ASC|DESC];
```

ORDER BY 可以使用其列表中的多种类型的元素：

- 按名称排序的列。 可以指定结果排序所依据的列的名称。 结果按第一列的顺序返回，然后依次按其他每个列进行次排序。
- 列的别名。 由于是先处理 SELECT 子句再处理 ORDER BY，因此 ORDER BY 可以访问 SELECT 列表中定义的别名。
- 按 SELECT 列表中的<u>序号位置</u>排序的列。 不建议在应用程序中使用该位置，因为可读性会降低，并且需要额外注意让 ORDER BY 列表保持最新。 但是，对于 SELECT 列表中的复杂表达式，在故障排除过程中使用位置编号可能很有用。
- **列未包含在 SELECT 列表中，但可从 FROM 子句中列出的表中获得**。 如果查询使用 DISTINCT 选项，则 ORDER BY 列表中的任何列都必须包含在 SELECT 列表中。



### 排序方向

除了指定用于确定排序顺序的列之外，还可以控制排序的方向。 可以使用 ASC 进行升序（A-Z、0-9）或使用 DESC 进行降序（Z-A，9-0）。 默认为升序排序。 每列都可以指定自己的方向，如以下示例所示：

```sql
SELECT ProductCategoryID AS Category, ProductName
FROM Production.Product
ORDER BY Category ASC, Price DESC;
```

多列排序的情况下例如`ORDER BY Category ASC, Price DESC;`中`Price`的排序只在`Category`相同的情况下才会生效。在这种情况下相同的情况下才会生效。在这种情况下，`Category` 排序可能对结果的排序影响较小，甚至可以说是次要的。如果希望 `Category` 排序更加突出，可以考虑修改排序的优先级（比如交换排序的列顺序）。



## 限制排序结果

TOP 子句是 SELECT 子句的 Microsoft 专属扩展。 TOP 可用于指定要返回的行数，可以是正整数，也可以是所有符合条件的行的百分比。 行数可以指定为常量或表达式。 TOP 经常与 ORDER BY 一起使用，但也可以与无序数据一起使用。

### 使用 TOP 子句

与 ORDER BY 一起使用的 TOP 子句的简化语法如下所示：

```sql
SELECT TOP (N) <column_list>
FROM <table_source>
WHERE <search_condition>
ORDER BY <order list> [ASC|DESC];
```

例如，要仅从“Production.Product”表中检索 10 个价格最高的产品，请使用以下查询：

```sql
SELECT TOP 10 Name, ListPrice
FROM Production.Product
ORDER BY ListPrice DESC;

/*
结果应如下所示：
名称                      ListPrice

Road-150 Red, 62          3578.27

Road-150 Red, 44          3578.27

Road-150 Red, 48          3578.27

Road-150 Red, 52          3578.27

Road-150 Red, 56          3578.27

Mountain-100 Silver, 38   3399.99

Mountain-100 Silver, 42   3399.99

Mountain-100 Silver, 44   3399.99

Mountain-100 Silver, 48   3399.99

Mountain-100 Black, 38    3374.99
*/
```

TOP 运算符依赖 ORDER BY 子句来提供所选行的有意义优先级。 使用 TOP 时可以不带 ORDER BY，但在这种情况下，无法预测将返回的行。 此示例中，如果没有 ORDER BY 子句，<u>可能会返回任意 10 个订单</u>。

### 使用 WITH  TIES

除了指定要返回的固定行数外，TOP 关键字还接受 WITH TIES 选项，该选项将检索可能在所选前 N 行找到其值的任何行。

在上一个示例中，查询按价格降序返回前 10 个产品。 但是，通过将 WITH TIES 选项添加到 TOP 子句，将看到更多行有资格包含价格最高产品前 10 名中：

```sql
SELECT TOP 10 WITH TIES Name, ListPrice
FROM Production.Product
ORDER BY ListPrice DESC;

/*
修改后的查询将返回以下结果：

名称                    ListPrice

Road-150 Red, 62        3578.27

Road-150 Red, 44        3578.27

Road-150 Red, 48        3578.27

Road-150 Red, 52        3578.27

Road-150 Red, 56        3578.27

Mountain-100 Silver, 38 3399.99

Mountain-100 Silver, 42 3399.99

Mountain-100 Silver, 44 3399.99

Mountain-100 Silver, 48 3399.99

Mountain-100 Black, 38  3374.99

Mountain-100 Black, 42  3374.99

Mountain-100 Black, 44  3374.99

Mountain-100 Black, 48  3374.99
```

如果查询中使用了 `WITH TIES`，那么不仅会返回这些前 10 名的产品（从 `Product A` 到 `Product J`），还会包括其他所有价格为 `3578.27` 的产品（即 `Product K`, `Product L`, 等等），即使它们的排名超出了前 10 名。

决定是否包含 WITH TIES 取决于对源数据的了解、其唯一值的可能性以及正在编写的查询的要求。



### 使用 PERCENT

要返回符合条件的行的百分比，请将 PERCENT 选项与 TOP 一起使用，而不是用固定数字。

```sql
SELECT TOP 10 PERCENT Name, ListPrice
FROM SalesLT.Product
ORDER BY ListPrice DESC;
```

PERCENT 还可以与 WITH TIES 选项一起使用。

**出于统计行数的目的，TOP (N) PERCENT 将向上取整到最接近的整数。**

许多 SQL Server 专业人员使用 TOP 选项作为一种方法，以仅检索某特个特定范围的行。 但在使用 TOP 时，请考虑以下事实：

- TOP 是 T-SQL 专有的。
- TOP 本身不支持跳过行。
- 由于 TOP 依赖于 ORDER BY 子句，因此无法：使用一种排序顺序来确定按 TOP 筛选的行，同时使用另一种排序顺序来确定输出顺序。



## 页结果

### OFFSET-FETCH 语法

OFFSET-FETCH 子句的语法在技术方面是 ORDER BY 子句的一部分，如下所示：

```sql
OFFSET { integer_constant | offset_row_count_expression } { ROW | ROWS }
[FETCH { FIRST | NEXT } {integer_constant | fetch_row_count_expression } { ROW | ROWS } ONLY]
```

### 使用 OFFSET-FETCH

要使用 OFFSET-FETCH，需要提供起始 OFFSET 值（可能为零）和可选的要返回的行数，如以下示例所示：

此示例将返回前 10 行，然后根据“ListPrice”返回接下来的 10 行产品数据：

```sql
SELECT ProductID, ProductName, ListPrice
FROM Production.Product
ORDER BY ListPrice DESC 
OFFSET 0 ROWS --Skip zero rows
FETCH NEXT 10 ROWS ONLY; --Get the next 10
```

要检索下一页产品数据，请使用 OFFSET 子句指定要跳过的行数：

```sql
SELECT ProductID, ProductName, ListPrice
FROM Production.Product
ORDER BY ListPrice DESC 
OFFSET 10 ROWS --Skip 10 rows
FETCH NEXT 10 ROWS ONLY; --Get the next 10
```

在语法定义中，可以看到需要 OFFSET 子句，但不需要 FETCH 子句。 如果省略 FETCH 子句，则将返回 OFFSET 之后的所有行。 用户还会发现，关键字 ROW 和 ROWS 可以互换，就像 FIRST 和 NEXT 一样，这可实现更自然语法。

要确保结果的准确性，尤其是切换不同的数据页时，<u>务必要构造一个 ORDER BY 子句，该子句将提供唯一排序并产生确定的结果</u>。 由于 SQL Server 的查询优化器的工作方式，在多页上显示行在技术上可以实现，除非行范围是确定性的。



## 删除重复项

尽管表中的行应始终唯一，但如果仅选择列的某个子集，结果行也可能不唯一，即使原始行是唯一的。 例如，可能有一个供应商表，其中要求城市和省州（或自治区/直辖市）是唯一的，以便任何城市中都不会有多个供应商。 但是，如果只想查看供应商所在的城市和国家/地区，则返回的结果可能不是唯一的。 假设编写下面的查询：

```sql
SELECT City, CountryRegion
FROM Production.Supplier
ORDER BY CountryRegion, City;

/*
该查询可能返回如下所示的结果：

城市                  CountryRegion

Aurora                加拿大

Barrie                加拿大

Brampton              加拿大

Brossard              加拿大

Brossard              加拿大

Burnaby               加拿大

Burnaby               加拿大

Burnaby               加拿大

Calgary               加拿大

Calgary               加拿大

...                   ...
*/
```

默认情况下，SELECT 子句包括一个导致此行为的隐式 ALL 关键字：

```sql
SELECT ALL City, CountryRegion
FROM Production.Supplier
ORDER BY CountryRegion, City;
```

T-SQL 还支持备用的 DISTINCT 关键字，该关键字删除所有重复的结果行：

```sql
SELECT DISTINCT City, CountryRegion
FROM Production.Supplier
ORDER BY CountryRegion, City;
/*
使用 DISTINCT 时，示例仅返回 SELECT 列表中值的其中一个唯一组合：

城市             CountryRegion

Aurora           加拿大

Barrie           加拿大

Brampton         加拿大

Brossard         加拿大

Burnaby          加拿大

Calgary          加拿大

...              ...
*/
```



## 用谓词筛选数据

仅包含 SELECT 和 FROM 子句的最简单 SELECT 语句将计算表中的每一行。 通过使用 WHERE 子句，可以定义确定将处理哪些行并可能减少结果集的条件。

### WHERE 子句的结构

WHERE 子句由一个或多个搜索条件组成，每个搜索条件对表中的每一行必须求值为 TRUE、FALSE 或“未知”。 仅当 WHERE 子句的计算结果为 TRUE 时，才会返回行。 <u>各个条件充当数据的筛选器，称为“谓词”</u>。 每个谓词包括一个正在测试的条件，通常使用基本比较运算符：

- =（等于）
- <>（不等于）
- \>（大于）
- \>=（大于或等于）
- <（小于）
- <=（小于或等于）

例如，以下查询返回 ProductCategoryID 值为 2 的所有产品：

```sql
SELECT ProductCategoryID AS Category, ProductName
FROM Production.Product
WHERE ProductCategoryID = 2;
```

同样，以下查询返回 ListPrice 小于 10.00 的所有产品：

```sql
SELECT ProductCategoryID AS Category, ProductName
FROM Production.Product
WHERE ListPrice < 10.00;
```

### IS NULL / IS NOT NULL

还可以使用“IS NULL”或“IS NOT NULL”轻松筛选以允许或排除“未知”或 NULL 值。

```sql
SELECT ProductCategoryID AS Category, ProductName
FROM Production.Product
WHERE ProductName IS NOT NULL;
```



## 多个条件

多个谓词可以与 AND 和 OR 运算符以及括号结合使用。 但是 SQL Server 一次只处理两个条件。 使用 AND 运算符连接多个条件时，所有条件都必须为 TRUE。 使用 OR 运算符连接两个条件时，对于结果集，其中的一个或两个条件可以为 TRUE。

例如，下面的查询返回类别 2 中成本低于 10.00 的产品：

```sql
SELECT ProductCategoryID AS Category, ProductName
FROM Production.Product
WHERE ProductCategoryID = 2
    AND ListPrice < 10.00;
```

除非使用括号，否则先处理 OR 运算符，然后再处理 AND 运算符。 最佳做法是，在使用两个以上的谓词时使用括号。 以下查询将返回类别 2 或 3 中成本低于 10.00 的产品：

```sql
SELECT ProductCategoryID AS Category, ProductName
FROM Production.Product
WHERE (ProductCategoryID = 2 OR ProductCategoryID = 3)
    AND (ListPrice < 10.00);
```



## 比较运算符

Transact-SQL 包括可帮助简化 WHERE 子句的比较运算符。

### IN

对于使用 OR 连接的同一个列，IN 运算符是多个等式条件的快捷方法。 在查询中使用多个 OR 条件没有任何问题，如以下示例所示：

```sql
SELECT ProductCategoryID AS Category, ProductName
FROM Production.Product
WHERE ProductCategoryID = 2
    OR ProductCategoryID = 3
    OR ProductCategoryID = 4;
```

但是，使用 IN 清晰简洁，而且查询的性能不会受到影响。

```sql
SELECT ProductCategoryID AS Category, ProductName
FROM Production.Product
WHERE ProductCategoryID IN (2, 3, 4);
```



### LIKE

最终比较运算符只能用于字符数据，并且允许使用通配符和正则表达式模式。 使用通配符，可以指定部分字符串。 例如，可以使用以下查询返回名称中包含单词“mountain”的所有产品：

```sql
SELECT Name, ListPrice
FROM SalesLT.Product
WHERE Name LIKE '%mountain%';

/*
% 通配符表示 0 个或更多字符的任何字符串，因此结果包括名称中任意位置有单词“mountain”的产品，如下所示：

名称                            ListPrice

山地自行车袜，M                  9.50

Mountain Bike Socks, L         9.50

HL Mountain Frame - Silver, 42 1364.0

HL Mountain Frame - Black, 42  1349.60

HL Mountain Frame - Silver, 38 1364.50

Mountain-100 Silver, 38        3399.99
*/
```

可以使用 _ 通配符来表示单个字符，如下所示：

```sql
SELECT ProductName, ListPrice
FROM SalesLT.Product
WHERE ProductName LIKE 'Mountain Bike Socks, _';

/*
以下结果仅包括以“山地自行车袜子”开头的产品，以及后面的单个字符：

ProductName            ListPrice

山地自行车袜，M          9.50

Mountain Bike Socks, L 9.50
```

还可以定义要查找的复杂模式字符串。 例如，以下查询搜索了名称以“山地-”开头的产品，后跟：

- 0 到 9 之间的三个字符
- 1 个空格
- 任何字符串
- 一个逗号
- 1 个空格
- 介于 0 和 9 之间的两个字符

```sql
SELECT ProductName, ListPrice
FROM SalesLT.Product
WHERE ProductName LIKE 'Mountain-[0-9][0-9][0-9] %, [0-9][0-9]';

/*
该查询的结果可能如下所示：

ProductName                ListPrice

Mountain-100 Silver, 38    3399.99

Mountain-100 Silver, 42    3399.99

Mountain-100 Black, 38     3399.99

Mountain-100 Black, 42     3399.99

Mountain-200 Silver, 38    2319.99

Mountain-200 Silver, 42    2319.99

Mountain-200 Black, 38     2319.99

Mountain-200 Black, 42     2319.99
```

要了解有关使用 **LIKE** 运算符进行模式匹配的更多信息，请参阅 [Transact-SQL 文档 ](https://docs.microsoft.com/sql/t-sql/language-elements/like-transact-sql)。



[练习 - 对查询结果进行排序和筛选](https://microsoftlearning.github.io/dp-080-Transact-SQL/Instructions/Labs/02-filter-sort.html)

```sql
SELECT Name
FROM SalesLT.Product
WHERE SellEndDate BETWEEN '2006/1/1' AND '2006/12/31';
```

```sql
SELECT ProductNumber, Name
FROM SalesLT.Product
WHERE (Color = 'Black' OR Color = 'Red' OR Color = 'White') AND (Size = 'S' OR Size = 'M');

-- 等效

 SELECT ProductNumber, Name
 FROM SalesLT.Product
 WHERE Color IN ('Black','Red','White') AND Size IN ('S','M');

```

[在 Transact-SQL 中对结果进行排序和筛选 （第 3 部分（共 7 部分） |使用 T-SQL 为初学者编程数据库](https://learn.microsoft.com/zh-cn/shows/programming-databases-with-t-sql-for-beginners/sort-and-filter-results-in-transact-sql-3-of-7-programming-databases-with-t-sql-for-beginners)

***

## 理解联接概念和语法

合并多个表中的数据的最基本且最常用的方法是使用 JOIN 运算。 有些人将 <u>JOIN 视为 SELECT 语句中的单独子句</u>，<u>而其他人则将其视为 FROM 子句的一部分</u>。 本模块主要将其视为 FROM 子句的一部分。 在本模块中，我们将了解 T-SQL SELECT 语句中的 FROM 子句如何创建中间虚拟表，这些虚拟表将在查询的后续阶段使用。

```sql title="JOIN 作为 FROM 子句的一部分"
-- 这是最常见的情况，JOIN 是直接写在 FROM 子句之后，表示不同表之间的连接。

SELECT 
    p.ProductNumber, p.Name, c.Name AS CategoryName
FROM 
    SalesLT.Product p
JOIN 
    SalesLT.ProductCategory c ON p.ProductCategoryID = c.ProductCategoryID
ORDER BY 
    p.Name;
```

- `JOIN` 在 `FROM` 子句中，直接连接了 `SalesLT.Product` 和 `SalesLT.ProductCategory` 两个表。
- `p.ProductCategoryID = c.ProductCategoryID` 作为连接条件。

这种写法是标准的 SQL 语法，`JOIN` 明确是 `FROM` 子句的一部分。

```sql title="将 JOIN 视为 SELECT 语句中的单独子句"
-- 在一些情况下，虽然 JOIN 仍然写在 FROM 子句中，但它可能被认为是单独的子句，特别是在某些 SQL 教程或文档中，强调 JOIN 是从表中“获取”数据的一种方式。

SELECT 
    p.ProductNumber, p.Name, c.Name AS CategoryName
FROM 
    SalesLT.Product p
WHERE 
    p.ListPrice > 100
JOIN 
    SalesLT.ProductCategory c ON p.ProductCategoryID = c.ProductCategoryID
ORDER BY 
    p.Name;
```

- 在这个查询中，`JOIN` 仍然是连接 `SalesLT.Product` 和 `SalesLT.ProductCategory` 表，但在某些教程或文档中，这种写法可能被看作 `JOIN` 是一个独立的操作，因为它位于 `WHERE` 子句之后（虽然这是不推荐的写法，标准 SQL 语法要求 `JOIN` 在 `FROM` 子句中）。

这种方式有时出现在非正式或教学性内容中，目的是帮助理解 SQL 的联接逻辑，但在实际应用中通常会遵循第一种方式（`JOIN` 在 `FROM` 子句中）。在标准 SQL 中，不应该把 `JOIN` 放在 `WHERE` 子句之后。

## [FROM 子句和虚拟表](https://learn.microsoft.com/zh-cn/training/modules/query-multiple-tables-with-joins/2-join-concepts)

如果你已了解 SQL Server 处理查询时执行的运算的逻辑顺序，则会发现 SELECT 语句的 FROM 子句是第一个要处理的子句。 此子句确定哪个表或哪些表将成为查询的行源。 FROM 可以引用单个表或将多个表组合在一起作为查询的数据源。 <u>可以将 FROM 子句视为创建和填充一个**虚拟表**。 该虚拟表将保存 FROM 子句的输出，并由稍后应用的 SELECT 语句的子句使用，例如 WHERE 子句</u>。 在向 FROM 子句添加额外的功能（例如[联接运算符](https://learn.microsoft.com/zh-cn/sql/relational-databases/performance/joins?view=sql-server-ver16)）时，可以将 FROM 子句元素的目的理解为在虚拟表中添加或删除行。

```sql title="FROM 子句元素的目的理解为在虚拟表中添加或删除行"
-- Products 表
ProductID | ProductName | CategoryID
----------------------------------
1         | Bike       | 10
2         | Helmet     | 20
3         | Tire       | 10
4         | Gloves     | 30

-- Categories 表
CategoryID | CategoryName
--------------------------
10         | Cycling
20         | Sports
30         | Accessories

SELECT p.ProductName, c.CategoryName
FROM Products p
JOIN Categories c ON p.CategoryID = c.CategoryID;
-- 通过联接操作，虚拟表中会生成新的行，包含 ProductName 和 CategoryName 字段。

ProductName | CategoryName
--------------------------
Bike        | Cycling
Tire        | Cycling
Helmet      | Sports
Gloves      | Accessories

SELECT p.ProductName, c.CategoryName
FROM Products p
LEFT JOIN Categories c ON p.CategoryID = c.CategoryID;
-- 通过左联接操作，Categories 表中的每一行都会尝试与 Products 表进行匹配。如果没有匹配项，Products 表中的行仍然会保留，只是 CategoryName 会返回 NULL。这里并没有行被“删除”，即使有某些商品没有匹配的类别，左联接会保留它们并用 NULL 填充。
ProductName | CategoryName
--------------------------
Bike        | Cycling
Tire        | Cycling
Helmet      | Sports
Gloves      | Accessories

-- 修改后的 Products 表
ProductID | ProductName | CategoryID
----------------------------------
1         | Bike       | 10
2         | Helmet     | 20
3         | Tire       | 10
4         | Gloves     | 30
5         | Unknown    | NULL  -- 新添加的没有类别的商品

SELECT p.ProductName, c.CategoryName
FROM Products p
INNER JOIN Categories c ON p.CategoryID = c.CategoryID;

-- 在这种情况下，Unknown 商品会被删除，因为它没有匹配的类别（NULL），INNER JOIN 只返回两个表中都有的匹配行。
ProductName | CategoryName
--------------------------
Bike        | Cycling
Tire        | Cycling
Helmet      | Sports
Gloves      | Accessories

SELECT p.ProductName, c.CategoryName
FROM Products p
LEFT JOIN Categories c ON p.CategoryID = c.CategoryID;

-- 这里 Unknown 商品的行被保留，虽然没有匹配的类别，但仍然保留了商品名称，并且 CategoryName 返回了 NULL。
ProductName | CategoryName
--------------------------
Bike        | Cycling
Tire        | Cycling
Helmet      | Sports
Gloves      | Accessories
Unknown     | NULL
```

- **`JOIN` 操作** 在 `FROM` 子句中，实际上是 **添加行** 或 **删除行** 的操作。`INNER JOIN` 会将不符合联接条件的行删除，而 `LEFT JOIN` 则会保留这些行并用 `NULL` 填充缺失的值。
- 在虚拟表中，`JOIN` 操作增加了新的行，或者按联接条件删除不匹配的行。

这就是为什么可以将 `FROM` 子句理解为“创建和填充一个虚拟表”的原因，`JOIN` 操作通过匹配条件将不同表中的数据合并，并根据联接类型（内联接、左联接等）来决定最终虚拟表中的行数。

由 FROM 子句创建的虚拟表只是逻辑实体。 在 SQL Server 中，不会创建任何物理表（无论是永久的还是临时的）来保存 FROM 子句的结果，因为该表将传递给 WHERE 子句或查询的其他部分。

由 FROM 子句创建的虚拟表包含来自所有联接表的数据。 可以将结果视为集，并将联接结果概念化为维恩图。

![显示联接到 SalesOrder 表的 Employee 表集的维恩图](../assets/T-SQL/join-venn-diagram.avif)

在 T-SQL 语言的整个历史中，它经过不断扩展，反映了 SQL 语言的美国国家标准协会 (ANSI) 标准的更改。 体现这些更改的最明显的地方之一是 FROM 子句中的联接语法。 在 [ANSI SQL-89](https://archive.org/details/federalinformati1271nati/mode/2up) 标准中，通过在以逗号分隔的列表中的 FROM 子句中包含多个表来指定联接。 用于确定要包括哪些行的任何筛选均在 WHERE 子句中执行，如下所示：

```sql
SELECT p.ProductID, m.Name AS Model, p.Name AS Product
FROM SalesLT.Product AS p, SalesLT.ProductModel AS m
WHERE p.ProductModelID = m.ProductModelID;
```

SQL Server 仍支持此语法，但是由于表示复杂联接的筛选器很复杂，因此不建议使用。 此外，如果意外省略了 WHERE 子句，则 ANSI SQL-89 样式的联接很容易成为**[笛卡尔乘积](https://zh.wikipedia.org/wiki/%E7%AC%9B%E5%8D%A1%E5%84%BF%E7%A7%AF)**，并返回过多的结果行，从而导致性能问题，并可能生成错误的结果。

...

在数据库中，笛卡尔乘积是将一个表中的每一行与另一个表中的每一行相结合的结果。 一个包含 10 行的表和一个包含 100 行的表的乘积是一个包含 1,000 行的结果集。 JOIN 运算的基本结果是笛卡尔乘积，但是对于大多数 T-SQL 查询来说，笛卡尔乘积并不是期望的结果。 在 T-SQL 中，当联接两个输入表，而不考虑它们之间的任何关系时，就会产生笛卡尔乘积。 如果没有关于关系的信息，SQL Server 查询处理器将返回所有可能的行组合。 尽管此结果可能有一定的实际应用价值，例如生成测试数据，但通常并没有用，并且可能会对性能产生严重影响。

随着 [ANSI SQL-92](https://www.contrib.andrew.cmu.edu/~shadow/sql/sql1992.txt) 标准的出现，添加了对关键字 JOIN 和 ON 子句的支持。 T-SQL 也支持此语法。 在 FROM 子句中通过使用相应的 JOIN 运算符来表示联接。 在 ON 子句中指定了表之间的逻辑关系，该关系成为筛选谓词。

下面的示例使用较新的语法重述了前面的查询：

```sql
SELECT p.ProductID, m.Name AS Model, p.Name AS Product
FROM SalesLT.Product AS p
JOIN SalesLT.ProductModel AS m
    ON p.ProductModelID = m.ProductModelID;
```

**ANSI SQL-92 语法使创建偶然的笛卡尔乘积变得更加困难。 添加关键字 JOIN 后，除非将 JOIN 指定为 CROSS JOIN，否则在缺少 ON 子句的情况下将引发语法错误。**

## 使用内部联接

T-SQL 查询中最常见的 JOIN 类型是 INNER JOIN。 内部联接用于解决许多常见的业务问题，尤其是在高度规范化的数据库环境中。 要在多个表中检索已存储的数据，通常需要通过 INNER JOIN 查询将其合并。 <u>INNER JOIN 以笛卡尔乘积的形式开始其逻辑处理阶段，然后对其进行筛选以删除与谓词不匹配的所有行</u>。

### 处理 INNER JOIN

我们来看看 SQL Server 对 JOIN 查询进行逻辑处理的步骤。 为了清楚起见，在以下假设示例中添加了行号：

```sql
1) SELECT emp.FirstName, ord.Amount
2) FROM HR.Employee AS emp 
3) JOIN Sales.SalesOrder AS ord
4)      ON emp.EmployeeID = ord.EmployeeID;
```

你应该知道，FROM 子句将在 SELECT 子句之前处理。 让我们从第 2 行开始跟踪处理过程：

- FROM 子句将“HR.Employee”表指定为输入表之一，并为其提供别名“emp”。
- 第 3 行中的 <u>JOIN 运算符反映了 INNER JOIN（T-SQL 中的默认类型）</u>的用法，并将“Sales.SalesOrder”指定为另一个输入表，其别名为“ord”。
- SQL Server 将对这些表执行逻辑笛卡尔联接，并将结果作为虚拟表传递到下一步。 （查询的物理处理实际上可能不会执行笛卡尔乘积运算，具体取决于优化器的决策。但是，可以想象一下正在创建笛卡尔乘积。）
- 使用 ON 子句，SQL Server 将筛选虚拟表，仅保留“emp”表中 EmployeeID 值与“ord”表中的 EmployeeID 匹配的那些行。
- 其余的行将留在虚拟表中，并移交给 SELECT 语句中的下一步。 在此示例中，虚拟表接下来由 SELECT 子句处理，并将两个指定的列返回到客户端应用程序。

已完成的查询生成一个员工及其订单数量的列表。 ON 子句会筛选掉没有任何关联订单的员工，也会筛选掉拥有任何恰巧包含 EmployeeID 但与“HR.Employee”表中条目不对应的订单的员工。

![显示 Employee 和 SalesOrder 集的匹配成员的维恩图](../assets/T-SQL/inner-join-venn-diagram.avif)

### INNER JOIN 语法

**INNER JOIN 是 JOIN 的默认类型，可选的 INNER 关键字在 JOIN 子句中是隐式的**。 在混合和匹配联接类型时，显式指定联接类型可能很有用，如以下假设示例所示：

```sql
SELECT emp.FirstName, ord.Amount
FROM HR.Employee AS emp 
INNER JOIN Sales.SalesOrder AS ord
    ON emp.EmployeeID = ord.EmployeeID;
```

使用内部联接编写查询时，请考虑以下准则：

- 表别名不仅是 SELECT 列表的首选，也是编写 ON 子句时的首选。
- 内部联接可以针对单个匹配列（例如 OrderID）执行，也可以针对多个匹配属性（例如 OrderID 和 ProductID 的组合）执行。 <u>指定多个匹配列的联接称为复合联接</u>。
- 对于 SQL Server 优化器来说，INNER JOIN 的 FROM 子句中列出的表的顺序并不重要。 从概念上讲，联接将从左到右进行评估。
- 对于 FROM 列表中的每对联接表使用一次 JOIN 关键字。 对于两个表的查询，请指定一个联接。 对于三个表的查询，将使用两次 JOIN；一次是在前两个表之间，一次是在前两个表和第三个表之间的 JOIN 输出之间。

```sql title="什么是复合连接" 
/*
Orders 表：
OrderID	OrderDate	CustomerID
1001	2025-01-01	A123
1002	2025-02-01	B456
1003	2025-03-01	C789

OrderDetails 表：
OrderID	ProductID	Quantity	Price
1001	101	        2	        15.5
1001	102	        1	        25.0
1002	103	        3	        10.0
1003	104	        5	        12.0
1003	105	        2	        18.0
*/

-- 我们通常会使用单一列来联接这两个表，例如 OrderID，来查询每个订单的商品详情。
SELECT ord.OrderID, ord.OrderDate, od.ProductID, od.Quantity, od.Price
FROM Orders ord
INNER JOIN OrderDetails od
    ON ord.OrderID = od.OrderID;

/*
结果：
OrderID	OrderDate	ProductID	Quantity	Price
1001	2025-01-01	101	        2	        15.5
1001	2025-01-01	102	        1	        25.0
1002	2025-02-01	103	        3	        10.0
1003	2025-03-01	104	        5	        12.0
1003	2025-03-01	105	        2	        18.0
*/

-- 复合联接则涉及多个列来进行匹配。在这种情况下，我们不仅需要 OrderID 来匹配订单，还可能需要 ProductID 来确定每个订单的具体商品。假设我们想要根据 OrderID 和 ProductID 两个条件来联合 Orders 表和 OrderDetails 表：

SELECT ord.OrderID, ord.OrderDate, od.ProductID, od.Quantity, od.Price
FROM Orders ord
INNER JOIN OrderDetails od
    ON ord.OrderID = od.OrderID
    AND ord.CustomerID = 'A123';  -- 复合联接，根据 OrderID 和 CustomerID
    
-- 假设我们不仅想要按 OrderID 匹配，而且还想根据 ProductID 匹配，来确保订单和每个商品的详细信息正确匹配：

SELECT ord.OrderID, ord.OrderDate, od.ProductID, od.Quantity, od.Price
FROM Orders ord
INNER JOIN OrderDetails od
    ON ord.OrderID = od.OrderID
    AND od.ProductID = 101;  -- 按照 OrderID 和 ProductID 两列进行复合联接

/*
结果：
OrderID	OrderDate	ProductID	Quantity	Price
1001	2025-01-01	101	        2	        15.5
*/
```

```sql title="为什么说优化器不关心表的顺序？"
-- FROM 子句
FROM HR.Employee AS emp 
INNER JOIN Sales.SalesOrder AS ord

/*
SQL Server 的优化器会尝试选择最有效的查询执行计划来执行查询，考虑多种因素，如索引、表大小、统计信息等。在执行 INNER JOIN 时，理论上，表的顺序不会影响最终的结果，因为 INNER JOIN 是一个对称的操作——无论哪个表先被列出，最终都会执行相同的联接操作。然而，优化器可以根据实际情况选择最优的执行顺序。
*/

-- 即使我们交换 FROM 和 JOIN 的表顺序，SQL Server 会根据其优化规则来决定执行顺序：
-- 表顺序交换
SELECT emp.FirstName, ord.Amount
FROM Sales.SalesOrder AS ord
INNER JOIN HR.Employee AS emp
    ON emp.EmployeeID = ord.EmployeeID;
    
-- 虽然表的顺序不同，但因为 INNER JOIN 是对称的，最终结果还是一样的。优化器会根据表的大小、索引等来选择最佳的执行顺序，而不一定是按你写的顺序来评估。
```

## 使用外部联接

```sql
SELECT emp.FirstName, ord.Amount
FROM HR.Employee AS emp
LEFT OUTER JOIN Sales.SalesOrder AS ord
    ON emp.EmployeeID = ord.EmployeeID;
```

本示例使用 LEFT OUTER JOIN 运算符，该运算符指示查询处理器保留左侧表（“HR.Employee”）中的所有行，并显示“Sales.SalesOrder”中匹配行的 Amount 值。 但是，无论员工是否取得了销售订单，都会返回所有员工。 对于没有匹配销售订单的员工，查询将返回 NULL 来代替 Amount 值。

![A Venn diagram showing the outer join results of the Employee and SalesOrder sets](../assets/T-SQL/outer-join-venn-diagram.avif)

### OUTER JOIN 语法

外部联接的表示方法是，在 OUTER JOIN 之前使用关键字 LEFT、RIGHT 或 FULL。 关键字的目的在于指示应保留哪个表（在关键字 JOIN 的哪一侧），并显示其所有行（匹配或不匹配）。

<u>使用 LEFT、RIGHT 或 FULL 定义联接时，可以省略 OUTER 关键字</u>，如下所示：

```sql
SELECT emp.FirstName, ord.Amount
FROM HR.Employee AS emp
LEFT JOIN Sales.SalesOrder AS ord
    ON emp.EmployeeID = ord.EmployeeID;
```

- 但是，像 INNER 关键字一样，编写关于所使用的联接类型的显式代码通常会很有帮助。

  使用 OUTER JOIN 编写查询时，请考虑以下准则：

  - 如你所见，<u>表别名不仅是 SELECT 列表的首选，也是 ON 子句的首选</u>。
  - 与 INNER JOIN 一样，OUTER JOIN 可以针对单个匹配列执行，也可以针对多个匹配属性执行。
  - 与 INNER JOIN 不同，<u>在 FROM 子句中列出和连接表的顺序对 OUTER JOIN 很重要，因为它将决定选择 LEFT 还是 RIGHT 进行连接</u>。
  - 当存在 OUTER JOIN 时，多表联接会更加复杂。 如果随后将中间结果联接到第三个表，则 OUTER JOIN 的结果中存在 NULL 可能会导致问题。 第二个联接的谓词可能会将包含 NULL 的行筛选掉。
  - 要仅显示不存在匹配项的行，请在 OUTER JOIN 谓词之后的 WHERE 子句中添加 NULL 测试。
  - FULL OUTER JOIN 很少使用。 它返回两个表之间的所有匹配行，第一个表中在第二个表内没有匹配项的所有行，以及第二个表中在第一个表内没有匹配项的所有行。
  - 如果没有 ORDER BY 子句，便无法预测行返回的顺序。 无法知道是先返回匹配的行，还是不匹配的行

```sql title="FULL OUTER JOIN和INNER JOIN"
/*
Employee 表：
EmployeeID	Name
1	        John
2	        Jane
3	        Alice

SalesOrder 表：
OrderID	EmployeeID	Amount
101	    1	        200
102	    2	        150
103	    4	        300
*/

SELECT emp.Name, ord.Amount
FROM Employee emp
INNER JOIN SalesOrder ord ON emp.EmployeeID = ord.EmployeeID;

/*
返回结果：
Name	Amount
John	200
Jane	150
*/

SELECT emp.Name, ord.Amount
FROM Employee emp
FULL OUTER JOIN SalesOrder ord ON emp.EmployeeID = ord.EmployeeID;

/*
返回结果：
Name	Amount
John	200
Jane	150
Alice	NULL
NULL	300
*/
```



## 使用交叉联接

交叉联接就是两个表的笛卡尔乘积。 **你可以使用 ANSI SQL-89 语法，通过省略连接两个表的筛选器来创建交叉联接。 使用 ANSI-92 语法会有点困难**；这很好，因为一般来说，交叉联接并不是你通常想要的。 使用 ANSI-92 语法，不太可能意外出现交叉联接。

```sql
/*
Products 表：
ProductID	ProductName
1	        Apple
2	        Banana
3	        Orange

Categories 表：
CategoryID	CategoryName
1	        Fruit
2	        Vegetable
*/

-- ANSI SQL-89 语法
SELECT Products.ProductName, Categories.CategoryName
FROM Products, Categories;

--  ANSI SQL-92 语法
SELECT Products.ProductName, Categories.CategoryName
FROM Products
CROSS JOIN Categories;

/*
返回的结果（笛卡尔积）：

ProductName	CategoryName
Apple	    Fruit
Apple	    Vegetable
Banana	    Fruit
Banana	    Vegetable
Orange	    Fruit
Orange	    Vegetable
*/
```

要显式创建笛卡尔乘积，请使用 CROSS JOIN 运算符。

此运算会创建包含输入行的所有可能组合的结果集：

```sql
SELECT <select_list>
FROM table1 AS t1
CROSS JOIN table2 AS t2;
```

虽然此结果通常不是所需的输出，但是编写显式的 CROSS JOIN 还是有以下一些实际应用：

- 创建数字表，范围内的每个可能值均用一行表示。
- 生成大量数据以进行测试。 当与自身进行交叉联接时，一个只有 100 行的表可以轻松生成 10,000 个输出行，而你几乎不需要执行任何操作。

```sql title="实际应用"
-- 数字表 是一个仅包含数字的表，通常用于生成一个有序的数字序列，或者用于执行某些特定的查询（比如生成日期序列、范围数字等）。它本身不包含业务数据，仅仅存储数字。

-- 你可以使用 CROSS JOIN 来生成一个数字表，而不需要手动列出每个数字。

-- 示例：生成数字 1 到 100
-- 首先，我们创建两个非常简单的表（table1 和 table2），这两个表分别包含 1 到 10 的数字。

-- 创建两个小表，每个表包含数字 1 到 10
SELECT 1 AS num UNION ALL SELECT 2 UNION ALL SELECT 3 UNION ALL SELECT 4 UNION ALL SELECT 5
UNION ALL SELECT 6 UNION ALL SELECT 7 UNION ALL SELECT 8 UNION ALL SELECT 9 UNION ALL SELECT 10;

-- 然后，我们可以使用 CROSS JOIN 将这两个表结合起来。由于 CROSS JOIN 会生成两个表的笛卡尔积，两个包含 10 个数字的表会生成 100 个结果行。
SELECT t1.num + t2.num - 1 AS number
FROM 
    (SELECT 1 AS num UNION ALL SELECT 2 UNION ALL SELECT 3 UNION ALL SELECT 4 UNION ALL SELECT 5 
     UNION ALL SELECT 6 UNION ALL SELECT 7 UNION ALL SELECT 8 UNION ALL SELECT 9 UNION ALL SELECT 10) AS t1
CROSS JOIN
    (SELECT 1 AS num UNION ALL SELECT 2 UNION ALL SELECT 3 UNION ALL SELECT 4 UNION ALL SELECT 5 
     UNION ALL SELECT 6 UNION ALL SELECT 7 UNION ALL SELECT 8 UNION ALL SELECT 9 UNION ALL SELECT 10) AS t2;
     
/*
结果集：
number
1
2
3
4
...
100
*/

-- 假设你想要生成一个日期范围从 2023-01-01 到 2023-12-31，可以借助 CROSS JOIN 来生成日期序列。你可以使用数字表（如上例）与日期函数配合，从而创建日期。
WITH Numbers AS (
    SELECT 1 AS num UNION ALL SELECT 2 UNION ALL SELECT 3 
    UNION ALL SELECT 4 UNION ALL SELECT 5 UNION ALL SELECT 6 
    UNION ALL SELECT 7 UNION ALL SELECT 8 UNION ALL SELECT 9 
    UNION ALL SELECT 10  -- 重复多次以生成更大的数字范围
)
SELECT 
    DATEADD(DAY, num - 1, '2023-01-01') AS generated_date
FROM Numbers
CROSS JOIN Numbers
WHERE num <= 365;  -- 控制最大日期为一年内的所有日期

/* 这个查询会生成 2023 年所有日期，从 2023-01-01 到 2023-12-31。*/

-- 你也可以使用 CROSS JOIN 生成大量测试数据。例如，如果你需要生成多个产品和多个客户的所有组合来进行测试，你可以将两个表交叉连接来生成所有可能的组合。

SELECT 
    p.ProductName,
    c.CustomerName
FROM 
    Products AS p
CROSS JOIN 
    Customers AS c;
    
/* 这会生成所有产品和客户的组合，通常用于生成测试数据。*/    

```



### CROSS JOIN 语法

使用 CROSS JOIN 编写查询时，请考虑以下准则：

- 没有执行行匹配，因此不使用 ON 子句。 （**将 ON 子句与 CROSS JOIN 一起使用是错误的。**）
- 要使用 ANSI SQL-92 语法，请使用 CROSS JOIN 运算符分隔输入表名称。

以下查询是使用 CROSS JOIN 创建员工和产品的所有组合的示例：

```sql
SELECT emp.FirstName, prd.Name
FROM HR.Employee AS emp
CROSS JOIN Production.Product AS prd;
```



## 使用自联接

```sql
-- 到目前为止，我们使用的联接涉及不同的表。 在某些情况下，你可能需要检索表中的行，并将其与同一表中的其他行进行比较。 例如，在人力资源应用程序中，“Employee”表可能包含有关每个员工的经理的信息，并将经理的 ID 存储在员工所在的那行中。 每个经理也列为一名员工。

/*
EmployeeID          FirstName           ManagerID

1                   Dan                 Null

2                   Aisha               1

3                   Rosie               1

4                   Naomi               3
*/

-- 要检索员工信息并将其与相关的经理进行匹配，可以在查询中使用该表两次，并将其与自身进行联接以实现查询目的。

SELECT emp.FirstName AS Employee, 
       mgr.FirstName AS Manager
FROM HR.Employee AS emp
LEFT OUTER JOIN HR.Employee AS mgr
  ON emp.ManagerID = mgr.EmployeeID;
  
-- 在该查询的结果中，每名员工一行，其中包含其经理的姓名。 公司的 CEO 没有经理。 为了在结果中包括 CEO，将使用外部联接，并且对于“ManagerID”字段没有匹配的“EmployeeID”字段的行，经理姓名将返回 NULL。

/*
Employee       Manager

Dan            Null

Aisha          Dan

Rosie          Dan

Naomi          Rosie
*/
```

在其他情况下，你可能希望将表中的行与同一表中的其他行进行比较。 如你所见，使用 T-SQL 比较同一行中的列非常容易，但是比较不同行（例如存储开始时间的行，以及同一表中存储相应停止时间的另一行）中的值的方法就不那么明显了。 对于这些类型的查询，自联接这一技术会很有用。

要完成此类任务，应考虑以下准则：

- 在 FROM 子句中定义同一表的两个实例，并根据需要使用内部或外部联接来联接它们。
- 使用表别名来区分同一个表的两个实例。
- 使用 ON 子句来提供筛选器，将表的一个实例的列与该表的另一个实例的列进行比较。



[练习 - 使用联接查询多个表](https://microsoftlearning.github.io/dp-080-Transact-SQL/Instructions/Labs/03a-joins.html)

使用 **LEFT**（或 **RIGHT）** 关键字会自动将联接标识为 **OUTER** 联接。

为什么所有后续的外部连接必须具有相同的方向（LEFT 或 RIGHT）？

- **一致的方向**（`LEFT` 或 `RIGHT`）能够保持查询的逻辑一致性，确保结果集中的数据按预期返回。
- **不同方向的外部连接** 可能导致数据重复、遗漏或结果混乱，因此最好在查询中保持方向一致，避免让查询变得复杂并难以预测。

***自联接实际上不是一种特定类型的联接，而是一种技术，用于通过定义表的两个实例（每个实例都有自己的别名）将表联接到自身。**

挑战三有点绕

[将多个表与 Transact-SQL 中的 JOINS 组合（第 4 部分（共 7 部分） |使用 T-SQL 为初学者编程数据库](https://learn.microsoft.com/zh-cn/shows/programming-databases-with-t-sql-for-beginners/combining-multiple-tables-with-joins-in-transact-sql-4-of-7-programming-databases-with-t-sql-for-beginners)

```sql
INSERT INTO #Employees（[Name], Title, Manager) 
-- # 表示临时表，用于创建局部临时表，在会话结束后会自动删除。
-- [] 用于标识符，包裹表名或列名等数据库对象，防止它们与 SQL 保留字或特殊字符冲突。
```

***

## 了解子查询

子查询是指嵌套在另一个查询中的 SELECT 语句。 <u>通过将一个查询嵌套在另一个查询中，可增强在 T-SQL 中创建有效查询的能力</u>。 通常，子查询将计算一次，并将结果提供给外部查询。

### 使用子查询

<u>子查询是指嵌套或嵌入在另一个查询中的 SELECT 语句。 嵌套的查询（即子查询）被称为内部查询。 包含嵌套查询的查询是外部查询。</u>

子查询的目的是向外部查询返回结果。 结果的形式决定子查询是标量子查询还是多值子查询：

- 标量子查询返回单个值。 外部查询必须处理单个结果。
- 多值子查询返回的结果很像是单列表格。 外部查询必须能够处理多个值。

```sql title="标量子查询和多值子查询"
-- 标量子查询 返回单个值，外部查询必须处理这个单个结果。通常用于 SELECT、WHERE、HAVING 或 ORDER BY 子句中，并且子查询必须保证只返回一行一列，否则会报错。
SELECT Name, ListPrice
FROM SalesLT.Product
WHERE ListPrice = (SELECT MAX(ListPrice) FROM SalesLT.Product);

-- 子查询：(SELECT MAX(ListPrice) FROM SalesLT.Product) 只返回一个值，即最高价格。
-- 外部查询：使用 WHERE ListPrice = 最高价格 过滤产品，找到最高价的产品。

-- 多值子查询 返回多行单列（相当于一个单列表格），外部查询必须能够处理多个值。常用于 IN、EXISTS、ANY、ALL 语句。
SELECT ProductID, Name
FROM SalesLT.Product
WHERE ProductID IN (SELECT ProductID FROM SalesLT.Product WHERE Color = 'Red');

-- 子查询：(SELECT ProductID FROM SalesLT.Product WHERE Color = 'Red') 可能返回多个 ProductID（多个值）。
-- 外部查询：使用 IN 关键字，匹配 ProductID 是否属于子查询返回的结果。
```

除标量子查询和多值子查询选项之外，子查询还可以是自包含子查询，或外部查询的关联子查询：

- 自包含子查询可以作为独立查询写入，不依赖于外部查询。 在外部查询运行时，自包含子查询将处理一次，并将其结果传递给该外部查询。
- 关联子查询会引用外部查询中的一个或多个列，因此依赖于外部查询。 关联子查询不能与外部查询分开运行。

```sql title="自包含子查询和关联子查询"
-- 自包含子查询 可以独立运行，不依赖外部查询中的任何列。它在外部查询执行前仅执行一次，然后将结果传递给外部查询。
SELECT Name, ListPrice
FROM SalesLT.Product
WHERE ListPrice > (SELECT MAX(ListPrice) FROM SalesLT.Product WHERE Color = 'Red');

-- 子查询 (SELECT MAX(ListPrice) FROM SalesLT.Product WHERE Color = 'Red')：该查询可以单独执行，它计算所有红色产品的最高价格。
-- 外部查询 WHERE ListPrice > 最高红色产品价格：过滤 ListPrice 高于红色产品最高价的产品。

-- 关联子查询 不能独立运行，它依赖外部查询中的列。子查询对外部查询中的每一行执行一次。
SELECT p1.Name, p1.Color, p1.ListPrice
FROM SalesLT.Product p1
WHERE p1.ListPrice > (
    SELECT AVG(p2.ListPrice)
    FROM SalesLT.Product p2
    WHERE p1.Color = p2.Color
);
-- 子查询 (SELECT AVG(p2.ListPrice) FROM SalesLT.Product p2 WHERE p1.Color = p2.Color)：
-- 它依赖外部查询的 p1.Color，计算相同 Color 的产品的平均价格。
-- 不能独立执行，因为 p1.Color 只有在外部查询运行时才有值。

-- 外部查询 WHERE p1.ListPrice > 该颜色的平均价格：
-- 逐行检查 p1.ListPrice 是否高于同色产品的平均价格。
```



## 使用标量或多值子查询

<u>标量子查询是外部查询中的内部 SELECT 语句，写入后可返回单个值</u>。 标量子查询可用于 T-SQL 语句中允许单值表达式的任何位置，例如 SELECT 子句、WHERE 子句、HAVING 子句，甚至是 FROM 子句。 此外，它们还可用于 UPDATE 或 DELETE 等数据修改语句。

顾名思义，多值子查询可以返回多个行。 不过，它们仍会返回单列。

### 标量子查询

假设要检索上次下单的详细信息，假定它是具有最高“SalesOrderID”值的订单。

要查找最高的“SalesOrderID”值，可以使用以下查询：

```sql
SELECT MAX(SalesOrderID)
FROM Sales.SalesOrderHeader
```

此查询将返回一个指示“SalesOrderHeader”表中“OrderID”最高值的值。

要获取该订单的详细信息，可能需要根据上述查询返回的任何值筛选“SalesOrderDetails”表。 通过将检索最高“SalesOrderID”的查询嵌套在检索订单详细信息的查询的 WHERE 子句中，可以完成此项任务。

```sql
SELECT SalesOrderID, ProductID, OrderQty
FROM Sales.SalesOrderDetail
WHERE SalesOrderID = 
   (SELECT MAX(SalesOrderID)
    FROM Sales.SalesOrderHeader);
```

要写入标量子查询，请考虑以下准则：

- 要将查询表示为子查询，请用括号将其括起来。
- **Transact-SQL 中支持多级子查询。 在此模块中，我们仅考虑两级查询（一个外部查询中有一个内部查询），但最多支持 32 个级别**。
- 如果子查询没有返回任何行（空集），则子查询的结果是 NULL。 <u>如果方案中可以不返回任何行，则应确保外部查询除处理其他预期结果之外，还能正常处理 NUL</u>L。
- <u>内部查询一般应返回单列</u>。 在子查询中选择多个列几乎都会出错。 唯一例外是，<u>使用 EXISTS 关键字引入子查询时</u>。

```sql
-- 子查询没有返回任何行（空集）时的处理

SELECT c.CustomerName, 
       (SELECT TOP 1 o.Amount
        FROM Orders o
        WHERE o.CustomerID = c.CustomerID
        ORDER BY o.OrderDate DESC) AS LastOrderAmount
FROM Customers c;

-- 改进：用 COALESCE 处理 NULL

SELECT c.CustomerName, 
       COALESCE(
           (SELECT TOP 1 o.Amount
            FROM Orders o
            WHERE o.CustomerID = c.CustomerID
            ORDER BY o.OrderDate DESC), 0) AS LastOrderAmount
FROM Customers c;

/*
输出示例：
CustomerName	LastOrderAmount
Alice	        100
Bob	            NULL
Charlie	        50

通过使用 COALESCE，当没有订单时，NULL 就被转换成了 0：

CustomerName	LastOrderAmount
Alice	        100
Bob	            0
Charlie	        50
*/
```

```sql
-- 子查询返回多个列会出错
-- 假设我们有两个表：Employees 和 Departments。我们想要找出在某个部门工作的员工。以下是一个错误的示例，尝试让子查询返回多个列。

SELECT e.EmployeeName
FROM Employees e
WHERE (e.DepartmentID, e.JobTitle) IN (
    SELECT d.DepartmentID, d.ManagerID  -- 错误：子查询返回多个列
    FROM Departments d
    WHERE d.Location = 'New York'
);

-- 使用 EXISTS 处理多个列
-- EXISTS 关键字是一个例外，它可以用于检查子查询是否返回至少一行数据，而不要求子查询返回单列。即使子查询返回多个列，只要它返回至少一行，EXISTS 就会返回 TRUE。

SELECT e.EmployeeName
FROM Employees e
WHERE EXISTS (
    SELECT 1  -- `EXISTS` 不关心返回的列内容
    FROM Departments d
    WHERE e.DepartmentID = d.DepartmentID
    AND d.Location = 'New York'
);

-- EXISTS 子查询中，即使我们选择了多个列（在此例中，DepartmentID 和 Location），EXISTS 只关心子查询是否有行返回。如果子查询返回至少一行数据，那么外部查询就会返回相应的员工名称。
-- SELECT 1 是一种常见的写法，因为 EXISTS 只关心是否存在匹配的行，而不关心返回哪些列。
```

标量子查询可用于查询中涉及值的任何位置，包括 SELECT 列表。 例如，我们可以将检索到最新订单详细信息的查询扩展到包括已订购商品的平均数量，这样我们就能将最新订单的订购数量与所有订单的平均订购数量进行比较。

### 多值子查询

多值子查询非常适合返回使用 IN 运算符的结果。 以下假设示例将返回加拿大客户下达的所有订单的“CustomerID”和“SalesOrderID”值。

```sql
SELECT CustomerID, SalesOrderID
FROM Sales.SalesOrderHeader
WHERE CustomerID IN ( -- IN 在 SQL 中确实用于查找列值是否等于列表中的任意一个值
    SELECT CustomerID
    FROM Sales.Customer
    WHERE CountryRegion = 'Canada');
```

在此示例中，如果只执行内部查询，则会返回“CustomerID”值列，每行代表加拿大的一位客户。

很多情况下，可以使用 join 轻松写入多值子查询。 例如，以下查询使用 join 来返回与前面示例相同的结果：

```sql
SELECT c.CustomerID, o.SalesOrderID
FROM Sales.Customer AS c
JOIN Sales.SalesOrderHeader AS o
    ON c.CustomerID = o.CustomerID
WHERE c.CountryRegion = 'Canada';
```

那么，<u>怎么决定是以 JOIN 方式写入涉及多个表的查询，还是使用子查询？ 有时，**这取决于你哪种更熟练**</u>。 大多数可轻松转换为 JOIN 的嵌套查询，实际上都会在内部转换为 JOIN。 对于这类查询，使用哪种方式写入并无实际差别。

<u>应谨记的一项限制是，使用嵌套查询时，返回到客户端的结果只能包含外部查询中的列。 因此，如果需要返回两个表中的列，则应使用 JOIN 来写入查询</u>。

最后，在某些情况下，内部查询需要执行示例中的简单检索无法做到的更复杂的操作。 使用 JOIN 重写复杂子查询可能非常困难。 **许多 SQL 开发人员发现，子查询非常适合用于复杂处理，因为使用它们可以将处理分解为更小的步骤**。

```sql
-- Employees 表：包含员工信息（EmployeeID, FirstName, LastName）。
-- SalesOrders 表：包含订单信息（OrderID, EmployeeID, OrderAmount）。
-- 如果我们使用嵌套查询来查找员工和他们的订单金额（假设我们只想得到销售订单中订单金额大于 1000 的员工信息），可以写如下查询：
SELECT EmployeeID, FirstName, LastName
FROM Employees
WHERE EmployeeID IN (
    SELECT EmployeeID
    FROM SalesOrders
    WHERE OrderAmount > 1000
);
--  限制： 在这个查询中，外部查询只能返回 Employees 表的列。如果你需要包括来自 SalesOrders 表的列（比如 OrderAmount），就不能通过这个嵌套查询来直接获取了。

-- 如果你需要从多个表中获取信息，JOIN 更合适。你可以通过 JOIN 连接两个表，并选择你需要的列：
SELECT e.EmployeeID, e.FirstName, e.LastName, s.OrderAmount
FROM Employees e
JOIN SalesOrders s ON e.EmployeeID = s.EmployeeID
WHERE s.OrderAmount > 1000;
-- 我们通过 JOIN 连接了 Employees 表和 SalesOrders 表。
-- 外部查询返回了 Employees 表的列（EmployeeID, FirstName, LastName）和 SalesOrders 表的列（OrderAmount）。
-- 通过 WHERE 子句过滤订单金额大于 1000 的记录。
```



## 使用自包含或关联子查询

以前，我们将自包含子查询看作是内部查询不依赖于外部查询，执行一次，并向外部查询返回结果。 T-SQL 还支持关联子查询，其中内部查询会引用外部查询中的列，理论上每行执行一次。

### 使用关联子查询

同自包含子查询一样，关联子查询也是嵌套在外部查询中的 SELECT 语句。 关联子查询也可以是标量子查询或多值子查询。 它们通常用于内部查询需要引用外部查询中值的情况。

但是，与自包含子查询不同的是，使用关联子查询时有一些特殊注意事项：

- <u>关联子查询不能与外部查询分开运行。 此限制会使测试和调试变得有些复杂。</u>
- 与自包含子查询处理一次不同，关联子查询将运行多次。 逻辑上，外部查询先运行，对于返回的每一行，内部查询都会处理。

```sql
SELECT EmployeeID, Name
FROM Employees e
WHERE EXISTS (
    SELECT 1
    FROM Orders o
    WHERE o.EmployeeID = e.EmployeeID
);

-- 这就是 关联子查询（Correlated Subquery），它会对 Employees 表中的每一行执行一次，检查该员工是否在 Orders 表中有对应的记录。

-- 由于子查询依赖于外部查询，它无法单独运行，因此无法直接测试子查询的结果。
-- 调试时，通常需要先单独运行 类似的非关联查询 来检查 Orders 表中是否有 EmployeeID，例如：

SELECT DISTINCT EmployeeID FROM Orders;

-- 然后再使用 EXISTS 语句进行联调。
```

以下示例将使用关联子查询返回每个客户的最近订单。 子查询引用外部查询，并在 WHERE 子句中引用其“CustomerID”值。 对于外部查询中的每一行，子查询都会查找该行中所引用客户的最大订单 ID，而外部查询将检查其正在查看的行是否是包含该订单 ID 的行。

```sql
SELECT SalesOrderID, CustomerID, OrderDate
FROM SalesLT.SalesOrderHeader AS o1
WHERE SalesOrderID =
    (SELECT MAX(SalesOrderID)
     FROM SalesLT.SalesOrderHeader AS o2
     WHERE o2.CustomerID = o1.CustomerID)
ORDER BY CustomerID, OrderDate;
```

### 写入关联子查询

要写入关联子查询，请考虑以下准则：

- <u>写入外部查询以接受来自内部查询的适当返回结果。 如果内部查询是标量查询，可以在 WHERE 子句中使用等于和比较运算符，例如 =、<、> 和 <>。 如果内部查询可能返回多个值，则使用 IN 谓词。 计划处理 NULL 结果。</u>
- 标识关联子查询将会引用的外部查询中的列。 对作为外部查询中列源的表声明一个别名。
- 标识内部表中要与外部表中列进行比较的列。 为源表创建一个别名，方法与针对外部查询的处理方式相同。
- 写入内部查询，以根据外部查询的输入值从其源中检索值。 例如，在内部查询的 WHERE 子句中使用外部列。

当内部查询引用外部值进行比较时，内外查询之间将会发生关联。 正是这种关联，让该子查询名副其实。

### 使用 EXISTS

除从子查询中检索值之外，T-SQL 还提供了一种检查查询是否会返回任何结果的机制。 <u>EXISTS 谓词可确定是否存在满足指定条件的任何行，但不会返回这些行，而是返回 TRUE 或 FALSE</u>。 这种方法对于验证数据非常有用，不会产生检索和处理结果的开销。

<u>当子查询与使用 EXISTS 谓词的外部查询关联时，SQL Server 将以特殊方式来处理子查询的结果。</u> EXISTS 只需检查结果中是否存在任何行，而不是从子查询中检索标量值或多值列表。

理论上，EXISTS 谓词相当于检索结果，对返回的行计数，再将计数与零进行比较。 比较以下查询，这些查询将返回已下单客户的详细信息：

第一个示例查询在子查询中使用 COUNT：

```sql
SELECT CustomerID, CompanyName, EmailAddress 
FROM Sales.Customer AS c 
WHERE
(SELECT COUNT(*) 
  FROM Sales.SalesOrderHeader AS o
  WHERE o.CustomerID = c.CustomerID) > 0;
```

第二个查询使用 EXISTS 返回相同的结果：

```sql
SELECT CustomerID, CompanyName, EmailAddress 
FROM Sales.Customer AS c 
WHERE EXISTS
(SELECT * 
  FROM Sales.SalesOrderHeader AS o
  WHERE o.CustomerID = c.CustomerID);
```

在第一个示例中，子查询必须对在“Sales.SalesOrderHeader”表中找到的每个“custid”匹配项计数，并将计数结果与零比较，这样才能指示该客户已下单。

在第二个查询中，只要在“Sales.SalesOrderHeader”表中找到相关订单，EXISTS 就会立即对该“custid”返回 TRUE。 不需要完整计算每个匹配项。 另请注意，使用 EXISTS 形式时，子查询并不局限于返回单列。 在此，我们使用 SELECT *。 返回的列无关紧要，因为我们只是检查有无返回任何行，而不是这些行中的值。

从逻辑处理角度来看，这两种查询形式效果相当。 从性能角度来看，数据库引擎对查询的处理可能有所不同，因为它会优化查询后执行处理。 请针对自己的使用情况测试每种查询。

**如果要将使用 COUNT(*) 的子查询转换为使用 EXISTS 的子查询，请确保子查询使用 SELECT *，而不是 SELECT COUNT(*)。 SELECT COUNT(*) 始终返回一行，因此 EXISTS 将始终返回 TRUE。**

EXISTS 的另一种有用情形是使用 NOT 对子查询求反，如以下示例所示，结果将返回从未下过订单的所有客户：

```sql
SELECT CustomerID, CompanyName, EmailAddress 
FROM SalesLT.Customer AS c 
WHERE NOT EXISTS
  (SELECT * 
   FROM SalesLT.SalesOrderHeader AS o
   WHERE o.CustomerID = c.CustomerID);
```

SQL Server不必一定返回已下过订单客户的相关订单数据。 如果在“Sales.SalesOrderHeader”表中找到“custid”，则 NOT EXISTS 计算为 FALSE，并将很快完成计算。

要对使用 EXISTS 的查询写入子查询，请考虑以下准则：

- 关键字 EXISTS 直接位于 WHERE 后。 除非也使用 NOT，否则前面不加任何列名称（或其他表达式）。
- 在子查询中使用 SELECT *。 子查询不返回任何行，因此无需指定任何列。



[练习 - 使用子查询](https://microsoftlearning.github.io/dp-080-Transact-SQL/Instructions/Labs/03b-subqueries.html)

```sql
--SELECT Name FROM SalesLT.Product
--WHERE ProductID IN
--	(SELECT  DISTINCT ProductID
--	 FROM SalesLT.SalesOrderDetail 
--     WHERE OrderQty >= 20);

--SELECT DISTINCT Name
--FROM SalesLT.Product As p
--JOIN SalesLT.SalesOrderDetail AS o
--	on p.ProductID = o.ProductID
--WHERE OrderQty >= 20;

-- 运行查询并注意它会返回相同的结果。通常，您可以使用子查询或连接来实现相同的结果，而且对于查看代码的开发人员来说，子查询方法通常比复杂的连接查询更容易理解，因为操作可以分解为离散的组件。在大多数情况下，等效的连接或子查询操作的性能是相似的，但在某些需要执行存在性检查的情况下，连接的性能更好。
     
--SELECT p.Name
--FROM SalesLT.Product AS p
--WHERE EXISTS (
--    SELECT 1
--    FROM SalesLT.SalesOrderDetail AS o
--    WHERE o.ProductID = p.ProductID
--      AND o.OrderQty >= 20
--);

--说到“存在性检查”，最标准、性能最稳的写法是 EXISTS：
--为什么 EXISTS 特别快？
--只关心：有没有
--找到第一行就返回 TRUE
--不生成中间结果集
--不关心返回列
--和 JOIN 一样可以被优化成半连接（Semi Join）
--在 SQL Server 里，EXISTS 很容易被编译成：
--Semi Join（半连接）执行计划
--这正是“存在性检查”的理想执行方式。
```

但在某些需要执行存在性检查的情况下，连接的性能更好。为什么会这么说？

你第一个写法本质是：

①【子查询**必须先算完**】先执行子查询：扫描 `SalesOrderDetail`→找出所有 `OrderQty >= 20` 的 `ProductID`→去重（`DISTINCT`）

②【即使外层只需要判断「存不存在」，它也得把**所有符合条件的 ProductID 全找出来**】把结果集“物化”：放到临时表 / 哈希结构中

③【`DISTINCT` 会引入 **排序 / 哈希去重成本**】再和外层 Product 表逐条比对

这是一种“先算全量，再过滤”的思路

对于第二写法：

**优化器可能采取的策略**

- **[Nested Loop](https://www.cnblogs.com/xiaohuizhenyoucai/p/10983783.html) + Index Seek**
- 对 `SalesOrderDetail(ProductID, OrderQty)` 做索引查找
- **一旦找到第一条满足条件的行，立刻停止扫描**

这叫：**Short-Circuit（短路执行）**

不需要把所有匹配行都扫出来
 不需要 DISTINCT 子查询的完整结果集



## 使用标量函数

**使用标量函数时需注意以下事项：**

确定性：若函数每次在相同输入和数据库状态下均返回相同值，则称其具有确定性。例如，ROUND(1.1, 0) 始终返回值 1.0。许多内置函数具有非确定性，例如 GETDATE() 返回当前日期和时间。非确定性函数的结果无法建立索引，这会影响查询处理器制定高效查询执行计划的能力。

排序规则：使用处理字符数据的函数时，应采用哪种排序规则？部分函数采用输入值的排序规则；若未提供输入排序规则，其他函数则采用数据库的排序规则。

## [Get started with Transact-SQL programming](https://learn.microsoft.com/en-us/training/modules/get-started-transact-sql-programming/)

### Describe T-SQL for programming

> Finally, you might want to create aliases, or pointers, to objects so that you can reference them by a different name or from a different location than where they're defined.
>
> 最后，您可能希望为对象创建别名（或指针），以便能够通过不同的名称或从与定义位置不同的地方来引用它们。

这里的“指针”其实是一个**类比**，它指的是一种**逻辑上的重定向**。在 SQL Server 或其他数据库中，这种功能真正的名字叫：**同义词（Synonym）**。

------

**1. 什么是同义词（Synonym）？**

同义词就是为一个已有的数据库对象（表、视图、存储过程等）起一个“别名”。

- **它是指针吗？** 是的，因为它不存储数据，它只是指向了另一个对象的“位置”。
- **它的价值：** 隐藏对象的真实名称、位置（甚至可以在不同的服务器上），并提供一个稳定的访问入口。

------

**2. 为什么需要这种“指针”？（应用场景）**

想象一下，你有一张真实的表存在远程服务器 `Server_B` 上，它的全名长得离谱： `Server_B.SalesDB.dbo.Tbl_Yearly_Sales_Report_2026`

**如果不使用同义词：**

你每次写查询都要写这一长串名字。如果明天数据库从 `Server_B` 搬到了 `Server_C`，你所有的代码都得改。

**如果使用同义词（“指针”）：**

你可以创建一个名为 `Sales` 的同义词：

现在，你的代码只需要写： `SELECT * FROM Sales;`

**如果后台数据库搬家了，你只需要改一下同义词的定义，你的应用程序代码一行都不用动。** 这就是它像“指针”的地方：你操作的是指针，指针指向哪里由后台配置。

------

**3. SQL 中几种“别名”的区别**

为了防止混淆，我们将你之前提到的几种“起名”方式做个对比：

------

**4. 总结**

在 SQL 的语境下，说“指针”是指**“位置透明性”**：

1. **跨服务器引用**：指向远程服务器的对象。
2. **重命名**：给难听的表名起个好听的名字。
3. **架构解耦**：当底层对象变化时，顶层代码保持不变。



### Describe batches


>批处理是变量作用域的边界，这意味着在一个批处理中定义的变量仅能在同一批处理中被其他代码引用。
>
>某些语句（通常是数据定义语句，如 `CREATE VIEW` 、 `CREATE FUNCTION` 和 `CREATE PROCEDURE` ）无法在同一批处理中与其他语句合并。

简单来说，**批处理（Batch）** 就是你交给数据库引擎一次性执行的一组命令。在 SQL Server Management Studio (SSMS) 等工具中，我们通常使用 **GO** 关键字来分隔不同的批处理。

你提到的规则主要规定了 **变量的生命周期** 和 **特定命令的执行要求**。

------

**1. 变量作用域的边界 (Scope Boundary)**

在 SQL Server 中，变量的寿命仅限于它所在的批处理。这意味着在 GO 之前定义的变量，在 GO 之后就会失效。

**错误代码示例：**

```SQL
DECLARE @MyName NVARCHAR(20) = 'Gemini';
SELECT @MyName;
GO  -- 批处理结束，变量 @MyName 在此处被注销

SELECT @MyName; -- 报错：必须声明标量变量 @MyName
```

**正确做法：** 如果你需要在后续代码中使用该变量，请不要在中间插入 GO，或者在新的批处理中重新声明并赋值。

------

**2. DDL 语句的独立执行要求 (Batch Requirement)**

某些数据定义语言（DDL），如 `CREATE VIEW`、`CREATE PROCEDURE`、`CREATE TRIGGER` 等，要求在当前的批处理中必须作为第一个语句出现。你不能在同一个批处理中先执行其他逻辑再创建这些对象。

**错误代码示例：**

```SQL
DECLARE @ViewName NVARCHAR(20) = 'TestView'; -- 动作 1

CREATE VIEW MyView AS                       -- 动作 2：此处会报错
SELECT * FROM Employees;
-- 错误提示：'CREATE VIEW' 必须是查询批处理中的第一个语句。
```

**正确做法（使用 GO 分批）：** 你需要使用 GO 关键字将语句拆散，确保 `CREATE VIEW` 成为其所在批处理的起始语句。

```SQL
DECLARE @Message NVARCHAR(50) = '开始创建视图...';
PRINT @Message;
GO  -- 结束当前批处理

CREATE VIEW MyView AS 
SELECT * FROM Employees;
GO  -- 结束创建视图的批处理

PRINT '视图创建完成！';
GO
```

------

**3. 为什么会有这种限制？**

- **编译与解析逻辑**：数据库在执行一个批处理之前，会先对其进行整体的语法校验和编译。将特定的 DDL 语句独立出来，可以确保编译器在处理对象定义时不会受到其他上下文逻辑的干扰。
- **内存与资源管理**：变量存储在内存的临时空间中。当一个批处理结束（遇到 GO）时，数据库会清空该批处理相关的执行上下文和局部变量，以释放内存并确保不同任务之间的隔离性。

> FUNCTION和PROCEDURE又有怎样的区别?

这是一个非常经典的问题。在 SQL 的世界里，**函数 (Function)** 和 **存储过程 (Procedure)** 就像是两种不同性格的工具：函数像是一个**“计算器”**，必须有输入输出；而存储过程像是一个**“脚本/批处理”**，它可以做任何事。

它们最核心的区别在于：**函数主要用于计算并返回结果，而存储过程主要用于执行一系列复杂的业务逻辑。**

------

**1. 核心差异对比表**

| **特性**     | **函数 (Function)**                          | **存储过程 (Procedure)**              |
| ------------ | -------------------------------------------- | ------------------------------------- |
| **返回值**   | **必须**返回一个值（或表）                   | 可以不返回，也可以返回多个结果集      |
| **调用方式** | 可以在 `SELECT` 或 `WHERE` 中直接调用        | 必须使用 `EXEC` 或 `CALL` 独立调用    |
| **副作用**   | **严禁**修改数据库数据（只能查，不能增删改） | **允许**进行增、删、改等各种操作      |
| **事务支持** | 不允许在内部使用事务                         | 支持完整的事务处理（Commit/Rollback） |
| **参数**     | 只有输入参数                                 | 支持输入参数和输出参数 (OUTPUT)       |

------

**2. 形象的比喻**

- **函数 (Function)：** 就像 Excel 里的 `SUM(A1:A10)`。你给它数据，它还你一个结果。它不会在你算账的时候偷偷去修改你的员工表名。
- **存储过程 (Procedure)：** 就像一个“一键入职脚本”。你运行它，它会先在员工表插一条数据，再去权限表开通账号，最后发一封邮件。它是一个完整的**动作流程**。

------

**3. 代码举例说明**

**函数 (Function) —— 侧重于“算”**

你可以在查询数据时，像处理内置函数一样使用它。

```SQL
-- 定义：计算含税价格
CREATE FUNCTION dbo.GetTaxPrice (@price DECIMAL(10,2))
RETURNS DECIMAL(10,2)
AS
BEGIN
    RETURN @price * 1.13;
END;
GO

-- 调用：直接嵌入在 SELECT 语句中
SELECT Name, dbo.GetTaxPrice(Price) AS PriceWithTax FROM Products;
```

**存储过程 (Procedure) —— 侧重于“做”**

它是一个独立的逻辑块，可以包含复杂的判断和修改操作。

```SQL
-- 定义：办理员工调薪，包含日志记录和修改
CREATE PROCEDURE dbo.UpdateEmployeeSalary
    @EmpID INT,
    @NewSalary DECIMAL(10,2)
AS
BEGIN
    BEGIN TRANSACTION; -- 支持事务
    
    UPDATE Employees SET Salary = @NewSalary WHERE ID = @EmpID;
    INSERT INTO SalaryLog(EmpID, UpdateDate) VALUES (@EmpID, GETDATE());
    
    COMMIT;
END;
GO

-- 调用：必须独立执行
EXEC dbo.UpdateEmployeeSalary @EmpID = 101, @NewSalary = 8000;
```

------

**4. 为什么函数不能修改数据？**

这是一个为了**确定性**的设计。想象一下，如果你在 `SELECT * FROM Orders` 的时候调用了一个会删除数据的函数，那么每查出一行，数据库就少了一行数据，这会导致查询结果完全不可预测。因此，数据库强制规定：函数必须是“只读”的。

------

**总结：我该选哪一个？**

- 如果你只是想把一段**通用的计算逻辑**（比如格式化字符串、算税率）封装起来，方便在各种查询里复用，选 **Function**。
- 如果你需要**处理业务逻辑**（比如转账、批量备份、跨表更新），或者需要执行 DDL 语句（建表等），选 **Procedure**。



```sql
-- 定义：办理员工调薪，包含日志记录和修改
CREATE PROCEDURE dbo.UpdateEmployeeSalary
    @EmpID INT,
    @NewSalary DECIMAL(10,2)
AS
BEGIN
    BEGIN TRANSACTION; -- 支持事务
    
    UPDATE Employees SET Salary = @NewSalary WHERE ID = @EmpID;
    INSERT INTO SalaryLog(EmpID, UpdateDate) VALUES (@EmpID, GETDATE());
    
    COMMIT;
END;
GO

-- 调用：必须独立执行
EXEC dbo.UpdateEmployeeSalary @EmpID = 101, @NewSalary = 8000;

-- 这里面的AS是起什么作用，BEGIN就是简单的开始吗，外面的BEGIN开始再BEGIN TRANSACTION，TRANSACTION是什么关键字吗?
```

这是一个非常棒的细节观察！这些关键字是 SQL 编程（特别是 T-SQL）中构建逻辑块的“骨架”。我们一个一个拆解开来看：

**1. `AS` 的作用：定义与实现的分界线**

在 `CREATE` 语句中，`AS` 的作用是**“衔接”**。

- **`AS` 之前**：是过程或函数的**签名**（名字、参数列表）。它告诉数据库：“我要创建一个叫什么名字、需要什么零件的东西”。
- **`AS` 之后**：是过程或函数的**具体内容**。它告诉数据库：“这个东西具体是怎么运作的”。

你可以把它翻译成：“其内容如下：” 或者 “作为：”。

------

**2. `BEGIN...END`：代码的“括号”**

`BEGIN` 和 `END` 在 SQL 中其实就是编程语言里的**大括号 `{ }`**。

- 它们的作用是将多条 SQL 语句打包成一个**逻辑块**。
- 在存储过程里，由于你有很多动作（更新、插入、提交），必须用 `BEGIN...END` 把它们圈起来，告诉编译器：这整堆东西都属于这个存储过程。

------

**3. `TRANSACTION` 与 `BEGIN TRANSACTION`**

这是数据库中最核心的概念之一：**事务**。

- **`TRANSACTION` 是什么？** 它是一个关键字，代表一个**不可分割的工作单位**。它的原则是：**要么全做，要么全不做**。
- **`BEGIN TRANSACTION` 是什么意思？** 它是在告诉数据库：“从现在起，我接下来的操作都是暂时的。除非我最后说 `COMMIT`（提交），否则不要把这些更改永久写死到硬盘里。”

------

**4. 为什么会有两个 `BEGIN`？它们不一样吗？**

这就是你代码里最容易混淆的地方。虽然都叫 `BEGIN`，但它们的职责完全不同：

- **外层的 `BEGIN` (与 `END` 配对)**： 这是存储过程的**结构边界**。它只是在划定代码范围。
- **内层的 `BEGIN TRANSACTION`**： 这是一个**功能指令**。它启动了数据库的“保护模式”。

**如果我们把这个过程比喻成去银行转账：**

1. **`CREATE PROCEDURE`**：这就是银行的柜台业务说明书。
2. **第一个 `BEGIN`**：业务员翻开说明书，准备开始操作。
3. **`BEGIN TRANSACTION`**：业务员在电脑里点击“启动事务”，此时他操作的一切都在缓存里，还没真的把钱划走。
4. **`UPDATE` 和 `INSERT`**：业务员在账本上写写画画（如果此时突然停电，钱不会丢，因为还没确认）。
5. **`COMMIT`**：业务员敲下回车键，数据正式写入硬盘，钱正式到账。
6. **最后一个 `END`**：业务员合上说明书，业务结束。

------

**💡 如果不写 `BEGIN TRANSACTION` 会怎样？**

如果不写这个，SQL 会进入 **“自动提交模式”**：

- 第一步 `UPDATE` 执行完，数据就立刻改了。
- 此时如果第二步 `INSERT` 失败（比如日志表满了），你会发现员工薪水改了，但日志没记下来。
- **这就会导致数据不一致。** 加上 `TRANSACTION` 就是为了防止这种“半吊子”情况发生。



### Declare and assign variables and synonyms

声明变量后，必须进行初始化或为其赋值。可通过以下三种方式完成：

- 在 SQL Server 2008 或更高版本中，您可以使用 DECLARE 语句声明并初始化变量。
- 在所有 SQL Server 版本中，均可通过 SET 语句为变量赋予单个（标量）值。
- 在所有 SQL Server 版本中，均可通过 SELECT 语句为变量赋值，但务必确保 SELECT 语句仅返回一行。若结果集为空，则变量保持原值；若返回多行结果，则将取最后一行作为赋值依据。

> 您可以创建一个指向尚不存在对象的同义词（SYNONYM），这被称为延迟名称解析。在运行时使用该同义词（SYNONYM）之前，SQL Server 引擎不会检查实际对象是否存在。

> 用户若需创建同义词（SYNONYM），则必须具备 CREATE SYNONYM 权限，同时需拥有目标架构的所有权或该架构的 ALTER SCHEMA 权限。

[Program with Transact-SQL](https://microsoftlearning.github.io/dp-080-Transact-SQL/Instructions/Labs/10-program-with-tsql.html)

## [Create tables, views, and temporary objects](https://learn.microsoft.com/en-us/training/modules/create-tables-views-temporary-objects/)

### Create and query views

> 视图是数据库中可创建的已保存查询。一个视图能够引用一张或多张表。与表类似，视图也由数据行和列构成。您可以像使用表一样，将视图作为查询的数据源。但视图本身并不持久存储数据：其定义会在运行时展开，并直接查询所引用的源对象。

<u>**我可以认为视图是缓存的一种形式吗?**</u>

**总结建议**

- 如果你是为了**安全控制**（隐藏敏感列）或**简化代码**（把复杂的 Join 封装起来），请使用**普通视图**。
- 如果你是为了**性能优化**（底层表数据量巨大，计算逻辑复杂），你应该考虑**物化视图**或应用层的缓存方案（如 Redis）。

**你可以把普通视图想象成一个“滤镜”：** 它没有改变物体本身，也没有复制一份物体，只是让你看物体的角度变了。

<u>**所以视图就是封装了下查询语句？**</u>

你可以把视图理解为数据库里的**“快捷方式”**或**“宏定义”**：

- **它不产生新数据。**
- **它不占用额外空间（除了存那几行定义语句）。**
- **它主要的目的是为了好用、安全和易于维护。**

```sql
CREATE VIEW <schema_name.view_name> [<column_alias_list>] 
[WITH <view_options>]
AS select_statement;
```

**A. `WITH CHECK OPTION`（最常用）**

视图不仅可以查，有时还可以通过视图往原表里**写数据**。这个选项能确保你写进去的数据，必须符合视图本身的过滤条件。

- **例子：** 你创建了一个“只看 18 岁以上员工”的视图。如果你加了 `WITH CHECK OPTION`，那么有人想通过这个视图插入一个 15 岁的员工时，数据库会直接报错拦截。

**B. `WITH SCHEMABINDING`（常见于 SQL Server）**

这是一种“强绑定”模式。

- **作用：** 只要这个视图存在，你就**不能修改或删除**它引用的底层表结构。
- **好处：** 防止别人不小心删了表，导致你的视图突然“挂掉”。

**C. `WITH ENCRYPTION`**

- **作用：** 把视图背后的 SQL 语句加密，不让别人通过系统表看到你的查询逻辑。



### Use temporary tables

**<u>全局临时表和表性能上有什么区别吗?</u>**

这是一个非常硬核的性能问题。在数据库（以 SQL Server 或 PostgreSQL 为例）中，**全局临时表（Global Temporary Tables，通常以 `##` 开头）**和**普通永久表（Regular Tables）**在底层存储上其实非常相似，但它们的“性格”和“生存环境”决定了性能差异。

简单来说：**短期爆发力临时表强，长期稳定性永久表胜。**

------

**1. 核心差异对比表**

| **特性**       | **全局临时表 (TempTable)**          | **普通永久表 (Regular Table)**         |
| -------------- | ----------------------------------- | -------------------------------------- |
| **存储位置**   | 专门的 `tempdb` 数据库（内存+磁盘） | 用户数据库的磁盘文件                   |
| **日志开销**   | **较低**（Minimal Logging）         | **较高**（Full Logging，为了数据恢复） |
| **锁竞争**     | **高**（所有会话共享 `tempdb`）     | **低**（只在所属数据库内竞争）         |
| **元数据维护** | 频繁创建/删除，增加系统表压力       | 结构稳定，元数据开销极小               |
| **生命周期**   | 会话结束或最后一个引用消失即销毁    | 永久存在，直到手动 DROP                |

------

**2.性能上的三大具体差异**

**① 事务日志（Write-Ahead Logging）—— 临时表更快**

这是性能差异最明显的地方。

- **普通表：** 为了保证崩溃后数据能恢复，每一条 `INSERT` 都会被完整记录在日志中（Full Logging）。
- **临时表：** 因为它是临时的，数据库通常采用“最小化日志记录”。这意味着在高并发写入时，临时表的 I/O 压力更小，写入速度往往比普通表快得多。

**② 资源竞争（Resource Contention）—— 临时表可能变慢**

虽然单个操作快，但全局临时表有一个致命伤：**`tempdb` 瓶颈**。

- 全球临时表存在于 `tempdb` 中。如果你的服务器上有很多程序都在疯狂使用临时表，大家会一起挤占 `tempdb` 的空间和文件分配锁（PFS/GAM 页面竞争）。
- **普通表**分散在各自的数据库文件中，互不干扰，更适合并行的大规模负载。

**③ 统计信息与执行计划 —— 永久表更稳定**

- **普通表：** 数据库有完整的统计信息（Statistics），优化器能算出最精准的查询路径。
- **临时表：** 统计信息可能是不准确的，因为数据波动太快。如果你的 `##` 表里有100万行数据，但优化器还以为它是空的，那你的查询计划会变得非常糟糕。

------

**3. 什么时候该选谁？**

- **选 全局临时表 的场景：**
  - 需要在多个不同连接（Session）之间共享一团中间数据。
  - 数据量不大（几万行以内），且处理完就扔，不想污染正式数据库。
  - 追求极速的写入，且不担心系统崩溃导致这部分数据丢失。
- **选 普通表 的场景：**
  - 数据量巨大（GB 级别），需要建立复杂的索引。
  - 需要严格的权限控制。
  - 数据需要持久化，或者需要进行数据库备份/还原。

**💡 一个关键的小坑**

很多开发者误以为“全局临时表”是存在内存里的。 **纠正：** 它依然会被写入磁盘（`tempdb` 的文件里），只是它比普通表更倾向于留在内存缓存中。如果 `tempdb` 所在的磁盘很慢，临时表的性能也会断崖式下跌。



### Use Common Table Expressions


> 公用表表达式（CTE）需要为表表达式命名，同时还需为 CTE 的 SELECT 子句中引用的每个列指定唯一名称。
>
> 公用表表达式（CTE）可为列使用内联或外部别名。
>
> 与派生表不同，同一个查询中可通过单一定义多次引用公用表表达式（CTE）。同一 WITH 子句中也可以定义多个公用表表达式（CTE）。
>
> CTE 支持递归，即在定义表达式时可以引用自身。递归 CTE 超出了本模块的讨论范围。

**1.内联别名 vs. 外部别名 (Inline vs. External Aliases)**

CTE 要求每一列都必须有唯一的名称。你可以像在普通 `SELECT` 里那样起名（内联），也可以在 CTE 头部统一声明（外部）。

**内联别名 (Inline)**

直接在 `SELECT` 语句中使用 `AS`。

```SQL
WITH DeptSalary AS (
    SELECT DeptID, SUM(Salary) AS TotalSalary -- 内联定义列名
    FROM Employees
    GROUP BY DeptID
)
SELECT * FROM DeptSalary;
```

**外部别名 (External)**

在 CTE 名称后的括号内统一列出所有列名。这种方式让结构更清晰。

```SQL
WITH DeptSalary (部门编号, 总薪水) AS ( -- 外部定义列名
    SELECT DeptID, SUM(Salary)
    FROM Employees
    GROUP BY DeptID
)
SELECT * FROM DeptSalary;
```

------

**2.同一查询中多次引用 (Multiple References)**

这是 CTE 碾压派生表的地方。你可以定义一次，然后在接下来的查询中像对待普通表一样多次 `JOIN` 它。

**场景：** 找出薪水高于所在部门平均薪水的员工。

```SQL
WITH DeptAvg AS (
    SELECT DeptID, AVG(Salary) AS AvgSal
    FROM Employees
    GROUP BY DeptID
)
SELECT e.Name, e.Salary, da.AvgSal
FROM Employees e
JOIN DeptAvg da ON e.DeptID = da.DeptID -- 第一次引用
WHERE e.Salary > da.AvgSal;
```

------

**3.WITH 子句定义多个 CTE (Multiple CTEs)**

如果你有多个逻辑块，不需要写多个 `WITH`，只需用**逗号**隔开即可。后面的 CTE 甚至可以引用前面已经定义好的 CTE。

**场景：** 先算部门平均分，再算全公司总平均分。

```SQL
WITH 
DeptStats AS ( -- 第一个 CTE
    SELECT DeptID, AVG(Salary) AS DeptAvg FROM Employees GROUP BY DeptID
),
CompanyStats AS ( -- 第二个 CTE，引用了第一个 CTE
    SELECT AVG(DeptAvg) AS GlobalAvg FROM DeptStats
)
SELECT * FROM DeptStats, CompanyStats 
WHERE DeptStats.DeptAvg > CompanyStats.GlobalAvg;
```

------

**4.递归 CTE (Recursive CTE) —— 简单看个一眼**

虽然你的模块说“超出范围”，但为了让你直观感受它的“自我引用”，看这个最经典的：**生成 1 到 5 的数字序列**。

```SQL
WITH RECURSIVE Counter AS (
    SELECT 1 AS n           -- 锚点部分：从 1 开始
    UNION ALL
    SELECT n + 1 FROM Counter -- 递归部分：引用自己，在原来的基础上 +1
    WHERE n < 5             -- 终止条件：到 5 停止
)
SELECT * FROM Counter;
```

------

总结一下

- **别名**：给封装的“盒子”和里面的“零件”贴标签。
- **多次引用**：一次定义，到处运行，不用复制粘贴。
- **多个定义**：像写文章的分段，一段逻辑一个 CTE，最后再拼起来。
- **递归**：逻辑上的“循环”，让 SQL 具备了处理复杂层次结构的能力。



### Write queries that use derived tables

#### 向派生表传递参数

> 定义派生表的嵌套 SELECT 语句必须为其指定别名。外部查询将在其 SELECT 语句中使用该别名，这基本上类似于在 FROM 子句中引用带别名的连接表的方式。
>
> 派生表 SELECT 子句中引用的所有列都应分配别名，这是在 Transact-SQL 中虽非强制但推荐的最佳实践。每个别名在表达式内必须保持唯一性。列别名可以在列声明时内联指定，也可以在子句外部单独声明。
>
> 定义派生表表达式的 SELECT 语句不得使用 ORDER BY 子句，除非同时包含 TOP 运算符、OFFSET/FETCH 子句或 FOR XML 子句。因此，派生表不提供排序顺序。您需要在外部查询中对结果进行排序。
>
> 用于定义派生表的 SELECT 语句可以编写为接受局部变量形式的参数。如果该 SELECT 语句嵌入在存储过程中，这些参数可编写为存储过程的参数。
>
> 嵌套在外层查询中的派生表表达式可以包含其他派生表表达式。嵌套是允许的，但不建议这样做，因为会增加复杂性并降低可读性。
>
> 在外层查询中，派生表不能多次引用。如果需要处理相同的结果，每次都需要定义派生表表达式，例如在 JOIN 运算符的每一侧。

为上面的情况举个例子

**1. 强制别名与引用 (Alias Required)**

**规则：** 嵌套的 `SELECT` 必须有别名，外部查询像引用普通表一样使用它。

```SQL
SELECT d.YearlyRevenue  -- 像引用表名一样使用别名 d
FROM (
    SELECT SUM(Amount) AS YearlyRevenue 
    FROM Sales
) AS d; -- 必须给这个括号里的东西起个名字（d）
```

------

**2.列别名的两种写法 (Inline vs. External)**

**规则：** 推荐给所有列起名，且必须唯一。

- **内联 (Inline):** 直接在子查询里写 `AS`。
- **外部 (External):** 在派生表名后的括号里统一定义。

```SQL
-- 外部声明列别名（推荐实践）
SELECT d.Total, d.Average
FROM (
    SELECT SUM(Price), AVG(Price)
    FROM Products
) AS d(Total, Average); -- 这里定义了列的名字
```

------

**3. ORDER BY 的限制**

**规则：** 除非有 `TOP` 或 `OFFSET`，否则派生表内部不能排序。排序动作应发生在最外层。

```SQL
-- 错误示例：SELECT * FROM (SELECT Name FROM Users ORDER BY Name) AS d; (会报错)

-- 正确做法：
SELECT d.Name
FROM (
    SELECT Name FROM Users
) AS d
ORDER BY d.Name; -- 在外部查询进行排序
```

------

**4. 接受局部变量或存储过程参数**

**规则：** 派生表不是死板的，它可以动态响应变量。

```SQL
DECLARE @MinSalary INT = 5000;

SELECT d.EmployeeName
FROM (
    SELECT Name AS EmployeeName
    FROM Employees
    WHERE Salary > @MinSalary -- 使用了局部变量
) AS d;
```

------

**5. 嵌套派生表 (Nesting)**

**规则：** 允许嵌套（套娃），但会变得很难读。

```SQL
SELECT d1.FinalResult
FROM (
    SELECT d2.SubTotal * 1.1 AS FinalResult
    FROM (
        SELECT SUM(Amount) AS SubTotal FROM Orders -- 第二层嵌套
    ) AS d2
) AS d1;
```

------

**6. 不能多次引用 (Multiple References)**

**规则：** 如果你想在 `JOIN` 两边用同样的数据，派生表必须写两次（这正是它的弱点，CTE 只需要写一次）。

```SQL
-- 必须定义两次完全一样的逻辑
SELECT *
FROM (SELECT ID, Score FROM Exams) AS d1
JOIN (SELECT ID, Score FROM Exams) AS d2 
  ON d1.ID = d2.ID;
```

------

**总结对比：派生表 vs. CTE**

从你给出的这些规则可以清楚地看到：

- **派生表**是“**就地解决**”，它嵌入在 `FROM` 子句中，适合一次性、简单的逻辑。
- **CTE** 是“**声明后再使用**”，它解决了派生表**无法多次引用**和**嵌套可读性差**的痛点。

**既然看到了派生表在 `JOIN` 中必须重复定义的缺点，你想对比看看同一个逻辑用 CTE 写出来会精简多少吗？**

[Create queries with table expressions](https://microsoftlearning.github.io/dp-080-Transact-SQL/Instructions/Labs/06-use-table-expressions.html)

## [Create stored procedures and user-defined functions](https://learn.microsoft.com/en-us/training/modules/create-stored-procedures-table-valued-functions/?utm_source=chatgpt.com)

### Call stored procedures

```sql
-- 要检查确切的系统过程名称，请使用目录视图：

sys.system_objects
sys.system_parameters

-- 系统存储过程以 sp_ 为前缀。系统存储过程并非由用户创建，而是所有用户定义和系统定义数据库的组成部分。执行它们不需要完全限定的名称，但最佳做法是包含 sys 架构名称。例如：

EXEC sys.sp_who;
```

> `sys.system_objects` 和 `sys.system_parameters` 是干嘛的？什么叫“不需要完全限定名”？

它们不是函数，而是**系统视图**（你可以把它们当成只读的表）。

- **`sys.system_objects`**：这是**“官方功能清单”**。 它记录了数据库自带的所有系统级对象（包括存储过程、视图、系统表等）。如果你想找某个 `sp_` 开头的功能在哪里，查它就行。

  ```sql
  -- 查找所有名字里带 'who' 的系统存储过程
  SELECT name, type_desc 
  FROM sys.system_objects 
  WHERE name LIKE '%who%';
  ```

- **`sys.system_parameters`**：这是**“官方功能参数说明书”**。 它记录了这些系统存储过程需要接收什么参数（比如你是要查所有人的进程，还是某个特定用户的进程）。

2. “不需要完全限定名” vs “最佳做法”

这部分其实是在聊**“寻址”**的问题。

**什么叫“不需要完全限定名”？**

通常我们要访问别的数据库的东西，要写 `数据库名.架构名.对象名`。 但系统过程太常用，数据库做了特殊处理：无论你在哪个数据库（Master, Sales, HR），只要你喊 `EXEC sp_who`，数据库引擎都会自动去 `sys` 架构里帮你找。

**为什么“最佳做法”是包含 `sys` 架构名？**

如果你直接写 `EXEC sp_who`，数据库会经历这样一个过程：

1. 在当前数据库的当前用户架构里找。
2. 没找到，再去系统架构里找。

如果你直接写 **`EXEC sys.sp_who`**：

- **直接定位**：你明确告诉数据库“去 sys 柜子里拿 sp_who”，省去了寻找的过程。
- **防止重名**：万一哪个调皮的同事在自己架构里也写了个叫 `sp_who` 的过程，带上 `sys.` 能保证你调用的永远是官方正版。

------

3. 举个直观的例子

假设你想查看当前数据库里有哪些人在连接，有多少进程：

- **偷懒写法（常用）：**

  ```SQL
  EXEC sp_who; 
  -- 就像在家里喊“开灯”，虽然能成，但如果不止一个灯，可能会乱。
  ```

- **专业写法（推荐）：**

  ```SQL
  EXEC sys.sp_who;
  -- 就像说“打开【客厅】的灯”，位置明确，绝不出错。
  ```

------

4. 关键点：`sp_` 前缀的含义

系统存储过程以 `sp_` 开头（Stored Procedure 的缩写）。 **注意：** 官方强烈建议你**不要**给自己写的存储过程起名以 `sp_` 开头。因为数据库每次看到 `sp_` 都会先去系统库里搜一圈，这会浪费一点点性能；万一以后官方更新了一个同名的功能，你的程序就崩了。

#### 自动执行存储过程

>对于无需并行执行的多个过程，可将其一指定为启动过程，并在该启动过程中调用其余过程，这样只需要占用一个辅助线程。

这句话描述的是一种**“节省资源”**的策略，特别是在数据库启动（SQL Server 启动）时自动运行某些逻辑的场景。

我们可以把它拆解为两个核心概念：**工作线程（Worker Thread）\**和\**调用链（Calling Chain）**。

------

1. 什么是“启动过程” (Startup Procedure)？

在 SQL Server 中，你可以把一个存储过程标记为“自动启动”。当 SQL Server 服务开启的那一刻，这些过程就会自动运行（通常用于初始化环境、检查状态或清理临时数据）。

- **标记方法：** 使用 `sp_procoption` 将过程设置为 `startup`。

------

2. “并行” vs “串行”：线程的消耗

想象一下，你有 3 个任务（过程 A, B, C）要在系统启动时完成。

**方案一：三个过程全是启动过程（并行/浪费）**

如果你把 A、B、C 都标记为 `startup`，SQL Server 会在启动时：

1. 为 A 开一个线程。
2. 为 B 开一个线程。
3. 为 C 开一个线程。

- **结果：** 瞬间占用了 **3 个工作线程**。如果启动任务很多，会抢占系统资源，甚至导致其他连接排队。

**方案二：套娃调用（串行/节省）**

你只把 A 标记为 `startup`，但在 A 的代码里面写上 `EXEC B; EXEC C;`。

1. 系统启动，只为 A 开 **1 个工作线程**。
2. A 运行完，接着在这个线程里跑 B。
3. B 跑完，接着在这个线程里跑 C。

- **结果：** 只占用了 **1 个工作线程**。虽然总时间可能变长了一点，但对系统资源的占用极低。

------

3. 为什么要强调“不需要并行”？

如果你的任务 B 必须等任务 A 算完的结果（比如 A 先建好临时表，B 往里填数据），那么**方案二**不仅省资源，还能保证顺序。

如果你的任务之间完全没关系，且每个任务都要跑 1 小时，那你可能还是得用方案一（用空间换时间）。

------

4. 形象的比喻：分发传单

- **方案一（并行）：** 你雇了 3 个人（3 个线程），每人拿一份传单去发。你得付 3 个人的工钱（消耗资源多）。
- **方案二（串行）：** 你只雇了 1 个人（1 个线程）。让他发完第一份，再去发第二份，最后发第三份。你只需要付 1 个人的钱（消耗资源少）。

------

5. 总结

这段话的意思是：**如果你的启动任务可以一个接一个地跑，就不要让它们齐头并进。** 通过在一个主启动过程里调用其他过程，你可以把对服务器性能的影响降到最低。

> **最佳实践：** > 很多 DBA 喜欢创建一个叫 `sp_MasterCleanup` 的过程设为启动，然后在里面依次调用各种清理脚本，而不是把几十个清理脚本全部设为启动。



### [Pass parameters to stored procedures](https://learn.microsoft.com/en-us/training/modules/create-stored-procedures-table-valued-functions/3-pass-parameters-to)

#### 输出参数

> 不过，您也可以向调用程序返回一个值。这称为输出参数。在 CREATE PROCEDURE 语句中，使用 OUTPUT 或 OUT 关键字可以指定一个输出参数。当存储过程执行完毕时，该过程会将输出参数的当前值返回给调用程序。

**1. 综合代码示例**

假设我们有一张产品表 `Products`。我们写一个存储过程：

```SQL
CREATE PROCEDURE usp_GetProductReport
    -- 1. 输入参数 & 4. 默认值
    @Category NVARCHAR(50) = 'Electronics', -- 默认查电子类
    @MaxRows INT = 5,                      -- 2. 用于 TOP 运算符
    
    -- 3. OUTPUT 参数：用于返回查到的商品平均价格
    @AvgPrice MONEY OUTPUT        
AS
BEGIN
    -- 使用 @MaxRows 控制返回行数，使用 @Category 过滤结果
    SELECT TOP (@MaxRows) ProductID, ProductName, ListPrice
    FROM Products
    WHERE CategoryName = @Category
    ORDER BY ListPrice DESC;

    -- 计算该类别的平均价格并赋值给输出参数
    SELECT @AvgPrice = AVG(ListPrice)
    FROM Products
    WHERE CategoryName = @Category;
END;
```

------

**2. 详细拆解这四个功能**

**① 运行时传递参数（灵活过滤）**

- **意思**：你不需要为每个城市或每个类别写不同的 SQL。
- **例子**：代码里的 `WHERE CategoryName = @Category`。你可以传“手机”，也可以传“电脑”，同一套逻辑跑不同的数据。

**② 用于 `TOP` 运算符（动态控制数量）**

- **意思**：你可以动态决定看前 5 名还是前 10 名。
- **例子**：`SELECT TOP (@MaxRows)`。在老版本 SQL 里这里只能写死数字，现在可以用变量，非常适合做“换一批”或者“查看更多”的功能。

**③ `OUTPUT` 参数（像函数一样返回值）**

- **意思**：除了给你一张表（结果集），存储过程还能单独甩给你一个数值。
- **例子**：`@AvgPrice MONEY OUTPUT`。调用者执行完后，可以直接拿到这个平均价去跑别的逻辑，而不需要再去解析整张结果表。

**④ 分配默认值（保底方案）**

- **意思**：如果调用者偷懒没给参数，程序不会崩溃，而是按预设跑。
- **例子**：`@Category ... = 'Electronics'`。你不传类别，它就默认给你看电子产品。

------

**3. 如何调用这个过程？**

调用带 `OUTPUT` 的过程时，稍微有点讲究，你需要先声明一个变量来“接住”那个值。

```SQL
DECLARE @ResultPrice MONEY; -- 准备一个容器

EXEC usp_GetProductReport 
    @Category = 'Books',       -- 传入参数
    @MaxRows = 3,              -- 传入参数
    @AvgPrice = @ResultPrice OUTPUT; -- 告诉程序：把结果装进 @ResultPrice

-- 打印出拿到的“找零”
SELECT @ResultPrice AS 'Average_Book_Price';
```

------

**总结**

- **输入参数**：是给存储过程的“原材料”。
- **默认值**：是没给原材料时的“库存件”。
- **TOP/WHERE**：是存储过程处理数据的“加工模具”。
- **OUTPUT**：是存储过程加工完后，单独递回来的“成品标签”。



### Create a stored procedure

> 存储过程使用 CREATE PROCEDURE 关键字创建。要创建存储过程，需要以下权限：
>
> 数据库中的 CREATE PROCEDURE 权限。
>
> 对创建存储过程所在架构的 ALTER 权限。

**怎么知道有没有这些权限呢?**

在 SQL Server 中，你可以通过以下几种方式来确认：

------

**1. 最简单的方法：直接“试错”**

SQL 程序员最常用的方法就是直接跑一下 `CREATE` 语句。

- 如果成功了，说明你有权限。
- 如果报错 **"The CREATE PROCEDURE permission was denied in database..."**，说明你没有第一条权限。
- 如果报错 **"The ALTER permission was denied on the schema..."**，说明你没有第二条权限。

------

**2. 使用系统函数查询 (推荐)**

你可以使用 `HAS_PERMS_BY_NAME` 函数，它会直接返回 `1`（有权限）或 `0`（没权限）。

**检查数据库级别的创建权限：**

```SQL
SELECT HAS_PERMS_BY_NAME(DB_NAME(), 'DATABASE', 'CREATE PROCEDURE');
```

**检查架构（Schema）级别的修改权限：**

如果你打算在 `dbo` 架构下创建过程：

```SQL
SELECT HAS_PERMS_BY_NAME('dbo', 'SCHEMA', 'ALTER');
```

------

**3. 查看当前用户的“特权清单”**

你可以查询 `sys.fn_my_permissions` 这个内置函数，它会列出你当前连接所拥有的所有权限。

```SQL
-- 查看在当前数据库里你能干的所有事
SELECT * FROM sys.fn_my_permissions(NULL, 'DATABASE')
WHERE permission_name = 'CREATE PROCEDURE';

-- 查看在某个架构（如 Sales）里你能干的所有事
SELECT * FROM sys.fn_my_permissions('Sales', 'SCHEMA')
WHERE permission_name = 'ALTER';
```

------

**4. 为什么需要这两种权限？（形象理解）**

我们可以把这个过程比作**在商场里开一家快闪店**：

1. **CREATE PROCEDURE 权限**：相当于你拿到了“在这个城市摆摊”的**营业执照**。如果没有它，你哪儿都去不了。
2. **SCHEMA 上的 ALTER 权限**：相当于你拿到了“某个特定摊位”的**钥匙**。
   - 比如数据库里有 `dbo`、`Sales`、`HR` 三个架构。
   - 你可能有执照，但如果你没有 `Sales` 架构的 ALTER 权限，你只能在 `dbo` 里建过程，不能在 `Sales` 里建。

------

**5. 常见的一种“特许情况”**

如果你在公司里的角色是 **`db_owner`**（数据库所有者），那么你不需要单独检查这些权限，因为 `db_owner` 拥有数据库内**所有**操作的最高权限。

你可以查查自己是不是“老板”：

```SQL
SELECT IS_MEMBER('db_owner'); -- 返回 1 代表你是老板，权限全开
```

### Use dynamic SQL with EXEC and sp-execute-sql

> sp_executesql 允许您执行带参数的 T-SQL 语句。当需要向语句传递不同值时，可以替代存储过程使用 sp_executesql。T-SQL 语句保持不变，仅参数值发生更改。与存储过程类似，SQL Server 查询优化器很可能会重用执行计划。
>
> sp_executesql 接受 T-SQL 语句作为参数，该参数可以是 Unicode 常量或 Unicode 变量。例如，以下两个代码示例均有效：

```sql
EXECUTE sp_executesql   
          N'SELECT * FROM SalesLT.Customer   
          WHERE CompanyName = @company',  
          N'@company nvarchar(128)',  
          @company = "Sharp Bikes";
```

### Create user-defined functions

#### 性能注意事项

> 查询优化器无法预估多语句表值函数（MSTVF）将返回的行数，但可以预估内联表值函数（内联 TVF）的行数。因此，为了获得更优性能，应尽可能使用内联 TVF。若无需将 MSTVF 与其他表进行联接，或已知结果仅包含少量数据行，则其性能影响相对有限。若预期返回的结果集较大，且需与其他表联接，可以改用临时表存储结果，然后再与临时表进行联接操作。

简单来说，这两个缩写代表了 SQL Server 中“表值函数”的两种不同**构建方式**。

- **TVF (Table-Valued Function)**：表值函数（返回一张表的函数）。
- **Inline TVF (内联 TVF)**：简单透明，像是一张“带参数的视图”。
- **MSTVF (Multi-Statement TVF，多语句 TVF)**：复杂强大，像是一个“带返回值的脚本”。

------

**1.直观对比：代码长什么样？**

**Inline TVF (内联)** —— 就像一张“透视纸”

它只有一条 `SELECT` 语句，数据库可以直接看透它的底层逻辑。

```SQL
CREATE FUNCTION dbo.GetCheapProducts(@Price money)
RETURNS TABLE
AS
RETURN (
    SELECT ProductID, Name FROM Product WHERE ListPrice < @Price
);
```

**MSTVF (多语句)** —— 就像一个“加工车间”

它有 `BEGIN...END`，需要先声明表变量，再往里插入数据。

```SQL
CREATE FUNCTION dbo.GetProductsComplex(@Price money)
RETURNS @Results TABLE (ID int, Name nvarchar(50))
AS
BEGIN
    INSERT INTO @Results
    SELECT ProductID, Name FROM Product WHERE ListPrice < @Price;
    -- 这里还可以写其它的逻辑，比如 IF、UPDATE 等
    RETURN;
END;
```

------

**2. 为什么 Query Optimizer（查询优化器）会“头疼”？**

你引用的那段话揭示了数据库引擎的一个秘密：

- **Inline TVF（好预测）**：优化器就像有“透视眼”。当你调用它时，优化器会把函数里的 SQL 拿出来，和外面的主查询合并。它知道这张表大概有多少行，从而选出最快的执行路径（比如用哪个索引）。
- **MSTVF（瞎子摸象）**：对优化器来说，MSTVF 就是一个“黑盒子”。由于里面可能有很多行代码，优化器**在执行之前无法预知**里面到底会吐出 10 行还是 100 万行数据。
  - **后果**：由于不知道行数，优化器通常会乱猜（旧版本默认猜 1 行，新版本猜 100 行）。如果实际有 100 万行，它选错执行计划（例如选了慢速的循环连接），你的查询就会卡死。

------

**3. 该选哪一个？（决策清单）**

| **场景**                                     | **推荐选择**   | **理由**                                                     |
| -------------------------------------------- | -------------- | ------------------------------------------------------------ |
| **逻辑简单**（一个 SELECT 搞定）             | **Inline TVF** | 性能最高，优化器能完美优化。                                 |
| **逻辑复杂**（需要 IF/ELSE, 循环, 临时处理） | **MSTVF**      | 只有它能处理多步逻辑。                                       |
| **结果集很大**（万级以上）                   | **慎用 MSTVF** | 建议改用 **临时表 (#TempTable)**，因为临时表有统计信息，优化器能看懂。 |
| **需要跟大表 JOIN**                          | **Inline TVF** | 避免“黑盒子”效应导致连接顺序错误。                           |

------

**4. 总结那段话的意思**

那段话是在教你**“避坑”**：

1. **首选 Inline**：因为它透明，快。
2. **次选 MSTVF**：除非逻辑太复杂不得不写多行，且结果集很小（几行数据），这时候优化器猜错也无伤大雅。
3. **终极方案**：如果数据量大、逻辑又复杂，别用函数了，先把结果塞进**临时表**，再拿临时表去跟别人连。

#### 标量型用户自定义函数

```sql
CREATE FUNCTION dbo.ufn_GetProductListPrice
(@ProductID [int], @OrderDate [datetime])
RETURNS [money] 
AS 
BEGIN
    DECLARE @ListPrice money;
        SELECT @ListPrice = plph.[ListPrice]
        FROM [Production].[Product] p 
        INNER JOIN [Production].[ProductListPriceHistory] plph 
        ON p.[ProductID] = plph.[ProductID] 
            AND p.[ProductID] = @ProductID 
            AND StartDate = @OrderDate
    RETURN @ListPrice;
END;
GO
```

在 SQL Server 中，这些方括号 `[]` 被称为**标识符分隔符（Identifier Quoters）**。

它们的主要作用是告诉数据库引擎：“括号里的内容是一个完整的名字，请把它当成一个整体来处理。”

------

**1. 为什么要用方括号？**

主要有以下三个原因：

**原因 A：处理空格或特殊字符**

如果你的表名或列名中间有空格，直接写会报错。

- `SELECT My Column FROM My Table` —— **错误**（SQL 会以为 `Column` 是个关键词）。
- `SELECT [My Column] FROM [My Table]` —— **正确**。

**原因 B：避开保留关键字**

如果你给表起名叫 `Order` 或 `User`，这在 SQL 中是关键字。

- `SELECT * FROM Order` —— **报错**。
- `SELECT * FROM [Order]` —— **正确**。

**原因 C：定义架构（Schema）和对象的层级关系**

在你的例子中：`[Production].[Product]`

- 第一个 `[Production]` 指的是 **架构 (Schema)**，类似于文件系统中的“文件夹”。
- 第二个 `[Product]` 指的是 **表 (Table)**，类似于文件夹里的“文件”。 使用括号可以清晰地划分界限，防止解析歧义。

------

**2. 你的代码中 `[int]` 为什么也带括号？**

在你的代码里看到了 `(@ProductID [int], ...)`。 实际上，`int`、`datetime`、`money` 是 SQL 的内置数据类型，它们**不需要**加括号。

**那为什么会出现呢？** 这通常是因为这段代码是由 SQL Server Management Studio (SSMS) **自动生成**的（通过“脚本化”功能）。SSMS 为了绝对的保险，会给所有的标识符（包括系统类型名）都套上括号，以防止任何潜在的解析冲突。这是一种“防御性编程”风格，不加也不会错。

------

**3. 什么时候可以不写？**

如果你的名字符合以下标准，完全可以省去括号：

1. 以字母开头。
2. 中间没有空格。
3. 不含特殊符号（如 `#`, `-`, `.` 等）。
4. 不是 SQL 关键字（如 `SELECT`, `FROM`, `TABLE`）。

**简写版（更清爽）：**

```SQL
CREATE FUNCTION dbo.ufn_GetProductListPrice
( @ProductID int, @OrderDate datetime )
RETURNS money 
AS 
...
```

> 创建函数时，SCHEMABINDING 为可选参数。指定 SCHEMABINDING 后，函数将与引用的对象绑定，此时若未同时修改函数，则无法修改对象。必须先修改或删除函数以解除依赖关系，才能修改对象。
>
> 在以下任一情况发生时，SCHEMABINDING 将被移除：
>
> 函数被删除
>
> 使用 ALTER 语句修改函数时未指定 SCHEMABINDING

简单来说，`SCHEMABINDING`（架构绑定）就像是给函数和它引用的表之间签了一份**“不离不弃”的法律合同**。

一旦签了这份合同，你就不能随便乱动底层的表了，因为函数正“盯着”它呢。

------

**1. 举个例子：签了合同的函数**

假设你有一张表 `Sales.Products`，里面有个字段叫 `Price`。你创建了一个带架构绑定的函数：

```SQL
CREATE FUNCTION dbo.fn_GetTax(@ProductID INT)
RETURNS MONEY
WITH SCHEMABINDING -- 签合同：架构绑定
AS
BEGIN
    RETURN (SELECT Price * 0.1 FROM dbo.Products WHERE ID = @ProductID);
END;
```

> **注意：** 使用 `SCHEMABINDING` 时，表名必须写成 `dbo.Products`（两部分名称），不能只写 `Products`。

------

**2. 这时候你想“毁约”会发生什么？**

如果你突然想把 `Products` 表里的 `Price` 字段删了，或者把这个字段改名：

```SQL
-- 尝试改名或删除字段
ALTER TABLE dbo.Products DROP COLUMN Price;
```

**结果：报错！** SQL Server 会跳出来拦住你：“住手！函数 `dbo.fn_GetTax` 引用了这个字段，并且它开启了架构绑定。你得先处理掉那个函数，才能动这张表。”

------

**3. 如何解除这种绑定？**

正如你提供的那段话所说，解除合同有两种方式：

**方式 A：直接把函数删了（Drop）**

```SQL
DROP FUNCTION dbo.fn_GetTax;
-- 好了，现在你想怎么改 Products 表都可以了。
```

**方式 B：修改函数，但不写那行字（Alter）**

如果你想保留函数，但不想再被它束缚：

```SQL
ALTER FUNCTION dbo.fn_GetTax(@ProductID INT)
RETURNS MONEY
-- 注意：这里我故意没写 WITH SCHEMABINDING
AS
BEGIN
    RETURN (SELECT Price * 0.1 FROM dbo.Products WHERE ID = @ProductID);
END;
-- 合同解除了！现在再去改表，SQL 就不会拦你了。
```

------

**4. 为什么要费劲用它？（它的好处）**

既然这么麻烦，为什么大家还要用它？

1. **安全性**：防止别人在你不知道的情况下改了表结构，导致你的函数突然失效（报错）。
2. **性能**：对于某些视图或计算列，如果你想在上面**创建索引**，SQL Server 强制要求必须指定 `SCHEMABINDING`。
3. **确定性**：优化器知道这些表结构不会变，有时候能生成更高效的执行计划。

------

**总结**

- **不带 SCHEMABINDING**：你是自由的，但如果表改了，函数跑起来可能会“炸”。
- **带 SCHEMABINDING**：表被“锁死”了，安全性极高，但你要改表时必须先去改函数。

[Create stored procedures and functions in Transact-SQL](https://microsoftlearning.github.io/dp-080-Transact-SQL/Instructions/Labs/11-create-stored-procedures.html)

## [Configure databases for optimal performance](https://learn.microsoft.com/en-us/training/modules/configure-databases-for-optimal-performance/)

### Explore database maintenance checks

#### 重建与重组

> 索引碎片是指索引页的逻辑顺序与其物理顺序不匹配的情况。在执行常规数据修改操作（例如 `UPDATE` 、 `DELETE` 和 `INSERT` ）时，页面可能变得无序排列。碎片化会引发性能问题，因为查找索引页中指针所指向的数据需要执行额外的 I/O 操作。

**<u>怎么理解"页面可能会变得无序。碎片化可能导致性能问题，因为定位索引页内指针所引用数据需要额外的 I/O 操作？"</u>**

简单来说，**“索引碎片”\**就像是一本\**页码被撕乱了的书**。

为了理解为什么“无序”会导致“额外的 I/O（磁盘读写）”，我们可以把索引想象成一叠整齐的纸。

------

**1. 什么是“页面变得无序”？**

在数据库里，数据是存在一张张 **“页 (Page)”** 上的（每页 8KB）。

- **理想状态（物理和逻辑一致）**：

  数据页 1 $\rightarrow$ 数据页 2 $\rightarrow$ 数据页 3。

  它们在硬盘上是挨在一起的。当你需要读这三页时，硬盘的磁头转一圈就能顺着读完。

- **碎片状态（无序）**：

  假设你在页 1 和页 2 之间突然插入了很多新数据，原本的页 1 放不下了，数据库就会在磁盘的**另一个遥远角落**找一张空页（比如页 99）来放新数据。

  现在的顺序变成了：页 1 $\rightarrow$ **页 99** $\rightarrow$ 页 2。

虽然逻辑上它们还是连贯的（页 1 的指针指向页 99），但在物理位置上，它们已经**“散架”**了。

------

**2. 为什么会导致性能问题（额外的 I/O）？**

这就是你提到的“定位数据需要额外的 I/O 操作”。我们可以打两个比方：

**比方 A：在超市买菜**

- **无碎片**：你想买葱、姜、蒜。它们都在同一个货架上。你走过去（一次 I/O），伸手一拿就齐了。
- **有碎片**：你想买葱、姜、蒜。结果葱在 1 楼，姜在 5 楼，蒜在地下室。你得跑三趟（三次 I/O）。虽然你最终买到了同样的菜，但你**走路的时间（延迟）**变长了。

**比方 B：机械硬盘的磁头旋转**

如果你用的是传统的机械硬盘（HDD）：

- **顺序读取**：磁头放在盘片上，像听 CD 一样顺着转就行，非常快。
- **碎片读取**：磁头读完页 1，必须**“跳一下”**去找页 99，读完再**“跳回来”**找页 2。
- **代价**：每一次“跳动（寻道）”都是一次昂贵的物理运动，会消耗大量时间。

------

**3. 什么是“指针引用数据”？**

索引页就像书的目录。

1. 你查目录（索引），目录告诉你：“你要的数据在第 10 页、第 500 页和第 20 页。”
2. 因为物理顺序乱了，数据库引擎必须跟着这些**“指针”**在磁盘上到处乱跳。
3. **结果**：为了读出本该连续的 1MB 数据，硬盘可能得前后反复横跳几十次。

------

**4. 总结**

- **无序** = 逻辑上连续的数据，在物理磁盘上被存得东一块西一块。
- **性能下降** = 数据库为了把这些碎片拼凑起来，被迫做了更多次的“寻找”和“读取”动作（即额外的 I/O）。

**这就是为什么 DBA 经常要进行“索引重建 (Rebuild)”的原因——它相当于把书拆了，按页码重新装订一遍，让它们在物理上也挨在一起。**

#### 重建与重组

> "索引重组是一种联机操作，可对索引（包括聚集索引和非聚集索引）的叶级别进行碎片整理。该整理过程会将叶级页按节点从左到右的逻辑顺序重新物理排序，同时根据设定的填充因子值压缩索引页。"

**<u>什么是聚集索引和非聚集索引，如何理解“配置的填充因子值对索引页进行压缩”</u>**

**1. 聚集索引 vs. 非聚集索引**

这是 SQL Server 中最核心的两个索引概念：

**聚集索引 (Clustered Index) —— “书就是索引”**

- **原理**：聚集索引决定了数据在硬盘上的**物理存储顺序**。
- **形象比喻**：就像一本**字典**。字典的内容是按拼音 A-Z 排列的，你找到了拼音，也就直接找到了那个字所在的完整内容。
- **特点**：
  - 一张表只能有一个聚集索引（因为数据物理上只能按一种顺序排）。
  - 叶级别（最底层）存储的是**实际的数据行**。

**非聚集索引 (Non-Clustered Index) —— “书后的索引表”**

- **原理**：它像是一个独立的结构，记录了关键词和它指向实际数据的“地址”。
- **形象比喻**：就像一本**技术书籍后面的索引页**。你查“数据库”这个词，它告诉你这个词在第 5、12、88 页。你需要根据这个页码（指针）翻到正文去看。
- **特点**：
  - 一张表可以有很多个非聚集索引。
  - 叶级别存储的是**索引键值**和一个**指向数据行的指针**（如果表有聚集索引，指针就是聚集索引键）。

------

**2. 如何理解“根据填充因子对索引页进行压缩”？**

这里有两个关键词：**填充因子 (Fill Factor)** 和 **压缩 (Compacted)**。

**什么是填充因子？**

这是你给数据库留的**“呼吸空间”**。

- 数据库的数据存在 8KB 大小的“页”里。
- 如果填充因子设为 **80**，意思就是：“新写数据时，每页只填满 80%，留下 20% 的空位”。
- **为什么要留空？** 为了防止下次插入数据时，因为没位置而导致“页拆分（Page Split）”，产生碎片。

**什么是“压缩”？**

在**索引重组（Reorganize）**过程中，系统会检查这些页面。

- **碎片整理**：如果发现有的页只用了 10%，有的用了 30%，它们物理位置又很乱。
- **重新打包**：重组过程会把这些零散的数据重新塞进连续的物理页中。
- **达到标准**：它会尽量让每一页的饱满程度回到你设定的“填充因子”水平。比如你设了 80%，它就会把散落在外的碎片挤进 80% 的空间里，腾出多余的空白页给系统回收。

------

**3. 总结这句话的内涵**

> “同时根据设定的填充因子值压缩索引页。”

这句话的意思是：**在把书页顺序排好的同时，顺便把每页纸上的文字重新排版，确保每页纸都按照你要求的比例（比如 80%）填满，把那些因为删除数据留下的“窟窿”给挤掉。**

------

💡 小贴士：

- **重组 (Reorganize)** 是**在线**的，不锁表，适合轻微碎片。
- **重新生成 (Rebuild)** 是**离线**的（默认），它会把旧索引删了重造，清理碎片最彻底，但会卡住业务。

#### 重建与重组

> 索引重建操作分为在线和离线两种方式，具体取决于执行的命令或使用的 SQL Server 版本。离线重建会直接删除并重新创建索引。而在线重建则会在保留现有索引的同时并行构建新索引，待新索引建成后，再删除旧索引并将新索引重命名为原有名称。请注意，在线重建需要更多存储空间，因为新旧索引会暂时共存。

> SQL Server 与 Azure SQL 平台提供了一系列动态管理视图（DMV），可用于检测数据库对象中的碎片情况。其中，针对 B 树索引最常使用的是 `sys.dm_db_index_physical_stats` ，而针对列存储索引则常用 `sys.dm_db_column_store_row_group_physical_stats` 。

**<u>“列索引和B树索引有什么区别，这两个方法是查看碎片化程度的吗？”</u>**

这两个 DMV 确实是用来**查看碎片化程度**的，它们就像是给索引做“超声波检查”。

至于 **B 树索引**和**列存储索引**，它们是数据库里两种截然不同的“存储逻辑”，适用于完全不同的工作场景。

------

**1. B 树索引 (B-Tree Index) —— “横向存储”**

这是 SQL Server 默认的索引类型（聚集和非聚集索引通常都是 B 树结构）。

- **存储方式**：它是按**行**存储的。一页（8KB）里存的是这条记录的列 A、列 B、列 C、列 D...
- **形象比喻**：就像书架上的**书**。你想看某人的信息，就把这整本书（整行）拿出来。
- **擅长场景**：**OLTP（事务处理）**。比如：查询特定一个人的订单、修改某一个人的地址。它定位“某一行”非常快。
- **碎片化表现**：页面的物理顺序和逻辑顺序不一致（就像之前聊的磁头乱跳）。

------

**2. 列存储索引 (Columnstore Index) —— “纵向存储”**

这是为了大数据分析而诞生的技术。

- **存储方式**：它是按**列**存储的。把所有人的“年龄”存在一堆，所有人的“收入”存在另一堆。
- **形象比喻**：就像**Excel 里的某一列**。如果你只想算“全公司平均年龄”，数据库只需要读“年龄”这一列的数据块，根本不用理会名字、地址等其他列。
- **擅长场景**：**OLAP（数据分析/报表）**。比如：从 1 亿条记录里计算去年的总销售额。它的压缩率极高（因为同一列的数据类型相似），速度比 B 树快几十倍。
- **碎片化表现**：列存储的碎片不叫页面乱序，而叫**“行组（Row Groups）”中有太多已删除的记录**。

------

**3. 这两个方法是查碎片的吗？**

是的。它们分别对应这两种不同的结构：

**针对 B 树索引：`sys.dm_db_index_physical_stats`**

这个视图最核心的指标是：`avg_fragmentation_in_percent`。

- **数值含义**：如果这个值 > 30%，你就该 Rebuild（重新生成）索引了。它告诉你物理页和逻辑页有多“不合拍”。

**针对列存储索引：`sys.dm_db_column_store_row_group_physical_stats`**

列存储不讲页面顺序，它看的是“行组”的状态。

- **检测逻辑**：列存储数据是分块（Row Group）存的。如果你频繁删除数据，这些块里就会有很多“空洞”。这个视图让你看到每个块里有多少被标记为“删除”的数据。
- **处理方式**：当被删除的数据太多，你就需要通过 `REORGANIZE` 来合并这些块。

> 微软自 SQL Server 2017 起引入了可恢复的索引重建操作。该功能为控制重建操作在特定实例上的耗时提供了更大的灵活性。SQL Server 2019 进一步增加了关联最大并行度的控制能力，使数据库管理员能够进行更精细的管控。

<u>**索引重建操作需要怎么操作?**</u>

索引重建（Index Rebuild）就像是把一本破旧、页码混乱的字典**彻底拆掉，按顺序重新装订成一本新书**。

在 SQL Server 中，你可以通过 **SSMS 图形界面**或 **T-SQL 脚本**来操作。

------

**1. 图形界面操作（适合新手）**

1. 在 **SSMS** 的“对象资源管理器”中，找到你的数据库。
2. 展开 **“表”** -> 找到目标表 -> 展开 **“索引”**。
3. 右键点击某个具体的索引（或整个“索引”文件夹）。
4. 选择 **“重新生成 (Rebuild)”**。
5. 在弹出的窗口中，点击“确定”。

------

**2. T-SQL 脚本操作（生产环境推荐）**

对于大型数据库，写脚本可以更精确地控制。

**基础重建命令：**

**进阶：开启“可恢复”模式 (SQL 2017+)**

正如你提供的那段话所说，传统的重建如果运行到 99% 报错了，就要从头再来。使用 `RESUMABLE = ON` 可以让它停下来，之后继续跑。

**进阶：控制“并行度” (SQL 2019+)**

如果你担心重建索引把 CPU 占满了，导致别的用户卡顿，可以用 `MAXDOP`（最大并行度）来限制它只用几个核心：

------

**3. 三种操作模式的对比**

在操作时，你需要根据业务情况选择“选项”：

------

**4. 什么时候该操作？**

通常我们通过之前提到的 `sys.dm_db_index_physical_stats` 来查看碎片率（`avg_fragmentation_in_percent`）：

- **碎片 < 5%**：啥也不用干。
- **5% - 30%**：建议使用 `REORGANIZE`（重组，比较轻量）。
- **> 30%**：必须使用 `REBUILD`（重建，彻底解决问题）。

------

**总结**

重建索引就像是给数据库做一次“深层大扫除”。

- 如果是**小表**或**维护时段**，直接点右键重建。
- 如果是**超大表**且**不能停机**，请务必使用 `ONLINE = ON` 和 SQL 2017+ 的 `RESUMABLE = ON`。

#### 统计

> 统计信息以二进制大型对象（blobs）的形式存储在用户数据库中。这些 blob 包含有关表或索引视图中一个或多个列的数据值分布的统计信息。
>
> 统计信息描述了数据在列中的分布情况。查询优化器利用这些列和索引的统计信息来估算基数，也就是查询预计将返回的行数。
>
> 随后，查询优化器会利用基数估计来生成执行计划。基数估计还能帮助优化器决定采用何种操作类型（例如索引查找或扫描）以获取所需数据。

**1. 形象比喻：图书馆的“索引卡”与“馆员预测”**

想象你要去一个拥有 100 万本书的图书馆找书：

- **数据（Data）**：就是那 100 万本书。
- **统计信息（Statistics）**：是馆长手里的一张**概览表**。上面没写每本书的名字，但记录了：姓“张”的作者写的书约有 10 万本，姓“欧阳”的只有 50 本。
- **基数估计（Cardinality Estimate）**：当你问“我要找姓张的作者写的书”时，馆员心里预估：“大概会出 10 万条结果”。这个 **“10 万”** 就是基数。
- **执行计划（Execution Plan）**：
  - **方案 A（全馆扫描）**：馆员从第一排书架走到最后一排，一本本看。（适合找“姓张”的，因为反正书多，边走边看就行）。
  - **方案 B（索引查找）**：馆员直接去索引柜看编号，然后精准跑到对应的架子上。（适合找“姓欧阳”的，因为一共才 50 本，没必要全馆跑）。

------

**2. SQL 里的真实案例**

假设你有一张员工表 `Employees`，里面有 10,000 行数据，有一列是 `Job_Title`（职位）。

**场景一：查询“初级职员”（数据量大）**

1. **统计信息说**：表中 80% 的人都是 `Junior`。
2. **基数估计**：优化器预估会返回 **8,000 行**。
3. **决定方案**：既然要拿走 80% 的数据，优化器觉得“与其拿着索引地址一个个去翻，不如直接把整张表从头扫到尾（**Table Scan**）快”。

**场景二：查询“CEO”（数据量极小）**

1. **统计信息说**：全公司只有 1 个 `CEO`。
2. **基数估计**：优化器预估返回 **1 行**。
3. **决定方案**：只要 1 行？那通过索引精准定位（**Index Seek**）最划算，连 1 毫秒都用不了。

------

**3. 为什么它存成 BLOB（二进制大对象）？**

正如你提供的那段话所说，统计信息存为 BLOB，是因为它里面包含了一个复杂的**直方图（Histogram）**。

如果你打开这个 BLOB，你会看到：

- **RANGE_HI_KEY**：数据的上限值。
- **EQ_ROWS**：等于这个值的有多少行。
- **DISTINCT_RANGE_ROWS**：这个区间内有多少不同的值。

------

**4. 如果统计信息“过期”了会怎样？**

这是最常见的性能杀手。 如果公司裁员了，`Junior` 只剩 10 个人，但**统计信息没更新**，它还以为有 8,000 人。

- **结果**：你查 10 个人，优化器却给你配了一台“收割机”（全表扫描），明明只要“一把小镊子”（索引查找）就能搞定。**这就是为什么数据库查询变慢时，DBA 第一步通常是执行 `UPDATE STATISTICS`。**

------

**总结**

- **统计信息**是数据库的“地图快照”。
- **基数估计**是基于地图的“路程预估”。
- **执行计划**是基于预估选出的“最优路线”。

#### 生成统计信息

> 如果启用 `AUTO_CREATE_STATISTICS` 选项 `ON` ，查询优化器将默认自动为索引列生成统计信息。此外，它也会为查询条件中的单列生成统计信息。
>
> 这些方法能为大多数查询生成高质量的查询计划。但某些情况下，您可能需要借助 CREATE STATISTICS 语句创建额外的统计信息，从而优化特定查询的执行计划。
>
> 建议启用 `AUTO_CREATE_STATISTICS` 选项，以便查询优化器能自动为查询谓词中的列生成统计信息。
>
> 在以下情况下，请考虑创建统计信息：
>
>
> 数据库引擎优化顾问建议创建统计信息
>
>
> 查询谓词涉及的多个列未被现有索引覆盖
>
>
> 该查询从数据子集中选取
>
> 该查询缺少统计信息

在 Azure SQL 的世界里，虽然都叫“SQL”，但由于底层架构（PaaS 的深度不同）的差异，自动化维护工具的选择确实大不相同。

主要可以分为以下**三个平台**：

------

**1. Azure SQL 数据库 (Azure SQL Database)**

这是最纯粹的 **PaaS（平台即服务）**。你只需要关注数据库，不需要管服务器。

- **平台特点**：不支持“SQL Server 代理 (SQL Agent)”。
- **可用维护工具**：
  - **弹性作业 (Elastic Jobs)**：专门为云设计的工具，可以跨多个数据库同时运行脚本（比如给 100 个数据库同时重建索引）。
  - **Azure 自动化 (Azure Automation)**：使用 PowerShell 或 Python 脚本，结合“运行簿 (Runbook)”来定时触发 SQL 命令。
  - **逻辑应用 (Logic Apps)**：通过图形化工作流，按时间计划或特定事件触发数据库维护任务。

------

**2. Azure SQL 托管实例 (Azure SQL Managed Instance)**

这是一种**近乎 100% 兼容本地 SQL Server** 的托管方案。

- **平台特点**：它保留了本地 SQL Server 的绝大部分特性，包括 VNET 隔离。
- **可用维护工具**：
  - **SQL Server 代理 (SQL Agent)**：这是它的王牌。和你在本地用的一模一样，直接在 SSMS 里创建“作业 (Job)”，设置步骤和计划即可。
  - **托管计划**：虽然有了 SQL Agent，你依然可以使用 Azure 自动化的方式。

------

**3. Azure 虚拟机上的 SQL Server (SQL Server on Azure VM)**

这本质上是 **IaaS（基础设施即服务）**，就是你在云端租了一台装了 Windows/Linux 的电脑。

- **平台特点**：你拥有最高权限，但也得负责操作系统的维护。
- **可用维护工具**：
  - **SQL 最佳实践评估 (SQL Best Practices Assessment)**：这是云端特有的，能自动扫描你的虚拟机设置，告诉你哪里配置不合理。
  - **自动备份与修补 (Automated Backup & Patching)**：Azure 平台可以直接管理这台虚机的补丁和备份，不需要你手动写脚本。
  - **维护计划向导**：经典的本地 SQL Server 维护计划。

------

**核心差异总结表**

------

**为什么不一样？**

本质原因在于**“责任分担模型”**：

- 在 **Azure SQL Database** 中，微软承担了大部分维护工作（如自动更新统计信息、自动备份），所以它把底层的 SQL Agent 隐藏了，让你用更现代化的云工具。
- 在 **VM（虚拟机）** 中，微软认为你是个“专业玩家”，所以把所有控制权都给你，由你自己决定怎么维护。

### Describe database scoped configuration options

#### 阐述数据库作用域配置选项

> SQL Server 历来提供数据库层面的配置选项。例如，恢复模式一直是数据库级别的设置。随着更复杂功能的引入，额外选项也逐渐增多。其中许多选项与数据库兼容级别相关，而兼容级别同样是数据库层面的配置。根据一个微小的区别，这些配置选项可归为两大类。
>
> 在 T-SQL 中使用 `ALTER DATABASE SCOPED CONFIGURATION` 语法配置的选项
>
> 在 T-SQL 中使用 `ALTER DATABASE` 语法配置的选项

> 这些选项的不同设置方法并无区别。通过 `ALTER DATABASE` 设置的选项包括：
>
> 数据库恢复模式 – 指数据库处于完整恢复模式还是简单恢复模式
>
> 自动调优选项 – 是否启用‘强制执行上次良好计划’功能
>
> 自动创建并更新统计信息 - 使数据库能够自动创建和更新统计信息，并支持异步统计信息更新选项
>
> 查询存储选项 – 查询存储的相关设置在此处进行配置
>
> 快照隔离 – 您可以配置快照隔离及读提交快照隔离

> 许多以往在服务器端配置的选项，现在已支持在数据库级别进行调整。其中包括：
>
>
> 最大并行度 – 支持数据库配置专属的 MaxDOP 设置，并覆盖服务器级设置。
>
> 传统基数估算功能 - 允许数据库继续使用早期版本的基数估算器。在某些情况下，采用新版基数估算器的查询可能性能会有所下降，此时启用此功能或许能带来改善。需注意的是，即使在较新的兼容性级别（如 140 或 150）中启用此选项，仍可享受相应兼容级别下智能查询处理带来的性能优势。
>
> 上次查询计划统计信息 – 用于捕获查询的最后实际执行计划数据。此功能仅可在兼容性级别 150 下使用。
>
> 即时工作负载优化 – 通过优化器在计划缓存中存储存根查询计划。对于包含大量单次查询的工作负载，这有助于减小计划缓存的大小。

#### 数据库兼容级别

> 每个数据库都有独立的兼容级别，这决定了该数据库查询优化器的工作方式。
>
> 在升级 SQL Server 时，可在此处管理该设置，从而确保查询能够获得与旧版本相似的执行计划。
>
> 微软支持数据库在较旧的兼容级别上长期运行。但为了使用智能查询处理的众多新功能，建议您尽可能升级到兼容级别 140 或 150，因为这些新特性大多仅在此类较新版本中提供。

在 SQL Server 中，**兼容级别（Compatibility Level）** 是一个非常关键的“软开关”。它就像是给你的数据库装了一个**“时光过滤器”**：即使你把数据库升级到了最新的 SQL Server 2022 引擎，只要你把兼容级别设为旧版，它在处理查询时就会像旧版本一样思考，从而避免升级后执行计划突然变差。

你可以通过以下两种方式查看和管理这个设置：

**1. 使用图形界面 (SSMS) —— 最直观的方法**

1. 打开 **SQL Server Management Studio (SSMS)** 并连接到实例。
2. 在“对象资源管理器”中，找到你的 **数据库**，右键点击它，选择 **属性 (Properties)**。
3. 在弹出的窗口左侧菜单中，点击 **选项 (Options)**。
4. 在右侧面板的第一行，你就会看到 **兼容级别 (Compatibility level)**。
   - **查看**：下拉框里显示的当前选中的值就是现在的级别。
   - **管理/设置**：直接在这个下拉框里选择你想要的版本（例如 SQL Server 2019 [150]），然后点击底部的“确定”即可。

------

**2. 使用 T-SQL 脚本 —— 最专业的方法**

如果你要一次性检查服务器上所有数据库的兼容级别，或者需要在脚本中修改它，可以使用以下命令：

**查看所有数据库的兼容级别：**

```SQL
SELECT name, compatibility_level 
FROM sys.databases;
```

- **160** = SQL Server 2022
- **150** = SQL Server 2019
- **140** = SQL Server 2017
- **130** = SQL Server 2016
- **120** = SQL Server 2014

**修改某个数据库的兼容级别：**

```SQL
-- 将数据库 MyDatabase 的兼容级别设置为 SQL Server 2019 (150)
ALTER DATABASE MyDatabase  
SET COMPATIBILITY_LEVEL = 150;  
GO
```

------

**3. 为什么升级时要特别在意这个“此处”？**

就像你提供的那段话所说，这其实是微软给 DBA 准备的**“后悔药”**：

- **平滑过渡**：当你从 SQL Server 2012 搬迁到 2022 时，你可以先保持兼容级别在 110。这样业务逻辑和查询速度基本不会变。
- **开启新技能**：等你确认一切稳定后，再手动把级别调到 160。这时，**智能查询处理 (Intelligent Query Processing)** 才会真正被激活，你的数据库才会突然变得更聪明。

------

**总结**

- **在哪里看？** 数据库属性 -> 选项 -> 第一行。
- **在哪里设？** 就在看的地方选下拉框，或者执行 `ALTER DATABASE ... SET COMPATIBILITY_LEVEL`。

### Describe automatic tuning

> 自动调优是一种监控分析功能，可不断学习您的工作负载特性，从而发现潜在问题并给出优化建议。
>
> 自动优化建议是基于从查询存储收集的数据提出的。执行计划会因架构变更、索引调整或数据变动引起的统计信息更新而不断演变。这种演变可能导致查询性能下降，因为原有的执行计划已无法适应当前查询的需求。
>
> 此外，自动调优能够基于性能指标收集并应用机器学习服务，从而提供改进建议，甚至实现自我纠正。
>
> 无论是在本地环境还是云端，自动调优功能都能帮助您识别因查询执行计划退化引发的问题。同时，在 Azure SQL 数据库中，您还可通过索引优化来提升查询性能。Azure SQL 数据库的自动调优能够识别出哪些索引应当被添加，甚至哪些应当从数据库中移除，从而优化查询效率。

**自动调优（Automatic Tuning）** 的设置位置取决于你是使用 **Azure SQL（云端）** 还是 **本地 SQL Server**。

它的核心逻辑就像是给数据库请了一个“AI 医生”，不仅能看病（监控），还能自己动手术（自动修复）。

------

**1. Azure SQL 数据库（云端）—— 功能最全**

在 Azure 门户中，自动调优是最好找且最强大的。

**设置路径：**

1. 登录 **Azure Portal**。
2. 找到你的 **Azure SQL Database**（单库）或 **SQL Server**（服务器级）。
3. 在左侧导航栏的 **“性能 (Performance)”** 栏目下，点击 **“自动调优 (Automatic tuning)”**。

**你可以开启的三个开关：**

- **FORCE_PLAN（强制计划）**：如果发现 SQL 变慢了（执行计划退化），AI 会自动切回之前那个快的计划。
- **CREATE_INDEX（自动创建索引）**：AI 发现你老是查某列但没索引，它会帮你建一个。
- **DROP_INDEX（自动删除索引）**：如果某个索引一年没人用，AI 会把它删了省空间。

------

**2. 本地 SQL Server (2017及以上)**

在本地版本中，它主要负责**“计划回归修正（Plan Regression Correction）”**，也就是帮你把变慢的 SQL 拉回正轨。

**设置路径（使用 SSMS）：**

1. 右键点击你的 **数据库** -> **属性 (Properties)**。
2. 点击左侧的 **“选项 (Options)”**。
3. 在右侧面板中找到 **“自动优化 (Automatic Tuning)”** 部分。
4. 将 **“所需的计划强制模式 (Desired Plan Forcing Mode)”** 设置为 **ON**。

**或者使用 T-SQL（更推荐）：**

```SQL
-- 开启自动强制计划
ALTER DATABASE CURRENT 
SET AUTOMATIC_TUNING ( FORCE_LAST_GOOD_PLAN = ON );
```

------

**3. 理解“自我纠正”的逻辑**

你提到它能“自我纠正”，这个过程非常巧妙，是一个 **PDCA（计划-执行-检查-处理）** 循环：

1. **监控**：查询存储（Query Store）记录每个 SQL 的运行速度。
2. **发现**：AI 发现同一个 SQL 昨天跑 1 秒，今天跑了 10 秒（发生**退化**）。
3. **应用**：AI 自动下令：“别用今天这个烂计划了，强制用昨天那个好计划！”
4. **验证**：AI 观察一会儿。如果变快了，就保持；如果没变快甚至更慢了，它会立刻撤销操作（**回滚**），重新寻找方案。

------

**4. 关键前提：查询存储 (Query Store)**

无论是云端还是本地，**自动调优必须依赖查询存储**。如果查询存储没开，自动调优就是瞎子。

**检查/开启查询存储：**

```SQL
ALTER DATABASE [你的数据库名] 
SET QUERY_STORE = ON;
```

------

**总结**

- **Azure 用户**：去 Portal 页面点点鼠标就行，甚至能让它帮你加减索引。
- **本地用户**：主要用它来对付“变慢的查询”，在数据库属性的选项里开启。

#### 自动计划修正

> "通过查询存储中的数据，数据库引擎能够判断查询执行计划何时发生性能衰退。除了可以通过用户界面手动识别性能衰退的计划外，查询存储还具备自动通知功能。"![Screenshot of the Query Store view for regressed plan correction.](../assets/T-SQL/module-44-optimize-resources-final-25.avif)在上例中，您可以看到计划 ID 1 旁显示勾号标记，这表示该计划已被强制实施。功能启用后，数据库引擎将在以下情况自动强制实施所有推荐的查询执行计划：
>
> 与推荐计划相比，先前计划的错误率更高
>
> 预估的 CPU 增益高于 10 秒
>
> 该行动计划相比前一个表现更佳
>
> 查询执行 15 次后，计划将还原至上一次已知的有效版本。
>
> 自动执行计划强制时，数据库引擎会采用最近一次已知的良好计划，并密切监视查询执行性能。若强制计划的性能未优于原计划，数据库引擎会取消强制并编译新计划；反之，若强制计划持续优于原有不良计划，则该计划将保持强制状态，直至重新编译发生。
>
> 您可通过 T-SQL 查询启用自动计划修正功能。为确保操作成功，需提前启用查询存储并确保其以读写模式运行。若未满足任一条件，ALTER 语句执行将失败。
>
> ```SQL
> ALTER DATABASE [WideWorldImporters] SET AUTOMATIC_TUNING (FORCE_LAST_GOOD_PLAN = ON);
> ```
>
> 您可通过动态管理视图（DMV）查看自动调优建议，该功能支持 SQL Server 2017 及以上版本，也可在 Azure SQL Database 解决方案中使用。此 DMV 会提供建议的原因、类型及状态等信息。如需确认某数据库是否已启用自动调优，请核查相应视图。

**一、 查看执行计划（核心分析工具）**

执行计划是理解 SQL Server 如何查找数据的“地图”，也是调优的起点。

**1. 显示“实际执行计划”**

- **操作路径**：在 SSMS 查询窗口工具栏，点击 **Include Actual Execution Plan**。
- **快捷键**：`Ctrl + M`
- **使用场景**：运行 SQL 语句，并在结果栏查看真实的物理操作（如 Index Seek、Scan 或 Hash Match）。

**2. 查看“估算执行计划”**

- **快捷键**：`Ctrl + L`
- **使用场景**：不实际执行查询，仅根据当前的统计信息预测执行路径。适合测试大耗时查询，避免直接运行拖慢服务器。

------

**二、 查看慢查询与资源使用情况**

**1. 活动监视器 (Activity Monitor)**

- **操作路径**：SSMS 对象资源管理器 -> 服务器右键 -> **Activity Monitor**。
- **监控维度**：
  - **CPU 使用率**：查看系统实时负载。
  - **等待任务**：识别是否存在阻塞（Blocking）或磁盘瓶颈。
  - **最近耗时查询**：快速定位当前哪些 SQL 正在大量消耗资源。

**2. 查询存储 (Query Store)**

- **操作路径**：数据库 -> 展开文件夹 -> **Query Store**。
- **主要报表**：
  - **Top Resource Consuming Queries**：查看消耗资源最多的查询。
  - **Queries with Regressed Hits**：识别由于执行计划变动导致性能下降的查询。
- **注意**：需先在数据库属性中开启 Query Store 功能。

------

**三、 查看索引与统计信息**

**1. 索引属性查看**

- **操作路径**：数据库 -> 表 -> 右键属性 (Properties) -> **Indexes**。
- **手动检查**：查看索引的填充因子、碎片率等物理属性。

**2. 动态管理视图 (DMV)**

使用 SQL 脚本查看真实的索引使用频率，判断是否存在冗余索引：

```SQL
SELECT * FROM sys.dm_db_index_usage_stats;
```

------

**四、 识别性能图表来源**

如果你在文档或教程中看到资源走势图，它们通常来自以下位置：

1. **Activity Monitor**：实时的实时波形图。
2. **Query Store 仪表盘**：按时间轴分布的条形图。
3. **Azure SQL 控制台**：Performance Overview（性能概览）界面。

------

**五、 实践练习：性能诊断实验**

你可以通过以下简单步骤训练自己的直觉：

1. **准备查询**：

   ```SQL
   SELECT * FROM Sales.Orders WHERE OrderDate = '2023-01-01';
   ```

2. **观察步骤**：

   - 开启 `Actual Execution Plan` 并执行。
   - **观察操作**：检查是 **Index Seek**（利用索引定位）还是 **Index Scan**（全表扫描）。
   - **观察开销**：查看各操作节点的 **Cost %**（成本百分比），找出耗时最长的步骤。

```SQL
-- 或者，您可以通过以下查询查看数据库中已启用的自动调优功能：
SELECT name,
    desired_state_desc,
    actual_state_desc,
    reason_desc
FROM sys.database_automatic_tuning_options
```



### Describe intelligent query processing

> 在 SQL Server 2017、2019 以及 Azure SQL 中，微软为兼容性级别 140 和 150 引入了众多新功能。其中许多功能针对以往的一些反模式（例如使用用户定义的标量值函数和使用表变量）进行了修正。
>
> 这些功能可划分为几个主要类别：
>
> ![Screenshot of Intelligent Query Processing architecture.](../assets/T-SQL/module-44-optimize-resources-final-27.avif)
>
> 智能查询处理包含一系列功能，能够以最小的实施工作量提升现有工作负载的性能。
>
> 若要使工作负载自动适用智能查询处理，只需将数据库兼容级别更改为 150 即可。例如：
>
> ```sql
> ALTER DATABASE [WideWorldImportersDW] SET COMPATIBILITY_LEVEL = 150;
> ```

#### 自适应查询处理技术

> 自适应查询处理提供了多种选项，可根据查询的执行上下文实现更动态的查询处理。这些选项包含多项增强查询处理的功能。
>
> - 自适应连接——数据库引擎根据参与连接的行数，动态推迟选择使用哈希连接或嵌套循环连接。自适应连接目前仅在批处理执行模式下生效。
> - 交错执行——目前该功能主要支持多语句表值函数。在 SQL Server 2017 之前的版本中，多语句表值函数采用固定的行数估计值（根据 SQL Server 具体版本不同，通常为 1 行或 100 行）。当函数实际返回的行数远超此估计值时，可能导致生成的查询计划并非最优。通过交错执行机制，系统会在编译查询计划的其他部分之前，先执行多语句表值函数以获得实际行数，从而优化后续的查询计划编译。
> - 内存授予反馈功能用于优化查询性能。SQL Server 在查询初始执行计划时，会依据统计信息估算行数并分配相应的内存授予额度。若数据存在严重倾斜，可能导致行数估算过高或过低，进而引发内存过度分配（影响并发性能）或分配不足（导致数据写入 tempdb 临时库）。通过内存授予反馈机制，SQL Server 可自动检测此类情况，动态调整授予查询的内存大小，从而有效避免内存溢出或资源浪费。
>
> 在兼容性模式 150 下，所有功能均已自动启用，无需额外设置。

> "如同多语句表值函数（MSTVFs）一样，SQL Server 执行计划中的表变量原先也采用固定的单行估计值。这种固定估计方式在实际行数较多时，同样会导致性能问题。自 SQL Server 2019 起，表变量会进行实际行数分析。延迟编译机制与 MSTVFs 的交错执行原理相似，不同之处在于延迟编译是在查询首次编译时完成的，而非在执行计划中动态调整。"

**<u>如何理解延迟编译机制和MSTVFs 的交错执行的原理？</u>**

要理解这两个概念，我们得先看清 SQL Server 以前的“固执”之处：在过去，由于表变量和多语句表值函数（MSTVF）的内容是动态生成的，SQL Server 在编译查询计划时，因为“看不透”它们到底有多少数据，干脆就假设它们只有 **1 行**。

这就好比你去饭店点餐，厨师（优化器）假设你只有 **1 个人**（1 行估计值），于是只准备了 1 个小盘子，结果你带了 **100 个人**（实际行数）来，饭店立刻就乱套了（性能崩溃）。

------

**1. 理解：MSTVF 的“交错执行 (Interleaved Execution)”**

这是 SQL Server 2017 引入的技术。它的核心思想是：**“先停下来，数一数，再继续”。**

- **原理**：当优化器遇到一个 MSTVF 时，它不再瞎猜是 1 行。它会先执行这个函数的一部分，把数据真实地算出来。
- **动作**：
  1. 运行到函数这里，**暂停**主查询的编译。
  2. 把函数里的数据跑一遍，**数出真实的行数**。
  3. 把这个准确的数字传回给优化器。
  4. 优化器根据这个真实数字，重新决定后面是用“大卡车”还是“小推车”。

------

**2. 理解：表变量的“延迟编译 (Deferred Compilation)”**

这是 SQL Server 2019 针对**表变量**引入的升级。

- **区别点**：MSTVF 是在执行过程中“交错”进行的。而表变量是在**查询真正要开始跑的那一刻**，才进行编译。
- **原理**：
  1. 当你定义一个表变量并往里塞了 5000 行数据后，紧接着写了一个 `SELECT` 关联这个表变量。
  2. SQL Server 不会在你写脚本时就定死计划。
  3. 它会等数据都塞进表变量后，在 `SELECT` 语句**第一次编译**时，去看一眼这个表变量当前真实的大小。
  4. 因为它“延迟”了编译的时机，所以它看到了真相。

------

**3. 两者的“神似”与“形散”**

你提供的这段话总结得非常精辟，我们可以通过这个对比表来看清它们的共性与区别：

| 特性           | MSTVF 交错执行                     | 表变量延迟编译                   |
| -------------- | ---------------------------------- | -------------------------------- |
| **解决的问题** | 固定的 1 行估计值错误              | 固定的 1 行估计值错误            |
| **发生时间**   | **查询执行期间**（动态暂停并探测） | **查询首次编译时**               |
| **核心逻辑**   | 遇到函数先跑出结果，反馈给优化器   | 延迟到数据填充完毕后，再进行估算 |
| **结果**       | 获得了准确的基数（Cardinality）    | 获得了准确的基数（Cardinality）  |

**4. 为什么这很重要？**

以前因为估计值是 **1**，SQL Server 可能会选择 **Nested Loops Join（嵌套循环连接）**，这在处理几行数据时很快，但在处理几万行数据时会慢得像蜗牛。

有了延迟编译和交错执行，SQL Server 发现实际有几万行，它就会改用 **Hash Join（哈希连接）**。这就像是从“用勺子喂饭”升级到了“用传送带上菜”，效率提升是量级的。

------

**总结**

- **交错执行**是“且战且退”，边跑边数。
- **延迟编译**是“最后通牒”，等数据到位了再看。

它们的目的只有一个：**让优化器睁开眼看清楚真实的行数，别再把 100 个人当成 1 个人。**

#### 针对行存储的批处理模式

> 批处理执行模式支持以批为单位处理数据，而非逐行处理。对于计算和聚合操作消耗大量 CPU 资源的查询，这种处理模式的提升效果最为显著。将批处理与列存储索引解耦后，更多类型的工作负载均可享受到批处理模式带来的性能优势。

简单来说，这就像是**“用勺子喂饭”**和**“用大铲子端菜”**的区别。

在 SQL Server 中，传统的处理方式叫 **行模式（Row Mode）**，而进阶的方式就是 **批处理模式（Batch Mode）**。

------

**1. 形象比喻：超市收银**

**行模式（Row Mode）—— 逐行处理**

想象收银员每扫描一个商品，都要问你一次：“还要别的吗？”然后打印一张收银条。

- 扫描 1 个苹果 $\rightarrow$ 问一句 $\rightarrow$ 打一张单子。
- 扫描 1 个梨子 $\rightarrow$ 问一句 $\rightarrow$ 打一张单子。
- **缺点**：如果你买了 900 个苹果，收银员重复动作的次数太多，效率极低（CPU 浪费在了反复切换任务上）。

**批处理模式 (Batch Mode) —— 批量处理**

收银员拿出一张大单子，一次性扫描 900 个苹果，然后只问一次，打一张总单。

- 一次性处理一筐（通常是 **900 行**数据作为一个 Batch）。
- **优点**：大幅减少了 CPU 的“行政开销”。

------

**2. 真实 SQL 例子**

假设你有一张销售表 `Sales`，里面有 1 亿行数据。你需要计算总销售额：

```SQL
SELECT SUM(Price) FROM Sales;
```

**在行模式下：**

CPU 必须执行 1 亿次“读取数值 -> 累加”的操作。每次操作都要检查数据类型、处理空值等。这 1 亿次的**指令切换**会消耗大量的 CPU 时钟周期。

**在批处理模式下：**

1. 引擎把 1 亿行切成很多个“批次”（每个批次约 900 行）。
2. CPU 利用现代处理器的 **SIMD (单指令多数据流)** 技术。
3. 就像用一把大排刷刷墙，一次刷一排，而不是拿牙刷一点点点。
4. **结果**：原本需要 10 秒的聚合运算，现在可能只要 1 秒。

------

**3. 什么是“与列存储索引解耦”？**

这是 SQL Server 2019 的重大进步。

- **以前（2016 及更早）**：只有你建立了“列存储索引（Columnstore Index）”的表，才能享受批处理模式。如果你用的是普通的 B 树索引（行存储），就只能老老实实逐行处理。
- **现在（2019+）**：微软把这套高效的算法从索引里“拿出来”了。即便你的表是普通的行存储表，只要你的 SQL 满足条件（比如大数据量的聚合查询），优化器也会尝试开启 **Batch Mode on Rowstore**。

------

**4. 谁受益最大？**

正如你提供的那段话所说，受益最大的是：

- **大数量聚合**：如 `SUM`, `AVG`, `COUNT`。
- **大规模排序**：如 `ORDER BY`。
- **复杂连接**：如多个大表的 `JOIN`。

**总结**

**批处理模式**就是把“小步快跑”变成了“大步跨越”。解耦意味着你不再需要为了提速而强行改变表的结构，SQL Server 会在后台自动决定何时使用这种“大铲子”来搬运数据。

#### 标量用户定义函数的内联

> 在旧版 SQL Server 中，标量函数因多种原因性能不佳。这些函数以迭代方式执行，本质上是逐行处理数据。它们在执行计划中缺乏准确的成本估算，且无法在查询计划中实现并行处理。借助用户定义函数内联技术，这些函数会在执行计划中被转换为标量子查询，从而替代原有的用户定义函数运算符。这一转换能显著提升涉及标量函数调用的查询性能。

<u>**什么意思？什么是标量函数？**</u>

简单来说，**标量函数（Scalar Function）** 就是那种“丢进去一个值，吐出来一个值”的函数。

你可以把它想象成 Excel 里的公式：你在某一个单元格输入公式，它根据这一行的数据算出一个结果。

------

**1. 什么是标量函数？**

- **输入**：零个或多个参数（比如：单价、数量）。
- **输出**：**单个**数值（比如：总价、格式化后的字符串、计算后的日期）。
- **地位**：它通常出现在 `SELECT` 列表里，或者 `WHERE` 子句中。

------

**2. 举个例子：计算“含税价”**

假设你有一张订单表，你想把每件商品的“不含税价格”转换成“含税价格”。

**定义标量函数：**

```SQL
CREATE FUNCTION dbo.fn_GetTaxPrice (@Price MONEY)
RETURNS MONEY
AS
BEGIN
    -- 假设税率是 13%
    RETURN @Price * 1.13;
END
```

**使用标量函数：**

```SQL
SELECT 
    ProductName, 
    UnitPrice, 
    dbo.fn_GetTaxPrice(UnitPrice) AS TaxPrice -- 这里的函数就是标量函数
FROM Sales.Products;
```

------

**3. 为什么旧版本性能很差？（逐行处理）**

想象你有 **100 万行**订单数据。

- **旧版的工作模式（RBAR）**： SQL Server 的执行引擎像个搬运工。它每搬起一行数据（一行记录），就要放下活儿，去敲函数（`dbo.fn_GetTaxPrice`）的门，问：“喂，这个数乘 1.13 是多少？” 拿回结果后再回来搬下一行。
  - **后果**：这个“敲门、进门、问话、出门”的动作（上下文切换）重复了 **100 万次**。即使函数里的逻辑非常简单，光是往返的开销就能让查询慢得像蜗牛。
  - **无法分身**：因为每一行都要等函数返回，SQL Server 无法让多个 CPU 核心同时干活（不支持并行）。

------

**4. 什么是“内联（Inlining）”？（SQL Server 2019+）**

“内联”就是优化器变聪明了。它不再去敲函数的门，而是**直接把函数里的逻辑“拆开”，塞进主查询里。**

- **内联后的工作模式**： 优化器在生成执行计划时，直接把 `dbo.fn_GetTaxPrice(UnitPrice)` 替换成了 `UnitPrice * 1.13`。
  - **进化**：查询引擎现在看一眼就知道：“哦，就是把单价乘个 1.13 嘛！”
  - **优势**：
    1. **没有门槛**：不再需要反复进出函数环境，消除了切换开销。
    2. **人多力量大**：既然逻辑变简单了，SQL Server 可以启动 **并行处理（Parallelism）**，让 8 个 CPU 核心一起算，速度瞬间起飞。
    3. **成本透明**：优化器现在能准确算出这个计算需要多少 CPU 资源，从而选出更好的路线图。

------

**总结**

- **标量函数**：处理一行，出一个值。
- **旧版问题**：逐行迭代，像用勺子喂饭，慢且无法并行。
- **内联技术**：把函数逻辑直接写进 SQL 逻辑，像用收割机割麦子，极大地提升了效率。

#### 近似去重计数

> 数据仓库中一种常见的查询模式是统计订单或用户的不重复数量，这种操作对大型数据表的性能开销较大。近似不重复计数函数采用分组处理的方式，能够更快速地获取统计结果。该函数可确保在 97%的置信水平下，误差率不超过 2%。

**近似不重复计数函数（Approximate Distinct Count）** 是 SQL Server 2019 引入的一种高性能统计工具。它的核心语法是 `APPROX_COUNT_DISTINCT`。

在处理数亿行的大数据时，传统的 `COUNT(DISTINCT Column)` 为了给出“绝对精确”的数字，必须把所有不重复的值都存入内存进行排重，这会导致巨大的内存消耗和 CPU 瓶颈。而近似函数则通过**概率算法**来换取极高的执行速度。

------

**1. 它是如何工作的？（算法原理）**

`APPROX_COUNT_DISTINCT` 使用了名为 **HyperLogLog (HLL)** 的算法。

- **原理**：它并不会真的去记下每一个出现过的值，而是通过对数据进行哈希（Hash）处理，并观察哈希值中二进制“0”出现的规律。
- **形象比喻**：想象你在一个广场上数有多少不同的人。
  - **精确计数**：你需要给每个人发一张名片，最后收回名片数一数有多少张。如果你有 100 万个客人，你就得准备 100 万张纸，还得一张张对。
  - **近似计数**：你观察每个人的特征。比如你发现每 100 个人里大约只有 1 个人戴红帽子。你只要数广场上有多少红帽子，然后乘以 100 就能猜出总人数。虽然不绝对精确，但速度极快，且不需要发名片。

------

**2. 核心性能优势**

对于千万级、亿级的数据表，这种函数的优势体现在：

1. **内存占用极低**：它只需要极小的固定内存空间（通常几 KB），而传统的 `DISTINCT` 随数据量增长，内存需求会爆炸式增长。
2. **扫描速度极快**：它非常适合与 **批处理模式（Batch Mode）** 配合，减少了数据在 CPU 缓存和内存之间的往返。
3. **不会阻塞查询**：因为它对系统资源的索取非常节俭，不会因为一个统计操作就让整台服务器变卡。

------

**3. 什么是“97% 的置信度”和“2% 的误差”？**

这是衡量该函数“靠谱程度”的标准：

- **2% 误差**：如果你算出来的结果是 1,000,000，那么真实数字极大概率在 980,000 到 1,020,000 之间。
- **97% 置信水平**：意味着如果你运行这个统计 100 次，其中 97 次的结果都能满足上述 2% 的误差范围。

------

**4. 什么时候该用它？**

既然它不完全精确，那么它的应用场景非常明确：

- **适合使用（追求速度）**：
  - **仪表盘报表**：比如展示“今日活跃用户数”或“历史订单总数”，100 万和 101 万在视觉呈现上没有本质区别。
  - **大数据预估**：在做复杂分析前，先快速看一眼数据的大致规模。
  - **监控系统**：实时监控海量日志中的唯一 IP 数量。
- **不适合使用（追求精准）**：
  - **财务结算**：涉及钱的统计，差一分钱都不行。
  - **库存管理**：必须确保每一件商品的进出都准确对账。

------

**总结**

`APPROX_COUNT_DISTINCT` 是用**细微的精度损失**换取**极速响应**的利器。它是专门为数据仓库和大数据分析场景设计的，让你在面对“亿级数据”时，依然能像查小表一样秒出结果。

[Detect and correct fragmentation issues](https://microsoftlearning.github.io/dp-300-database-administrator/Instructions/Labs/07-detect-correct-fragmentation-issues.html)

## [Optimize query performance in Azure SQL](https://learn.microsoft.com/en-us/training/paths/optimize-query-performance-sql-server/)

### Explore query performance optimization

#### 简介

> 数据库性能调优中，最需要掌握的关键技能之一便是读懂查询执行计划。执行计划揭示了数据库引擎处理查询与返回结果的具体过程，通过深入分析，可精准定位性能瓶颈、优化查询效率，从而保障数据库平稳高效运行。

> 查询存储是一款不可或缺的工具，它能帮助您快速定位最消耗资源的查询，并追踪性能随时间的变化趋势。通过自动化收集查询计划和执行运行时的详细数据，您可以精准识别性能瓶颈，从而做出有效决策以优化查询效率。

> SQL Server 采用锁定与阻塞机制来管理并发操作，从而确保数据的一致性。这些机制能够防止多用户同时访问数据库时产生冲突，有效维护数据的完整性。此外，您还可以调整 SQL Server 的隔离级别，以精细调控并发管理策略。通过选择合适的隔离级别，可在数据一致性与系统性能之间取得平衡，确保数据库在不同工作负载下均能高效稳定运行。

#### 理解查询计划

> 在深入了解执行计划细节前，有必要先理解数据库优化器的工作原理。SQL Server 采用基于成本的查询优化器，它会依据相关列的统计信息和查询计划中各操作可能用到的索引，对多种可能的执行方案进行成本估算，从而为每个方案计算出总成本。对于复杂的查询，可能存在成千上万种执行计划，但优化器并不会逐一评估，而是通过启发式方法筛选出潜在的高效方案，最终从中选取成本最低的执行计划。

> 查询优化器是基于成本的优化器，为其提供准确的输入至关重要，这直接关系到其决策质量。SQL Server 依靠统计信息来追踪列和索引中的数据分布情况，且必须及时更新这些统计信息，以免生成低效的执行计划。尽管 SQL Server 会在表数据变更时自动更新统计信息，但对于数据频繁变动的情况，可能需要进行更频繁的更新。优化器在生成执行计划时会综合考虑多种因素，包括数据库兼容级别、基于统计信息的行数估算以及可用索引等。

> 用户向数据库引擎发起查询后，将执行以下流程：
>
> 系统会解析查询的语法，若语法无误，则生成数据库对象的解析树。
>
> 解析树随后被送入名为“代数器”的数据库引擎组件进行绑定。此步骤会验证查询涉及的列和对象是否存在，并识别所处理的数据类型。最终输出的是一个查询处理器树结构，它将作为后续步骤的输入内容。
>
> 查询优化过程对 CPU 资源消耗较大，因此数据库引擎会将执行计划存储在一个名为“计划缓存”的专用内存区域内。若该查询的执行计划已存在，则直接从缓存中提取。缓存中的每个查询都会根据其 T-SQL 语句生成一个称为 query_hash 的哈希值。引擎会为当前查询生成 query_hash，并在计划缓存中查找与之匹配的现有计划。
>
> 如若不存在现有执行计划，查询优化器将采用其基于成本的优化器，依据相关列、表及索引的统计信息生成若干备选执行方案，最终输出查询执行计划。
>
> 查询使用计划缓存中的执行计划或前一步生成的新计划来执行，输出即为查询结果。

**若要深入了解查询处理器的工作原理，请参阅[《查询处理体系结构指南》](https://learn.microsoft.com/en-us/sql/relational-databases/query-processing-architecture-guide)**

> ```sql
> SELECT orderdate,
>      AVG(salesAmount)
> FROM FactResellerSales
> WHERE ShipDate = '2013-07-07'
> GROUP BY orderdate;
> ```
>
> 本示例中，SQL Server 首先检查 FactResellerSales 表中是否存在 OrderDate、ShipDate 和 SalesAmount 列。若存在，系统将为该查询生成哈希值，并在计划缓存中搜索匹配项。若找到匹配的哈希值，则引擎尝试复用对应执行计划；若未找到，SQL Server 将转而检查 OrderDate 和 ShipDate 列上的可用统计信息。
>
> 该查询中引用 ShipDate 列的 `WHERE` 子句称为谓词。若存在包含 ShipDate 列的非聚集索引，只要其成本低于从聚集索引获取数据，SQL Server 很可能会在查询计划中采用该索引。查询优化器随后会从可用方案中选择成本最低的执行计划并运行查询。
>
> 查询计划通过组合一系列关系运算符来检索数据，其中包含了诸如估计行数等信息。执行计划的另一要素是执行连接或数据排序等操作所需的内存，称为内存授权。内存授权突显了统计信息的重要性。若 SQL Server 估计某运算符将返回 10,000,000 行数据，而实际仅返回 100 行，系统就会为该查询分配过大的内存授权。过大的内存授权可能引发两个问题：其一，查询可能遇到 `RESOURCE_SEMAPHORE` 等待，这表示它正在等待 SQL Server 分配大量内存。SQL Server 默认会等待查询执行成本的 25 倍时长（以秒计，最长可达 24 小时）后才执行查询。其二，若查询执行时可用内存不足，数据将溢出到 tempdb 中处理，其速度远低于内存操作。
>
> 查询计划通过组合一系列关系运算符来检索数据，其中包含了诸如估计行数等信息。执行计划的另一要素是执行连接或数据排序等操作所需的内存，称为内存授权。内存授权突显了统计信息的重要性。若 SQL Server 估计某运算符将返回 10,000,000 行数据，而实际仅返回 100 行，系统就会为该查询分配过大的内存授权。过大的内存授权可能引发两个问题：其一，查询可能遇到 `RESOURCE_SEMAPHORE` 等待，这表示它正在等待 SQL Server 分配大量内存。SQL Server 默认会等待查询执行成本的 25 倍时长（以秒计，最长可达 24 小时）后才执行查询。其二，若查询执行时可用内存不足，数据将溢出到 tempdb 中处理，其速度远低于内存操作。
>
> 执行计划还存储查询的其他元数据信息，例如数据库兼容级别、并行度，以及若查询已参数化时所提供的参数。

> 查询计划既可通过图形界面查看，也可采用基于文本的格式显示。基于文本的选项需使用 SET 命令激活，且仅对当前连接生效。您可以在任何能执行 T-SQL 查询的环境中查看这些计划。
>
> 大多数数据库管理员倾向于使用图形化计划，因为它能直观展示计划的整体结构和流程。查看和保存图形化查询执行计划有多种方式，其中最常用的工具是 SQL Server Management Studio。此外，也存在一些支持查看图形化执行计划的第三方工具。

**1. 基于文本的格式显示（Text-based）**

这种方法不需要图形界面，直接在查询结果窗口以纯文本形式输出。它通过 `SET` 命令开启，且**仅对当前窗口（连接）有效**。

常用的命令有三种：

- **仅查看估计计划（不执行 SQL）**：

  ```SQL
  SET SHOWPLAN_TEXT ON;   -- 简单的文本描述
  GO
  SELECT * FROM Employees WHERE ID = 1;
  GO
  SET SHOWPLAN_TEXT OFF;
  ```

- **查看详细估计计划（包含开销估算）**：

  ```SQL
  SET SHOWPLAN_ALL ON;    -- 输出包含行数、IO、CPU开销等详细列表
  GO
  -- 执行你的查询...
  SET SHOWPLAN_ALL OFF;
  ```

- **查看实际执行计划（执行 SQL 后输出统计）**：

  ```SQL
  SET STATISTICS PROFILE ON; 
  GO
  -- 执行你的查询...
  SET STATISTICS PROFILE OFF;
  ```

------

**2. 通过图形界面显示 (Graphical)**

这是最主流的方式，通过图标、箭头和颜色来展示数据的流向。

**在 SSMS 中操作：**

- **快捷键**：`Ctrl + M`（开启实际执行计划）或 `Ctrl + L`（显示估计执行计划）。
- **按钮**：点击工具栏上的“包含实际执行计划”图标。
- **交互性**：你可以把鼠标悬停在每个图标（操作符）上，查看详细的弹出窗口。

------

**3. 支持图形化执行计划的第三方工具**

除了微软自家的 SSMS 和 Azure Data Studio，DBA 们经常会用到一些功能更强大的专业工具：

**1. SentryOne Plan Explorer (强烈推荐)**

- **特点**：这是 DBA 圈子里最出名的免费工具。
- **优势**：它的图形比 SSMS 更清晰，能自动高亮显示“开销最大”的路径，并提供更准确的统计颜色对比。

**2. Redgate SQL Prompt / SQL Monitor**

- **特点**：高级辅助插件。
- **优势**：在编写 SQL 的同时就能快速预览执行计划，并给出优化建议。

**3. ApexSQL Plan**

- **特点**：专门用于分析执行计划的工具。
- **优势**：支持查看执行计划的历史对比，方便你找出为什么昨天的计划比今天快。

**4. SolarWinds Database Performance Analyzer (DPA)**

- **特点**：企业级监控平台。
- **优势**：从时间维度展示执行计划的变化，适合查找突发的性能衰减。

------

**总结**

- **文本格式**：适合在自动化脚本中抓取信息，或者在没有图形界面的终端环境下使用。
- **图形格式**：适合肉眼快速定位瓶颈（比如看哪个箭头最粗、哪个节点报错）。

#### 预估执行计划

> 这类执行计划由查询优化器生成。其内存授予的元数据及大小均依据查询编译时数据库中的统计信息进行估算。如需查看文本形式的预估执行计划，可在执行查询前运行 `SET SHOWPLAN_ALL ON` 命令。执行该查询时，您将看到执行计划的各个步骤，但查询本身不会实际运行，也不会返回结果。SET 选项的设置将持续生效，直至手动将其设为 OFF。

#### 实际执行的计划

> 此类计划与预估计划类似，但额外包含了查询的执行上下文信息。该上下文涵盖预估与实际行数、执行过程中的任何警告、实际并行度（使用的处理器数量），以及执行消耗的运行时间与 CPU 时间。若需查看文本格式的实际执行计划，可在运行查询前执行命令 `SET STATISTICS PROFILE ON` 。随后查询将正常执行，并同时返回执行计划与结果集。

#### 实时查询统计

> "此项计划查看功能将预估执行计划与实际执行计划合成为动态视图，通过运算符动画展示查询执行进度。该视图每秒刷新，实时显示流经各运算符的数据行数。实时查询统计的另一优势在于能够清晰呈现运算符间的数据传递过程，有助于性能问题诊断。需要注意的是，由于采用动画形式，此功能仅支持图形化执行计划展示。"

### Explain estimated and actual query plans

> 实际执行计划与预估执行计划常令人混淆。两者的主要区别在于：实际计划包含了运行时统计信息，而预估计划则没有。在绝大多数情况下，它们所使用的运算符和执行顺序是相同的。还需注意的是，获取实际执行计划需要真正执行查询，这可能会耗费时间，甚至有时无法实现（例如，某些 `UPDATE` 语句只能执行一次）。不过，若您需要同时查看查询结果和执行计划，则必须选择实际执行计划的获取方式。
>
> ![Screenshot of an estimated execution plan generated in SQL Server Management Studio.](../assets/T-SQL/module-55-optimize-queries-final-01.avif)
>
> 如图所示，您可以在 SQL Server Management Studio (SSMS) 中，点击“估计查询计划”按钮（或使用快捷键 Ctrl+L）来生成估计执行计划。若需生成实际执行计划，可点击相应图标（或使用快捷键 Ctrl+M），然后执行查询。这两个功能按钮的运行机制有所不同：“包含估计查询计划”会立即对当前选中的查询（若未选中任何内容则针对整个工作区）生效；而“包含实际查询计划”则需要先执行查询才能生成计划。
>
> 由于执行查询和生成预计执行计划均会产生开销，因此在生产环境中查看执行计划时需格外谨慎。
>
> 通常，在编写查询时，您可以使用预估执行计划来了解查询的性能特点，找出缺失的索引或发现查询中的异常。而实际执行计划则更适合用于理解查询在运行时的实际性能，关键是能发现统计信息中的不足之处，这些不足会导致查询优化器基于现有数据做出不够理想的优化选择。

**<u>实际执行计划就是实际数据库统计到的数量和消耗的性能吗？预估执行计划就是数据库估计到需要查询到的数量？</u>**

**1. 核心区别：估计值 vs 真实值**

**预估执行计划 (Estimated Plan)**

- **数据来源**：它完全依赖于数据库中的**统计信息（Statistics）**。
- **逻辑**：优化器查了查“地图”，根据上次更新统计信息时记录的数据分布情况，**掐指一算**：“我觉得这次大概要扫描 1000 行，消耗 50 毫秒。”
- **特点**：**不执行代码**。所以即使你的 `DELETE` 语句会删掉 1 亿行数据，点“预估”也不会真的删数据。

**实际执行计划 (Actual Plan)**

- **数据来源**：来自**查询执行时的真实反馈**。
- **逻辑**：数据库真的去跑了这段代码。它记录下：“我本来以为只有 1000 行，结果实际跑出来发现有 **80 万行**（这叫基数估计错误）。”
- **特点**：**必须执行代码**。你会看到真实的 CPU 耗时、真实的内存占用和真实的物理 IO 次数。

------

**2. 为什么实际计划能发现“统计信息的不足”？**

这是实际执行计划最有价值的地方。当你把鼠标悬停在图标上时，你会看到两行数据：

- **Estimated Number of Rows (预估行数)**
- **Actual Number of Rows (实际行数)**

**如果这两个数字相差巨大（例如：预估 1 行，实际 100 万行）：**

这就是你提到的“统计信息不足”。说明数据库的“地图”太旧了，或者数据分布太诡异，导致优化器做出了错误的判断（比如选了慢速路而非高速路）。**这种情况在预估计划里是看不出来的**，因为预估计划只显示它“以为”的样子。

------

**3. 对比总结表**

| **维度**         | **预估执行计划 (Ctrl+L)**      | **实际执行计划 (Ctrl+M + 执行)**       |
| ---------------- | ------------------------------ | -------------------------------------- |
| **是否运行 SQL** | **否**                         | **是**                                 |
| **性能开销**     | 极小（仅编译开销）             | **大**（等于 SQL 运行本身的开销）      |
| **包含的信息**   | 预估行数、预估成本             | 预估行数、**实际行数**、实际 IO/时间   |
| **生产环境风险** | 安全（除非触发某些特殊架构锁） | **危险**（如果是增删改操作，数据会变） |
| **主要用途**     | 语法检查、快速查看索引缺失     | 性能调优、排查统计信息过期             |

------

**4. 生产环境的“避坑”建议**

正如你提供的文字所说，在生产环境中查看执行计划要谨慎：

1. **对于 SELECT 查询**：如果要查实际计划，且查询本身很慢，可能会导致服务器瞬间压力过大（CPU 飙升）。

2. **对于 UPDATE/DELETE 查询**：

   - **绝对不要**为了看执行计划而在生产环境直接按 `Ctrl+M` 运行。
   - **替代方案**：先用 `Ctrl+L` 看预估计划。如果非要看实际的，建议放入 **事务 (TRANSACTION)** 中执行后回滚，或者在测试环境模拟。

   ------

   **总结**

   **预估执行计划**是数据库“想象中”的查询过程；**实际执行计划**是数据库“真实发生”的查询过程。

   当你想知道“为什么这个 SQL 还没跑完”时，你需要**实际计划**；当你正在写代码想知道“加个索引会不会快一点”时，**预估计划**通常就够了。

#### **查看查询计划**

执行计划显示了数据库引擎为满足查询需求而获取数据时执行的具体任务。下面我们来详细解析该计划。

```SQL
SELECT [stockItemName]
 ,[UnitPrice] * [QuantityPerOuter] AS CostPerOuterBox
 ,[QuantityonHand]

FROM [Warehouse].[StockItems] s
 JOIN [Warehouse].[StockItemHoldings] sh ON s.StockItemID = sh.StockItemID
ORDER BY CostPerOuterBox;
```

> 该查询会将 StockItems 表与 StockItemHoldings 表进行关联，关联条件为两表的 StockItemID 列值相等。在执行查询的后续操作前，数据库引擎首先需要识别出这些匹配的行。
>
> ![Screenshot of a query execution plan.](../assets/T-SQL/module-55-optimize-queries-final-02.avif)
>
> 执行计划中的每个图标均代表一项特定操作，对应着构成执行计划的各类行动与决策。SQL Server 数据库引擎包含 100 多种查询运算符，均可作为执行计划的组成部分。每个运算符图标下方标注的成本百分比，是相对于查询总成本的比例。即使显示为 0%成本的操作，实际上仍会产生开销。该 0%显示是由于四舍五入规则所致——图形执行计划中的成本始终以整数形式呈现，而实际百分比小于 0.5%。
>
> 执行计划的流程遵循从右到左、从上到下的方向。 在此计划中，针对 StockItemHoldings.PK_Warehouse_StockItemHoldings 丛集索引的「丛集索引扫描」作是查询的首个步骤。 连接各运算子的线条粗细，取决于流向下一个运算子的预估数据列数量。 若出现较粗的箭头，代表运算子间传输的数据量较大，这可能意味着查询有优化的空间。 此外，将鼠标光标悬停在任一运算子上，还能通过工具提示查看更多详细信息。

**<u>我记得是有执行顺序，先找到FROM [Warehouse].[StockItems] 然后预估数量？然后下一个算子SELECT [stockItemName],[UnitPrice] * [QuantityPerOuter] AS CostPerOuterBox,[QuantityonHand]再预估数量吗？</u>**

你的理解非常有洞察力，但这里需要区分两个容易混淆的概念：**SQL 语句的逻辑执行顺序**和**执行计划的物理执行流程**。

你记忆中的“先找到 `FROM`”，其实是 SQL 语言的**逻辑解析顺序**。而执行计划展示的是数据库引擎为了拿到结果，真实在内存和硬盘上执行的**物理操作流程**。

------

**1. 逻辑顺序 vs. 物理流向**

**你记忆中的逻辑顺序（代码怎么被读懂）：**

当你写下这段 SQL 时，数据库是这样“理解”它的：

1. **FROM / JOIN**：先确定数据源（`StockItems` 和 `StockItemHoldings`）。
2. **SELECT**：计算 `UnitPrice * QuantityPerOuter` 并选出需要的列。
3. **ORDER BY**：最后对结果集进行排序。

**执行计划的物理流向（数据怎么跑）：**

但在**图形化执行计划**中，数据的流动是从**最右边**开始，像流水线一样流向左边的：

- **第一步（右侧末端）**：引擎分别从两个表的索引里去“抓取”数据。
  - 此时，你会看到 **Clustered Index Scan**（扫描）。
  - **预估数量**：就在这一步发生。引擎看一眼统计信息，预估这里会吐出多少行。
- **第二步（中间）**：执行 **Inner Join**（通常是 Hash Match 或 Merge Join）。
  - 两个表的数据在这里汇合。
- **第三步（向左移动）**：执行 **Compute Scalar**。
  - 这就是你写的 `[UnitPrice] * [QuantityPerOuter]`。它不是在最开始算的，而是在数据流动的过程中计算出来的。
- **第四步（接近左侧）**：执行 **Sort**。
  - 因为你有 `ORDER BY`，数据流在这里被拦截并重新排列。
- **最后一步（最左侧）**：**SELECT**。
  - 将最终结果呈现给你。

------

**2. 怎么理解“箭头的粗细”与预估数量？**

在执行计划中，每一个**箭头（线条）**上其实都背负着“预估值”：

1. **初始箭头**：从最右边的“扫描”操作出来的箭头，它的粗细取决于表里总共有多少行。
2. **筛选后的箭头**：如果中间有 `WHERE` 过滤，箭头会变细。
3. **JOIN 后的箭头**：两个箭头合二为一，其粗细代表了连接后产生的结果集大小。
4. **SELECT 前的箭头**：这就是你最终会拿到的行数预估。

**为什么关注粗箭头？** 如果从 `FROM` 表出来的第一条线就特别粗（比如 1 亿行），而最后的线特别细（比如 10 行），说明你让数据库搬运了太多没用的废料，这就是优化空间——比如考虑加个索引，让它在第一步就变细。

------

**3. 针对你这段代码的执行计划示意**

你的查询包含了一个 `JOIN` 和一个 `ORDER BY`，执行计划大概长这样：

- **(右下)** 扫描 `StockItemHoldings` 表 → **[预估行数 A]**
- **(右上)** 扫描 `StockItems` 表 → **[预估行数 B]**
- **(汇合)** **Hash Match (Inner Join)** → **[预估连接后的行数]**
- **(处理)** **Compute Scalar** (计算乘法)
- **(排序)** **Sort** (因为你要排序，这是个重体力活)
- **(终点)** **SELECT** (输出结果)

------

**4. 关键点拨：SELECT 不是一个“动作”**

在执行计划里，`SELECT` 通常只是最左边的一个图标（Result），它本身不产出数据，它只是整个流水线的**出口**。

你提到的 `[UnitPrice] * [QuantityPerOuter]` 这种计算，在执行计划里通常对应一个叫 **Compute Scalar（计算标量）** 的小图标。它会根据前一个步骤传过来的每一行数据，实时算出这个乘法结果。

**总结**

- **逻辑上**：先 `FROM` 再 `SELECT` 是对的。
- **物理上**：数据从最右边的**索引扫描**开始，沿着**箭头**流过计算节点和排序节点，最后到达最左边的 **SELECT** 终点。
- **预估行数**：是分布在**每一个箭头**上的，你可以看到每一阶段数据是怎么变多或变少的。

> 执行计划的流程遵循从右到左、从上到下的方向。 在此计划中，针对 StockItemHoldings.PK_Warehouse_StockItemHoldings 丛集索引的「丛集索引扫描」作是查询的首个步骤。 连接各运算子的线条粗细，取决于流向下一个运算子的预估数据列数量。 若出现较粗的箭头，代表运算子间传输的数据量较大，这可能意味着查询有优化的空间。 此外，将鼠标光标悬停在任一运算子上，还能通过工具提示查看更多详细信息。
>
> ![module-55-optimize-queries-final-03.png](../assets/T-SQL/module-55-optimize-queries-final-03.avif)
>
> 工具提示会高亮显示预估执行计划的成本与估算数据；对于实际执行计划，还会包含与实际行数和成本的对比信息。每个运算符都拥有比工具提示更详尽的属性信息。右键单击特定运算符，从上下文菜单中选择“属性”选项即可查看完整属性列表。此操作将在 SQL Server Management Studio 右侧（默认位置）打开独立属性窗格。打开属性窗格后，选择任一运算符即可查看其详细属性。此外，也可通过 SQL Server Management Studio 主菜单的“视图”项选择“属性”来打开该窗格。
>
> ![module-55-optimize-queries-final-04.png](../assets/T-SQL/module-55-optimize-queries-final-04.avif)
>
> 属性窗格提供更多信息并列出输出列，这些列将被传递给下一操作符。分析聚集索引扫描时，这些列可能意味着需要借助非聚集索引来优化查询性能。由于聚集索引扫描会读取整张表，在此场景中，为每个表的 StockItemID 列建立非聚集索引可能会更高效。

**<u>如何理解"分析聚集索引扫描时，这些列可能意味着需要借助非聚集索引来优化查询性能。由于聚集索引扫描会读取整张表，在此场景中，为每个表的 StockItemID 列建立非聚集索引可能会更高效。"?</u>**

这句话是数据库调优中最经典的逻辑之一：**用“更薄的书”代替“整本字典”**。

为了理解为什么“输出列”能暗示你需要非聚集索引，我们可以拆解成三个层次来看：

------

**1. 为什么聚集索引扫描（Clustered Index Scan）性能差？**

**聚集索引（Clustered Index）本身就是表数据。** * **形象比喻**：想象一本《现代汉语词典》，聚集索引就是按拼音排好序的正文。

- **物理代价**：当你进行“聚集索引扫描”时，SQL Server 必须把字典从第一页翻到最后一页。哪怕你只需要查每个字对应的“笔画数”，你也得把包含字义、例句、拼音的所有页面全部读入内存。
- **结论**：它太“胖”了，导致了大量的 IO 开销。

------

**2. “这些输出列”是如何泄露秘密的？**

当你查看执行计划中“聚集索引扫描”算子的**属性窗格（Properties）**时，会看到一个 **输出列表（Output List）**。

假设你的查询只需要三个列：`StockItemID`, `StockItemName`, `UnitPrice`。

- 如果输出列表里只有这三列，但你却在做“聚集索引扫描”，这意味着：数据库为了拿到这三样东西，不得不搬运了整行（可能包含备注、图片、长字符串等几十列）数据。
- **这就是暗示点**：如果我能把这三列单独抽出来，做成一个小册子（**非聚集索引**），数据库就不需要去搬那本厚字典了。

------

**3. 非聚集索引为什么更高效？**

非聚集索引（Non-Clustered Index）是一个独立的存储结构。

- **更瘦、更窄**：它只包含你指定的列（比如 `StockItemID`）以及指向原书的页码。
- **IO 优势**：
  - 字典正文（聚集索引）可能有 1000 页。
  - 提取出来的“ID 索引表”（非聚集索引）可能只有 10 页。
- **场景分析**：在你的 `JOIN` 例子中，数据库需要匹配两表的 `StockItemID`。
  - 如果有非聚集索引，引擎只需要扫描那 10 页纸就能完成匹配。
  - 如果没有，引擎必须扫描 1000 页纸来找 ID。

------

**4. 实例说明**

回到你之前的查询：

```SQL
SELECT s.[stockItemName], sh.[QuantityonHand]
FROM [Warehouse].[StockItems] s
JOIN [Warehouse].[StockItemHoldings] sh ON s.StockItemID = sh.StockItemID
```

**如果没有非聚集索引：** 执行计划会显示 **Clustered Index Scan**。

- **原因**：它必须遍历整张表来找到所有 `StockItemID` 进行匹配，然后再顺便带出 `stockItemName`。

**如果你在 `StockItemID` 上建了非聚集索引：** 执行计划可能会变为 **Index Scan**（注意没有 Clustered 单词）或 **Index Seek**。

- **结果**：引擎只读取索引页。因为索引页非常小，加载到内存的速度极快，查询性能通常会有量级的提升。

------

**总结**

这句话的意思是： 通过观察算子输出了哪些列，你可以发现查询其实只关心表中的一小部分信息。既然如此，与其扫描**包含所有信息**的厚表（聚集索引），不如建一个**只包含关键列**的薄表（非聚集索引），让数据库能以最少的体力（IO）干完活。

<u>**非聚集索引就是有索引指向，聚集索引扫描就是没索引指向吗？**</u>

你可以这么粗略地理解，但更精准地说，两者的区别不在于“有没有指向”，而在于**“你手里的那本书是怎么摆放的”**。

我们可以把数据库比作一间巨大的**储藏室**：

**1. 聚集索引扫描（Clustered Index Scan）：这就是“翻遍全库”**

**你的理解**：没索引指向。 **修正理解**：其实你是**顺着书架一本书一本书地摸过去**。

- 在 SQL Server 中，**聚集索引本身就是数据**。
- 当执行计划显示“聚集索引扫描”时，意味着数据库引擎没有找到更快捷的入口，只能从这张表的第一行一直读到最后一行，直到把整张表翻完。
- **代价**：非常大。因为每一行数据（包含所有字段）都会被加载进内存，就像为了找一张发票，你把家里所有的档案袋全部拆开看了一遍。

------

**2. 非聚集索引（Non-Clustered Index）：这就是“翻看目录”**

**你的理解**：有索引指向。 **核心原理**：你手里拿着一本**缩印版的“小目录”**。

- 非聚集索引是一个独立于数据表之外的小结构，它只记录了**索引列的值**和**一个指向实际数据的指针**。
- 当数据库使用非聚集索引时，它先去翻这本薄薄的目录，找到对应的页码，然后再精准地跳到那一行。
- **代价**：非常小。因为目录很薄，翻得极快。

------

**3. 为什么扫描聚集索引时要建议建非聚集索引？**

回到你之前看的那段话，它的逻辑是这样的：

1. **现状**：你现在为了找 `StockItemID`，正在翻整本厚厚的“聚集索引字典”。
2. **发现**：系统通过“输出列（Output List）”发现，你其实只需要 `StockItemID` 这一个信息。
3. **建议**：既然你只要这一个信息，为什么不建一个只有 `StockItemID` 的“小目录（非聚集索引）”呢？
4. **效果**：以后你只要翻那几页纸的“目录”，不用再搬整台字典了。

**总结**

- **聚集索引扫描**：因为没有更窄的路径可用，被迫搬运整张表（全表物理扫描）。
- **非聚集索引**：为你提供了一条“瘦身版”的路径，让你绕过那些沉重的、不需要的数据列。

**所以，“聚集索引扫描”不代表没索引，而是代表你现在用的这个索引（主键/聚集索引）太胖了，干这活不划算。**

**想知道怎么在执行计划里一眼看出一个“非聚集索引”是不是真的帮到了忙吗？我们可以看看 Index Seek（查找）和 Index Scan（扫描）的区别。**

<u>**非聚集索引是一个独立于数据表之外的小结构，它只记录了索引列的值和一个指向实际数据的指针。就是索引列的值就是StockItemID，然后指向实际数据的指针就是StockItemName和UnitPrice这个键值对的地址吗？**</u>

这个理解已经非常接近底层逻辑了，但有一个关键的**“导航路径”**需要纠正。

在 SQL Server 中，非聚集索引指向实际数据的“指针”，并不是直接指向某个具体的键值对地址，而是指向**整行数据的“门牌号”**。

具体结构是这样的：

**1. 索引里存了什么？**

假设你在 `StockItemID` 上建了非聚集索引：

- **索引页内容**：只有 `StockItemID` 的值 + 一个**书签（Bookmark）**。
- 这个“书签”就是你说的指针，但它的内容取决于表是怎么组织的：
  - **如果表有聚集索引**：指针就是该行对应的**聚集索引键（通常是主键）**。
  - **如果表是堆（Heap，无聚集索引）**：指针就是该行的**物理行 ID (RID)**。

**2. 它是如何“找到” StockItemName 的？**

当你执行 `SELECT StockItemName FROM ... WHERE StockItemID = 10` 时，过程如下：

1. **查目录**：在非聚集索引里快速找到 `StockItemID = 10`。
2. **看书签**：索引说：“对应的聚集索引键是 `PK_10`”。
3. **回表（Key Lookup）**：数据库拿着这个 `PK_10`，**回到聚集索引（主键）那张大表里**，去翻那一整行数据。
4. **取值**：从那一整行里把 `StockItemName` 读出来。

------

**3. 一个极具价值的进阶概念：覆盖索引 (Covering Index)**

你刚才的理解里提到了“指向 `StockItemName` 的地址”，这其实引出了 SQL 优化中最高级的技巧：**包含列 (INCLUDE)**。

如果你经常需要根据 ID 查 Name，你可以建立这样一个索引：

**这时候，你的理解就完全正确了：**

- 现在非聚集索引里**直接存了** `StockItemName` 的值。
- 数据库查到 ID 后，**不需要**再回大表（聚集索引）里翻书了，直接在索引里就能把名字报出来。
- **效果**：这就是传说中的**“索引覆盖”**，性能比普通的非聚集索引还要快出一大截，因为它省去了“回表”的动作。

------

**总结**

- **普通非聚集索引**：索引里只存 ID，要名字得回大表再查一次（像查目录后再翻书）。
- **带 INCLUDE 的索引**：索引里存了 ID 和名字，查到 ID 的瞬间名字也就到手了（像目录里直接写了简介，不用翻书了）。

**在执行计划里，如果你看到一个图标叫“Key Lookup”，就说明你的索引没包含全部需要的列，数据库正在辛苦地“回表”。**

#### 轻量级查询性能分析

> 在生成实际执行计划时，无论通过 SSMS 还是扩展事件监控架构，都可能带来显著的系统开销。因此，该操作通常仅限于生产环境故障排查的场景。所谓观测开销，即监控运行中应用程序所产生的性能损耗。在某些情况下，这类损耗可能仅占用少量 CPU 资源（约几个百分点）；但在捕获实际执行计划等场景中，可能导致单个查询性能明显下降。SQL Server 引擎的传统性能分析机制在捕获查询信息时可能产生高达 75%的开销，而轻量级分析机制的最大开销则可控制在 2%左右。
>
> 在轻量级分析的初始版本中，系统会收集行数统计与 I/O 使用情况信息（即数据库引擎为执行指定查询所进行的逻辑读写和物理读写次数）。同时，新引入了名为 query_thread_profile 的扩展事件，用于检查查询计划中各个运算符的数据。在该版本中，需全局启用跟踪标志 7412 方可使用此功能。

**<u>跟踪标志 7412是什么？如何启用它？</u>**

**跟踪标志 7412 (Trace Flag 7412)** 就像是给 SQL Server 的监控引擎安装了一个“节能开关”。

在 SQL Server 的旧版本中，要看到 SQL 正在运行的实时进度，系统需要耗费巨大的力气去记录每一个动作，这会导致服务器变慢（观测开销高达 75%）。而开启 7412 后，SQL Server 会切换到**轻量级查询分析（Lightweight Query Profiling）**模式。

------

**1. 跟踪标志 7412 的作用**

它的主要作用是**允许系统以极低的代价（约 2%）收集查询的实时状态**。

启用它后，你可以享受到以下功能：

- **实时查询统计 (Live Query Statistics)**：在 SSMS 中，你可以看到数据在算子之间流动的“动画”，实时观察哪一步卡住了。
- **查看正在运行的计划**：通过特定的函数（如 `sys.dm_exec_query_statistics_xml`），你可以看到一个还没跑完的 SQL 已经处理了多少行。

------

**2. 如何启用跟踪标志 7412？**

根据你的需求，有两种常用的启用方式：

**方式 A：临时启用（立即生效，重启失效）**

这种方式适合你现在就要排查问题，不需要重启数据库服务。执行以下 T-SQL 语句：

*参数 `-1` 表示在全局范围内生效，而不仅仅是当前的连接。*

**方式 B：永久启用（重启依然有效）**

如果你希望服务器始终保持这个轻量级监控状态，需要将其添加到 SQL Server 的**启动参数**中：

1. 打开 **SQL Server 配置管理器 (SQL Server Configuration Manager)**。
2. 在左侧选择 **SQL Server 服务**。
3. 右键点击你的实例（如 `SQL Server (MSSQLSERVER)`），选择 **属性**。
4. 切换到 **启动参数 (Startup Parameters)** 选项卡。
5. 在参数框中输入 `-T7412`，点击 **添加 (Add)**。
6. 点击确定。注意：**此操作需要重启 SQL Server 服务才能生效。**

------

**3. 我还需要启用它吗？（版本提示）**

虽然 7412 在 SQL Server 2016 (SP1) 和 2017 中非常重要，但请注意：

- **SQL Server 2019 及更高版本**：轻量级分析功能**默认就是开启的**。你不需要设置任何跟踪标志就能享受这种高性能监控。
- **Azure SQL 数据库**：也是默认开启的。

------

**总结**

- **7412 是什么？** 一个让 SQL Server 能以“省电模式”实时监控查询进度的开关。
- **如何开启？** 临时用 `DBCC TRACEON`，永久用启动参数 `-T7412`。

> "若未全局启用轻量级性能分析功能，可通过结合使用 `USE HINT` 查询提示与 `QUERY_PLAN_PROFILE` ，在查询级别启用该功能。当包含此提示的查询执行完毕后，系统将生成 query_plan_profile 扩展事件，其中包含实际执行计划。以下为使用此提示的查询示例："
>
> ```sql
> SELECT [stockItemName]
>  ,[UnitPrice] * [QuantityPerOuter] AS CostPerOuterBox
>  ,[ QuantityonHand]
> FROM [Warehouse].[StockItems] s
>     JOIN [Warehouse].[StockItems] sh ON s.StockItemID = sh.StockItemID
> ORDER BY CostPerOuterBox 
> OPTION(USE HINT ('QUERY_PLAN_PROFILE'));
> ```

**<u>那轻量级分析机制是不是必开的?</u>**

简单直接的回答是：**在现代版本的 SQL Server 中，它确实是“必开”的，甚至微软已经帮你默认开启了。**

你可以把它理解为飞机的“黑匣子”。老款黑匣子记录数据时会拖慢飞机的速度（高开销），而新款黑匣子（轻量级分析）几乎不影响飞行，却能在出事时提供关键证据。

以下是为什么要开启（以及是否需要手动开启）的详细分析：

------

**1. 为什么它是“必开”的？**

如果不开启轻量级分析（Lightweight Profiling），当你遇到生产环境数据库突然变慢时，你会陷入非常被动的局面：

- **无法“动态”诊断**：你无法看到正在运行中的 SQL 跑到哪一步了（是卡在 Join 还是卡在 Scan？）。
- **诊断代价巨大**：如果你为了排查问题而开启“传统分析（Standard Profiling）”，原本就卡顿的服务器可能会因为那 **75% 的额外开销**直接崩溃。
- **事后复盘难**：很多性能指标（如每个算子的实际行数）将无法通过简单的动态管理视图（DMV）获取。

开启轻量级分析后，你可以随时查看**实时查询统计（Live Query Statistics）**，观察数据流在执行计划中的实时移动。

------

**2. 你的版本需要手动开吗？**

虽然它很好，但并不是所有版本都需要你手动去折腾 7412 标志：

- **SQL Server 2014 / 2016 (早期版本)**：不支持或支持不完善，建议升级。
- **SQL Server 2016 (SP1) / 2017**：**强烈建议开启**。你需要手动启用 **跟踪标志 7412**。
- **SQL Server 2019 / 2022**：**默认开启**。微软已经意识到这个功能太好用了，所以出厂时就是打开的，你不需要做任何操作。
- **Azure SQL (云端)**：**默认开启**。

------

**3. 开启后会有副作用吗？**

**几乎没有。**

正如你提供的那段文字所说，它的开销只有 **2% 左右**。对于绝大多数业务系统来说，这 2% 的 CPU 损耗换取的是极其珍贵的“可观测性”。在数据库运维中，这属于“高性价比”投资。

------

**4. 开启后的“杀手锏”功能：LAST_QUERY_PLAN_STATS**

当你开启了轻量级分析（或使用 2019+ 版本），你还可以开启一个更牛的功能：**捕获最后一次实际执行计划**。

**开启这个后，即使 SQL 已经跑完了，你也可以通过系统视图直接抓取它上次运行时的“实际执行计划”，而不需要在它运行时盯着屏幕看。** 这对于排查“半夜突然变慢的 SQL”简直是救命稻草。

------

**总结**

- 如果你用的是 **SQL Server 2019 或更高版本**：不用担心，它已经是必开且已开了。
- 如果你用的是 **2016/2017**：它是**准必选**的，赶紧去把 `-T7412` 启动参数加上吧。

#### 最近查询计划统计

> 轻量级查询分析在 SQL Server 2019、Azure SQL 数据库及托管实例中默认启用。该功能也可通过名为 `LIGHTWEIGHT_QUERY_PROFILING` 的数据库范围配置选项进行管理。借助此选项，您可以为任意用户数据库单独启用或禁用该功能，各数据库的设置互不影响。
>
> 此外，存在一个名为 `sys.dm_exec_query_plan_stats` 的动态管理函数，该函数可显示指定计划句柄对应的最近一次已知实际查询执行计划。若要通过该函数获取最近的实际查询计划，可在服务器范围启用跟踪标志 2451。另一种方法是，通过名为 `LAST_QUERY_PLAN_STATS` 的数据库作用域配置选项来启用此功能。
>
> 您可以结合此功能与其他对象，来获取所有缓存查询的最新执行计划：
>
> ```sql
> SELECT *
> FROM sys.dm_exec_cached_plans AS cp
>     CROSS APPLY sys.dm_exec_sql_text(plan_handle) AS st
>     CROSS APPLY sys.dm_exec_query_plan_stats(plan_handle) AS qps; 
> GO
> ```
>
> 该功能可以帮助您以极低的开销，快速获取系统中任一查询最近一次执行的运行时统计信息。下图演示了如何提取执行计划。若您选中结果第一列中的执行计划 XML 文件，系统将展示如下第二张图所示的执行计划详情。
>
> ![module-55-optimize-queries-final-05](../assets/T-SQL/module-55-optimize-queries-final-05.avif)
>
> 从下图列存储索引扫描的属性可以看出，缓存中获取的查询计划包含了本次查询实际检索到的行数。
>
> ![module-55-optimize-queries-final-06](../assets/T-SQL/module-55-optimize-queries-final-06.avif)

<u>**sys.dm_exec_query_plan_stats是启用最近一次已知实际查询执行计划的缓存的意思吗？**</u>

在没有这个功能之前，如果你想看“实际执行计划”，你必须在 SQL 运行的那一刻盯着屏幕看；如果 SQL 已经跑完了，你只能看到“预估计划”。

开启这个功能后，SQL Server 会在内存里**额外划出一块地**，专门记录这个 SQL 最近一次**真实跑完**时的样子。

------

**1. 深度拆解：它到底存了什么？**

普通计划缓存（Plan Cache）存的是“施工图纸”，而 `sys.dm_exec_query_plan_stats` 存的是“**带批注的施工报告**”。

- **记录内容**：它不仅包含 SQL 的逻辑，还记录了最后一次执行时的**实际行数**、**实际消耗的内存**、**实际产生的 IO** 等运行时统计信息。
- **如何提取**：你需要一个 `plan_handle`（计划句柄，相当于这张图纸的条形码），把它丢进这个函数里，它就会吐出那张带实际数据的执行计划。

------

**2. 为什么需要 2451 或 LAST_QUERY_PLAN_STATS？**

默认情况下，SQL Server 为了节省性能，**不存储**这些实际数据。

- **2451 (跟踪标志)**：这是一个全局开关。一旦开启，服务器上所有的数据库都会开始记录最近一次的实际计划。
- **LAST_QUERY_PLAN_STATS (数据库配置)**：这是一个更精细的开关。你可以只针对某个重要的数据库开启，而不影响其他不重要的数据库。

**注意：** 虽然这被称为“缓存”，但它是**覆盖式**的。它只记录**最后一次（Last Known）**。如果同一个 SQL 跑了 10 次，你只能看到第 10 次的实际情况，前 9 次的就消失了。

------

**3. 这个功能的“救命”时刻**

想象一下这个场景：

1. 半夜 3 点，有个报表 SQL 突然跑了 2 小时才结束。
2. 早上 9 点你上班时，SQL 早就结束了。
3. **如果没有这个功能**：你只能看到“预估计划”，看起来一切正常。你无法知道昨晚它到底在哪一步卡住了。
4. **有了这个功能**：你调用 `sys.dm_exec_query_plan_stats`，就能看到昨晚那次运行的**真实数据流**，一眼发现：“噢！原来昨晚这个 Join 产生了 1 亿行数据，难怪这么慢！”

------

**4. 总结**

`sys.dm_exec_query_plan_stats` 并不是“启用缓存”的动作，它是一个**查看工具**。

- **开关**是：`LAST_QUERY_PLAN_STATS`。
- **存储的内容**是：最后一次运行的**真实统计数据**。
- **意义**：让你具备“事后查账”的能力，而不需要实时守候。

### Describe dynamic management views and functions

> SQL Server 提供了数百种动态管理对象，它们包含监控服务器实例运行状况、诊断问题以及优化性能所需的系统信息。动态管理视图（DMV）和函数（DMF）能够返回数据库或实例内部状态数据。虽然这类对象包括视图和函数两种形式，但通常人们会统一使用"DMV"这一简称来指代它们。
>
> 动态管理视图分为两个级别：服务器级和数据库级。
>
> - 服务器范围的对象 - 需在服务器级别拥有 `VIEW SERVER STATE` 权限
> - 数据库作用域对象要求具备数据库内的 `VIEW DATABASE STATE` 权限

**<u>动态管理视图（DMV）是系统的封装的函数？，函数（DMF）是聚合函数这些吗？</u>**

这是一个非常棒的问题！虽然名字里都有“视图”和“函数”，但它们和你平时写的普通视图或聚合函数（如 `SUM`, `AVG`）在用途和逻辑上完全不同。

我们可以把 SQL Server 想象成一台正在高速运转的**跑车**：

- **仪表盘和传感器**：就是 **DMV/DMF**。它们实时显示引擎转速、油耗、胎压。
- **普通视图和函数**：则是你放在后备箱里的**货物**或**工具箱**。

------

**1. 动态管理视图 (DMV) 是什么？**

**不是你理解的“封装函数”**，它更像是一张**“实时变化的系统状态表”**。

- **特点**：它看起来像一张普通的表，你可以直接用 `SELECT * FROM` 来查。
- **数据来源**：它不存储在磁盘上，而是直接从 SQL Server 的**内存（引擎内部状态）**中读取数据。
- **例子**：`sys.dm_exec_requests`
  - 这张“表”里记录了此时此刻，服务器上正有哪些人在运行 SQL，已经运行了多久。

------

**2. 动态管理函数 (DMF) 是什么？**

**它不是聚合函数（SUM/COUNT）**，它更像是一个**“需要钥匙才能开启的保险箱”**。

- **特点**：它不能直接 `SELECT *`，你必须给它一个具体的**参数**，它才能告诉你结果。
- **区别**：普通函数（如 `GETDATE()`）是算值的；而 DMF 是用来**提取复杂的系统结构数据**的。
- **例子**：`sys.dm_exec_query_plan(plan_handle)`
  - 你得给它一个“计划句柄”（钥匙），它才会把那个复杂的 XML 执行计划吐给你。

------

**3. 三者对比表：带你彻底分清**

| **对象类型**     | **缩写** | **它的本质**               | **你怎么用它**                   |
| ---------------- | -------- | -------------------------- | -------------------------------- |
| **动态管理视图** | **DMV**  | 引擎内部状态的“快照”       | `SELECT * FROM sys.dm_...`       |
| **动态管理函数** | **DMF**  | 传入参数才能查到的深层数据 | `SELECT * FROM sys.dm_...(参数)` |
| **普通聚合函数** | **UDF**  | 逻辑计算工具（求和、计数） | `SELECT SUM(金额) FROM ...`      |

------

**4. 为什么大家统一叫它们“DMV”？**

就像我们平时把“移动电源、充电头、数据线”统称为“充电套件”一样。

因为 DMV 和 DMF 的**核心目的**是一样的：**偷窥 SQL Server 的内部运行真相。**

- 如果你想知道谁在拖慢速度，你查的是 DMV。
- 如果你想知道那个人运行的 SQL 长什么样，你调用的是 DMF。

它们通常是配合使用的，通过 `JOIN` 连在一起。

------

**总结建议**

- **DMV** = 系统自动维护的一张“实时动态表”。
- **DMF** = 系统提供的一个“需要输入参数的查询接口”。
- **聚合函数** = 你用来算账的数学工具。

> 这是一段 DBA（数据库管理员）在排障时最常用的“黄金组合”代码。它完美展示了 **DMV（视图）** 和 **DMF（函数）** 是如何协同工作的。
>
> 复制并在你的 SQL Server 查询窗口运行：
>
> ```sql
> SELECT TOP 5
>     r.session_id,                 -- 会话ID
>     r.status,                     -- 状态
>     r.cpu_time,                   -- 已消耗的 CPU 时间（毫秒）
>     r.logical_reads,              -- 逻辑读（内存操作）
>     t.text AS [SQL Text],         -- 通过 DMF 转换出的 SQL 文本
>     p.query_plan                  -- 通过 DMF 转换出的执行计划
> FROM sys.dm_exec_requests r                               -- 【DMV】正在运行的请求
> CROSS APPLY sys.dm_exec_sql_text(r.sql_handle) t          -- 【DMF】传入句柄，换取 SQL 文字
> CROSS APPLY sys.dm_exec_query_plan(r.plan_handle) p       -- 【DMF】传入句柄，换取图形计划
> WHERE r.session_id <> @@SPID                               -- 排除掉你自己正在运行的这行查询
> ORDER BY r.cpu_time DESC;
> ```
>
> ### 为什么这段代码能解释你的疑问？
>
> 1. **`sys.dm_exec_requests` (DMV)**： 它就像一张**自动填写的表格**。每一行代表一个正在跑的任务。你直接 `SELECT` 它，就能看到 `cpu_time` 这种数值。但它里面存的 SQL 内容是一串人类看不懂的二进制编码（`sql_handle`）。
> 2. **`sys.dm_exec_sql_text` (DMF)**： 它是一个**翻译机**。你把那个看不懂的 `sql_handle` 丢给它，它就吐出具体的 SQL 语句。
> 3. **`sys.dm_exec_query_plan` (DMF)**： 它是一个**绘图机**。你把 `plan_handle` 丢给它，它就吐出一个可以点击查看的 XML 蓝色链接，点开就是图形化的执行计划。
>
> ### 你会看到什么结果？
>
> - **[SQL Text] 列**：你会直接看到是谁在写垃圾代码，或者是哪个存储过程在疯狂运行。
> - **[query_plan] 列**：点击那个蓝色的 XML 链接，SQL Server 会直接为你打开对应的**图形执行计划**，你可以立刻开始分析它的箭头粗细和索引扫描情况。
>
> ### 总结
>
> 这就是动态管理对象的魅力：**DMV 提供线索，DMF 提供细节。**
>
> 如果你在结果里发现某个 `cpu_time` 特别高，且 `query_plan` 里全是“厚厚的粗箭头”，那么恭喜你，你已经抓到了拖慢服务器的“真凶”。

> 所有动态管理视图的名称均以 "sys.dm_" 开头，后接功能区域和对象的具体功能说明。SQL Server 支持三种类型的动态管理视图：
>
> - 与数据库相关的动态管理对象
> - 与查询执行相关的动态管理对象
> - 与事务有关的动态管理对象

> 在查询存储功能不可用的旧版 SQL Server 中，您可以通过视图 `sys.dm_exec_cached_plans` 配合函数 `sys.dm_exec_sql_text` 和 `sys.dm_exec_query_plan` 来获取执行计划的相关信息。但需注意，与查询存储不同，此方法无法显示特定查询的执行计划变更情况。

> "Azure SQL Database 与 SQL Server 在可用的 DMV（动态管理视图）上略有差异：部分对象仅存在于 Azure SQL Database 中，另一部分则仅存在于 SQL Server 中。有些 DMV 的作用域限定在服务器级别，不适用于 Azure 模型（例如 `waits_stats` DMV 就属于服务器作用域）；而像 `sys.dm_db_resource_stats` 这样的 DMV 则是 Azure SQL Database 独有的，它们提供了 SQL Server 中不存在（或不适用）的 Azure 平台特定信息。"

### Explore Query Store

> SQL Server 查询存储是一项按数据库提供的功能，可自动捕获查询历史记录、执行计划及运行时统计信息，从而简化性能故障排除与查询调优工作。此外，它还能帮助深入理解数据库的使用模式和资源消耗情况。
>
> The Query Store consists of three stores:
> 查询存储包含三个组成部分：
>
> 计划存储：用于存储预估执行计划信息。
>
> "运行时统计存储的功能是存储执行统计信息。"
>
> "等待统计存储：用于持久保存等待统计信息。"
>
> ![Screenshot of the Query Store components.](../assets/T-SQL/module-55-optimize-queries-final-09.avif)

#### 启用查询存储

> "Query Store 功能在 Azure SQL 数据库中默认已启用。若要在 SQL Server 或 Azure Synapse Analytics 中使用此功能，需先手动启用。你可根据环境运行以下查询以启用 Query Store："
>
> ```sql
> -- SQL Server
> ALTER DATABASE <database_name> SET QUERY_STORE = ON (OPERATION_MODE = READ_WRITE);
> 
> -- Azure Synapse Analytics
> ALTER DATABASE <database_name> SET QUERY_STORE = ON;
> ```

#### 如何收集数据工作

> 查询存储与查询处理管道在多个环节实现集成。各集成环节会在内存中收集数据，并采用异步方式写入磁盘，从而最大限度减少 I/O 开销。具体集成环节包括：
>
> 查询首次执行时，其查询文本及初始预估执行计划将发送至查询存储中进行持久化保存。
>
> 查询重新编译时，查询存储中的计划会相应更新。若重新编译后生成了新的执行计划，该计划也会持久保存至查询存储，以补充此前的历史计划。同时，查询存储会记录每个查询计划的执行统计数据，以便进行对比分析。
>
>
> 在查询的编译及重新编译检查阶段，查询存储会判断该查询是否存在被强制使用的执行计划。若查询存储提供的强制计划与过程缓存中的现有计划不一致，系统将重新编译该查询。
>
> 查询执行时，其运行时统计数据会持久化存储在查询存储中。查询存储将对这些数据进行聚合处理，从而为每个查询计划提供准确的呈现。
>
> ![Screenshot of the Query Store integration points in the query execution pipeline displayed as a flow chart.](../assets/T-SQL/module-55-optimize-queries-final-10.avif)
>
> 如需详细了解查询存储如何收集数据，请参阅[《查询存储的数据收集机制》](https://learn.microsoft.com/en-us/sql/relational-databases/performance/how-query-store-collects-data?view=sql-server-ver17)。

#### 常规场景

> SQL Server Query Store 能够为数据库操作性能提供有价值的洞察。常见使用场景包括：
>
> 识别和修复因查询执行计划选择不当导致的性能衰退。
>
> 识别并调优资源消耗最高的查询。
>
> A/B 测试，用于评估数据库和应用程序变更的影响。
>
> 确保 SQL Server 升级后性能保持稳定。
>
> 确定使用最频繁的查询
>
> 审计查询的查询计划历史记录。
>
> 识别并优化计划外工作负载。
>
> 理解数据库常见的等待类别，以及导致等待时间增加的查询语句和执行计划。
>
> 分析数据库资源（CPU、I/O、内存）消耗随时间变化的使用模式。

#### 探索查询存储视图

> 启用数据库的查询存储功能后，即可在对象资源管理器中看到该数据库对应的查询存储文件夹。在 Azure Synapse Analytics 中，查询存储视图位于系统视图目录下。这些视图能够聚合呈现 SQL Server 数据库的关键性能指标，帮助用户快速掌握系统运行状况。
>
> ![Screenshot of S S M S Object Explorer with the Query Store views highlighted.](../assets/T-SQL/module-55-optimize-queries-final-11.avif)

#### 性能退化的查询

> 当一个查询的性能因执行计划变更而逐渐下降时，我们称之为查询性能回归。导致预估执行计划变化的因素众多，例如架构改动、统计信息更新或索引调整。虽然首先想到的是检查过程缓存，但它仅保存查询最近一次的执行计划，并且这些计划可能因系统内存压力而被清除。与之相比，查询存储会持久化存储每个查询的多个执行计划，使得我们可以通过强制选择特定计划的方式，灵活应对因计划变更引发的查询性能退化问题。
>
> "回归查询视图可在指定时间段内，精准识别因执行计划变更而导致性能指标退化的查询。该视图支持按选定指标（如持续时间、CPU 时间、行数等）和统计方式（总计、平均值、最小值、最大值或标准差）进行筛选，并列出受影响最严重的前 25 条查询。默认以条形图形式直观展示查询结果，也可根据需要切换为网格视图进行查看。"
>
> 从左上角查询窗格中选择一个查询后，计划摘要窗格会展示该查询关联的各个历史持久化查询计划。在计划摘要窗格中选定某个查询计划时，底部窗格即会显示相应的图形化查询计划。计划摘要窗格和图形查询计划窗格中的工具栏按钮均支持为所选查询强制执行选定计划。此窗格布局与操作逻辑在所有 SQL 查询视图中保持一致。
>
> ![Screenshot of the Query Store Regressed Queries view displaying each of the different panes.](../assets/T-SQL/module-55-optimize-queries-final-12.avif)
>
> 另外，您也可以通过调用 `sp_query_store_force_plan` 存储过程来实现执行计划强制。
>
> ```SQL
> EXEC sp_query_store_force_plan @query_id=73, @plan_id=79
> ```

#### 总体资源消耗

> "总体资源消耗视图可用于分析指定时间段内多项执行指标（如执行次数、持续时间、等待时间等）的总资源消耗情况。图表具有交互功能：从任一图表中选定指标后，系统将在新标签页中展示对应的钻取视图，并列出与该指标相关的查询。"
>
> ![Screenshot of the SQL Query Store Overall resource consumption view with a configuration dialog indicating the different metrics available for display.](../assets/T-SQL/module-55-optimize-queries-final-13.avif)

## [Monitor and optimize operational resources in Azure SQL](https://learn.microsoft.com/en-us/training/modules/describe-performance-monitoring/2-describe-performance-monitoring-tools)

### Describe critical performance metrics

#### 需要关注的 SQL Server 指标

> Microsoft SQL Server 内置完善的性能数据采集机制，可收集大量性能元数据。数据库引擎提供的各项指标可用于监控并帮助识别、优化性能问题。部分操作系统指标仅能通过性能监视器查看，另一些则可通过 T-SQL 查询访问——特别是通过查询动态管理视图（DMV）获取。某些指标在两种途径中均可查看，因此明确特定指标的查看位置十分重要。例如，仅能通过 DMV 捕获的数据包括 `sys.dm_os_volume_stats` 中呈现的数据文件与事务日志文件的读写延迟。另一方面，SQL Server 无法直接提供的操作系统指标示例是磁盘卷的每次磁盘读写耗时（秒）。综合这两类指标，有助于更准确地判断性能问题源于数据库结构设计，还是物理存储瓶颈。

### Establish baseline metrics

#### 等待时间统计

> 当线程正在执行并被迫等待不可用资源时，SQL Server 会持续追踪这些指标。通过动态管理视图 (DMV) `sys.dm_os_wait_stats` 可以轻松识别这些信息。这些信息对于理解数据库的基准性能至关重要，能帮助识别查询执行和硬件限制方面的具体性能问题。识别适当的等待类型及相应解决方案对解决性能问题极为关键。等待统计信息在整个 Azure SQL 平台中均可获取。

### Explore extended events

> 扩展事件建立在 SQL Server Profiler 功能的基础上，支持追踪查询并公开更多可监控的数据（事件）。您可以通过扩展事件排查的一些问题示例如下：
>
> 排查阻塞和死锁性能问题。
>
>
> 识别长时间运行的查询。
>
> 监控数据定义语言（DDL）操作。
>
> 记录缺失列统计信息。
>
> 观察数据库中的内存压力。
>
> 长时间运行的物理 I/O 操作。

![Extended Event running on a SQL Database](../assets/T-SQL/extended-events.avif)

在此示例中，xe_deadlocks 是运行在 AdventureWorks 数据库上的扩展事件会话名称（位于图像左侧）。位于事件会话节点下的 event_counter 目标节点，用于统计事件会话中每个事件的发生次数。要在 SSMS 对象资源管理器中查看目标数据，您可以选中目标节点，然后选择“查看目标数据”。SSMS 将如我们在图像左侧所见显示数据，以及每个事件的计数结果。

**<u>什么是死锁？</u>**

简单来说，**死锁 (Deadlock)** 就像是在狭窄的单行道上，两辆车面对面相撞，谁也不肯退让，结果导致交通完全瘫痪。

在数据库（如 SQL Server）中，死锁是一种特定的竞争状态：**两个或多个任务都在等待对方持有的资源，导致逻辑上陷入了永久的循环等待。**

------

**1. 死锁发生的经典场景**

想象以下两个事务（Transaction）同时运行：

- **事务 A**：锁定了“表 1”，准备去更新“表 2”。
- **事务 B**：锁定了“表 2”，准备去更新“表 1”。

**结果**：

- 事务 A 在等事务 B 释放“表 2”。
- 事务 B 在等事务 A 释放“表 1”。
- **结局**：如果没有外部干预，这两个事务将永远等下去。

------

**2. SQL Server 如何处理死锁？**

数据库引擎拥有一个**死锁检测器 (Deadlock Monitor)**，它会定期扫描是否有循环等待。一旦发现死锁，它会：

1. **牺牲者选择**：自动选择其中一个事务作为“牺牲者”（通常是撤销成本最低、回滚数据最少的那个）。
2. **强制回滚**：终止并回滚该事务，并向应用程序抛出 **1205 错误码**。
3. **释放资源**：牺牲者释放了锁，另一个事务就能顺利完成。

------

**3. 为什么需要用“扩展事件 (Extended Events)”来监控它？**

虽然 SQL Server 会自动处理死锁，但死锁频繁发生会导致：

- **用户体验差**：部分操作总是随机失败。
- **系统吞吐量下降**：频繁的回滚和重试浪费了计算资源。

正如你提供的文档所言，**扩展事件**可以帮你捕获死锁发生的瞬间状态，包括：

- **死锁图 (Deadlock Graph)**：直观显示哪条 SQL 语句锁了哪个索引，哪条语句被阻塞。
- **受影响的查询**：查看导致冲突的具体代码。
- **等待类型**：分析是因为行锁、页锁还是其他资源导致的。

------

**4. 常见的死锁原因**

- **操作顺序不一致**：程序 A 访问顺序是 1->2，程序 B 是 2->1。
- **长事务**：事务开启后长时间不提交，占着资源不撒手。
- **缺少索引**：导致 SQL 必须扫描整张表（锁住更多行），增加了冲突概率。

#### 通过扩展事件可以监控哪些内容？

> 扩展事件全面覆盖 SQL Server 的各个层面，并划分为四个通道，用以区分不同事件的目标受众群体。
>
> 管理员事件面向最终用户和管理员。其中包含的事件揭示了管理员在执行一系列明确定义的操作时可能出现的问题。例如，生成 XML 死锁报告有助于定位死锁的根本原因。
>
> 操作事件 - 操作事件用于分析和诊断常见问题。这类事件可基于其发生来触发相应的操作或任务。例如，可用性组中的数据库状态变更，即表示发生了一次故障转移。
>
> 分析事件通常与性能事件相关，发布频率很高。例如，跟踪存储过程或查询执行就属于分析事件。
>
> 调试 - 调试事件尚未完全记录，仅限在与 Microsoft 技术支持进行故障排除时使用。

事件被添加到会话中，单个会话可承载多个事件。通常会将多个事件分组集合在同一会话中，以捕获相关联的信息集。

您可以通过运行以下查询获取可用事件、操作和目标列表：

```SQL
SELECT
    obj.object_type,
    pkg.name AS [package_name],
    obj.name AS [object_name],
    obj.description AS [description]
FROM sys.dm_xe_objects  AS obj
    INNER JOIN sys.dm_xe_packages AS pkg  ON pkg.guid = obj.package_guid
WHERE obj.object_type in ('action',  'event',  'target')
ORDER BY obj.object_type,
    pkg.name,
    obj.name;
```

| **Target 目标**                                             | **Description 说明**                                         | **Processing 处理中** |
| :---------------------------------------------------------- | :----------------------------------------------------------- | :-------------------- |
| **Event Counter 事件计数器**                                | 统计扩展事件会话期间发生的所有事件。用于在无需完整事件收集开销的情况下，获取工作负载的相关工作负载特征信息。 | Synchronous 同步的    |
| **Event File 事件文件**                                     | 将事件会话输出从内存写入磁盘上的持久文件。                   | Asynchronous 异步     |
| **Event Pairing 事件配对**                                  | 许多事件通常成对发生（例如锁定获取、锁定释放），此集合可用于识别这些事件何时未在匹配集合中出现。 | Asynchronous 异步     |
| **Event Tracing for Windows (ETW) Windows 事件跟踪（ETW）** | 用于关联 SQL Server 事件与 Windows 操作系统事件数据。        | Synchronous 同步的    |
| **Histogram 直方图**                                        | 这与事件计数器类似，用于统计事件发生的次数。不同之处在于，直方图可以基于特定的事件列或操作进行统计。 | Asynchronous 异步     |
| **Ring Buffer 环形缓冲区**                                  | 用于在内存中保存数据。数据不会持久保存到磁盘，可能会频繁地从缓冲区中刷新 | Asynchronous 异步     |

或者，您可以使用 T-SQL 创建扩展事件会话。以下 T-SQL 命令提供了如何创建扩展事件会话的示例：

```sql
IF EXISTS (SELECT * FROM sys.server_event_sessions WHERE name='test_session')
    DROP EVENT session test_session ON SERVER;
GO

CREATE EVENT SESSION test_session
ON SERVER
    ADD EVENT sqlos.async_io_requested,
    ADD EVENT sqlserver.lock_acquired
    ADD TARGET package0.etw_classic_sync_target (SET default_etw_session_logfile_path = N'C:\demo\traces\sqletw.etl' )
    WITH (MAX_MEMORY=4MB, MAX_EVENT_SIZE=4MB);
GO
```

事件会话可以限定于服务器或数据库范围。本示例中将添加两个事件，并使用带文件位置的 Windows 事件追踪（ETW）路径。创建会话后需要手动启动，可通过 T-SQL 执行并配合 `STATE` 选项来 `ALTER` 会话，也可以使用 SQL Server Management Studio 进行操作。

[Isolate performance problems through monitoring](https://microsoftlearning.github.io/dp-300-database-administrator/Instructions/Labs/06-isolate-performance-problems.html)



## [Configure SQL Server resources for optimal performance](https://learn.microsoft.com/en-us/training/modules/configure-sql-server-resources-optimal-performance/)

### Explain how to optimize Azure storage for SQL Server virtual machines

>Azure 存储是一个稳健且安全的平台，旨在满足各种应用程序的不同需求。它提供多种可扩展的解决方案，确保所有类型的存储都支持静态加密。用户可以选择 Microsoft 托管的加密密钥或用户定义的加密密钥以增强安全性。
>
>Blob 存储 - Blob 存储即所谓的基于对象的存储，包括冷存储、热存储和存档存储层级。在 SQL Server 环境中，Blob 存储通常用于数据库备份，利用 SQL Server 的“备份到 URL”功能。
>
>文件存储 - 文件存储实际上是一个文件共享，可以在虚拟机内部挂载，无需设置任何硬件。SQL Server 可以将文件存储用作故障转移群集实例的存储目标。
>
>磁盘存储 - Azure 托管磁盘提供块存储，并呈现给虚拟机。这些磁盘就像本地服务器中的物理磁盘一样进行管理，不同之处在于它们是虚拟化的。根据工作负载的不同，托管磁盘有多种性能层级。这种类型的存储是 SQL Server 数据和事务日志文件最常用的类型。

### Optimize database storage

#### 配置 SQL Server 中的 Tempdb

> SQL Server 中的 tempdb 不仅用于存储用户定义的临时表，还承担着多种任务，例如存放中间查询结果的工作表、执行排序操作，以及作为行版本控制的版本存储等。鉴于其广泛用途，务必将 tempdb 部署在延迟最低的可用存储上，并妥善配置其数据文件。
>
> `tempdb` 的数据库文件始终会自动存储在本地 SSD 驱动器上，因此 I/O 性能不会成为问题。
>
> SQL Server 专业人员通常使用多个数据库文件来对 `tempdb` 表进行分配分区。对于 Azure SQL 数据库，文件数量会根据 vCore 数量按比例调整（例如，两个 vCore 对应四个文件），上限为 16 个。无法通过针对 `tempdb` 执行 T-SQL 来配置文件数量，但可通过更改部署选项进行调整。 `tempdb` 的最大容量随 vCore 数量按比例扩展。而在 SQL 托管实例中，无论 vCore 数量多少，均可获得 12 个文件。
>
> 数据库选项 `MIXED_PAGE_ALLOCATION` 设置为关闭状态，而 `AUTOGROW_ALL_FILES` 设置为开启状态。虽然无法手动配置这些选项，但与 SQL Server 一样，这些设置为推荐的默认配置。

<u>**什么是tempdb?**</u>

简单来说，**tempdb** 是 SQL Server 的**“草稿纸”**或**“临时工作区”**。

它是系统预设的一个全局共享数据库。每当 SQL Server 需要处理一些“暂时拿不住”的数据时，就会把它丢到 tempdb 里。因为它是所有用户、所有数据库共享的，所以它的性能直接决定了整个服务器的快慢。

------

**1. tempdb 里面到底存了什么？**

既然是“草稿纸”，它主要处理三类东西：

- **临时对象**：你显式创建的临时表（如 `#TempTable`）、全局临时表（`##GlobalTable`）或存储过程中的表变量。
- **内部对象**：数据库引擎在背后偷偷用的。比如你执行一个巨大的 `ORDER BY`（排序）或 `GROUP BY`（分组）时，如果内存放不下，它就会在 tempdb 里建临时工作表。
- **行版本控制**：如果你开启了快照隔离级别（Snapshot Isolation），旧版本的数据行会暂存在这里，供其他查询读取。

------

**2. 为什么你提到的那段话这么强调文件数量？**

这涉及到一个著名的性能瓶颈：**PFS/GAM 页争用（分配冲突）**。

想象一下：`tempdb` 就像一个巨大的文件柜。如果这个柜子只有一个“抽屉”（即只有一个物理文件数据流），当 16 个 CPU 同时想往里塞临时数据时，它们必须在抽屉门口排队领号（锁争用）。

- **多文件方案**：如果你把 tempdb 拆成 8 个文件（8 个抽屉），不同的 CPU 就可以同时操作不同的文件，大大减少了排队时间。
- **8 个文件的由来**：经验表明，当文件数达到 8 个时，绝大多数情况下的争用都会消失。如果 CPU 更多且负载极重，才考虑增加更多。

------

**3. 那段话里的“跟踪标志”是什么鬼？**

以前的 DBA 必须手动开启这两个开关，现在已经内置了：

- **1117（等比增长）**：如果你有 8 个 tempdb 文件，当空间不够时，8 个文件同时长胖。这能保证数据在 8 个文件里分布均匀。
- **1118（统一区分配）**：不再一次只给一张表分 1 个小格子，而是一次给 8 个，减少分配时的管理开销。

------

**4. tempdb 的一个“神奇”特性**

**它每次重启都会“格式化”！** 每次 SQL Server 重启，tempdb 都会被删掉重建。这意味着：

1. 你存在里面的数据重启后就没了（所以叫临时库）。
2. 它不会像业务库那样因为日志没收缩而无限撑爆磁盘（前提是你设置了合理的初始大小）。

------

**总结**

`tempdb` 是全公司的**公共办公桌**。如果桌子太小（文件太少），大家就会挤在一起打架；如果你按照 CPU 数量多准备几张桌子（配置多个文件），大家干活就都快了。

> SQL Server 2019 中引入的元数据优化功能（标记为 `tempdb` ）能够缓解严重的闩锁争用，但目前该功能尚未在 Azure SQL 数据库或 Azure SQL 托管实例中提供。

**什么叫“闩锁争用”？**

简单来说，**闩锁争用（Latch Contention）** 就是数据库内部的一种**“排队交通堵塞”**。

如果把“锁（Lock）”比作你要锁住一扇门（一张表或一行数据），那么“闩锁（Latch）”就是为了防止别人在你翻书的时候把书页撕了。

------

**1. 什么是闩锁（Latch）？**

在数据库引擎内部，数据是存在“内存页（Page）”里的。

- **锁（Lock）**：保护的是**逻辑数据**（比如：不准改这行工资）。它通常持续整个事务时间，时间较长。
- **闩锁（Latch）**：保护的是**物理结构**（比如：不准在我读取这个内存地址时修改它）。它通常只持续几微秒，极短。

**闩锁争用**：当成千上万个线程（CPU 核心）试图在同一微秒内读写同一个内存结构时，它们就会互相等待。虽然每一次等待只有几微秒，但积累起来就会让 CPU 处于“忙等”状态，系统吞吐量直线下降。

------

**2. 为什么 tempdb 特别容易发生闩锁争用？**

`tempdb` 是全服务器最忙的数据库。每当有人创建临时表时，SQL Server 必须修改一些**管理页（Metadata）**，比如：

- **PFS (Page Free Space)**：记录哪一页还有空位。
- **GAM (Global Allocation Map)**：记录哪些区已经分配出去了。

想象全公司的人都在同一秒冲向同一个柜台（管理页）去领表格（创建临时表），柜台窗口太少，大家就撞在一起了。这就是 SQL Server 2019 之前最头疼的“分配瓶颈”。

------

**3. 2019 引入的“元数据优化”解决了什么？**

在 SQL Server 2019 之前，这些关于临时表的元数据（表名、列信息等）是作为**普通表数据**存在磁盘和内存里的。修改它们必须经过传统的闩锁机制。

**元数据优化（Memory-Optimized TempDB Metadata）** 做了两个大动作：

1. **搬家**：把存储临时表信息的系统表（如 `sys.objvalues`）从普通的 B 树结构挪到了**内存优化表（In-Memory OLTP）**中。
2. **去锁**：内存优化表使用的是“无锁（Lock-free）”结构。

**结果**：以前大家抢一个柜台领号，现在变成了一个“全自动感应柜台”，大家各拿各的，不再需要互相等待（闩锁）。

------

**4. 你如何知道自己遇到了闩锁争用？**

如果你在性能监视器或系统视图（DMV）中看到大量的 **Wait Type（等待类型）** 为以下内容，那就是在闹“闩锁争用”了：

- **`PAGELATCH_UP`**（更新闩锁等待）
- **`PAGELATCH_EX`**（排他闩锁等待）
- 且等待的对象通常在 `tempdb`（数据库 ID 为 2）上。

#### MAXDOP 设置

> 最大并行度（MAXDOP）设置会影响单个查询的性能。SQL Server 与 Azure SQL 在此机制的处理上保持一致。当 MAXDOP 设置较高时，每个查询会使用更多并行线程，从而可能提升查询执行速度。然而，这种并行度的增加会消耗额外的内存资源，可能引发内存压力，进而影响存储性能。例如，在将行组压缩为列存储格式时，并行操作需要更多内存，可能导致内存压力及行组修剪现象。
>
> 相反，将 MAXDOP 设置为较低值有助于缓解内存压力，从而提升存储系统的运行效率。这对于内存资源有限或存储需求较高的环境尤为重要。通过合理配置 MAXDOP，可以在查询性能与存储效率之间取得平衡，确保 CPU 与存储资源均得到优化利用。

在 Azure SQL 中，您可通过以下方式配置 MAXDOP，其操作方式与 SQL Server 类似：

- 对于 Azure SQL，支持配置 `ALTER DATABASE SCOPED CONFIGURATION` 到 `MAXDOP` 。
- SQL 托管实例支持用于配置“最大并行度”的存储过程 `sp_configure` 。
- 查询提示功能得到完整支持。
- SQL 托管实例支持配置 `MAXDOP` 与资源调控器。

### **Control SQL Server resources**

> 尽管部分 SQL Server 或 Azure SQL 托管实例专用于单个应用程序的数据库（这在关键任务应用中很常见），但多数服务器需承载多个应用程序的数据库，这些数据库的性能要求和峰值工作负载周期各不相同。管理员在满足这些不同需求之间找到平衡点可能颇具挑战。使用 SQL Server 2008 引入的资源调控器（Resource Governor）是一个有效的管理手段。
>
> 资源调控器是 SQL Server 与 Azure SQL 托管实例中的一项功能，可对应用程序请求所占用的 CPU、物理 I/O 及内存资源进行精细化管控。该功能在实例级别启用后，会通过分类器函数界定连接的处理规则，将会话划分为不同的工作负载组，每个工作负载组均分配有专属的系统资源池供其使用。

在 SQL Server 中，**资源调控器 (Resource Governor)** 并不是一个简单的开关按钮，它是一套**管理体系**。

如果你使用的是 **SQL Server Management Studio (SSMS)**，你可以按照以下路径找到并操作它：

**1.资源调控器在 SSMS 中的位置**

1. 打开 **SSMS** 并连接到你的数据库实例。
2. 在左侧的 **“对象资源管理器 (Object Explorer)”** 中，展开最顶层的服务器节点。
3. 找到 **“管理 (Management)”** 文件夹并展开。
4. 你就能看到 **“资源调控器 (Resource Governor)”** 了。

------

**2. 它是如何“选择”和工作的？（核心三要素）**

资源调控器不是直接给某个数据库打勾，而是通过以下三个步骤来“分配”资源：

**第一步：资源池 (Resource Pools)**

这是“大饼”。你把服务器的总资源（CPU、内存）切成几块。比如：

- **Pool_VIP**: 给核心业务，保证 70% 的 CPU。
- **Pool_Report**: 给跑报表的，最多只给 20% 的 CPU，防止它把服务器跑挂。

**第二步：工作负载组 (Workload Groups)**

这是“标签”。你可以把进来的连接分类。比如来自“财务软件”的归一类，来自“外部查询”的归一类。

**第三步：分类器函数 (Classifier Function) —— 真正的“选择器”**

这是资源调控器的**灵魂**。它是一段你写的 **T-SQL 函数**，用来识别身份。

- **逻辑**：当一个用户连接进来时，函数会判断：“你是谁？你从哪台电脑连过来的？”
- **结果**：如果是老板连进来，函数就把他扔进 `Pool_VIP`；如果是实习生连进来，就扔进 `Pool_Report`。

------

**3. 如何启用它？**

即使你配置好了上述内容，它默认也是关闭的。

- **图形界面**：右键点击“资源调控器”，选择“启用 (Enable)”。

- **T-SQL 命令**：

  ```SQL
  ALTER RESOURCE GOVERNOR RECONFIGURE;
  ```

------

**4. 重要的限制（避坑指南）**

1. **版本限制**：在本地版本中，资源调控器通常只在 **Enterprise (企业版)** 中可用。如果你用的是 Standard (标准版)，虽然能看到菜单，但可能无法启用。
2. **Azure 情况**：在 **Azure SQL 托管实例 (Managed Instance)** 中是支持的，但在普通的 **Azure SQL Database (单库)** 中，资源由微软自动管理，你无法手动配置资源调控器。

------

**总结**

资源调控器就像是数据库里的**“交通警察”**。它在“管理”菜单下，通过你编写的“分类逻辑”，确保重要的业务永远不会被次要的、耗资源的查询挤到没路走。

#### 资源池

> 资源池代表服务器上可用的物理资源。SQL Server 始终具有两个池：默认池和内部池，即使资源调控器未启用也是如此。内部池专用于关键的 SQL Server 功能，并且无法限制。默认池以及您显式定义的任何资源池，都可以配置其可使用的资源限制。对于每个非内部池，您可以指定以下限制：
>
> 百分比最小值/最大值
>
> CPU 百分比上限
>
> 内存百分比最小值/最大值
>
> 节点关联性
>
> 每个卷的最小/最大 IOPS

除最小和最大 CPU 百分比外，所有资源池设置都代表不能超过的硬性限制。最小/最大 CPU 百分比仅在有 CPU 争用时适用。例如，如果设置最大为 70%，则在没有争用的情况下，工作负载可能最多使用 100%的可用 CPU 周期。但是，如果其他工作负载正在运行，该工作负载将被限制在 70%。

#### 工作负载组

> 工作负载组是会话请求的容器，由分类器函数进行分类。类似于资源池，有两个内置组：默认组和内部组。每个工作负载组与单个资源池关联，但一个资源池可以承载多个工作负载组。默认情况下，所有连接都被定向到默认工作负载组，除非分类器函数将它们分配给用户定义的组。默认工作负载组使用分配给默认资源池的资源。

#### 分类器函数

> 分类器函数在建立到 SQL Server 实例的连接时运行，并将每个连接分类到给定的工作负载组中。如果函数返回 NULL、默认值或不存在的工作负载组名称，则会话将被转移到默认工作负载组。由于分类器在每个连接上运行，因此应测试其效率。下图显示了一个示例分类器函数，该函数根据用户名对用户进行分类

```sql
CREATE FUNCTION dbo.RGClassifier()
RETURNS SYSNAME
WITH SCHEMABINDING
AS
BEGIN
DECLARE @WorkloadGroup AS SYSNAME
IF(SUSER_NAME() = 'ReportUser')
    SET @WorkloadGroup = 'ReportServerGroup'

ELSE IF (SUSER_NAME() = 'PrimaryUser')
    SET @WorkloadGroup = 'PrimaryServerGroup'
ELSE
    SET @WorkloadGroup = 'default'

RETURN @WorkloadGroup
END

-- 可以增加示例中函数定义的复杂性，但应确保更复杂的函数不会影响用户性能。
```

> 资源调控器主要用于多租户方案，在该方案中，一组数据库共享一个 SQL Server 实例，并且需要为服务器的所有用户保持一致的性能。您还可以使用资源调控器来限制一致性检查和索引重建等维护操作所用的资源，以尽量在维护时段内保证用户查询有充足的资源。

[Detect and correct fragmentation issues](https://microsoftlearning.github.io/dp-300-database-administrator/Instructions/Labs/07-detect-correct-fragmentation-issues.html)

---

**_参考：_**

[Transact-SQL 简介](https://learn.microsoft.com/zh-cn/training/modules/introduction-to-transact-sql/)

[对 T-SQL 中的结果进行排序和筛选](https://learn.microsoft.com/zh-cn/training/modules/sort-filter-queries/)

[采用 T-SQL 通过 JOIN 合并多个表](https://learn.microsoft.com/zh-cn/training/modules/query-multiple-tables-with-joins/)

[在 T-SQL 中写入子查询](https://learn.microsoft.com/zh-cn/training/modules/write-subqueries/)
