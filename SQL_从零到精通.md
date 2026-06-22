# SQL 从零到精通 —— 数据分析师版

> **适用对象**：零基础、想用 SQL 做数据分析的初学者  
> **学习方式**：概念 → 语法 → 实战 → 练习，逐级深入  
> **环境建议**：MySQL / PostgreSQL / SQLite 任意一种即可

---

## 目录

1. [第一章：什么是 SQL？](#第一章什么是-sql)
2. [第二章：你的第一句 SQL](#第二章你的第一句-sql)
3. [第三章：筛选数据 WHERE](#第三章筛选数据-where)
4. [第四章：排序与取前 N 条](#第四章排序与取前-n-条)
5. [第五章：聚合函数](#第五章聚合函数)
6. [第六章：分组 GROUP BY](#第六章分组-group-by)
7. [第七章：连接 JOIN](#第七章连接-join)
8. [第八章：子查询](#第八章子查询)
9. [第九章：窗口函数](#第九章窗口函数)
10. [第十章：CTE 公用表表达式](#第十章cte-公用表表达式)
11. [第十一章：增删改操作](#第十一章增删改操作)
12. [第十二章：实战分析案例](#第十二章实战分析案例)
13. [附：常用速查表](#附常用速查表)

---

## 第一章：什么是 SQL？

### 1.1 数据库是什么？

想象一个超大 Excel 表格——每个 Sheet 就是一张 **表（Table）**：

`
┌──────────────────────────────────────────────┐
│  users 表                                     │
├──────┬──────────┬──────┬─────────────────────┤
│ id   │ name     │ age  │ email               │
├──────┼──────────┼──────┼─────────────────────┤
│ 1    │ 张三     │ 28   │ zhangsan@mail.com   │
│ 2    │ 李四     │ 35   │ lisi@mail.com       │
│ 3    │ 王五     │ 22   │ wangwu@mail.com     │
└──────┴──────────┴──────┴─────────────────────┘
`

- **行（Row/Record）** = 一条数据，比如"张三"这个人
- **列（Column/Field）** = 一个属性，比如"age"列
- **表（Table）** = 同一类数据的集合

### 1.2 SQL 是什么？

**SQL** = Structured Query Language（结构化查询语言）

说白了就是"**和数据库对话的语言**"。你想查什么数据，就用 SQL 写出来。

`sql
-- 这就是一句 SQL：查出所有年龄大于 25 的用户
SELECT name, age FROM users WHERE age > 25;
`

### 1.3 SQL 能干什么？

| 操作 | 关键词 | 说明 |
|------|--------|------|
| 查询数据 | SELECT | 从表中取数据 |
| 插入数据 | INSERT | 往表里加新行 |
| 更新数据 | UPDATE | 修改已有数据 |
| 删除数据 | DELETE | 删掉数据 |
| 创建表 | CREATE | 新建表结构 |

> **数据分析师 80% 的工作都是用 SELECT 查询数据**，所以我们的重点在查询上。

---

## 第二章：你的第一句 SQL

### 2.1 SELECT —— 选列

`sql
-- 语法
SELECT 列名1, 列名2 FROM 表名;

-- 示例：查所有用户的名字和年龄
SELECT name, age FROM users;
`

### 2.2 SELECT * —— 选全部列

`sql
-- 查出 users 表的所有列
SELECT * FROM users;
`

### 2.3 别名 AS

给列或表起个临时名字，让结果更易读：

`sql
SELECT name AS 姓名, age AS 年龄 FROM users;
`

### 2.4 去重 DISTINCT

`sql
-- 看看有哪些不同的城市
SELECT DISTINCT city FROM users;
`

### 2.5 常量和计算列

`sql
-- 加一列常量 "中国"
SELECT name, '中国' AS country FROM users;

-- 计算列：月薪 = 年薪 / 12
SELECT name, salary, salary / 12 AS monthly_salary FROM employees;
`

---

## 第三章：筛选数据 WHERE

### 3.1 比较运算符

`sql
SELECT * FROM products WHERE price > 100;      -- 大于
SELECT * FROM products WHERE price >= 100;      -- 大于等于
SELECT * FROM products WHERE price < 50;        -- 小于
SELECT * FROM products WHERE price <= 50;       -- 小于等于
SELECT * FROM products WHERE price = 99;         -- 等于
SELECT * FROM products WHERE price <> 99;        -- 不等于（或 !=）
`

### 3.2 逻辑运算符 AND / OR

`sql
-- 两个条件都满足
SELECT * FROM users WHERE age > 20 AND city = '北京';

-- 满足任一条件即可
SELECT * FROM users WHERE city = '北京' OR city = '上海';
`

> **注意**：AND 的优先级高于 OR，有歧义时加括号：
>
> `sql
> SELECT * FROM users WHERE (city = '北京' OR city = '上海') AND age > 20;
> `

### 3.3 BETWEEN

`sql
-- 查询价格在 50 到 100 之间的商品
SELECT * FROM products WHERE price BETWEEN 50 AND 100;
-- 等价于：price >= 50 AND price <= 100
`

### 3.4 IN

`sql
-- 查询城市是北京、上海、广州的用户
SELECT * FROM users WHERE city IN ('北京', '上海', '广州');
-- 等价于：city = '北京' OR city = '上海' OR city = '广州'
`

### 3.5 LIKE —— 模糊匹配

`sql
-- % 代表任意多个字符
SELECT * FROM users WHERE name LIKE '张%';    -- 以"张"开头
SELECT * FROM users WHERE name LIKE '%明';     -- 以"明"结尾
SELECT * FROM users WHERE email LIKE '%@qq%';  -- 包含 @qq

-- _ 代表恰好一个字符
SELECT * FROM users WHERE name LIKE '张_';     -- "张某"（两个字）
`

### 3.6 IS NULL / IS NOT NULL

`sql
-- 查没有填邮箱的用户
SELECT * FROM users WHERE email IS NULL;

-- 查已填邮箱的用户
SELECT * FROM users WHERE email IS NOT NULL;
`

> **NULL 不是 0，也不是空字符串，它表示"未知"！**

---

## 第四章：排序与取前 N 条

### 4.1 ORDER BY

`sql
-- 按年龄升序（默认 ASC）
SELECT * FROM users ORDER BY age;

-- 按年龄降序
SELECT * FROM users ORDER BY age DESC;

-- 多列排序：先按城市升序，同城市内按年龄降序
SELECT * FROM users ORDER BY city ASC, age DESC;
`

### 4.2 LIMIT（或 TOP / FETCH）

不同数据库写法略有不同：

`sql
-- MySQL / PostgreSQL / SQLite
SELECT * FROM users ORDER BY age DESC LIMIT 5;

-- SQL Server
SELECT TOP 5 * FROM users ORDER BY age DESC;
`

---

## 第五章：聚合函数

聚合函数把多行数据"压缩"成一个值。

### 5.1 常用聚合函数

| 函数 | 作用 | 示例 |
|------|------|------|
| COUNT() | 数行数 | COUNT(*) 数所有行 |
| SUM() | 求和 | SUM(salary) 工资总和 |
| AVG() | 平均值 | AVG(score) 平均分 |
| MAX() | 最大值 | MAX(price) 最高价 |
| MIN() | 最小值 | MIN(price) 最低价 |

`sql
-- 统计员工总数、总工资、平均工资、最高工资、最低工资
SELECT
    COUNT(*)   AS 总人数,
    SUM(salary)  AS 薪资总额,
    AVG(salary)  AS 平均薪资,
    MAX(salary)  AS 最高薪资,
    MIN(salary)  AS 最低薪资
FROM employees;
`

### 5.2 COUNT 的细节

`sql
-- COUNT(*)  统计所有行（包括 NULL）
-- COUNT(列名) 统计该列非 NULL 的行数
-- COUNT(DISTINCT 列名) 统计该列去重后的非 NULL 行数

SELECT
    COUNT(*)           AS 总行数,
    COUNT(email)       AS 有邮箱的人数,
    COUNT(DISTINCT city) AS 城市种类数
FROM users;
`

---

## 第六章：分组 GROUP BY

### 6.1 基本用法

GROUP BY 把数据按某一列分组，然后对每组做聚合计算。

`sql
-- 每个城市的用户数
SELECT city, COUNT(*) AS user_count
FROM users
GROUP BY city;
`

结果：

`
┌────────┬────────────┐
│ city   │ user_count │
├────────┼────────────┤
│ 北京   │        120 │
│ 上海   │         98 │
│ 广州   │         76 │
└────────┴────────────┘
`

### 6.2 多列分组

`sql
-- 每个城市、每个性别的用户数
SELECT city, gender, COUNT(*) AS cnt
FROM users
GROUP BY city, gender;
`

### 6.3 HAVING —— 过滤分组后结果

WHERE 在分组**前**过滤；HAVING 在分组**后**过滤。

`sql
-- 找出用户数大于 50 的城市
SELECT city, COUNT(*) AS cnt
FROM users
GROUP BY city
HAVING COUNT(*) > 50;
`

**WHERE vs HAVING 对比**：

`sql
-- WHERE 过滤行 → 分组 → HAVING 过滤组
SELECT city, COUNT(*) AS cnt
FROM users
WHERE age > 18              -- 先筛选成年人
GROUP BY city
HAVING COUNT(*) > 100;      -- 再筛人数>100的城市
`

### 6.4 SELECT 中能放什么？

分组后，SELECT 里只能放：
- GROUP BY 中的列
- 聚合函数

`sql
-- ✅ 正确
SELECT city, COUNT(*), AVG(age) FROM users GROUP BY city;

-- ❌ 错误：name 不在 GROUP BY 中，也不是聚合函数
SELECT city, name, COUNT(*) FROM users GROUP BY city;
`

---

## 第七章：连接 JOIN

> JOIN 是 SQL 的灵魂，也是数据分析师最核心的技能。

### 7.1 为什么需要 JOIN？

`
订单表 orders                      用户表 users
┌────┬──────────┬────────┐        ┌────┬──────┐
│ id │ user_id  │ amount │        │ id │ name │
├────┼──────────┼────────┤        ├────┼──────┤
│ 1  │ 101      │  500   │        │101 │ 张三  │
│ 2  │ 102      │  300   │        │102 │ 李四  │
│ 3  │ 101      │  200   │        └────┴──────┘
└────┴──────────┴────────┘
`

订单表只有 user_id，要看到下单人名字，就需要 JOIN。

### 7.2 INNER JOIN（最常用）

只返回**两张表都匹配**的行。

`sql
SELECT
    o.id   AS 订单号,
    u.name AS 客户名,
    o.amount AS 金额
FROM orders o
INNER JOIN users u ON o.user_id = u.id;
`

结果：

`
┌────────┬────────┬──────┐
│ 订单号 │ 客户名  │ 金额 │
├────────┼────────┼──────┤
│   1    │ 张三   │ 500  │
│   2    │ 李四   │ 300  │
│   3    │ 张三   │ 200  │
└────────┴────────┴──────┘
`

### 7.3 LEFT JOIN（保留左表全部）

`sql
-- 所有用户 + 他们的订单（没订单也显示 NULL）
SELECT u.name, o.amount
FROM users u
LEFT JOIN orders o ON u.id = o.user_id;
`

`
┌──────┬────────┐
│ name │ amount │
├──────┼────────┤
│ 张三 │  500   │
│ 张三 │  200   │
│ 李四 │  300   │
│ 王五 │  NULL  │   ← 没订单但保留了
└──────┴────────┘
`

### 7.4 四种 JOIN 对比

`
INNER JOIN  : 取交集  A ∩ B
LEFT JOIN   : 保留 A 全部  A（含交集）
RIGHT JOIN  : 保留 B 全部  B（含交集）
FULL JOIN   : 保留全部 A ∪ B  （MySQL 不支持，用 UNION 模拟）
`

### 7.5 多表 JOIN

`sql
SELECT
    u.name   AS 客户,
    o.amount AS 金额,
    p.name   AS 商品名
FROM orders o
JOIN users     u ON o.user_id    = u.id
JOIN products  p ON o.product_id = p.id;
`

---

## 第八章：子查询

子查询就是查询里套查询。

### 8.1 在 WHERE 中使用子查询

`sql
-- 查询工资高于平均工资的员工
SELECT name, salary
FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees);
`

### 8.2 在 FROM 中使用子查询（派生表）

`sql
-- 先算出每个城市的平均年龄，再从中筛选
SELECT *
FROM (
    SELECT city, AVG(age) AS avg_age
    FROM users
    GROUP BY city
) AS city_stats
WHERE avg_age > 30;
`

### 8.3 IN / NOT IN + 子查询

`sql
-- 查有过订单的用户
SELECT name FROM users
WHERE id IN (SELECT DISTINCT user_id FROM orders);

-- 查从没下单的用户
SELECT name FROM users
WHERE id NOT IN (SELECT DISTINCT user_id FROM orders);
`

### 8.4 EXISTS / NOT EXISTS

`sql
-- 查至少下过一单的用户
SELECT name FROM users u
WHERE EXISTS (
    SELECT 1 FROM orders o WHERE o.user_id = u.id
);
`

> **性能提示**：在大数据量下，EXISTS 通常比 IN 更快，因为它找到第一条就停了。

---

## 第九章：窗口函数

> 窗口函数是 SQL 进阶的标志，也是数据分析师的"大杀器"。

### 9.1 什么是窗口函数？

普通聚合（SUM + GROUP BY）会把多行压成一行；窗口函数**保留每一行，同时计算聚合值**。

`sql
-- 对比：GROUP BY  vs  窗口函数
SELECT city, SUM(sales) AS total FROM sales GROUP BY city;
-- → 每个城市一行

SELECT city, sales, SUM(sales) OVER (PARTITION BY city) AS city_total
FROM sales;
-- → 每行都保留，但加了城市小计列
`

### 9.2 基本语法

`sql
函数名() OVER (
    PARTITION BY 列名      -- 分组（可选）
    ORDER BY 列名           -- 排序（可选）
)
`

### 9.3 ROW_NUMBER() —— 编序号

`sql
-- 给每个城市的用户按年龄排序编号
SELECT
    city, name, age,
    ROW_NUMBER() OVER (PARTITION BY city ORDER BY age DESC) AS rn
FROM users;
`

### 9.4 RANK() / DENSE_RANK() —— 排名

`sql
SELECT name, score,
    RANK()       OVER (ORDER BY score DESC) AS rank_no,      -- 有跳跃：1,1,3
    DENSE_RANK() OVER (ORDER BY score DESC) AS dense_no      -- 无跳跃：1,1,2
FROM students;
`

### 9.5 前后行对比 LAG / LEAD

`sql
-- 看每个月的销售环比变化
SELECT
    month,
    sales,
    LAG(sales) OVER (ORDER BY month) AS prev_month_sales,
    sales - LAG(sales) OVER (ORDER BY month) AS change
FROM monthly_sales;
`

### 9.6 累计求和 SUM() OVER

`sql
-- 按日期累计金额
SELECT
    date, amount,
    SUM(amount) OVER (ORDER BY date) AS running_total
FROM transactions;
`

### 9.7 窗口函数全景

| 函数 | 作用 |
|------|------|
| ROW_NUMBER() | 行号（1,2,3,4...） |
| RANK() | 排名（有跳跃） |
| DENSE_RANK() | 排名（无跳跃） |
| NTILE(n) | 分桶（等分 n 组） |
| LAG(col, n) | 往前 n 行 |
| LEAD(col, n) | 往后 n 行 |
| FIRST_VALUE(col) | 窗口内第一个值 |
| LAST_VALUE(col) | 窗口内最后一个值 |
| SUM/AVG/MAX/MIN | 窗口内聚合 |

---

## 第十章：CTE 公用表表达式

CTE（WITH 子句）让你把复杂查询拆成多个"临时视图"，大幅提升可读性。

`sql
-- 不用 CTE（嵌套很乱）
SELECT ... FROM (
    SELECT ... FROM (
        SELECT ... FROM ...
    )
);

-- 用 CTE（清晰易懂）
WITH
    第一步 AS (
        SELECT city, COUNT(*) AS cnt FROM users GROUP BY city
    ),
    第二步 AS (
        SELECT * FROM 第一步 WHERE cnt > 100
    )
SELECT * FROM 第二步;
`

### 实战：找出销量前三的产品类别

`sql
WITH category_sales AS (
    SELECT category, SUM(amount) AS total
    FROM orders
    GROUP BY category
),
ranked AS (
    SELECT category, total,
           RANK() OVER (ORDER BY total DESC) AS rk
    FROM category_sales
)
SELECT category, total
FROM ranked
WHERE rk <= 3;
`

---

## 第十一章：增删改操作

### 11.1 INSERT

`sql
-- 插入一行
INSERT INTO users (name, age, city) VALUES ('赵六', 28, '深圳');

-- 插入多行
INSERT INTO users (name, age, city) VALUES
    ('钱七', 32, '杭州'),
    ('孙八', 26, '成都');

-- 从另一张表导入
INSERT INTO users_archive
SELECT * FROM users WHERE created_at < '2023-01-01';
`

### 11.2 UPDATE

`sql
-- 更新单行（一定要加 WHERE！）
UPDATE users SET city = '北京' WHERE id = 1;

-- 批量更新
UPDATE users SET age = age + 1 WHERE city = '上海';
`

> ⚠️ **不加 WHERE 的 UPDATE 会修改全部行！** 执行前务必确认。

### 11.3 DELETE

`sql
-- 删除指定行
DELETE FROM users WHERE id = 999;

-- 清空整张表（保留结构）
DELETE FROM temp_log;
-- 或更快：
TRUNCATE TABLE temp_log;
`

### 11.4 事务（救命稻草）

`sql
START TRANSACTION;         -- 开始事务
UPDATE accounts SET balance = balance - 500 WHERE id = 1;
UPDATE accounts SET balance = balance + 500 WHERE id = 2;
COMMIT;                    -- 确认提交
-- 或 ROLLBACK;             -- 撤销回滚
`

---

## 第十二章：实战分析案例

假设我们有三张表：

`sql
-- 用户表
users (id, name, city, register_date)

-- 订单表
orders (id, user_id, amount, order_date)

-- 商品表
products (id, name, category, price)
`

### 案例 1：各城市用户数和消费总额

`sql
SELECT
    u.city,
    COUNT(DISTINCT u.id) AS 用户数,
    SUM(o.amount)         AS 消费总额,
    AVG(o.amount)         AS 人均消费
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
GROUP BY u.city
ORDER BY 消费总额 DESC;
`

### 案例 2：找出每个类别的销量冠军

`sql
WITH cat_sales AS (
    SELECT p.category, p.name,
           SUM(o.amount) AS total_sales,
           RANK() OVER (PARTITION BY p.category ORDER BY SUM(o.amount) DESC) AS rk
    FROM orders o
    JOIN products p ON o.product_id = p.id
    GROUP BY p.category, p.name
)
SELECT category, name, total_sales
FROM cat_sales
WHERE rk = 1;
`

### 案例 3：用户留存分析（次日留存）

`sql
WITH first_orders AS (
    SELECT user_id, MIN(order_date) AS first_date
    FROM orders GROUP BY user_id
)
SELECT
    fo.first_date,
    COUNT(DISTINCT fo.user_id) AS 新用户数,
    COUNT(DISTINCT o.user_id) AS 次日回访数
FROM first_orders fo
LEFT JOIN orders o
    ON fo.user_id = o.user_id
    AND o.order_date = DATE_ADD(fo.first_date, INTERVAL 1 DAY)
GROUP BY fo.first_date;
`

### 案例 4：RFM 分析（最近、频率、金额）

`sql
WITH rfm AS (
    SELECT
        user_id,
        DATEDIFF('2024-12-31', MAX(order_date)) AS recency,
        COUNT(*)                                   AS frequency,
        SUM(amount)                                AS monetary
    FROM orders
    GROUP BY user_id
)
SELECT
    user_id,
    NTILE(4) OVER (ORDER BY recency DESC) AS r_score,
    NTILE(4) OVER (ORDER BY frequency)    AS f_score,
    NTILE(4) OVER (ORDER BY monetary)     AS m_score
FROM rfm;
`

---

## 附：常用速查表

### SQL 执行顺序

`sql
FROM → JOIN → WHERE → GROUP BY → HAVING → SELECT → ORDER BY → LIMIT
`

> 理解这个顺序，你就理解了 SQL 的"脾气"。

### 常用函数速查

| 类别 | 函数 | 示例 |
|------|------|------|
| 字符串 | CONCAT, SUBSTRING, UPPER, LOWER, TRIM, LENGTH | CONCAT(name, '-', city) |
| 日期 | NOW(), DATE_ADD, DATEDIFF, YEAR, MONTH, DAY | DATEDIFF('2024-12-31', order_date) |
| 数值 | ROUND, FLOOR, CEIL, ABS, MOD | ROUND(price * 1.1, 2) |
| 条件 | CASE WHEN ... THEN ... ELSE ... END | 见下方 CASE 示例 |
| 类型转换 | CAST(x AS type) | CAST('123' AS INT) |

### CASE WHEN —— SQL 里的 if-else

`sql
SELECT name, score,
    CASE
        WHEN score >= 90 THEN '优秀'
        WHEN score >= 80 THEN '良好'
        WHEN score >= 60 THEN '及格'
        ELSE '不及格'
    END AS grade
FROM students;
`

### 学习建议

1. **动手写**：SQL 是练出来的，不是看出来的
2. **先查后改**：熟悉 SELECT 再碰 INSERT/UPDATE/DELETE
3. **从小问题开始**：先查单表，再 JOIN，再窗口函数
4. **推荐练习平台**：LeetCode（SQL 题）、SQLZoo、HackerRank
5. **装一个本地数据库**：MySQL 或 PostgreSQL，导入一些 CSV 数据练习

---

> **记住：SQL 不是背出来的，是查出来的。每当你不知道怎么写，就问自己三个问题：**
> 1. 我要什么列？（SELECT）
> 2. 从哪张表来？（FROM + JOIN）
> 3. 满足什么条件？（WHERE / HAVING）
>
> 这三个问题答清楚，SQL 就写对了一半。
