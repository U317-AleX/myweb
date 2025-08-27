# 📚 DBMS

## 事务隔离级别
### 事务隔离级别的对象

事务隔离级别（`READ UNCOMMITTED`、`READ COMMITTED`、`REPEATABLE READ`、`SERIALIZABLE`）主要是 **针对事务中 DML（Data Manipulation Language，如 SELECT/INSERT/UPDATE/DELETE）的并发读写行为** 而定义的。

重点在于控制 **脏读**、**不可重复读**、**幻读** 这些现象。

这些现象本质上都发生在 **数据的读写（DML）** 上。

### DDL 是否受事务隔离级别限制？

**DDL（Data Definition Language，如 CREATE TABLE/ALTER TABLE/DROP TABLE）** 的行为和 **DML** 不一样：

大多数数据库（如 MySQL、PostgreSQL）中，DDL 操作通常是 **隐式提交事务**：

* 在执行 DDL 前，会自动提交当前事务。
* 执行完 DDL 后，又会自动提交一次。

所以 DDL 往往不在用户可控的事务上下文里。

因此 DDL 一般不受事务隔离级别的限制，因为它不是在事务中“并发可见性”的问题，而是涉及到 **元数据锁（metadata lock, MDL）** 或 **模式锁（schema lock）** 来保证一致性。

例如：

* MySQL 里，`ALTER TABLE` 会获得元数据锁，阻塞其他读写该表的事务。
* PostgreSQL 里，DDL 会拿 `ACCESS EXCLUSIVE` 锁，确保没有并发的 DML 影响表结构修改。

### 例外情况

* Oracle 与 PostgreSQL 支持某些 DDL 可以在事务里回滚（称为 transactional DDL），但本质上还是通过锁和元数据一致性控制，而不是通过隔离级别。
* MySQL（InnoDB）里几乎所有 DDL 都是非事务性的（除了 `CREATE/DROP TEMPORARY TABLE` 这种小范围情况）。

---

## PostgreSQL 的隔离级别实现
PostgreSQL 的四个标准事务隔离级别（`READ UNCOMMITTED`、`READ COMMITTED`、`REPEATABLE READ`、`SERIALIZABLE`）都是通过 MVCC 和锁的组合来实现的，但具体细节与 MySQL 略有不同：

### READ UNCOMMITTED
* **MVCC**: 不使用。
* **锁**: 写操作会加锁。
* **行为**: PostgreSQL 实际上将 `READ UNCOMMITTED` **提升到了 READ COMMITTED 级别**。这意味着即使你将隔离级别设置为 `READ UNCOMMITTED`，它仍然会表现得像 `READ COMMITTED` 一样，从而避免脏读。这是一个为了保证数据完整性的设计选择。

### READ COMMITTED
* **MVCC**: 使用。每个 `SELECT` 语句都会看到一个最新的、已提交的数据版本。事务中的每个语句都会创建一个新的快照。
* **锁**: 写操作加排他锁。
* **行为**: 这是 PostgreSQL 的默认隔离级别。它通过 MVCC 确保读到的数据都是已提交的，但由于每个 `SELECT` 都会获取新快照，因此在同一个事务中多次读取同一行可能会看到不同的数据，导致“不可重复读”。

### REPEATABLE READ
* **MVCC**: 使用，且更严格。事务开始时会创建一个快照，在整个事务期间，所有读操作都只读取这个快照中的数据。
* **锁**: 写操作加排他锁。
* **行为**: 这种模式确保了事务中的“可重复读”，因为其他事务的修改在当前事务提交前是不可见的。PostgreSQL 在这个级别下可以解决幻读，因为它会锁定范围内的行（通过一种类似于间隙锁的机制），从而阻止其他事务在该范围内插入新数据。这与 MySQL 的实现略有不同。

### SERIALIZABLE
* **MVCC**: 使用，但与锁结合。
* **锁**: 读写操作都加锁。PostgreSQL 使用**快照隔离（Serializable Snapshot Isolation, SSI）**来实现这个级别。它比传统的基于锁的串行化效率更高。
* **行为**: 这是 PostgreSQL 最严格的隔离级别。它通过复杂的 SSI 算法，在不使用传统读写锁阻塞的情况下，确保事务的行为就如同串行执行一样。如果 SSI 检测到潜在的并发冲突，会回滚其中一个事务。这个机制比简单的读写加锁更精巧，因为它在保证最高隔离性的同时，尽可能地提高了并发性。