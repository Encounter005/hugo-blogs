+++
title = "SQL视图"
tags = [ "数据库" ]
categories = [ "编程语言"]
date = "2024-09-17T10:57:07+08:00"
+++
## 视图

SQL 中的视图（View）是一个虚拟表，它基于一个或多个表的查询结果。视图本身不存储数据，而是存储查询的定义。当你查询视图时，数据库引擎会执行视图定义的查询，并返回结果。视图可以简化复杂的查询，提供数据的安全性，并隐藏底层表的复杂性。


### 1. 创建视图（CREATE VIEW）

你可以使用 `CREATE VIEW` 语句来创建视图。视图的定义可以包含 `SELECT` 语句中的所有元素，如 `WHERE`、`GROUP BY`、`JOIN` 等。

#### 语法：

```sql
CREATE VIEW view_name AS
SELECT columns
FROM tables
WHERE conditions;
```

#### 示例：

假设我们有一个 `employees` 表和一个 `departments` 表，我们希望创建一个视图，显示每个员工的姓名、部门名称和工资：

```sql
CREATE VIEW employee_department_view AS
SELECT employees.employee_name, departments.department_name, employees.salary
FROM employees
JOIN departments ON employees.department_id = departments.department_id;
```

### 2. 查询视图（SELECT）

查询视图与查询表的方式相同。你可以像查询表一样查询视图。

#### 示例：

```sql
SELECT * FROM employee_department_view;
```

### 3. 更新视图（UPDATE VIEW）

在某些情况下，你可以通过视图更新底层表的数据。视图必须是可更新的，即视图的定义必须满足一定的条件（如没有 `GROUP BY`、`HAVING`、`DISTINCT` 等）。

#### 示例：

```sql
UPDATE employee_department_view
SET salary = 6000
WHERE employee_name = 'Alice';
```

### 4. 删除视图（DROP VIEW）

你可以使用 `DROP VIEW` 语句删除视图。

#### 语法：

```sql
DROP VIEW view_name;
```

#### 示例：

```sql
DROP VIEW employee_department_view;
```

### 5. 修改视图（ALTER VIEW）

在某些数据库系统中，你可以使用 `ALTER VIEW` 语句修改视图的定义。

#### 语法：

```sql
ALTER VIEW view_name AS
SELECT columns
FROM tables
WHERE conditions;
```

#### 示例：

```sql
ALTER VIEW employee_department_view AS
SELECT employees.employee_name, departments.department_name, employees.salary
FROM employees
JOIN departments ON employees.department_id = departments.department_id
WHERE employees.salary > 5000;
```

### 6. 视图的优点

- 简化查询:视图可以将复杂的查询封装起来，简化用户的查询操作。
- 数据安全:视图可以限制用户只能访问特定的数据，从而提高数据的安全性。
- 数据独立性:视图可以隐藏底层表的复杂性，使用户无需了解底层表的结构。
- 逻辑数据分组:视图可以将多个表的数据逻辑上组合在一起，方便用户进行数据分析。

### 7. 视图的缺点

- 性能问题:视图的查询性能可能不如直接查询底层表，尤其是在视图定义复杂的情况下。
- 更新限制:视图的更新操作受到限制，不是所有的视图都可以进行更新操作。
- 存储开销:虽然视图本身不存储数据，但视图的定义会占用存储空间。


### 注意

```sql
CreatE VIEW view_name AS 子查询语句 [WITH CHECK OPTION]
```

`WITH CHECK OPTION`是指当创建后，如果更新视图中的数据，是否要满足子查询中的条件表达式，不满足将无法插入，创建后，我们就可以使用`SELECT`来直接查询视图上的数据，因此还能在视图上的基础导出其他视图

1. 若视图是由两个以上的基本表导出的，则此视图不允许更新
2. 若视图的字段来自地段表达式或常数，则不允许对此视图执行`INSERT`或`UPDATE`操作，但允许执行`DELETE`操作
3. 若视图的字段来自集函数，则此视图不允许更新
4. 若视图定义中含有`GROUP BY`或`HAVING`，则不允许更新
5. 若视图定义中含有`DISTINCT`短语，则不允许更新
6. 若视图定义中含有嵌套查询，并且内层查询的`FROM`子句中涉及的表也是导出该图的基本表，则此视图不允许更新。  
   例如将成绩在平均成绩以上的元组定义成一个视图`GOOD_SC`:

```sql
CREATE VIEW GOOD_SC AS SELECT Sno, Cno, Grade FROM SC
WHERE Grade > (SElECt AVG(Grade) FROM SC);
```

导出视图`GOOD_SC`的基本表是`SC`表，内层查询中涉及的基本表也是`SC`，所以视图`GOOD_SC`是不允许更新的

7. 一个不允许更新的视图上定义的视图也不允许更新

