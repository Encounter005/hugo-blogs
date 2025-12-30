+++
title = "SQL语句"
tags = [ "数据库" ]
categories = [ "编程语言",]
date = "2024-09-17T10:01:46+08:00"
+++
## 数据库定义语言(DDL)

### 创建数据库

```sql
create database <database_name>
```

为了支持中文，我们需要在创建的时候可以修改编码格式

```sql
CREATE DATABASE IF NOT EXISTS <database_name> DEFAULT CHARSET utf8 COLLATE utf8_general_ci
```

### 创建表

```sql
create table <table_name>(列名 数据类型[列级约束条件],
                          列名 数据类型[列级约束条件],
                          ...
                          [, 表级约束条件])

```

### SQL数据类型

#### 数值类型 (Numeric Types)

1. **整数类型**：

   - `TINYINT`: 存储很小的整数，通常范围是 `-128` 到 `127`。
   - `SMALLINT`: 存储较小的整数，通常范围是 `-32768` 到 `32767`。
   - `MEDIUMINT`: MySQL 特有的类型，适合存储中等大小的整数。
   - `INT` 或 `INTEGER`: 存储普通大小的整数，通常范围是 `-2147483648` 到 `2147483647`。
   - `BIGINT`: 存储非常大的整数，通常范围是 `-9223372036854775808` 到 `9223372036854775807`。
   - `UNSIGNED`: 以上整数类型可以加上此属性以存储非负整数，范围从 `0` 开始。

2. **小数类型**：

   - `DECIMAL` 或 `NUMERIC`: 用于存储精确的小数，格式为 `DECIMAL(M, D)`，其中 `M` 是数字总长度，`D` 是小数点后的位数。
   - `FLOAT`: 用于存储浮点数，可能丢失精度。
   - `DOUBLE` 或 `REAL`: 用于存储更高精度的浮点数。

3. **位类型**：
   - `BIT`: 用于存储位字段值。

#### 字符串类型 (String Types)

1. **定长字符串**：

   - `CHAR(n)`: 存储定长的字符串，其中 `n` 是字符串的最大长度。如果输入的字符串长度不够，会用空格填充。

2. **变长字符串**：

   - `VARCHAR(n)`: 存储变长的字符串，其中 `n` 是字符串的最大长度。
   - `BINARY(n)` 和 `VARBINARY(n)`: 分别存储定长和变长的二进制字符串。

3. **大对象类型**：
   - `TEXT`: 用于存储较大的文本数据。
   - `BLOB`: 用于存储较大的二进制数据。
   - `MEDIUMTEXT` 和 `MEDIUMBLOB`: MySQL 特有的类型，用于存储中等大小的文本或二进制数据。
   - `LONGTEXT` 和 `LONGBLOB`: MySQL 特有的类型，用于存储非常大的文本或二进制数据。

#### 日期/时间类型 (Date/Time Types)

1. **日期类型**：

   - `DATE`: 存储日期值。
   - `TIME`: 存储时间值。
   - `YEAR`: MySQL 特有的类型，用于存储四位数的年份。

2. **日期时间类型**：
   - `DATETIME`: 存储日期和时间值，支持的范围通常是 `1000-01-01 00:00:00` 至 `9999-12-31 23:59:59`。
   - `TIMESTAMP`: 存储日期和时间值，并且在插入或更新时可以自动设置为当前的日期和时间。
   - `INTERVAL`: PostgreSQL 特有的类型，用于存储时间间隔。

#### 其他类型

1. **枚举类型**：

   - `ENUM('value1', 'value2', ... )`: 存储一系列预先定义的值中的一个。

2. **集合类型**：

   - `SET('value1', 'value2', ... )`: 存储一系列预先定义的值中的一个或多个值。

3. **JSON 类型**：

   - `JSON`: MySQL 8.0 之后支持的类型，用于存储 JSON 文档。

4. **空间类型**：
   - `POINT`, `LINESTRING`, `POLYGON`, `MULTIPOINT`, `MULTILINESTRING`, `MULTIPOLYGON`, `GEOMETRYCOLLECTION`: 用于存储地理空间数据。

### 列级约束条件

有六种:主键(Primary Key),非空(NotNull)/空值null, 默认(Default),外键(外键约束),唯一键(Unique Key),检查约束(Check)(MySql不支持)、默认Default。

### 表级约束条件

有四种：主键、外键、唯一、检查

```sql
[CONSTRAINT <外键名>] FOREIGN KEY 字段名 [..., 字段名2, ...] REFERENCES <主表名> 主键列1 [, 主键列2, ...]
```

现在来创建三个表

```sql
mysql> CREATE TABLE study(sid INT PRIMARY KEY,
    -> name VARCHAR(10) NOT NULL,
    -> sex ENUM('male', 'female') NOT NULL DEFAULT 'male');

mysql> CREATE TABLE teacher(tid INT PRIMARY KEY,
    -> name VARCHAR(10) NOT NULL);


CREATE TABLE teach (
    tid INT NOT NULL,
    sid INT NOT NULL,
    CONSTRAINT f_tid FOREIGN KEY (tid) REFERENCES teacher(tid)
);

-- mysql> CREATE TABLE teach(tid int not null, sid int not null);
-- mysql> ALTER TABLE teach
--     -> ADD CONSTRAINT f_tid
--     -> FOREIGN KEY(tid)
--     -> REFERENCES teacher(tid);
```

### 修改表

如果我们想要修改表结构，可以通过`alter table`进行修改

```sql

ALTER TABLE 表名[ADD 新列名 数据类型[列级约束条件]]
                [DROP COLUMN 列名[restrict|cascade]]
                [ALTER COLUMN 列名 数据类型]

```

### 删除表

```sql
DROP TABLE 表名[restrict|cascade]
```

## 数据库操作语言(DML)

### 插入数据

```sql
INSERT INTO 表名 VALUES(值1, 值2, ...)
```

如果插入的数据与列一一对应，那么可以省略列名，但是如果希望向指定列上插入数据，就需要给出列名

```sql
INSERT INTO 表名(列名1, 列名2, ...) VALUES(值1, 值2, ...)
```

也可以一次性向数据库中插入多条数据

```sql
INSERT INTO 表名(列名1, 列名2, ...) VALUES(值1, 值2, ...), (值1, 值2, ...)
```

### 修改数据

```sql
UPDATE 表名 SET 列名 = 值, 列名 = 值, ... WHERE 条件
```

> 注意，SQL语句中等于判断是`=`

    如果忘记加where来限定条件，那么将使得整个表的数据都被修改

### 删除数据

删除全部数据

```sql
DELETE FROM 表名
```

删除指定数据

```sql
DELETE FROM 表名 WHERE 条件
```

## 数据库查询语言DQL

### 单表查询

只需要在一张表中查找数据，使用`select`语句即可

```sql

-- 指定查询某一列数据
SELECT 列名[, 列名] FROM 表名
-- 会以别名来显示此列
SELECT 列名 AS 别名 FROM 表名
-- 查询所有列的数据
SELECT * FROM 表名
-- 只查询不重复的值
SELECT DISTINCT 列名 FROM 表名
-- 使用where来限定条件
SELECT * FROM 表名 WHERE 条件

```

### 常用查询条件

- 一般的比较运算符`=, >, <, <=, >=, !=`
- 是否会在集合中`in, not in`
- 字符模糊匹配`like, not like`
- 多重条件连接查询`and, or, not`

### 排序查询

```sql
SELECT * FROM 表名 WHERE 条件 ORDER BY 列名 ASC|DESC
```

ASC 升序 DESC 降序 默认为升序

添加多个排序

```sql
SELECT * FROM 表名 WHERE 条件 ORDER BY 列名1 ASC|DESC, 列名2 ASC|DESC
```

会先按照列名1进行排序，再按照列名2进行排序

### 聚集函数

SQL 中的聚集函数（Aggregate Functions）用于对一组值执行计算，并返回单个值。这些函数通常用于对表中的数据进行汇总和分析。以下是一些常见的 SQL 聚集函数：

#### 1. `COUNT()`

- **用途**：计算表中行的数量或满足特定条件的行的数量。
- **示例**：
  ```sql
  SELECT COUNT(*) FROM employees;  -- 计算 employees 表中的总行数
  SELECT COUNT(DISTINCT department) FROM employees;  -- 计算 employees 表中不同部门的数量
  ```

#### 2. `SUM()`

- **用途**：计算某一列值的总和。
- **示例**：
  ```sql
  SELECT SUM(salary) FROM employees;  -- 计算 employees 表中所有员工的总工资
  ```

#### 3. `AVG()`

- **用途**：计算某一列值的平均值。
- **示例**：
  ```sql
  SELECT AVG(salary) FROM employees;  -- 计算 employees 表中所有员工的平均工资
  ```

#### 4. `MIN()`

- **用途**：返回某一列的最小值。
- **示例**：
  ```sql
  SELECT MIN(salary) FROM employees;  -- 返回 employees 表中最低的工资
  ```

#### 5. `MAX()`

- **用途**：返回某一列的最大值。
- **示例**：
  ```sql
  SELECT MAX(salary) FROM employees;  -- 返回 employees 表中最高的工资
  ```

#### 6. `GROUP_CONCAT()`

- **用途**：将某一列的值连接成一个字符串，通常用于将分组后的值合并。
- **示例**：
  ```sql
  SELECT department, GROUP_CONCAT(name) FROM employees GROUP BY department;  -- 将每个部门的员工名字连接成一个字符串
  ```

#### 7. `VARIANCE()` 和 `STDDEV()`

- **用途**：计算某一列值的方差和标准差。
- **示例**：
  ```sql
  SELECT VARIANCE(salary) FROM employees;  -- 计算 employees 表中工资的方差
  SELECT STDDEV(salary) FROM employees;  -- 计算 employees 表中工资的标准差
  ```

#### 8. `BIT_AND()`, `BIT_OR()`, `BIT_XOR()`

- **用途**：对某一列的值执行按位与、按位或、按位异或操作。
- **示例**：
  ```sql
  SELECT BIT_AND(flags) FROM employees;  -- 对 employees 表中的 flags 列执行按位与操作
  ```

#### 使用聚集函数的注意事项：

- **NULL 值**：聚集函数通常会忽略 `NULL` 值。例如，`SUM()` 函数不会将 `NULL` 值计入总和。
- **GROUP BY 子句**：聚集函数通常与 `GROUP BY` 子句一起使用，以便对数据进行分组并计算每个组的汇总值。
- **HAVING 子句**：用于过滤分组后的结果，通常与 `GROUP BY` 子句一起使用。

#### 示例：

```sql
SELECT department, AVG(salary) AS avg_salary
FROM employees
GROUP BY department
HAVING AVG(salary) > 50000;
```

这个查询会计算每个部门的平均工资，并只返回平均工资大于 50000 的部门。

### 分组分页查询

通过`group by`来对查询结果进行分组，需要结合聚集函数一起使用

在 MySQL 中，分组分页查询通常涉及使用 `GROUP BY` 子句对数据进行分组，然后使用 `LIMIT` 子句对结果进行分页。以下是详细解释和示例：

#### 1. `GROUP BY` 子句

`GROUP BY` 子句用于将数据按照一个或多个列进行分组。通常与聚集函数（如 `COUNT`, `SUM`, `AVG`, `MIN`, `MAX` 等）一起使用，以便对每个组进行汇总计算。

#### 2. `LIMIT` 子句

`LIMIT` 子句用于限制查询结果的行数。通常用于分页查询，通过指定起始行和返回的行数来实现分页。

#### 示例：分组分页查询

假设我们有一个 `orders` 表，结构如下：

```sql
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT,
    order_date DATE,
    amount DECIMAL(10, 2)
);
```

我们希望按 `customer_id` 分组，并计算每个客户的订单总金额，然后对结果进行分页。

#### 查询每个客户的订单总金额

```sql
SELECT customer_id, SUM(amount) AS total_amount
FROM orders
GROUP BY customer_id;
```

#### 分页查询每个客户的订单总金额

假设我们希望每页显示 10 个客户，查询第 2 页的数据（即第 11 到第 20 个客户）：

```sql
SELECT customer_id, SUM(amount) AS total_amount
FROM orders
GROUP BY customer_id
LIMIT 10 OFFSET 10;
```

#### 解释：

- **`LIMIT 10 OFFSET 10`**：
  - `LIMIT 10` 表示每页显示 10 行。
  - `OFFSET 10` 表示从第 11 行开始（因为 `OFFSET` 是从 0 开始计数的）。
  - `LIMIT 起始位置, 数量`

#### 使用变量进行分页

如果你希望动态地指定页码和每页的行数，可以使用变量：

```sql
SET @page_number = 2;  -- 页码，从 1 开始
SET @page_size = 10;   -- 每页的行数

SELECT customer_id, SUM(amount) AS total_amount
FROM orders
GROUP BY customer_id
LIMIT @page_size OFFSET ((@page_number - 1) * @page_size);
```

#### 注意事项：

1. **性能问题**：分组和分页查询可能会影响性能，尤其是在处理大量数据时。可以考虑使用索引来优化查询。
2. **`HAVING` 子句**：如果你需要在分组后对结果进行过滤，可以使用 `HAVING` 子句。例如，只显示订单总金额大于 1000 的客户：
   ```sql
   SELECT customer_id, SUM(amount) AS total_amount
   FROM orders
   GROUP BY customer_id
   HAVING total_amount > 1000
   LIMIT 10 OFFSET 10;
   ```

### 多表查询

多表查询是同时查询两个或两个以上的表，多表查询会通过连接转换为单表查询

```sql
SELECT * FROM 表1, 表2
```

直接这样查询会得到两张笛卡尔积，也就是每一项数据和另一张表的每一项数据都结合一次，会产生庞大的数据

```sql
SELECT * FROM 表1, 表2 WHERE 条件
```

这样只会从笛卡尔积的结果中得到满足条件的数据

### 自身连接查询

自身连接就是将表自身和表进行笛卡尔积计算，得到结果，但是由于表名相同，因此要先起一个别名

```sql
SELECT * FROM 表1 AS t1, 表1 AS t2
```

### 外连接查询

SQL 中的外连接（Outer Join）用于从两个或多个表中获取数据，即使某些行在连接条件中没有匹配的行。外连接分为三种类型：左外连接（Left Outer Join）、右外连接（Right Outer Join）和全外连接（Full Outer Join）。  

假设我们现在有一张存储所有用户的表，还有一张用户详细信息的表，我们希望将这两张表结合到一起来查看完整数据



#### 1. 左外连接（Left Outer Join）

左外连接返回左表中的所有行，即使右表中没有匹配的行。如果右表中没有匹配的行，结果集中对应右表的列将为 `NULL`。

##### 语法：

```sql
SELECT columns
FROM left_table
LEFT JOIN right_table
ON left_table.column = right_table.column;
```

##### 示例：

假设有两个表 `employees` 和 `departments`：

```sql
CREATE TABLE employees (
    employee_id INT PRIMARY KEY,
    employee_name VARCHAR(100),
    department_id INT
);

CREATE TABLE departments (
    department_id INT PRIMARY KEY,
    department_name VARCHAR(100)
);
```

我们希望获取所有员工及其所属部门的信息，包括那些没有部门的员工：

```sql
SELECT employees.employee_name, departments.department_name
FROM employees
LEFT JOIN departments
ON employees.department_id = departments.department_id;
```

#### 2. 右外连接（Right Outer Join）

右外连接返回右表中的所有行，即使左表中没有匹配的行。如果左表中没有匹配的行，结果集中对应左表的列将为 `NULL`。

##### 语法：

```sql
SELECT columns
FROM left_table
RIGHT JOIN right_table
ON left_table.column = right_table.column;
```

##### 示例：

我们希望获取所有部门及其员工的信息，包括那些没有员工的部门：

```sql
SELECT employees.employee_name, departments.department_name
FROM employees
RIGHT JOIN departments
ON employees.department_id = departments.department_id;
```

#### 3. 全外连接（Full Outer Join）

全外连接返回左表和右表中的所有行，即使它们在连接条件中没有匹配的行。如果某个表中没有匹配的行，结果集中对应另一个表的列将为 `NULL`。

##### 语法：

```sql
SELECT columns
FROM left_table
FULL OUTER JOIN right_table
ON left_table.column = right_table.column;
```

##### 示例：

我们希望获取所有员工和所有部门的信息，包括那些没有部门或没有员工的记录：

```sql
SELECT employees.employee_name, departments.department_name
FROM employees
FULL OUTER JOIN departments
ON employees.department_id = departments.department_id;
```

#### 注意事项：

- **MySQL 不支持全外连接**：MySQL 不直接支持 `FULL OUTER JOIN`，但可以通过 `UNION` 操作符来模拟全外连接。
- **模拟全外连接**：
  ```sql
  SELECT employees.employee_name, departments.department_name
  FROM employees
  LEFT JOIN departments
  ON employees.department_id = departments.department_id
  UNION
  SELECT employees.employee_name, departments.department_name
  FROM employees
  RIGHT JOIN departments
  ON employees.department_id = departments.department_id;
  ```

#### 总结

外连接允许你从两个或多个表中获取数据，即使某些行在连接条件中没有匹配的行。左外连接返回左表中的所有行，右外连接返回右表中的所有行，全外连接返回两个表中的所有行。通过合理使用外连接，你可以更灵活地处理数据关联和缺失情况。



### 嵌套查询

可以将查询的结果作为另一个查询的条件

```sql
SELECT * FROM 表名 WHERE 列名 IN (SELECT 列名 FROM 表名 WHERE 条件);
```


SQL 中的嵌套查询（Nested Query），也称为子查询（Subquery），是指在一个查询语句中嵌套另一个查询语句。子查询可以出现在 `SELECT`、`FROM`、`WHERE`、`HAVING` 等子句中，用于从数据库中获取更复杂的数据。

#### 1. 子查询的基本概念
子查询是一个完整的 `SELECT` 语句，它可以返回一个标量值（单个值）、一个行、一个列或一个表。子查询通常用括号 `()` 括起来，并且可以嵌套在主查询的各个部分。

#### 2. 子查询的类型
根据子查询返回的结果类型，子查询可以分为以下几种类型：

##### 2.1 标量子查询（Scalar Subquery）
标量子查询返回单个值（即一行一列）。它通常用于 `SELECT` 子句、`WHERE` 子句或 `HAVING` 子句中。

###### 示例：
假设我们有一个 `employees` 表和一个 `departments` 表，我们希望查询每个员工的姓名及其所属部门的名称：

```sql
SELECT employee_name,
       (SELECT department_name
        FROM departments
        WHERE departments.department_id = employees.department_id) AS department_name
FROM employees;
```

##### 2.2 行子查询（Row Subquery）
行子查询返回一行数据（即多列）。它通常用于 `WHERE` 子句中。

###### 示例：
假设我们希望查询工资最高的员工的姓名和工资：

```sql
SELECT employee_name, salary
FROM employees
WHERE (salary, employee_id) = (SELECT MAX(salary), employee_id
                               FROM employees);
```

##### 2.3 列子查询（Column Subquery）
列子查询返回一列数据（即多行一列）。它通常用于 `WHERE` 子句中。

###### 示例：
假设我们希望查询所有工资高于平均工资的员工：

```sql
SELECT employee_name, salary
FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees);
```

##### 2.4 表子查询（Table Subquery）
表子查询返回一个表（即多行多列）。它通常用于 `FROM` 子句中。

###### 示例：
假设我们希望查询每个部门的平均工资，并按平均工资排序：

```sql
SELECT department_name, avg_salary
FROM (SELECT department_id, AVG(salary) AS avg_salary
      FROM employees
      GROUP BY department_id) AS dept_avg
JOIN departments ON dept_avg.department_id = departments.department_id
ORDER BY avg_salary DESC;
```

#### 3. 子查询的位置
子查询可以出现在以下位置：

##### 3.1 `SELECT` 子句
子查询可以出现在 `SELECT` 子句中，用于计算每个行的特定值。

###### 示例：
```sql
SELECT employee_name,
       salary,
       (SELECT AVG(salary) FROM employees) AS avg_salary
FROM employees;
```

##### 3.2 `FROM` 子句
子查询可以出现在 `FROM` 子句中，用于生成一个临时表，供主查询使用。

###### 示例：
```sql
SELECT department_name, avg_salary
FROM (SELECT department_id, AVG(salary) AS avg_salary
      FROM employees
      GROUP BY department_id) AS dept_avg
JOIN departments ON dept_avg.department_id = departments.department_id;
```

##### 3.3 `WHERE` 子句
子查询可以出现在 `WHERE` 子句中，用于过滤数据。

###### 示例：
```sql
SELECT employee_name, salary
FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees);
```

##### 3.4 `HAVING` 子句
子查询可以出现在 `HAVING` 子句中，用于过滤分组后的数据。

###### 示例：
```sql
SELECT department_id, AVG(salary) AS avg_salary
FROM employees
GROUP BY department_id
HAVING AVG(salary) > (SELECT AVG(salary) FROM employees);
```

#### 4. 注意事项
- **性能问题**：嵌套查询可能会影响性能，尤其是在处理大量数据时。可以考虑使用连接（Join）或其他优化方法来替代嵌套查询。
- **相关子查询**：相关子查询是指子查询依赖于外部查询的值。相关子查询通常会导致性能问题，因为它们会对外部查询的每一行执行一次子查询。



### 数据库控制语言DCL

#### 创建用户

```sql
CREATE USER <username> IDENTIFIED BY <password>
```

也可以不带密码

```sql
CREATE USER <username>
```

#### 登录用户

```
mysql -u <username> -p <password>
```

### 用户授权


#### 1. `GRANT`
`GRANT` 命令用于授予用户或角色对数据库对象（如表、视图、存储过程等）的访问权限。

##### 语法：
```sql
GRANT privileges ON object TO user_or_role;
```

##### 参数说明：
- `privileges`：要授予的权限，可以是 `SELECT`, `INSERT`, `UPDATE`, `DELETE`, `ALL PRIVILEGES` 等。
- `object`：要授予权限的数据库对象，如表、视图、存储过程等。
- `user_or_role`：要授予权限的用户或角色。

##### 示例：
```sql
-- 授予用户 'alice' 对表 'employees' 的 SELECT 权限
GRANT SELECT ON employees TO alice;

-- 授予用户 'bob' 对表 'employees' 的所有权限
GRANT ALL PRIVILEGES ON employees TO bob;

-- 授予角色 'developers' 对数据库 'mydb' 的所有权限
GRANT ALL PRIVILEGES ON DATABASE mydb TO developers;
```

#### 2. `REVOKE`
`REVOKE` 命令用于撤销用户或角色对数据库对象的访问权限。

##### 语法：
```sql
REVOKE privileges ON object FROM user_or_role;
```

##### 参数说明：
- **`privileges`**：要撤销的权限，可以是 `SELECT`, `INSERT`, `UPDATE`, `DELETE`, `ALL PRIVILEGES` 等。
- **`object`**：要撤销权限的数据库对象，如表、视图、存储过程等。
- **`user_or_role`**：要撤销权限的用户或角色。

##### 示例：
```sql
-- 撤销用户 'alice' 对表 'employees' 的 SELECT 权限
REVOKE SELECT ON employees FROM alice;

-- 撤销用户 'bob' 对表 'employees' 的所有权限
REVOKE ALL PRIVILEGES ON employees FROM bob;

-- 撤销角色 'developers' 对数据库 'mydb' 的所有权限
REVOKE ALL PRIVILEGES ON DATABASE mydb FROM developers;
```

#### 3. `DENY`
`DENY` 命令用于显式拒绝用户或角色对数据库对象的访问权限。与 `REVOKE` 不同，`DENY` 会阻止用户通过角色或其他方式继承权限。

##### 语法：
```sql
DENY privileges ON object TO user_or_role;
```

##### 参数说明：
- **`privileges`**：要拒绝的权限，可以是 `SELECT`, `INSERT`, `UPDATE`, `DELETE`, `ALL PRIVILEGES` 等。
- **`object`**：要拒绝权限的数据库对象，如表、视图、存储过程等。
- **`user_or_role`**：要拒绝权限的用户或角色。

##### 示例：
```sql
-- 拒绝用户 'alice' 对表 'employees' 的 SELECT 权限
DENY SELECT ON employees TO alice;

-- 拒绝用户 'bob' 对表 'employees' 的所有权限
DENY ALL PRIVILEGES ON employees TO bob;

-- 拒绝角色 'developers' 对数据库 'mydb' 的所有权限
DENY ALL PRIVILEGES ON DATABASE mydb TO developers;
```

#### 4. `CREATE ROLE` 和 `DROP ROLE`
`CREATE ROLE` 和 `DROP ROLE` 命令用于创建和删除角色。角色是一组权限的集合，可以授予用户或其他角色。

##### 语法：
```sql
CREATE ROLE role_name;
DROP ROLE role_name;
```

##### 示例：
```sql
-- 创建一个名为 'developers' 的角色
CREATE ROLE developers;

-- 删除名为 'developers' 的角色
DROP ROLE developers;
```

#### 5. `ALTER ROLE`
`ALTER ROLE` 命令用于修改角色的属性，如重命名角色、修改角色的默认配置等。

##### 语法：
```sql
ALTER ROLE role_name WITH options;
```

##### 示例：
```sql
-- 重命名角色 'developers' 为 'devs'
ALTER ROLE developers RENAME TO devs;

-- 修改角色 'devs' 的默认配置
ALTER ROLE devs WITH LOGIN;
```

#### 6. `SET ROLE`
`SET ROLE` 命令用于在当前会话中切换角色。

##### 语法：
```sql
SET ROLE role_name;
```

##### 示例：
```sql
-- 在当前会话中切换到角色 'developers'
SET ROLE developers;
```


