# MDX 从零到精通 —— 数据分析师版

> **适用对象**：零基础、想用 MDX 做多维数据分析的初学者  
> **前置知识**：建议先了解 SQL 基础，因为会频繁对比  
> **环境建议**：Pentaho Mondrian + Saiku / SSAS / Power BI（任选其一）

---

## 目录

1. [第一章：什么是 OLAP 和 MDX？](#第一章什么是-olap-和-mdx)
2. [第二章：MDX 核心概念](#第二章mdx-核心概念)
3. [第三章：你的第一句 MDX](#第三章你的第一句-mdx)
4. [第四章：切片与切块 WHERE](#第四章切片与切块-where)
5. [第五章：成员、元组、集合](#第五章成员元组集合)
6. [第六章：层次导航](#第六章层次导航)
7. [第七章：常用 MDX 函数](#第七章常用-mdx-函数)
8. [第八章：计算成员](#第八章计算成员)
9. [第九章：命名集合](#第九章命名集合)
10. [第十章：时间智能函数](#第十章时间智能函数)
11. [第十一章：MDX vs SQL 对照速查](#第十一章mdx-vs-sql-对照速查)
12. [第十二章：实战分析案例](#第十二章实战分析案例)
13. [附：常见错误与调试](#附常见错误与调试)

---

## 第一章：什么是 OLAP 和 MDX？

### 1.1 OLAP —— 在线分析处理

SQL 处理的是**二维表格**（行 + 列），而 OLAP 处理的是**多维立方体（Cube）**。

`
想象一个立体的 Excel：

            ┌──────────────┐
           /    2024-Q1    /│
          /    销售额:50万  / │
         /──────────────/   │
        │   电子产品     │   │
        │   北京         │   │
        └──────────────┘   /
         └─────────────────┘

三个维度：
- 时间维度：年 → 季 → 月 → 日
- 产品维度：类别 → 子类别 → 单品
- 地区维度：国家 → 省 → 市
`

| 概念 | OLAP（多维） | SQL（关系型） |
|------|-------------|--------------|
| 数据存储 | Cube（立方体） | Table（表） |
| 查询语言 | MDX | SQL |
| 分析方式 | 切片、切块、钻取 | 筛选、分组、聚合 |
| 适用场景 | 汇总分析、报表 | 事务处理、明细查询 |

### 1.2 MDX 是什么？

**MDX** = Multidimensional Expressions（多维表达式）

当 SQL 无法优雅地处理"按地区、按时间、按产品同时分析"时，MDX 就是答案。

`sql
-- SQL：要写一长串
SELECT city, year, product, SUM(amount)
FROM sales
GROUP BY city, year, product;

-- MDX：天然支持多维
SELECT
    [Measures].[Sales] ON COLUMNS,
    [Time].[2024].Children ON ROWS
FROM [SalesCube]
WHERE [Product].[Electronics];
`

### 1.3 MDX 用在哪儿？

- **SSAS**（Microsoft SQL Server Analysis Services）
- **Pentaho Mondrian**（开源 OLAP 引擎）
- **Power BI**（背后就是 SSAS 的 MDX）
- **IBM Cognos**
- **SAP BW**

---

## 第二章：MDX 核心概念

### 2.1 五大核心对象

| 对象 | 英文 | 说明 | 类比 SQL |
|------|------|------|----------|
| 度量 | Measure | 要计算的数值，如销售额、数量 | SELECT 中的聚合列 |
| 维度 | Dimension | 分析的角度，如时间、产品、地区 | GROUP BY 的列 |
| 层次 | Hierarchy | 维度内的层级结构，如年→月→日 | 无直接对应 |
| 级别 | Level | 层次中的一层，如"月"这一层 | 无直接对应 |
| 成员 | Member | 级别中的具体值，如"2024年1月" | 某一行数据的值 |

### 2.2 可视化理解

`
维度：Time（时间）
  └─ 层次：Calendar（日历）
       ├─ 级别：Year    → 2023, 2024, 2025
       ├─ 级别：Quarter → Q1, Q2, Q3, Q4
       └─ 级别：Month   → Jan, Feb, Mar, ...

维度：Product（产品）
  └─ 层次：Category
       ├─ 级别：大类    → 电子、服装、食品
       ├─ 级别：中类    → 手机、电脑、电视
       └─ 级别：单品    → iPhone 15, MacBook Pro

度量：Sales, Quantity, Profit
`

### 2.3 引用语法

`mdx
-- 完全限定名（推荐，不会歧义）
[Dimension].[Hierarchy].[Level].[Member]

-- 示例
[Time].[Calendar].[Year].[2024]
[Product].[Category].[All].[Electronics]
[Measures].[Sales Amount]

-- 简写（如果只有一层层次，可省略层次名）
[Time].[2024]
[Product].[Electronics]
`

---

## 第三章：你的第一句 MDX

### 3.1 基本查询结构

`mdx
SELECT
    什么 ON COLUMNS,     -- 列轴（通常是度量或时间）
    什么 ON ROWS         -- 行轴（通常是产品、地区等）
FROM
    [Cube名]
WHERE
    切片条件;             -- 固定某个维度
`

### 3.2 最简单的查询

`mdx
-- 查询所有产品的销售额
SELECT
    [Measures].[Sales] ON COLUMNS,
    [Product].[Category].Members ON ROWS
FROM [SalesCube];
`

结果类似：

`
                  │ Sales
──────────────────┼──────
Electronics       │ 500000
Clothing          │ 320000
Food              │ 180000
`

### 3.3 指定多维度到轴上

`mdx
-- 列上是时间，行上是产品
SELECT
    { [Time].[2024].Children } ON COLUMNS,
    { [Product].[Category].Members } ON ROWS
FROM [SalesCube];
`

结果：

`
                  │ Q1     Q2     Q3     Q4
──────────────────┼─────────────────────────
Electronics       │ 120   130   125   125
Clothing          │  80    85    75    80
Food              │  45    50    40    45
`

### 3.4 交叉连接 CROSSJOIN

两个维度放在同一轴上：

`mdx
-- 行轴同时显示产品和地区
SELECT
    [Measures].[Sales] ON COLUMNS,
    CROSSJOIN(
        [Product].[Category].Members,
        [Region].[City].Members
    ) ON ROWS
FROM [SalesCube];
`

等价简写：

`mdx
SELECT
    [Measures].[Sales] ON COLUMNS,
    [Product].[Category].Members * [Region].[City].Members ON ROWS
FROM [SalesCube];
`

---

## 第四章：切片与切块 WHERE

### 4.1 切片（固定一个维度）

`mdx
-- 只看 2024 年的数据
SELECT
    [Measures].[Sales] ON COLUMNS,
    [Product].[Category].Members ON ROWS
FROM [SalesCube]
WHERE ([Time].[2024]);
`

### 4.2 多条件切片

`mdx
-- 2024 年 + 北京地区
SELECT
    [Measures].[Sales] ON COLUMNS,
    [Product].[Category].Members ON ROWS
FROM [SalesCube]
WHERE ([Time].[2024], [Region].[Beijing]);
`

### 4.3 WHERE vs 轴过滤

`mdx
-- 方式1：WHERE 切片（推荐，性能更好）
SELECT ... FROM [Cube] WHERE [Time].[2024];

-- 方式2：放在轴上过滤（更灵活）
SELECT ... FROM [Cube]
WHERE FILTER([Time].Members, [Time].CurrentMember.Name = '2024');
`

> WHERE 在 MDX 中叫 **Slicer**，切掉不关心的维度，不同于 SQL 的 WHERE。

---

## 第五章：成员、元组、集合

### 5.1 成员 Member

一个维度上的一个具体值。

`mdx
[Time].[2024]                    -- 一个成员
[Product].[Electronics].[Phones]  -- 一个成员
`

### 5.2 元组 Tuple

**多个成员的笛卡尔积组合**，用 () 包裹。

`mdx
([Time].[2024], [Product].[Electronics])
-- 表示：2024年 电子产品 的交叉点
`

一个成员本身也是元组（单成员元组），但多个成员时必须括起来。

### 5.3 集合 Set

**零个或多个元组的有序集合**，用 {} 包裹。

`mdx
-- 三个成员的集合
{ [Time].[2023], [Time].[2024], [Time].[2025] }

-- 两个元组的集合
{
    ([Time].[2024], [Product].[Electronics]),
    ([Time].[2024], [Product].[Clothing])
}

-- 空集合
{}

-- 范围（用冒号）
{ [Time].[Jan] : [Time].[Dec] }
`

### 5.4 集合函数

`mdx
-- .Members：某级别的所有成员
[Product].[Category].Members

-- .Children：某成员的所有子成员
[Time].[2024].Children          -- Q1, Q2, Q3, Q4

-- UNION：合并两个集合
UNION({A, B}, {C, D})           -- {A, B, C, D}

-- EXCEPT：差集
EXCEPT({A, B, C}, {B})          -- {A, C}

-- INTERSECT：交集
INTERSECT({A, B}, {B, C})       -- {B}
`

---

## 第六章：层次导航

### 6.1 层次结构示例

`
[Time] 维度
 └─ [Calendar] 层次
      ├─ [All]           (所有时间)
      │   ├─ [2023]
      │   │   ├─ [Q1]
      │   │   │   ├─ [Jan]
      │   │   │   ├─ [Feb]
      │   │   │   └─ [Mar]
      │   │   └─ ...
      │   └─ [2024]
      └─ [Fiscal] 层次  (财年，另一个层次)
`

### 6.2 导航函数

`mdx
-- .Parent：父成员
[Time].[2024].[Q1].[Jan].Parent   → [Q1]

-- .Children：子成员集合
[Time].[2024].Children            → {Q1, Q2, Q3, Q4}

-- .FirstChild / .LastChild
[Time].[2024].FirstChild          → [Q1]

-- .FirstSibling / .LastSibling
[Time].[2024].[Q1].LastSibling    → [Q4]

-- .PrevMember / .NextMember：兄弟间移动
[Time].[2024].[Q2].PrevMember     → [Q1]
[Time].[2024].[Q2].NextMember     → [Q3]

-- .Ancestor(member, level)：祖先
ANCESTOR([Time].[2024].[Q1].[Jan], [Time].[Year])  → [2024]

-- .Cousin：同层级对应
COUSIN([Time].[2024].[Q1], [Time].[2025])           → [2025].[Q1]
`

### 6.3 钻取操作

`mdx
-- 钻下（Drill Down）：展开到子级别
DRILLDOWNLEVEL([Product].[Category].Members)

-- 钻上（Drill Up）：收缩到父级别
DRILLUPLEVEL([Product].[Phones].[iPhone 15])

-- 钻到指定级别
DRILLDOWNMEMBER(
    { [Time].[2024] },
    { [Time].[2024].[Q1] }
)
`

---

## 第七章：常用 MDX 函数

### 7.1 聚合函数

`mdx
-- SUM：求和
SUM({[Time].[2024].Children}, [Measures].[Sales])

-- AVG：平均值
AVG({[Time].[2024].Children}, [Measures].[Sales])

-- COUNT：计成员数
COUNT({[Product].[Category].Members})

-- MIN / MAX
MIN({[Time].[2024].Children}, [Measures].[Sales])
MAX({[Time].[2024].Children}, [Measures].[Sales])

-- AGGREGATE（自动选聚合方式，推荐）
AGGREGATE({[Time].[2024].Children}, [Measures].[Sales])
`

### 7.2 集合生成函数

`mdx
-- TOPCOUNT：前 N 个
TOPCOUNT(
    [Product].[Category].Members,
    3,
    [Measures].[Sales]
)

-- BOTTOMCOUNT：后 N 个
BOTTOMCOUNT(
    [Product].[Category].Members,
    3,
    [Measures].[Sales]
)

-- TOPSUM：累积到前 N 个
-- TOPSUM(set, N, measure)

-- ORDER：排序
ORDER(
    [Product].[Category].Members,
    [Measures].[Sales],
    DESC
)

-- HEAD / TAIL
HEAD({A, B, C, D}, 2)    → {A, B}
TAIL({A, B, C, D}, 2)    → {C, D}

-- SUBSET：子集
SUBSET(set, start_index, count)
-- 注意：索引从 0 开始
SUBSET({A, B, C, D}, 0, 2)    → {A, B}
`

### 7.3 筛选函数

`mdx
-- FILTER：条件过滤
FILTER(
    [Product].[Category].Members,
    [Measures].[Sales] > 100000
)

-- NONEMPTY：去掉空值行
NON EMPTY [Product].[Category].Members ON ROWS

-- NONEMPTYCROSSJOIN：非空交叉连接（性能更好）
NONEMPTYCROSSJOIN(
    [Product].[Category].Members,
    [Time].[2024].Children,
    [Measures].[Sales],
    1
)
`

### 7.4 条件逻辑 IIF

`mdx
-- IIF(条件, 真值, 假值)
WITH MEMBER [Measures].[Status] AS
    IIF(
        [Measures].[Sales] > 100000,
        "高",
        "低"
    )
SELECT
    { [Measures].[Sales], [Measures].[Status] } ON COLUMNS,
    [Product].[Category].Members ON ROWS
FROM [SalesCube];
`

---

## 第八章：计算成员

### 8.1 WITH MEMBER

在查询中临时定义一个计算指标。

`mdx
WITH MEMBER [Measures].[Profit Margin] AS
    ([Measures].[Sales] - [Measures].[Cost]) / [Measures].[Sales],
    FORMAT_STRING = "Percent"
SELECT
    { [Measures].[Sales], [Measures].[Cost], [Measures].[Profit Margin] } ON COLUMNS,
    [Product].[Category].Members ON ROWS
FROM [SalesCube];
`

### 8.2 同期对比

`mdx
WITH MEMBER [Measures].[YoY Growth] AS
    ([Measures].[Sales] - ([Measures].[Sales], [Time].CurrentMember.PrevMember))
    /
    ([Measures].[Sales], [Time].CurrentMember.PrevMember),
    FORMAT_STRING = "Percent"
SELECT
    { [Measures].[Sales], [Measures].[YoY Growth] } ON COLUMNS,
    { [Time].[2024].Children } ON ROWS
FROM [SalesCube];
`

### 8.3 占比计算

`mdx
WITH MEMBER [Measures].[Pct of Total] AS
    [Measures].[Sales] / ([Measures].[Sales], [Product].[All]),
    FORMAT_STRING = "Percent"
SELECT
    { [Measures].[Sales], [Measures].[Pct of Total] } ON COLUMNS,
    [Product].[Category].Members ON ROWS
FROM [SalesCube];
`

### 8.4 常用 FORMAT_STRING

`mdx
FORMAT_STRING = "#,##0"          -- 千分位
FORMAT_STRING = "#,##0.00"       -- 两位小数
FORMAT_STRING = "Percent"        -- 百分比
FORMAT_STRING = "Currency"       -- 货币
FORMAT_STRING = "Short Date"     -- 短日期
`

---

## 第九章：命名集合

### 9.1 WITH SET

把常用的集合起个名字复用。

`mdx
WITH
    SET [Top3Products] AS
        TOPCOUNT(
            [Product].[Category].Members,
            3,
            [Measures].[Sales]
        )
    SET [RecentQuarters] AS
        { [Time].[2024].[Q1] : [Time].[2024].[Q4] }
SELECT
    [RecentQuarters] ON COLUMNS,
    [Top3Products] ON ROWS
FROM [SalesCube];
`

### 9.2 SCOPE（SSAS 独有）

持久化定义，不在查询内，而是写在 Cube 脚本里：

`mdx
SCOPE ([Measures].[Sales]);
    FORMAT_STRING = "#,##0.00";
END SCOPE;
`

---

## 第十章：时间智能函数

### 10.1 同期比较

`mdx
-- 去年同期（ParallelPeriod）
WITH MEMBER [Measures].[Sales LY] AS
    (ParallelPeriod(
        [Time].[Calendar].[Year],   -- 回溯级别
        1,                           -- 回溯步数
        [Time].CurrentMember         -- 当前成员
    ), [Measures].[Sales])
SELECT
    { [Measures].[Sales], [Measures].[Sales LY] } ON COLUMNS,
    { [Time].[2024].Children } ON ROWS
FROM [SalesCube];
`

### 10.2 累计值 YTD / MTD

`mdx
-- YTD（年初至今）
WITH MEMBER [Measures].[Sales YTD] AS
    SUM(
        YTD([Time].CurrentMember),
        [Measures].[Sales]
    )

-- QTD（季初至今）
WITH MEMBER [Measures].[Sales QTD] AS
    SUM(
        QTD([Time].CurrentMember),
        [Measures].[Sales]
    )
`

### 10.3 移动平均

`mdx
-- 3 期移动平均
WITH MEMBER [Measures].[3M Avg] AS
    AVG(
        { [Time].CurrentMember.Lag(2) : [Time].CurrentMember },
        [Measures].[Sales]
    )
`

### 10.4 时间函数总结

| 函数 | 作用 |
|------|------|
| ParallelPeriod(level, n, member) | 同期对比（去年同期） |
| YTD(member) | 年初至今的期间集合 |
| QTD(member) | 季初至今的期间集合 |
| MTD(member) | 月初至今的期间集合 |
| .Lag(n) | 往前 n 个兄弟 |
| .Lead(n) | 往后 n 个兄弟 |
| PeriodsToDate(level, member) | 截至某级别的时间段 |
| LastPeriods(n, member) | 最近 n 个期间 |

---

## 第十一章：MDX vs SQL 对照速查

| 需求 | SQL | MDX |
|------|-----|-----|
| 查所有列 | SELECT * | SELECT {dimension}.Members ON ROWS |
| 选几列 | SELECT col1, col2 | { [M1], [M2] } ON COLUMNS |
| 过滤行 | WHERE col = value | FILTER(set, condition) |
| 切片 | WHERE year = 2024 | WHERE ([Time].[2024]) |
| 排序 | ORDER BY col DESC | ORDER(set, measure, DESC) |
| 取前 N | LIMIT 5 | HEAD(set, 5) |
| 分组 | GROUP BY col | 天然按维度分组 |
| 去重 | DISTINCT | 自动去重（维度成员唯一） |
| 连接 | JOIN | CROSSJOIN / * |
| 空值处理 | IS NULL | NON EMPTY |
| 条件逻辑 | CASE WHEN | IIF() |
| 计算列 | AS 别名 | WITH MEMBER ... AS |
| 子查询 | 子查询 | SUBSELECT |

### 经典转化示例

**SQL → MDX**：

`sql
-- SQL：各城市各产品的销售额
SELECT city, product, SUM(amount)
FROM sales
WHERE year = 2024
GROUP BY city, product
ORDER BY SUM(amount) DESC;
`

`mdx
-- MDX：各城市各产品的销售额
SELECT
    [Measures].[Sales] ON COLUMNS,
    NON EMPTY
    ORDER(
        [Region].[City].Members * [Product].[Category].Members,
        [Measures].[Sales],
        DESC
    ) ON ROWS
FROM [SalesCube]
WHERE ([Time].[2024]);
`

---

## 第十二章：实战分析案例

### 案例 1：销售仪表盘

`mdx
WITH
    MEMBER [Measures].[YoY%] AS
        ([Measures].[Sales] - ([Measures].[Sales], [Time].CurrentMember.PrevMember))
        / ([Measures].[Sales], [Time].CurrentMember.PrevMember),
        FORMAT_STRING = "0.00%"

    MEMBER [Measures].[Pct Total] AS
        [Measures].[Sales] / ([Measures].[Sales], [Product].[All]),
        FORMAT_STRING = "0.00%"
SELECT
    {
        [Measures].[Sales],
        [Measures].[YoY%],
        [Measures].[Pct Total]
    } ON COLUMNS,
    NON EMPTY
    [Product].[Category].Members ON ROWS
FROM [SalesCube]
WHERE ([Time].[2024]);
`

### 案例 2：Top 10 客户分析

`mdx
WITH
    SET [Top10Customers] AS
        TOPCOUNT(
            [Customer].[Name].Members,
            10,
            [Measures].[Sales]
        )

    MEMBER [Measures].[Avg Order] AS
        [Measures].[Sales] / [Measures].[Order Count],
        FORMAT_STRING = "#,##0.00"
SELECT
    {
        [Measures].[Sales],
        [Measures].[Order Count],
        [Measures].[Avg Order]
    } ON COLUMNS,
    [Top10Customers] ON ROWS
FROM [SalesCube]
WHERE ([Time].[2024]);
`

### 案例 3：趋势分析（移动平均）

`mdx
WITH
    MEMBER [Measures].[3M Moving Avg] AS
        AVG(
            LastPeriods(3, [Time].CurrentMember),
            [Measures].[Sales]
        ),
        FORMAT_STRING = "#,##0"

    MEMBER [Measures].[Trend] AS
        IIF(
            [Measures].[Sales] > [Measures].[3M Moving Avg],
            "↑ 上升",
            "↓ 下降"
        )
SELECT
    {
        [Measures].[Sales],
        [Measures].[3M Moving Avg],
        [Measures].[Trend]
    } ON COLUMNS,
    [Time].[2024].[Month].Members ON ROWS
FROM [SalesCube];
`

### 案例 4：地区差异分析

`mdx
WITH
    MEMBER [Measures].[Sales per Capita] AS
        [Measures].[Sales] / [Measures].[Population],
        FORMAT_STRING = "#,##0.00"

    MEMBER [Measures].[Region Rank] AS
        RANK(
            [Region].CurrentMember,
            ORDER(
                [Region].[City].Members,
                [Measures].[Sales],
                DESC
            )
        )
SELECT
    {
        [Measures].[Sales],
        [Measures].[Sales per Capita],
        [Measures].[Region Rank]
    } ON COLUMNS,
    NON EMPTY
    ORDER(
        [Region].[City].Members,
        [Measures].[Sales],
        DESC
    ) ON ROWS
FROM [SalesCube]
WHERE ([Time].[2024]);
`

### 案例 5：贡献度矩阵（父项占比）

`mdx
WITH
    MEMBER [Measures].[% of Parent] AS
        [Measures].[Sales] / ([Measures].[Sales], [Product].CurrentMember.Parent),
        FORMAT_STRING = "0.00%"

    MEMBER [Measures].[Contribution] AS
        IIF(
            [Measures].[% of Parent] > 0.3, "核心",
            IIF([Measures].[% of Parent] > 0.1, "重要", "一般")
        )
SELECT
    {
        [Measures].[Sales],
        [Measures].[% of Parent],
        [Measures].[Contribution]
    } ON COLUMNS,
    DESCENDANTS(
        [Product].[All],
        [Product].[SubCategory]
    ) ON ROWS
FROM [SalesCube]
WHERE ([Time].[2024]);
`

---

## 附：常见错误与调试

### 错误 1：混淆 {} 和 ()

| 用法 | 正确 | 错误 |
|------|------|------|
| 集合 | {A, B, C} | (A, B, C) |
| 元组 | (A, B) | {A, B} |

### 错误 2：漏写 .Members

`mdx
-- ❌ 错误
[Product].[Category] ON ROWS

-- ✅ 正确
[Product].[Category].Members ON ROWS
`

### 错误 3：WHERE 里放集合而不是元组

`mdx
-- ❌ 错误：集合不能放在 WHERE
WHERE { [Time].[2024], [Time].[2023] }

-- ✅ 正确：元组
WHERE ([Time].[2024])

-- ✅ 正确：当需要多个成员时，用 SUBSELECT
FROM (
    SELECT { [Time].[2024], [Time].[2023] } ON COLUMNS
    FROM [SalesCube]
)
`

### 错误 4：忽略空值导致性能问题

`mdx
-- ❌ 可能返回大量空行
[Product].[Name].Members ON ROWS

-- ✅ 加上 NON EMPTY
NON EMPTY [Product].[Name].Members ON ROWS
`

### 调试技巧

1. **从最简查询开始** —— 先只查一个度量，逐步加维度
2. **分步验证** —— 用 WITH SET 定义中间集合，单独检查
3. **查看层次结构** —— 搞清楚 .Members、.Children、.Levels 的区别
4. **Mondrian 限制** —— 注意 ORDER 必须在顶层、嵌套 WITH SET 等特殊规则

---

## 学习路线建议

`
第1周：理解 OLAP 概念，跑通第一个 SELECT 查询
第2周：掌握 Members / Tuple / Set，能写单维度查询
第3周：学会 CROSSJOIN 和层次导航函数
第4周：掌握计算成员 WITH MEMBER 和时间函数
第5周：实战 —— 用真实 Cube 做仪表盘查询
`

> **核心心法：MDX 本质上是"在立方体上画十字坐标"——**
> - **COLUMNS** 放 X 轴
> - **ROWS** 放 Y 轴
> - **WHERE** 锁定 Z 轴
> - **度量**决定每个交叉点显示什么数字
>
> 想象你在操作一个三维数据立方体，每句 MDX 都是在告诉它"把哪个面展示给我看"。
