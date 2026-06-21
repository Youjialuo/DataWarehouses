# TD_MDX 简化版 —— Mondrian 分层语法

> FoodMart Sales Cube | `[Measures].[Unit Sales]`

---

## 习题 1：Month / Quarter / Year 三层交叉

### 思路
同维度放两轴会有 auto-exists 冲突 → 用 `Ancestor` 在列轴上下文里上溯取值。

### 代码

```mdx
WITH
MEMBER [Measures].[Mois] AS
    ([Measures].[Unit Sales], [Time].CurrentMember)

MEMBER [Measures].[Trimestre] AS
    ([Measures].[Unit Sales], Ancestor([Time].CurrentMember, [Time].[Quarter]))

MEMBER [Measures].[Annee] AS
    ([Measures].[Unit Sales], Ancestor([Time].CurrentMember, [Time].[Year]))

SELECT
    Descendants([Time].[1997], [Time].[Month]) ON COLUMNS,
    { [Measures].[Mois], [Measures].[Trimestre], [Measures].[Annee] } ON ROWS
FROM [Sales]
```

### 要点
- `Ancestor(m, level)` 从当前月跳到季度/年度
- `(Measure, Member)` 元组切换上下文重算值

---

## 习题 2：CA / WA × Q1~Q4 + 小计

### 思路
直接枚举 CA、WA，`SUM` 做小计。

### 代码

```mdx
WITH
MEMBER [Store].[T_State] AS SUM({[Store].[CA], [Store].[WA]})
MEMBER [Time].[T_Quarter] AS SUM(Descendants([Time].[1997], [Time].[Quarter]))

SELECT
    { Descendants([Time].[1997], [Time].[Quarter]), [Time].[T_Quarter] } ON COLUMNS,
    { [Store].[CA], [Store].[WA], [Store].[T_State] } ON ROWS
FROM [Sales]
WHERE ([Measures].[Unit Sales])
```

### 要点
- 不用 `Filter`，直接 `{[Store].[CA], [Store].[WA]}`
- `SUM` 比 `Aggregate` 更短，普通度量完全够用
- 小计挂到维度下，才能在交叉时自动参与运算

---

## 习题 3：Drink/Food → 部门 → 小计 × Q1/Q2 → 月份 → 小计

### 思路
手动枚举层次集合，小计用 `SUM(.Children)`。

### 代码

```mdx
WITH
MEMBER [Product].[Drink].[Sub] AS SUM([Product].[Drink].Children)
MEMBER [Product].[Food].[Sub] AS SUM([Product].[Food].Children)
MEMBER [Product].[Total_Prod] AS SUM({[Product].[Drink], [Product].[Food]})

MEMBER [Time].[1997].[Q1].[Sub] AS SUM([Time].[1997].[Q1].Children)
MEMBER [Time].[1997].[Q2].[Sub] AS SUM([Time].[1997].[Q2].Children)
MEMBER [Time].[Total_Temps] AS SUM({[Time].[1997].[Q1], [Time].[1997].[Q2]})

SELECT
    {
        [Time].[1997].[Q1], [Time].[1997].[Q1].Children, [Time].[1997].[Q1].[Sub],
        [Time].[1997].[Q2], [Time].[1997].[Q2].Children, [Time].[1997].[Q2].[Sub],
        [Time].[Total_Temps]
    } ON COLUMNS,
    {
        [Product].[Drink], [Product].[Drink].Children, [Product].[Drink].[Sub],
        [Product].[Food], [Product].[Food].Children, [Product].[Food].[Sub],
        [Product].[Total_Prod]
    } ON ROWS
FROM [Sales]
WHERE ([Measures].[Unit Sales])
```

### 要点
- `.Children` = 直接子成员（Alcoholic, Beverages... / Jan, Feb, Mar...）
- `SUM(.Children)` = 父成员下所有子成员之和
- 命名用 `Sub` 替代 `Sub_T1` 更短

---

## 习题 4：月度移动平均（前2月窗口）

### 思路
`AVG()` 替代手动 (a+b+c)/3，逻辑更干净。

### 代码

```mdx
WITH
MEMBER [Measures].[Moyen Mobile] AS
    IIF(
        [Time].CurrentMember.PrevMember IS NULL,
        -- Jan：只有自己
        [Measures].[Unit Sales],
        IIF(
            [Time].CurrentMember.PrevMember.PrevMember IS NULL,
            -- Feb：当月 + 上月
            AVG({[Time].CurrentMember.PrevMember, [Time].CurrentMember}),
            -- 其他：3月滑动窗口
            AVG({
                [Time].CurrentMember.PrevMember.PrevMember,
                [Time].CurrentMember.PrevMember,
                [Time].CurrentMember
            })
        )
    )

SELECT
    Descendants([Time].[1997], [Time].[Month]) ON COLUMNS,
    { [Measures].[Unit Sales], [Measures].[Moyen Mobile] } ON ROWS
FROM [Sales]
```

### 要点
- `AVG({...})` 自动求集合的平均值，省去 `/3`、`/2`
- 三层 `IIF` 从特殊到通用：Jan → Feb → 其余
- 去掉 Level.Name 防御（只要列轴是 Month，CurrentMember 就是 Month）

---

## 习题 5：中心化移动平均

### 思路
同理用 `AVG()`，对称窗口加 `.NextMember`。

### 代码

```mdx
WITH
MEMBER [Measures].[Moyen Mobile Centre] AS
    IIF(
        [Time].CurrentMember.PrevMember IS NULL,
        -- Jan：当月 + 下月
        AVG({[Time].CurrentMember, [Time].CurrentMember.NextMember}),
        IIF(
            [Time].CurrentMember.NextMember IS NULL,
            -- Dec：上月 + 当月
            AVG({[Time].CurrentMember.PrevMember, [Time].CurrentMember}),
            -- 中间：上月 + 当月 + 下月
            AVG({
                [Time].CurrentMember.PrevMember,
                [Time].CurrentMember,
                [Time].CurrentMember.NextMember
            })
        )
    )

SELECT
    Descendants([Time].[1997], [Time].[Month]) ON COLUMNS,
    { [Measures].[Unit Sales], [Measures].[Moyen Mobile Centre] } ON ROWS
FROM [Sales]
```

### 要点
- 先判首（无 Prev）→ 再判尾（无 Next）→ 通用3点窗口
- `AVG` 让三行核心代码极致简洁

---

## 习题 6：完整移动平均（月 + 季度双层）

### 思路
`Level.Name` 判断层级，月走习题4，季走2季滑动。

### 代码

```mdx
WITH
MEMBER [Measures].[Moyen Mobile Complet] AS
    IIF(
        [Time].CurrentMember.Level.Name = "Month",
        -- 月度：同习题4
        IIF(
            [Time].CurrentMember.PrevMember IS NULL,
            [Measures].[Unit Sales],
            IIF(
                [Time].CurrentMember.PrevMember.PrevMember IS NULL,
                AVG({[Time].CurrentMember.PrevMember, [Time].CurrentMember}),
                AVG({[Time].CurrentMember.PrevMember.PrevMember,
                     [Time].CurrentMember.PrevMember,
                     [Time].CurrentMember})
            )
        ),
        IIF(
            [Time].CurrentMember.Level.Name = "Quarter",
            -- 季度：(当前季 + 上季) / 2
            IIF(
                [Time].CurrentMember.PrevMember IS NULL,
                [Measures].[Unit Sales],
                AVG({[Time].CurrentMember.PrevMember, [Time].CurrentMember})
            ),
            [Measures].[Unit Sales]
        )
    )

SELECT
    Hierarchize({
        Descendants([Time].[1997], [Time].[Month]),
        Descendants([Time].[1997], [Time].[Quarter])
    }) ON COLUMNS,
    { [Measures].[Unit Sales], [Measures].[Moyen Mobile Complet] } ON ROWS
FROM [Sales]
```

### 要点
- `Level.Name` 做路由：`"Month"` → 月逻辑 / `"Quarter"` → 季逻辑
- 季度窗口仅2点（一年才4个季度，3点太宽）
- `Hierarchize` 让输出 Month→Quarter 自然分层

---

## 六题速查

| 题 | 一句话 | 核心函数 |
|----|--------|---------|
| 1 | 月/季/年三层通过 Ancestor 上溯 | `Ancestor`, `Descendants` |
| 2 | 直接枚举州 + SUM小计 | `SUM`, 直接成员引用 |
| 3 | 手动枚举层次 + .Children小计 | `.Children`, `SUM` |
| 4 | 前向3月AVG | `AVG`, `PrevMember`, `IIF` |
| 5 | 对称3月AVG（前+中+后） | `AVG`, `PrevMember`, `NextMember` |
| 6 | Level.Name路由月/季双逻辑 | `Level.Name`, 嵌套`IIF` |
