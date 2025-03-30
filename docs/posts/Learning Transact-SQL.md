---
date:
  created: 2025-03-26
tags:
  - T-SQL
authors: [Tenax]
description: >
  Learning Learning Transact-SQL
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

```sql title = "实际应用"
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

---

**_参考：_**

[Transact-SQL 简介](https://learn.microsoft.com/zh-cn/training/modules/introduction-to-transact-sql/)

[对 T-SQL 中的结果进行排序和筛选](https://learn.microsoft.com/zh-cn/training/modules/sort-filter-queries/)

[采用 T-SQL 通过 JOIN 合并多个表](https://learn.microsoft.com/zh-cn/training/modules/query-multiple-tables-with-joins/)

[在 T-SQL 中写入子查询](https://learn.microsoft.com/zh-cn/training/modules/write-subqueries/)
