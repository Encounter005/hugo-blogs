+++
title = "SQL事务"
tags = [ "数据库" ]
categories = [ "编程语言",]
mathjax = false
date = "2024-09-22T17:28:24+08:00"
+++
在SQL中，事务（Transaction）是一组数据库操作的逻辑单元，这些操作要么全部成功执行，要么全部不执行。事务的主要目的是确保数据库的一致性和完整性，尤其是在并发访问和系统故障的情况下。

### 事务的基本概念

1. **事务的特性（ACID）**:

   - **原子性（Atomicity）**: 事务中的所有操作要么全部成功执行，要么全部不执行。如果事务中的任何一个操作失败，整个事务都会回滚到初始状态。
   - **一致性（Consistency）**: 事务执行前后，数据库必须保持一致性状态。事务不能违反数据库的完整性约束。
   - **隔离性（Isolation）**: 并发执行的多个事务之间是相互隔离的，一个事务的执行不能被其他事务干扰。
   - **持久性（Durability）**: 一旦事务成功提交，其结果将永久保存在数据库中，即使系统发生故障也不会丢失。

2. **事务的控制语句**:
   - **BEGIN TRANSACTION**: 开始一个事务。
   - **COMMIT**: 提交事务，将所有操作永久保存到数据库中。
   - **ROLLBACK**: 回滚事务，撤销所有未提交的操作。
   - **SAVEPOINT**: 在事务中设置保存点，可以在回滚时只回滚到保存点。

```sql

BEGIN; #开始事务
...
...
ROLLBACK; #回滚事务
SAVEPOINT 回滚点; #添加回滚点
ROLLBACK TO 回滚点; # 回到指定回滚点
...
COMMIT; #提交事务
-- 一旦提交，就无法进行回滚
```


### 事务的实例

假设我们有一个银行账户表 `accounts`，我们希望实现一个转账操作，从一个账户转钱到另一个账户。为了确保转账操作的原子性和一致性，我们需要使用事务。

#### 表结构

```sql
CREATE TABLE accounts (
    account_id INT PRIMARY KEY,
    balance DECIMAL(10, 2)
);
```

#### 插入初始数据

```sql
INSERT INTO accounts (account_id, balance) VALUES (1, 1000.00);
INSERT INTO accounts (account_id, balance) VALUES (2, 500.00);
```

#### 转账操作

```sql
BEGIN TRANSACTION;

-- 从账户1转出500元
UPDATE accounts SET balance = balance - 500.00 WHERE account_id = 1;

-- 向账户2转入500元
UPDATE accounts SET balance = balance + 500.00 WHERE account_id = 2;

-- 检查账户1的余额是否足够
IF (SELECT balance FROM accounts WHERE account_id = 1) < 0 THEN
    ROLLBACK;  -- 如果余额不足，回滚事务
ELSE
    COMMIT;    -- 否则，提交事务
END IF;
```

在这个例子中，我们使用事务来确保转账操作的原子性。如果账户1的余额不足，事务将回滚，撤销所有的操作；否则，事务将提交，转账操作成功完成。

### 事务的隔离级别

SQL标准定义了四种事务隔离级别，用于控制并发事务之间的相互影响：

1. **读未提交（Read Uncommitted）**: 允许一个事务读取另一个事务未提交的数据。可能会导致脏读、不可重复读和幻读。
2. **读已提交（Read Committed）**: 允许一个事务读取另一个事务已提交的数据。可以避免脏读，但可能会导致不可重复读和幻读。
3. **可重复读（Repeatable Read）**: 确保一个事务在执行期间多次读取同一数据时，结果是一致的。可以避免脏读和不可重复读，但可能会导致幻读。
4. **串行化（Serializable）**: 最高的隔离级别，确保事务串行执行，完全避免脏读、不可重复读和幻读。

#### 设置隔离级别

```sql
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;
```

### 事务的使用场景

1. **银行转账**:

   - 确保从一个账户转钱到另一个账户的操作是原子的，要么全部成功，要么全部失败。

2. **订单处理**:

   - 确保订单的创建、库存的更新和支付的处理是原子的，避免出现订单创建成功但库存未更新的情况。

3. **数据一致性**:
   - 确保多个相关操作的一致性，例如在插入新记录时，同时更新相关统计信息。

