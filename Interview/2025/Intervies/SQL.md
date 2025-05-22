# 窗口函数解法详解：连续登录问题

窗口函数是解决连续登录问题最优雅和高效的方法，下面我将从底层原理到实际应用全面解析这种解法。

## 一、核心思路

窗口函数解法的**核心洞察**是：**对于连续日期，当用日期减去其序号时，结果相同**。

### 基本原理
1. 对每个用户的登录日期排序
2. 计算 `登录日期 - 序号天数` 
3. 相同结果的日期属于同一个连续区间

## 二、分步拆解

### 步骤1：数据准备（去重）

```sql
-- 原始数据可能存在同一用户同一天多次登录，需要去重
SELECT DISTINCT user_id, login_date 
FROM login_records;
```

**为什么需要去重**：
- 避免同一日期被多次计数
- 减少后续计算的数据量

### 步骤2：按用户分组并排序

```sql
SELECT 
    user_id,
    login_date,
    ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY login_date) AS row_num
FROM 
    (SELECT DISTINCT user_id, login_date FROM login_records) t;
```

**窗口函数解析**：
- `PARTITION BY user_id`：按用户分组计算
- `ORDER BY login_date`：按日期排序
- `ROW_NUMBER()`：为每组数据生成序号

### 步骤3：计算连续日期标识

```sql
SELECT 
    user_id,
    login_date,
    DATE_SUB(login_date, INTERVAL ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY login_date) DAY) AS date_group
FROM 
    (SELECT DISTINCT user_id, login_date FROM login_records) t;
```

**关键点**：
- `login_date - row_num`：连续日期的这个差值相同
- 非连续日期会导致差值变化

### 步骤4：分组统计连续天数

```sql
WITH date_groups AS (
    SELECT 
        user_id,
        login_date,
        DATE_SUB(login_date, INTERVAL ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY login_date) DAY) AS date_group
    FROM 
        (SELECT DISTINCT user_id, login_date FROM login_records) t
)
SELECT 
    user_id,
    date_group,
    MIN(login_date) AS start_date,
    MAX(login_date) AS end_date,
    COUNT(*) AS consecutive_days
FROM 
    date_groups
GROUP BY 
    user_id, date_group
HAVING 
    COUNT(*) >= 3;  -- 筛选连续登录≥3天的记录
```

## 三、执行过程示例

假设用户A的登录记录：

```
user_id | login_date
------- | ----------
A       | 2023-01-01
A       | 2023-01-02 
A       | 2023-01-03
A       | 2023-01-05
A       | 2023-01-06
```

### 步骤1：去重（本例已无重复）

### 步骤2：生成序号

```
user_id | login_date | row_num
------- | ---------- | -------
A       | 2023-01-01 | 1
A       | 2023-01-02 | 2  
A       | 2023-01-03 | 3
A       | 2023-01-05 | 4
A       | 2023-01-06 | 5
```

### 步骤3：计算date_group

```
user_id | login_date | row_num | login_date - row_num
------- | ---------- | ------- | --------------------
A       | 2023-01-01 | 1       | 2022-12-31
A       | 2023-01-02 | 2       | 2022-12-31  -- 相同
A       | 2023-01-03 | 3       | 2022-12-31  -- 相同
A       | 2023-01-05 | 4       | 2023-01-01  -- 变化（因为1月4日缺失）
A       | 2023-01-06 | 5       | 2023-01-01  -- 相同
```

### 步骤4：分组统计

```
user_id | date_group | start_date | end_date   | consecutive_days
------- | ---------- | ---------- | ---------- | ----------------
A       | 2022-12-31 | 2023-01-01 | 2023-01-03 | 3
A       | 2023-01-01 | 2023-01-05 | 2023-01-06 | 2
```

### 最终结果

```
user_id | start_date | end_date   | consecutive_days
------- | ---------- | ---------- | ----------------
A       | 2023-01-01 | 2023-01-03 | 3
```

## 四、技术细节解析

### 1. 为什么`date - row_num`能标识连续区间？

- **连续日期**：每天减序号相当于减天数，结果相同
  ```
  2023-01-01 - 1 = 2022-12-31
  2023-01-02 - 2 = 2022-12-31
  2023-01-03 - 3 = 2022-12-31
  ```
  
- **非连续日期**：跳过的天数会导致差值变化
  ```
  2023-01-05 - 4 = 2023-01-01  -- 因为跳过了1月4日
  ```

### 2. 窗口函数执行过程

1. **数据分区**：按user_id分组，每组数据独立计算
2. **排序**：每组内按login_date排序
3. **计算**：对每行计算ROW_NUMBER()和日期差值
4. **物化**：将结果物化为临时结果集

### 3. 性能优化点

- **索引建议**：确保`(user_id, login_date)`有复合索引
- **分区裁剪**：如果按日期分区，先过滤时间范围
- **并行计算**：窗口函数天然支持分区内并行计算

## 五、变种问题解决方案

### 1. 计算最大连续登录天数

```sql
WITH date_groups AS (
    -- 同上
),
consecutive_stats AS (
    SELECT 
        user_id,
        COUNT(*) AS consecutive_days
    FROM 
        date_groups
    GROUP BY 
        user_id, date_group
)
SELECT 
    user_id,
    MAX(consecutive_days) AS max_consecutive_days
FROM 
    consecutive_stats
GROUP BY 
    user_id;
```

### 2. 筛选特定长度的连续登录

```sql
-- 查找正好连续登录5天的用户
HAVING COUNT(*) = 5;

-- 查找连续登录3-7天的用户  
HAVING COUNT(*) BETWEEN 3 AND 7;
```

### 3. 带业务约束的连续登录

```sql
-- 连续登录且每次登录时长>1小时
WITH valid_logins AS (
    SELECT DISTINCT user_id, login_date
    FROM login_sessions
    WHERE duration > 3600
),
date_groups AS (
    SELECT 
        user_id,
        login_date,
        DATE_SUB(login_date, INTERVAL ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY login_date) DAY) AS date_group
    FROM valid_logins
)
-- 其余相同
```

## 六、不同数据库实现

### MySQL
```sql
SELECT 
    user_id,
    login_date - INTERVAL ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY login_date) DAY AS date_group
FROM 
    (SELECT DISTINCT user_id, login_date FROM login_records) t;
```

### PostgreSQL
```sql
SELECT 
    user_id,
    login_date - ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY login_date)::INT AS date_group
FROM 
    (SELECT DISTINCT user_id, login_date FROM login_records) t;
```

### Spark SQL
```sql
SELECT 
    user_id,
    date_sub(login_date, ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY login_date)) AS date_group
FROM 
    (SELECT DISTINCT user_id, login_date FROM login_records) t;
```

## 七、性能对比

| 方法 | 10万用户×100天 | 100万用户×1000天 | 适用场景 |
|------|----------------|-------------------|----------|
| 窗口函数 | 2-5秒 | 30-60秒 | 通用场景 |
| 自连接 | 10-30秒 | 超时 | 极小数据量 |
| 递归CTE | 5-10秒 | 2-5分钟 | 特定数据库 |

## 八、常见问题解答

### Q1：如何处理跨年日期？
**方案**：使用`DATE_SUB`或日期减法，数据库会自动处理跨年

### Q2：如果登录日期有时间戳怎么办？
**方案**：先转换为日期类型
```sql
CAST(login_time AS DATE) AS login_date
```

### Q3：如何提高大用户量的性能？
**优化建议**：
1. 先过滤时间范围减少数据量
2. 对中间结果使用临时表
3. 增加`spark.sql.shuffle.partitions`（Spark环境）

窗口函数解法以其简洁性和高效性成为解决连续登录问题的首选方案，适用于大多数SQL数据库和数据处理平台。理解其核心原理后，可以灵活应用到各种连续事件分析场景中。



Spark SQL:

-- 步骤1：对用户登录日期去重
WITH distinct_logins AS (
    SELECT DISTINCT user_id, login_date 
    FROM login_records
),

-- 步骤2：标记连续区间
login_groups AS (
    SELECT 
        user_id,
        login_date,
        SUM(login_flag) OVER (PARTITION BY user_id ORDER BY login_date) AS group_id
    FROM (
        SELECT 
            user_id,
            login_date,
            CASE WHEN DATEDIFF(login_date, LAG(login_date, 1) OVER (PARTITION BY user_id ORDER BY login_date)) = 1 
                 THEN 0 ELSE 1 END AS login_flag
        FROM distinct_logins
    ) t
)

-- 步骤3：统计结果
SELECT 
    user_id,
    group_id,
    MIN(login_date) AS start_date,
    MAX(login_date) AS end_date,
    COUNT(*) AS consecutive_days
FROM login_groups
GROUP BY user_id, group_id
HAVING COUNT(*) >= 3;