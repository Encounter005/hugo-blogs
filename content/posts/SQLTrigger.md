+++
title = "SQL触发器"
tags = [ "数据库" ]
categories = [ "编程语言"]
date = "2024-09-22T17:01:34+08:00"
+++
在SQL中，触发器（Trigger）是一种特殊的存储过程，它在特定的数据库操作（如插入、更新或删除）发生时自动执行。触发器通常用于维护数据完整性、执行审计跟踪、记录日志等任务。触发器可以在表级别定义，并且可以与 `INSERT`、`UPDATE` 和 `DELETE` 操作相关联。

触发器所依附的表为基本表，当触发器表上发生`select/update/delete`操作时，会自动生成两个临时的表(`new/old`，只能由触发器使用)

比如:

- 在`insert`操作时，会生成`new`表，记录插入的数据，`old`表则为空。
- 在`delete`操作时，会生成`new`表，记录删除的数据，`old`表记录被删除的数据。
- 在`update`操作时，会生成`new`表，记录更新的数据，`old`表记录更新前的数据。

创建触发器

```sql
CREATE TRIGGER <trigger_name> [BEFORE|AFTER] [INSERT|UPDATE|DELETE] ON <table_name>/<view_name>
FOR EACH ROW [INSERT|UPDATE|DELETE ] FROM <table_name> WHERE <condition>;
```

`FOR EACH ROW` 表示针对每一行都会生效，无论哪行进行指定操作都会执行触发器

查看触发器

```sql
SHOW TRIGGERS;
```

删除触发器

```sql
DROP TRIGGER <trigger_name>;
```

### 触发器的基本概念

1. **触发器的类型**:

   - **BEFORE 触发器**: 在触发操作（如插入、更新或删除）之前执行。
   - **AFTER 触发器**: 在触发操作之后执行。
   - **INSTEAD OF 触发器**: 用于视图，替代触发操作执行。

2. **触发器的创建**:

   - 使用 `CREATE TRIGGER` 语句创建触发器。
   - 触发器可以与 `INSERT`、`UPDATE` 和 `DELETE` 操作相关联。

3. **触发器的优点**:

   - **自动执行**: 触发器在特定事件发生时自动执行，无需手动调用。
   - **维护数据完整性**: 触发器可以用于强制执行复杂的业务规则和约束。
   - **审计跟踪**: 触发器可以用于记录对数据的修改，便于审计和追踪。

4. **触发器的缺点**:
   - **性能开销**: 触发器的执行会增加数据库操作的开销，尤其是在频繁操作的情况下。
   - **复杂性**: 触发器的逻辑可能变得复杂，难以维护和调试。

### 触发器的实例

#### 实例1：插入操作触发器

假设我们有一个 `employees` 表，我们希望在每次插入新员工时，自动记录插入操作的时间。

```sql
CREATE TABLE employees (
    id INT PRIMARY KEY,
    name VARCHAR(100),
    hire_date DATETIME
);

CREATE TABLE audit_log (
    log_id INT AUTO_INCREMENT PRIMARY KEY,
    action VARCHAR(10),
    log_date DATETIME
);

DELIMITER //

CREATE TRIGGER trg_employees_insert
AFTER INSERT ON employees
FOR EACH ROW
BEGIN
    INSERT INTO audit_log (action, log_date)
    VALUES ('INSERT', NOW());
END //

DELIMITER ;
```

在这个例子中，`trg_employees_insert` 触发器在每次向 `employees` 表插入新记录时自动执行，并将插入操作记录到 `audit_log` 表中。

#### 实例2：更新操作触发器

假设我们希望在每次更新 `employees` 表中的 `hire_date` 字段时，自动记录更新前后的值。

```sql
DELIMITER //

CREATE TRIGGER trg_employees_update
BEFORE UPDATE ON employees
FOR EACH ROW
BEGIN
    INSERT INTO audit_log (action, log_date)
    VALUES ('UPDATE', NOW());
END //

DELIMITER ;
```

在这个例子中，`trg_employees_update` 触发器在每次更新 `employees` 表中的记录时自动执行，并将更新操作记录到 `audit_log` 表中。

#### 实例3：删除操作触发器

假设我们希望在每次删除 `employees` 表中的记录时，自动记录删除操作。

```sql
DELIMITER //

CREATE TRIGGER trg_employees_delete
AFTER DELETE ON employees
FOR EACH ROW
BEGIN
    INSERT INTO audit_log (action, log_date)
    VALUES ('DELETE', NOW());
END //

DELIMITER ;
```

在这个例子中，`trg_employees_delete` 触发器在每次删除 `employees` 表中的记录时自动执行，并将删除操作记录到 `audit_log` 表中。

### 触发器的使用场景

1. **数据完整性**:

   - 触发器可以用于强制执行复杂的业务规则和约束，例如在插入或更新数据时检查数据的合法性。

2. **审计跟踪**:

   - 触发器可以用于记录对数据的修改，便于审计和追踪。

3. **自动更新**:
   - 触发器可以用于自动更新相关表中的数据，例如在插入新订单时自动更新库存表。
