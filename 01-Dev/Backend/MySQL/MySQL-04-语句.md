# 一.语句相关
## 1.count(1),count(星),count(name)有什么区别？
- count(1)和count(星)**完全相同**，且**都会读null值**
- count(name)要**对null值进行判断**，因此会慢一点。
- 实际上如果没索引，count(name)会慢点，如果有索引或者字段为主键，那么差别不大。
一般**推荐 count(星)，语义清晰，优化器也会优化**。

## 2.varchar和char有什么区别？
- varchar变长，但char定长。
- varchar会使用**额外一两个字符来记录长度**，而**char会将没使用的字符设置为空**。
- varchar如果更新后的字段比原来长，可能因为页无法容纳新数据造成**页分裂**，产生磁盘碎片影响IO性能。

## 3.一条sql语句怎么执行？
- 连接器：建立连接、权限校验
- 查询缓存：MySQL 8.0 已移除，了解即可（频繁更新会导致**维护缓存开销过大**）。
- 分析器：词法、语法分析
	- 词法分析（识别**关键字，表名，列名**）。
	- 语法分析（检查语句是否符合mysql**语法规则**）。
- 优化器：**选择索引、生成执行计划**。
- 执行器：调用存储引擎接口执行。
- 存储引擎：真正读写数据。

# 二.语句
`sql`判空必须用`is`不能用`=`
`cross join`得到笛卡尔积
`round(xx)`表示四舍五入到整数，`round(xx, 2)`表示四舍五入到小数点后2位
`ifnull(xx, a)`如果为空则值为a

## 0.sql执行顺序
```text
FROM
JOIN
WHERE
GROUP BY
HAVING
SELECT
ORDER BY
LIMIT
```
## 1.sql最小模板
1. 查询顺序
```sql
SELECT 字段
FROM 表
WHERE 条件
GROUP BY 分组字段
HAVING 分组后条件
ORDER BY 排序字段 DESC
LIMIT 10;
```
2. group by统计
```sql
SELECT user_id, COUNT(*) AS cnt
FROM orders
GROUP BY user_id;
```
注意，使用`group by`之后，分组前的多行会被压缩为一行，所以只有以下两个处理情况：
- 对分组字段本身进行`select`
- 对分组字段使用聚合函数
3. having
```sql
SELECT user_id, COUNT(*) AS cnt
FROM orders
GROUP BY user_id
HAVING COUNT(*) >= 3;
```
4. join
```sql
SELECT u.id, u.name, o.id AS order_id
FROM user u
JOIN orders o ON u.id = o.user_id;
```
5. left join查询没有订单的用户
```sql
SELECT u.id, u.name
FROM user u
LEFT JOIN orders o ON u.id = o.user_id
WHERE o.id IS NULL;
```
6. 子查询
```sql
SELECT *
FROM employee
WHERE salary > (
    SELECT AVG(salary)
    FROM employee
);
```
7. 每组第一名
```sql
SELECT *
FROM (
    SELECT e.*,
           ROW_NUMBER() OVER(PARTITION BY department_id ORDER BY salary DESC) AS rn
    FROM employee e
) t
WHERE rn = 1;
```
8. 每组topN
```sql
SELECT *
FROM (
    SELECT e.*,
           ROW_NUMBER() OVER(PARTITION BY department_id ORDER BY salary DESC) AS rn
    FROM employee e
) t
WHERE rn <= 3;
```
## 2.连接
```text
COUNT(*)：统计所有行
COUNT(字段)：统计该字段不为 NULL 的行

INNER JOIN：两边匹配才保留
LEFT JOIN：左表全部保留，右表没有就是 NULL

ROW_NUMBER：不并列，强行编号 1,2,3
RANK：并列跳名次 1,1,3
DENSE_RANK：并列不跳名次 1,1,2
```

# 三.聚合函数与常见函数
`datediff(a, b)`得到相差天数
例如：查询温度比上一天高的天
```sql
select a.id
from Weather as a, Weather as b
where a.Temperature > b.Temperature
and datediff(a.recordDate, b.recordDate) = 1;

select a.id 
from Weather as a inner join Weather b on datediff(a.recordDate, b.recordDate) = 1 
where a.temperature > b.temperature
```

# 四.窗口函数
每组取第一条：
```sql
SELECT *
FROM (
    SELECT
        t.*,
        ROW_NUMBER() OVER (
            PARTITION BY 分组字段
            ORDER BY 排序字段 DESC
        ) AS rn
    FROM 表 t
) x
WHERE rn = 1;
```
每组排名：
```sql
SELECT
    *,
    RANK() OVER (
        PARTITION BY 分组字段
        ORDER BY 排序字段 DESC
    ) AS ranking
FROM 表;
```
每组求平均但不合并：
```sql
SELECT
    *,
    AVG(score) OVER (PARTITION BY class) AS avg_score
FROM Scores;
```

## 1.窗口函数基础
窗口函数是在“**不合并行**”的前提下，对每一行计算它所在分组里的**排名、累计值、前后行数据**等。
窗口函数 = GROUP BY 的分组能力 + ORDER BY 的排序能力，但**不压缩行**。
和`group by`相比：
- GROUP BY：多行变一行
- 窗口函数：行数不变，只是在每一行旁边多算一个结果

窗口函数基本格式：
```sql
窗口函数() OVER (
    PARTITION BY 分组字段
    ORDER BY 排序字段
)
```
例如：每个班每个学生的班级排名
```sql
SELECT
    student,
    class,
    score,
    RANK() OVER (
        PARTITION BY class
        ORDER BY score DESC
    ) AS ranking
FROM Scores;
```
其中：
```sql
RANK()             -- 我要排名
OVER (...)         -- 在一个窗口里算
PARTITION BY class -- 按班级分组，每个班单独排
ORDER BY score DESC -- 按分数从高到低排
```

## 2.排名类窗口函数
```sql
ROW_NUMBER()：不管是否并列，都给唯一编号：1, 2, 3, 4
RANK()：并列会跳名次：1, 2, 2, 4
DENSE_RANK()：并列不跳名次：1, 2, 2, 3
```
例如：每个用户最新的一条记录
```sql
ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY create_time DESC)
```

## 3.聚合类窗口函数
- 普通聚合会合并行：`SELECT class, AVG(score) FROM Scores GROUP BY class;`结果每个班只有一行。
- 聚合窗口函数：不会合并行。

例如：每一行都保留，同时算出这个学生所在班级的平均分
```sql
SELECT
    student,
    class,
    score,
    AVG(score) OVER (PARTITION BY class) AS class_avg_score
FROM Scores;
```

## 4.前后行函数
```sql
LAG()：取上一行
LEAD()：取下一行
```
例如：查用户这次登录和上次登录间隔
```sql
SELECT
    user_id,
    login_time,
    LAG(login_time) OVER (
        PARTITION BY user_id
        ORDER BY login_time
    ) AS last_login_time
FROM Logins;
```