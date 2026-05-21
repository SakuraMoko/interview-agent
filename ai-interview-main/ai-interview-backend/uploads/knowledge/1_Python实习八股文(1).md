# Python 实习八股文

---

## 一、Python 语言基础

### 1. Python 是什么语言？有哪些特点？

- 解释型语言：逐行解释执行，不需要编译
- 动态类型：变量类型在运行时确定
- 强类型：不同类型不能随意隐式转换
- 跨平台：依赖解释器即可运行在不同操作系统
- 面向对象：支持类、继承、多态等面向对象特性

### 2. 列表、元组、字典的区别？

| 特性 | 列表（list） | 元组（tuple） | 字典（dict） |
|------|-------------|--------------|-------------|
| 可变性 | 可变 | 不可变 | 可变 |
| 有序性 | 有序 | 有序 | 无序（Python 3.7+ 保持插入顺序） |
| 元素重复 | 可重复 | 可重复 | 键唯一，值可重复 |
| 适用场景 | 需要增删改的有序集合 | 写保护数据、函数返回多值 | 键值对映射、快速查找 |

列表常用删除方法：
- `list.remove(value)` — 按值删除第一个匹配项
- `list.pop(index)` — 按索引删除并返回该元素
- `del list[index]` — 按索引删除
- `list.clear()` — 清空列表

### 3. 深拷贝和浅拷贝的区别？

- `copy.copy()` — 浅拷贝：复制对象本身，但内部嵌套对象仍指向原对象（共享引用）
- `copy.deepcopy()` — 深拷贝：递归复制对象及其所有子对象，完全独立

### 4. 什么是可变对象与不可变对象？

- 可变对象：创建后内容可修改，如 `list`、`dict`、`set`
- 不可变对象：创建后内容不可修改，如 `str`、`tuple`、`int`、`float`、`frozenset`

### 5. `*args` 和 `**kwargs` 有什么作用？

- `*args`：接收任意数量的位置参数，以元组形式传入
- `**kwargs`：接收任意数量的关键字参数，以字典形式传入

### 6. Python 中如何实现装饰器？

装饰器是一个接受函数作为参数并返回新函数的高阶函数，用于在不修改原函数代码的情况下增加功能。

```python
def log_decorator(func):
    def wrapper(*args, **kwargs):
        print(f"调用函数: {func.__name__}")
        return func(*args, **kwargs)
    return wrapper

@log_decorator
def hello():
    print("Hello")
```

常见使用场景：权限校验、日志记录、缓存、性能统计、重试机制。

### 7. 什么是生成器和迭代器？

- 迭代器（Iterator）：实现了 `__iter__()` 和 `__next__()` 方法的对象，支持逐个访问元素
- 生成器（Generator）：使用 `yield` 关键字的函数，调用时返回一个迭代器，惰性计算，节省内存

### 8. Python 的内存管理与垃圾回收机制

- 引用计数：每个对象维护一个引用计数器，计数为 0 时立即回收
- 分代回收：将对象分为 0、1、2 三代，新对象在第 0 代，存活越久代数越高，回收频率越低
- 循环引用检测：引用计数无法处理循环引用，分代回收机制负责检测并清理
- 小整数池：Python 缓存 -5 到 256 的整数对象，避免重复创建
- 字符串驻留：短字符串会被缓存复用，优化内存和比较性能

### 9. Python 中的异常处理机制

使用 `try-except-finally` 捕获和处理异常：

```python
try:
    result = 10 / 0
except ZeroDivisionError as e:
    print(f"错误: {e}")
finally:
    print("无论是否异常都会执行")
```

### 10. Python 函数的特性

- 一等公民：函数可以作为变量、参数、返回值
- 闭包：内部函数引用外部作用域的变量
- 匿名函数：`lambda` 表达式，用于简化简单函数定义
- 可变参数：`*args` 和 `**kwargs`

### 11. Python 常见的性能优化手段

- I/O 密集型任务 → 多线程 或 异步编程（asyncio）
- CPU 密集型任务 → 多进程 或 C 扩展
- 性能分析工具：`cProfile`
- 函数缓存：`functools.lru_cache`


---

## 二、GIL 与并发编程

### 1. 解释 Python 的 GIL（全局解释器锁）

GIL 是 CPython 解释器中的一把全局锁，同一时刻只允许一个线程执行 Python 字节码。

- 目的：保护 Python 对象的引用计数，避免多线程并发修改导致内存问题
- 影响：多线程无法利用多核 CPU 并行执行计算任务
- 注意：GIL 是 CPython 的实现细节，Jython、PyPy 等解释器不一定有 GIL

### 2. 进程、线程、协程的区别

| 特性 | 进程（Process） | 线程（Thread） | 协程（Coroutine） |
|------|----------------|---------------|-------------------|
| 定义 | 操作系统分配资源的最小单位 | 操作系统调度执行的最小单位 | 用户态的轻量级任务调度 |
| 内存 | 独立内存空间 | 共享进程内存 | 共享线程内存 |
| 切换开销 | 大（需保存/恢复上下文） | 较小 | 极小（无内核态切换） |
| 调度方式 | 操作系统调度 | 操作系统调度 | 程序自身控制（协作式） |
| 适用场景 | CPU 密集型任务 | I/O 密集型任务 | 高并发 I/O 密集型任务 |
| Python 实现 | `multiprocessing` | `threading` | `asyncio` + `async/await` |

### 3. 多线程

- 一个进程包含多个线程时，就是在使用多线程
- 优点：并发执行多个任务，共享进程资源，线程间通信方便
- 风险：线程安全问题（多个线程同时修改共享变量），需要使用锁（`Lock`）或线程安全的数据结构

### 4. 异步与并发的区别

| 对比 | 并发（Concurrency） | 异步（Asynchronous） |
|------|--------------------|--------------------|
| 本质 | 程序的运行状态 | 编程手段 |
| 概念 | 多个任务在同一时间段交替执行 | 任务发出后不阻塞，完成后再处理结果 |
| 实现方式 | 多线程、多进程、协程 | 协程、异步回调、async/await |
| 适用场景 | 需要同时处理多个任务 | I/O 密集型任务 |

### 5. 异步为什么快？

- 协程是异步的主要载体，本质是单线程的轻量级任务调度
- 单线程上下文切换开销小，比多线程轻量
- 协程利用等待 I/O 时的空闲时间执行其他任务，实现单线程高并发
- 在 I/O 密集型场景下，异步程序性能显著优于同步程序

### 6. I/O 密集型 vs CPU 密集型

**I/O 密集型任务：**
- 程序大部分时间在等待 I/O 操作（文件读写、网络请求、数据库查询）
- CPU 空闲时间多，瓶颈在 I/O
- 解决方案：多线程 或 协程（asyncio）

**CPU 密集型任务：**
- 程序大部分时间消耗在 CPU 计算上（数学计算、图像处理、加密解密）
- CPU 利用率高，瓶颈在计算能力
- 解决方案：多进程（`multiprocessing`）或 C/C++ 扩展

### 7. 面试常见问答

**Q: Python 多线程为什么效率不高？**
A: 因为 CPython 有 GIL，同一时刻只有一个线程执行字节码，无法利用多核并行。

**Q: 怎么解决 CPU 密集型任务？**
A: 用多进程（`multiprocessing`），让多个 CPU 核心并行工作。

**Q: I/O 密集型任务怎么优化？**
A: 用多线程或协程，协程效率更高。

---

## 三、异步编程（async/await）

### 1. async + await 实现异步函数

使用 `async` 定义异步函数，`await` 等待异步操作完成。定义异步函数的依据是功能是否存在 I/O 阻塞。

```python
import asyncio

async def fetch_data():
    await asyncio.sleep(1)  # 模拟 I/O 操作
    return "数据"

async def main():
    result = await fetch_data()
    print(result)

asyncio.run(main())
```

### 2. 普通函数不要滥用 async

如果函数内部没有 I/O 操作，加上 `async` 不会提升性能，反而会因为协程调度增加额外开销。

### 3. 异步原理

- 异步运行在同一个线程内
- 高效的原因是：遇到 I/O 等待时挂起当前任务，去执行其他就绪的任务
- 事件循环（Event Loop）负责调度所有协程的执行

### 4. 只加 async 不加 await 的场景

有时函数只需要返回协程对象供外部调度，此时只加 `async` 关键字即可，不需要内部 `await`。

---

## 四、Celery 分布式任务队列

### 1. 简介

Celery 是一个分布式任务队列，用来处理异步任务和定时任务。

常见应用场景：
- 异步执行耗时操作（发送邮件、生成报表）
- 定时任务调度（每天 0 点清理数据）
- 分布式任务执行（多台机器协作）

### 2. 核心概念

| 组件 | 作用 |
|------|------|
| Broker（消息代理） | 存放任务消息，常用 Redis 或 RabbitMQ |
| Task（任务） | 用 `@app.task` 装饰器声明的 Python 函数 |
| Worker（工作进程） | 负责从 Broker 取出任务并执行，支持多机部署 |
| Result Backend（结果存储） | 存储任务执行结果（可选），如 Redis、数据库 |
| Beat（调度器） | 执行周期性任务，类似 Linux 的 crontab |

### 3. 运行流程

```
任务生产者调用 my_task.delay(...)
        ↓
Celery 将任务消息存入 Broker（如 Redis）
        ↓
Worker 监听 Broker，取出任务并执行
        ↓
执行结果存入 Result Backend（如果配置了）
```

### 4. 应用场景

- 用户注册后异步发送验证邮件
- 大文件异步处理（视频转码、数据分析）
- 周期性任务（定时清理缓存、定时统计数据）
- 分布式爬虫任务调度

### 5. 在项目中使用 Celery（推荐做法）

1. 在项目根目录创建 `celery.py`，配置 Celery 实例
2. 在 `__init__.py` 中导入 `celery_app`，确保项目启动时能识别 Celery
3. 在各模块中编写 `tasks.py`，用 `@shared_task` 定义任务
4. 在业务逻辑中用 `.delay()` 异步调用任务
5. 启动 Worker：`celery -A your_project worker -l info`
6. 启动 Beat（定时任务）：`celery -A your_project beat -l info`

---

## 五、MySQL 数据库

### 1. 什么是索引？

索引是帮助数据库高效查询数据的数据结构，类比书的目录。MySQL InnoDB 引擎默认使用 B+ 树实现索引。

**按功能分类：**

| 索引类型 | 说明 |
|---------|------|
| 主键索引（Primary Key） | 唯一且不为空，每张表只能有一个，InnoDB 中是聚簇索引 |
| 唯一索引（Unique Key） | 值必须唯一，但允许 NULL |
| 普通索引（Index） | 加速查询，无唯一性约束 |
| 复合索引（联合索引） | 多个字段组合建索引，遵循最左前缀原则 |
| 全文索引（FullText） | 大文本搜索，一般推荐用 Elasticsearch 替代 |
| 前缀索引（Prefix Index） | 对字段前 N 个字符建索引，节省空间 |

**按物理实现分类：**

| 类型 | 说明 |
|------|------|
| 聚簇索引 | 数据和索引存在一起，InnoDB 主键索引就是聚簇索引，叶子节点存放整行数据 |
| 非聚簇索引（二级索引） | 索引和数据分开存储，叶子节点存放主键 ID，查到后需要回表查完整数据 |

### 2. 最左前缀原则

复合索引 `(a, b, c)` 只有查询条件从最左列开始连续匹配时，索引才会生效：

| 查询条件 | 是否走索引 |
|---------|-----------|
| `WHERE a = 1` | ✅ |
| `WHERE a = 1 AND b = 2` | ✅ |
| `WHERE a = 1 AND b = 2 AND c = 3` | ✅ |
| `WHERE b = 2` | ❌（跳过了 a） |
| `WHERE b = 2 AND c = 3` | ❌（跳过了 a） |

### 3. 索引什么时候失效？

- 对索引列使用函数或计算：`WHERE YEAR(create_time) = 2024`
- 前缀模糊查询：`WHERE name LIKE '%张%'`（`LIKE '张%'` 可以走索引）
- 隐式类型转换：字段是 varchar，查询用数字 `WHERE phone = 13800138000`
- OR 条件中有一侧没有索引
- 使用 `!=` 或 `NOT IN`

### 4. 覆盖索引

查询的字段全部包含在索引中，不需要回表查数据，性能最优。

```sql
-- 假设有复合索引 (name, age)
SELECT name, age FROM users WHERE name = '张三';
-- 索引中已包含 name 和 age，无需回表
```

### 5. 为什么用 B+ 树而不是 B 树或哈希？

- B+ 树叶子节点形成有序链表，支持范围查询（`BETWEEN`、`>`、`<`）
- B+ 树非叶子节点只存索引不存数据，单个节点能存更多索引项，树更矮，磁盘 IO 更少
- 哈希索引只支持等值查询，不支持范围查询和排序

### 6. 事务与 ACID

事务是数据库操作的最小执行单元，一组操作要么全部成功，要么全部回滚。

| 特性 | 说明 | 实现机制 |
|------|------|---------|
| 原子性（Atomicity） | 事务不可分割，要么全做要么全不做 | undo log（回滚日志） |
| 一致性（Consistency） | 事务前后数据保持一致 | 由其他三个特性共同保证 |
| 隔离性（Isolation） | 不同事务之间互不干扰 | 锁 + MVCC |
| 持久性（Durability） | 事务提交后数据不会丢失 | redo log（重做日志） |

### 7. 事务隔离级别

| 隔离级别 | 脏读 | 不可重复读 | 幻读 |
|---------|------|-----------|------|
| READ UNCOMMITTED（读未提交） | ✅ 可能 | ✅ 可能 | ✅ 可能 |
| READ COMMITTED（读已提交） | ❌ 解决 | ✅ 可能 | ✅ 可能 |
| REPEATABLE READ（可重复读） | ❌ 解决 | ❌ 解决 | ✅ 可能 |
| SERIALIZABLE（串行化） | ❌ 解决 | ❌ 解决 | ❌ 解决 |

- MySQL InnoDB 默认隔离级别：REPEATABLE READ
- 脏读：读到其他事务未提交的数据
- 不可重复读：同一事务内多次查询同一数据，结果不同（别人修改了）
- 幻读：同一事务内多次查询，结果集行数不同（别人插入/删除了）

### 8. MySQL 锁机制

**按粒度分：**

| 锁类型 | 说明 |
|--------|------|
| 表级锁 | 锁住整张表，开销小但并发度低，MyISAM 使用 |
| 行级锁 | 锁住某一行，并发度高但开销大，InnoDB 使用（基于索引实现） |

> InnoDB 行级锁是基于索引实现的，如果查询没有走索引，会退化为表锁。

**按用途分：**

| 锁类型 | 说明 |
|--------|------|
| 共享锁（S 锁 / 读锁） | 多个事务可以同时读，但不能写。`SELECT ... LOCK IN SHARE MODE` |
| 排他锁（X 锁 / 写锁） | 获得锁后，其他事务不能读也不能写。`SELECT ... FOR UPDATE` |

**InnoDB 行锁细分：**

| 锁类型 | 说明 |
|--------|------|
| 记录锁（Record Lock） | 锁定某一行 |
| 间隙锁（Gap Lock） | 锁定一个范围（不含已有记录），防止幻读 |
| 临键锁（Next-Key Lock） | 记录锁 + 间隙锁，InnoDB 默认加的锁类型 |

### 9. 死锁

两个事务互相持有对方需要的锁，导致都无法继续。

**解决方式：**
- 设置锁等待超时：`innodb_lock_wait_timeout`
- 按固定顺序访问资源，避免交叉加锁
- 拆分大事务，减少锁持有时间

### 10. SQL 优化

**常见优化手段：**
- 合理建索引，使用联合索引和覆盖索引
- 用 `EXPLAIN` 分析执行计划，关注 type、key、rows 字段
- 避免 `SELECT *`，只查需要的字段
- 深分页优化：用 `WHERE id > 上次最后ID LIMIT N` 代替 `OFFSET`
- 避免在 WHERE 中对字段使用函数
- 复杂子查询改写为 JOIN
- 减少长事务，避免锁冲突

**EXPLAIN 关键字段：**

| 字段 | 说明 |
|------|------|
| type | 访问类型，从好到差：const > eq_ref > ref > range > index > ALL |
| key | 实际使用的索引 |
| rows | 预估扫描行数，越少越好 |
| Extra | 额外信息，出现 `Using filesort` 或 `Using temporary` 需要优化 |

### 11. 千万级数据分页

```sql
-- 错误做法：OFFSET 越大越慢
SELECT * FROM orders LIMIT 10 OFFSET 9999990;

-- 正确做法：基于索引列的范围查询
SELECT * FROM orders WHERE id > 9999990 LIMIT 10;
```

### 12. 内连接与外连接

| 连接类型 | 说明 |
|---------|------|
| INNER JOIN（内连接） | 只返回两个表中匹配的记录 |
| LEFT JOIN（左连接） | 返回左表所有记录，右表无匹配则为 NULL |
| RIGHT JOIN（右连接） | 返回右表所有记录，左表无匹配则为 NULL |
| FULL OUTER JOIN（全外连接） | 左连接 + 右连接的并集（MySQL 不直接支持，需用 UNION 模拟） |

### 13. ORM 的优缺点与 N+1 问题

**优点：** 提高开发效率，代码可读性好，自动防 SQL 注入
**缺点：** 复杂查询性能可能不如原生 SQL，有学习成本

**N+1 问题：** 查询 N 条主记录后，每条记录又单独查一次关联表，共 N+1 次查询。

**解决方式（以 Django ORM 为例）：**
- `select_related()`：一次 JOIN 查询解决外键关联（一对一、多对一）
- `prefetch_related()`：额外一次查询预加载关联数据（多对多、一对多）

**SQLAlchemy 中：**
- `joinedload()`：等价于 `select_related`
- `subqueryload()`：等价于 `prefetch_related`

### 14. 如何防止 SQL 注入？

- 使用参数化查询（占位符），不要拼接 SQL 字符串
- 使用 ORM 框架（自动参数化）
- 对用户输入做校验和过滤

```python
# 危险：字符串拼接
cursor.execute(f"SELECT * FROM users WHERE id = {user_id}")

# 安全：参数化查询
cursor.execute("SELECT * FROM users WHERE id = %s", (user_id,))
```

### 15. InnoDB vs MyISAM

| 特性 | InnoDB | MyISAM |
|------|--------|--------|
| 事务支持 | ✅ 支持 | ❌ 不支持 |
| 行级锁 | ✅ 支持 | ❌ 只有表锁 |
| 外键 | ✅ 支持 | ❌ 不支持 |
| 崩溃恢复 | ✅ 支持（redo log） | ❌ 不支持 |
| 全文索引 | ✅ 支持（5.6+） | ✅ 支持 |
| 适用场景 | 高并发读写、事务场景 | 读多写少、不需要事务 |

> MySQL 5.5+ 默认存储引擎是 InnoDB。

### 16. CHAR vs VARCHAR

| 特性 | CHAR | VARCHAR |
|------|------|---------|
| 存储方式 | 固定长度，不足补空格 | 可变长度，按实际长度存储 |
| 性能 | 读取稍快（长度固定） | 节省空间 |
| 适用场景 | 长度固定的数据（如手机号、MD5） | 长度不固定的数据（如用户名、地址） |

### 17. MySQL 常见面试问答

**Q: MySQL 索引的底层实现？**
A: InnoDB 使用 B+ 树，叶子节点存放数据（聚簇索引）或主键 ID（非聚簇索引）。

**Q: 聚簇索引和非聚簇索引的区别？**
A: 聚簇索引叶子节点存整行数据，非聚簇索引存主键 ID，查到后需要回表。

**Q: InnoDB 默认的锁是什么？**
A: 行级锁，基于索引实现。无索引时退化为表锁。

**Q: 什么是 MVCC？**
A: 多版本并发控制，通过保存数据的历史版本，让读操作不加锁就能实现事务隔离，提高并发性能。InnoDB 在 READ COMMITTED 和 REPEATABLE READ 级别下使用 MVCC。

**Q: 数据库三大范式？**
A: 1NF（字段不可再分）、2NF（非主键字段完全依赖主键）、3NF（非主键字段不传递依赖主键）。实际开发中为了查询性能，会适当反范式（冗余字段）。


---

## 六、Redis

### 1. Redis 是什么？

Redis 是基于内存的高性能键值数据库，常用作缓存、消息队列、分布式锁等。单线程模型（6.0 之前），通过 IO 多路复用实现高并发，读写速度极快（10 万+ QPS）。

### 2. 数据结构与应用场景

| 数据结构 | 说明 | 典型场景 |
|---------|------|---------|
| String | 最基本类型，可存字符串、整数、浮点数 | 缓存 JSON、计数器（点赞数、阅读量）、分布式锁 |
| Hash | 键值对集合，类似 Python 字典 | 存储用户信息、Session、对象属性 |
| List | 有序列表，支持两端操作 | 消息队列（LPUSH + RPOP）、最新消息列表 |
| Set | 无序集合，元素唯一 | 去重、共同关注、标签系统 |
| Sorted Set（ZSet） | 有序集合，每个元素有一个 score | 排行榜、延时队列、带权重的任务调度 |

### 3. 缓存穿透、缓存击穿、缓存雪崩

| 问题 | 描述 | 解决方案 |
|------|------|---------|
| 缓存穿透 | 请求的数据缓存和数据库中都不存在，每次都打到数据库 | 布隆过滤器拦截不存在的 key；缓存空值（设短过期时间） |
| 缓存击穿 | 某个热点 key 突然过期，大量请求瞬间打到数据库 | 互斥锁（只让一个请求查库并回填）；热点数据永不过期 |
| 缓存雪崩 | 大量 key 同时过期，请求全部涌向数据库 | 过期时间加随机值，避免同时失效；多级缓存；限流降级 |

> 记忆技巧：穿透 = 数据根本不存在；击穿 = 单个热点 key 过期；雪崩 = 大批 key 同时过期。

**布隆过滤器：** 一种概率型数据结构，用于快速判断元素"是否可能存在"。空间效率高、查询速度快，但有一定误判率（可能误判为存在，但不会误判为不存在）。

### 4. 持久化机制：RDB vs AOF

| 对比 | RDB（快照） | AOF（追加日志） |
|------|-----------|----------------|
| 原理 | 定期将内存数据快照写入磁盘 | 每次写操作追加到日志文件 |
| 文件大小 | 小（二进制压缩） | 大（记录所有写命令） |
| 恢复速度 | 快 | 慢（需要重放命令） |
| 数据安全 | 可能丢失最后一次快照后的数据 | 最多丢 1 秒数据（配置 everysec） |
| 适用场景 | 备份、灾难恢复 | 数据安全要求高的场景 |

> 生产环境通常 RDB + AOF 结合使用。Redis 4.0+ 支持混合持久化（RDB 格式 + AOF 增量）。

### 5. 缓存与数据库一致性

**Cache Aside 模式（最常用）：**
- 读：先查缓存 → 缓存命中直接返回 → 未命中则查数据库 → 写入缓存
- 写：先更新数据库 → 再删除缓存

**为什么是"删除缓存"而不是"更新缓存"？**
避免并发下脏数据覆盖。两个请求同时修改数据，如果都去更新缓存，后写数据库的可能先更新了缓存，导致缓存中是旧值。删除缓存让下次读时重新从数据库加载，保证最终一致。

**进一步保障：**
- 延迟双删：更新数据库后删缓存，延迟几百毫秒再删一次
- 缓存设置过期时间：兜底策略，即使删除失败，过期后也会重新加载
- 消息队列：通过 MQ 异步删除缓存，保证最终一致性

### 6. 分布式锁

通过 `SETNX`（SET if Not eXists）+ 过期时间实现：

```bash
# 加锁：key 不存在时设置成功，同时设置过期时间防止死锁
SET lock_key unique_value NX EX 30

# 解锁：用 Lua 脚本保证原子性（先判断值是否是自己的，再删除）
if redis.call("get", KEYS[1]) == ARGV[1] then
    return redis.call("del", KEYS[1])
end
```

**注意事项：**
- 必须设置过期时间，防止持有锁的进程崩溃导致死锁
- 解锁时要验证是不是自己加的锁（用唯一值标识）
- 生产环境可用 Redisson（Java）或 redis-py 的 Lock 封装

### 7. 高并发下的并发控制

| 方案 | 说明 |
|------|------|
| 原子命令 | 使用 `INCR`、`DECR`、`SETNX` 等原子操作，避免读-改-写的竞态 |
| 分布式锁 | `SETNX + EXPIRE`，同一时间只有一个进程操作 |
| Lua 脚本 | 多个操作封装到 Lua 脚本中，Redis 保证脚本执行的原子性 |
| WATCH + 事务 | 乐观锁机制，WATCH 监视 key，key 被修改则事务失败重试 |
| 队列化 | 请求放入 List/Stream，单一消费者顺序处理 |

### 8. 大 Key 优化

当某个 key 对应的数据过大（几十 MB），会导致 Redis 和网络压力。

**优化方案：**
- 拆分：将大 key 拆成多个小 key（如按字段拆分 Hash）
- 精简：只缓存热点字段，去掉冗余数据
- 压缩：存储前用 gzip/zlib 压缩
- 异步删除：大 key 删除用 `UNLINK`（异步）代替 `DEL`（同步阻塞）

### 9. 过期策略与内存淘汰

**过期删除策略：**
- 惰性删除：访问 key 时才检查是否过期，过期则删除
- 定期删除：Redis 定时随机抽查一批 key，删除已过期的

**内存淘汰策略（内存满时）：**

| 策略 | 说明 |
|------|------|
| noeviction | 不淘汰，写入报错（默认） |
| allkeys-lru | 从所有 key 中淘汰最近最少使用的 |
| volatile-lru | 从设置了过期时间的 key 中淘汰 LRU |
| allkeys-random | 随机淘汰 |
| volatile-ttl | 淘汰剩余过期时间最短的 key |

> 缓存场景推荐 `allkeys-lru`。

### 10. Redis 为什么快？

- 纯内存操作，读写不涉及磁盘 IO
- 单线程模型，避免了线程切换和锁竞争的开销
- IO 多路复用（epoll），单线程高效处理大量并发连接
- 高效的数据结构（跳表、压缩列表、哈希表等底层实现）

### 11. Redis 常见面试问答

**Q: Redis 是单线程还是多线程？**
A: 核心命令执行是单线程。Redis 6.0 引入了多线程 IO（网络读写），但命令执行仍然是单线程，保证了线程安全。

**Q: Redis 和 Memcached 的区别？**
A: Redis 支持丰富的数据结构（String/Hash/List/Set/ZSet），支持持久化和主从复制；Memcached 只支持简单的 key-value，纯内存无持久化，但多线程模型在简单缓存场景下吞吐量高。

**Q: 如何用 Redis 实现排行榜？**
A: 用 Sorted Set，`ZADD` 添加成员和分数，`ZREVRANGE` 按分数从高到低获取排名。

**Q: 如何用 Redis 实现限流？**
A: 简单方案：用 String 计数器 + `EXPIRE` 过期时间，每次请求 `INCR`，超过阈值则拒绝。进阶方案：滑动窗口（Sorted Set）或令牌桶算法。


---

## 七、计算机网络

### 1. OSI 七层模型

| 层级 | 名称 | 作用 | 常用协议/技术 |
|------|------|------|-------------|
| 7 | 应用层 | 为应用程序提供网络服务 | HTTP、HTTPS、FTP、SMTP、DNS |
| 6 | 表示层 | 数据格式转换、加密解密 | SSL/TLS |
| 5 | 会话层 | 建立、管理、终止会话 | RPC |
| 4 | 传输层 | 提供端到端的可靠或不可靠传输 | TCP、UDP |
| 3 | 网络层 | 路由选择、逻辑寻址 | IP、ICMP |
| 2 | 数据链路层 | 物理地址寻址、帧传输 | MAC、PPP |
| 1 | 物理层 | 比特流的物理传输 | 网线、光纤、Wi-Fi |

> 面试中更常问的是 TCP/IP 四层模型：应用层、传输层、网络层、网络接口层。OSI 是理论模型，TCP/IP 是实际使用的模型。

### 2. TCP vs UDP

| 对比 | TCP | UDP |
|------|-----|-----|
| 连接方式 | 面向连接（三次握手） | 无连接 |
| 可靠性 | 可靠传输，保证顺序和完整性 | 不保证可靠性、顺序和完整性 |
| 传输方式 | 面向字节流 | 面向报文，每个数据包独立 |
| 速度 | 较慢（有确认、重传、拥塞控制） | 快（无握手、无确认） |
| 流量/拥塞控制 | 有 | 无 |
| 适用场景 | HTTP/HTTPS、FTP、SMTP | DNS 查询、视频直播、在线游戏、VoIP |

### 3. TCP 三次握手

目的：建立可靠连接，双方确认发送和接收能力。

```
客户端                    服务器
  |--- SYN (seq=x) ------->|    第一次：客户端发起连接请求
  |<-- SYN+ACK (seq=y, ack=x+1) --|    第二次：服务器确认并回复
  |--- ACK (ack=y+1) ----->|    第三次：客户端确认
  |      连接建立，开始传输数据      |
```

**为什么是三次而不是两次？**
防止已失效的连接请求到达服务器。如果只有两次握手，客户端发的一个延迟请求到达服务器，服务器会误以为是新连接并分配资源，造成浪费。第三次握手让服务器确认客户端确实要建立连接。

### 4. TCP 四次挥手

目的：双方确认数据传输完毕，安全关闭连接。

```
客户端                    服务器
  |--- FIN (seq=u) ------->|    第一次：客户端请求关闭
  |<-- ACK (ack=u+1) ------|    第二次：服务器确认收到
  |    （服务器可能还有数据要发）    |
  |<-- FIN (seq=w) --------|    第三次：服务器也请求关闭
  |--- ACK (ack=w+1) ----->|    第四次：客户端确认
  |      连接关闭                  |
```

**为什么是四次而不是三次？**
因为 TCP 是全双工的，双方都可以发送数据。客户端说"我发完了"，服务器可能还没发完，所以服务器先回复"收到"，等自己数据发完后再发 FIN。

### 5. HTTP 协议

**基本概念：**
- 应用层协议，用于客户端和服务器之间传输数据（HTML、JSON、图片等）
- 基于 TCP 传输，默认端口 80（HTTPS 为 443）
- 无状态协议：每次请求独立，服务器不记住之前的请求

**HTTP 请求结构：**

```
请求行：GET /api/users HTTP/1.1
请求头：Host、User-Agent、Cookie、Content-Type、Accept
请求体：POST/PUT 请求携带的数据（JSON、表单等）
```

**HTTP 响应结构：**

```
状态行：HTTP/1.1 200 OK
响应头：Set-Cookie、Cache-Control、Content-Type、Content-Length
响应体：HTML、JSON、文件等
```

### 6. HTTP 状态码

| 分类 | 含义 | 常见状态码 |
|------|------|-----------|
| 1xx | 信息性 | 100 Continue |
| 2xx | 成功 | 200 OK、201 Created、204 No Content |
| 3xx | 重定向 | 301 永久重定向、302 临时重定向、304 未修改（缓存） |
| 4xx | 客户端错误 | 400 Bad Request、401 未认证、403 禁止访问、404 未找到、405 方法不允许 |
| 5xx | 服务器错误 | 500 Internal Server Error、502 Bad Gateway、503 Service Unavailable |

### 7. HTTP 请求方法

| 方法 | 用途 | 幂等性 |
|------|------|--------|
| GET | 获取资源 | 幂等 |
| POST | 创建资源 | 非幂等 |
| PUT | 更新资源（全量替换） | 幂等 |
| PATCH | 更新资源（部分修改） | 非幂等 |
| DELETE | 删除资源 | 幂等 |

> 幂等性：同一个请求执行多次，结果和执行一次相同。

### 8. HTTP vs HTTPS

| 对比 | HTTP | HTTPS |
|------|------|-------|
| 端口 | 80 | 443 |
| 安全性 | 明文传输，不安全 | SSL/TLS 加密，安全 |
| 证书 | 不需要 | 需要 CA 证书 |
| 性能 | 快 | 稍慢（加密解密开销） |

**HTTPS 建立连接过程（简化）：**
1. 客户端发起 HTTPS 请求
2. 服务器返回 SSL 证书（包含公钥）
3. 客户端验证证书合法性
4. 客户端生成随机密钥，用公钥加密后发给服务器
5. 服务器用私钥解密，得到随机密钥
6. 双方用该随机密钥进行对称加密通信

### 9. HTTP/1.1 vs HTTP/2 vs HTTP/3

| 特性 | HTTP/1.1 | HTTP/2 | HTTP/3 |
|------|----------|--------|--------|
| 传输层 | TCP | TCP | QUIC（基于 UDP） |
| 多路复用 | ❌（队头阻塞） | ✅ | ✅ |
| 头部压缩 | ❌ | ✅（HPACK） | ✅（QPACK） |
| 服务器推送 | ❌ | ✅ | ✅ |
| 连接建立 | TCP 三次握手 + TLS | TCP + TLS | 0-RTT / 1-RTT |

### 10. Cookie、Session、Token 的区别

| 对比 | Cookie | Session | Token（JWT） |
|------|--------|---------|-------------|
| 存储位置 | 客户端（浏览器） | 服务器端 | 客户端 |
| 安全性 | 较低（可被篡改） | 较高 | 较高（签名验证） |
| 服务器压力 | 无 | 需要存储 Session 数据 | 无（无状态） |
| 跨域 | 受同源策略限制 | 受限 | 支持跨域 |
| 适用场景 | 记住登录状态 | 传统 Web 应用 | 前后端分离、API 认证 |

### 11. GET vs POST

| 对比 | GET | POST |
|------|-----|------|
| 参数位置 | URL 中（查询字符串） | 请求体中 |
| 数据长度 | 受 URL 长度限制（约 2KB） | 无限制 |
| 安全性 | 参数暴露在 URL 中 | 相对安全（参数在请求体） |
| 缓存 | 可被缓存 | 不会被缓存 |
| 幂等性 | 幂等 | 非幂等 |
| 用途 | 获取数据 | 提交数据 |

### 12. DNS 解析过程

域名 → IP 地址的转换过程：

```
1. 浏览器缓存 → 有则直接返回
2. 操作系统缓存（hosts 文件） → 有则返回
3. 本地 DNS 服务器（运营商提供） → 有则返回
4. 根域名服务器 → 返回顶级域名服务器地址
5. 顶级域名服务器（.com） → 返回权威域名服务器地址
6. 权威域名服务器 → 返回最终 IP 地址
```

### 13. 从输入 URL 到页面显示的过程

这是面试超高频题，完整流程：

1. DNS 解析：域名 → IP 地址
2. TCP 连接：三次握手建立连接
3. TLS 握手（如果是 HTTPS）
4. 发送 HTTP 请求
5. 服务器处理请求，返回 HTTP 响应
6. 浏览器解析 HTML，构建 DOM 树
7. 加载 CSS/JS 等资源
8. 渲染页面
9. TCP 四次挥手关闭连接

### 14. WebSocket

- 全双工通信协议，客户端和服务器可以互相主动发送消息
- 基于 TCP，通过 HTTP 升级握手建立连接
- 适用场景：实时聊天、在线游戏、股票行情、协同编辑
- 与 HTTP 的区别：HTTP 是请求-响应模式（客户端主动），WebSocket 是双向通信

### 15. 网络调试常用命令

| 命令 | 作用 | 示例 |
|------|------|------|
| `ping` | 测试网络连通性 | `ping www.baidu.com` |
| `nslookup` | 查询 DNS 解析 | `nslookup www.baidu.com` |
| `netstat` | 查看端口占用 | `netstat -ano \| findstr 80` |
| `lsof` | 查看端口占用（Linux） | `lsof -i:3306` |
| `telnet` | 测试远程端口连通性 | `telnet www.baidu.com 80` |
| `curl` | 测试 HTTP 请求 | `curl -I http://www.baidu.com` |

### 16. 网站无法访问的排查步骤

```
1. ping 目标地址 → 检查网络连通性
2. nslookup 域名 → 检查 DNS 解析是否正常
3. telnet/netstat 检查端口 → 确认服务端口是否开放
4. curl 测试 HTTP 请求 → 确认 Web 服务是否正常响应
5. 查看服务器日志 → 定位具体错误原因
```

### 17. 计算机网络常见面试问答

**Q: TCP 和 HTTP 是什么关系？**
A: HTTP 是应用层协议，定义数据交换的格式；TCP 是传输层协议，负责可靠传输数据。HTTP 的请求和响应都通过 TCP 连接发送。

**Q: 什么是跨域？怎么解决？**
A: 浏览器的同源策略限制不同域名/端口/协议之间的请求。解决方案：CORS（服务器设置响应头 `Access-Control-Allow-Origin`）、代理转发（Nginx/Vite proxy）。

**Q: 什么是 RESTful API？**
A: 一种 API 设计风格，用 URL 表示资源，用 HTTP 方法表示操作（GET 获取、POST 创建、PUT 更新、DELETE 删除），返回标准 HTTP 状态码。

**Q: 什么是 SSE（Server-Sent Events）？**
A: 服务器向客户端单向推送数据的技术，基于 HTTP 长连接。适合实时通知、AI 流式输出等场景。与 WebSocket 的区别是 SSE 只能服务器向客户端推送，WebSocket 是双向通信。


---

## 八、Web 框架（Django / Flask / FastAPI）

### 1. 三大框架对比

| 对比 | Django | Flask | FastAPI |
|------|--------|-------|---------|
| 类型 | 重量级全栈框架 | 轻量级微框架 | 现代异步框架 |
| 异步支持 | 3.1+ 部分支持 | 不原生支持 | 原生 async/await |
| ORM | 内置 Django ORM | 无（常用 SQLAlchemy） | 无（常用 SQLAlchemy） |
| Admin 后台 | 内置 | 无 | 无 |
| API 文档 | 需要 DRF + drf-yasg | 需要 Flask-RESTful | 自动生成 Swagger/ReDoc |
| 学习曲线 | 陡峭（功能多） | 平缓（简单灵活） | 中等（需要类型提示基础） |
| 性能 | 一般 | 一般 | 高（异步 + Starlette） |
| 适用场景 | 大型 Web 应用、CMS、企业级项目 | 小型 API、微服务、原型开发 | 高性能 API、微服务、AI 应用 |

### 2. Django

#### 2.1 MVT 模式

| 层 | 职责 |
|----|------|
| Model（模型层） | 定义数据模型，负责与数据库交互 |
| View（视图层） | 处理业务逻辑，接收请求、返回响应 |
| Template（模板层） | 页面展示，将数据渲染为 HTML |

> 与 MVC 的对应关系：Model = Model，View = Controller，Template = View。

#### 2.2 Django ORM

ORM 将 Python 类映射为数据库表，通过查询接口（`.filter()`、`.get()`、`.create()`）操作数据，底层自动生成 SQL。

**常见优化：**
- N+1 问题：用 `select_related()`（外键 JOIN）和 `prefetch_related()`（预加载）解决
- 慢查询：加索引、用 `EXPLAIN` 分析执行计划、避免 `SELECT *`
- 批量操作：用 `bulk_create()` / `bulk_update()` 减少数据库交互次数

#### 2.3 中间件

中间件是请求和响应的钩子，在视图处理前后执行。

```python
class CustomMiddleware:
    def __init__(self, get_response):
        self.get_response = get_response

    def __call__(self, request):
        # 请求到达视图之前执行
        print("Before view")
        response = self.get_response(request)
        # 视图返回响应之后执行
        print("After view")
        return response
```

**常见内置中间件：**
- `AuthenticationMiddleware`：用户认证
- `SessionMiddleware`：会话管理
- `CsrfViewMiddleware`：CSRF 防护
- `CorsMiddleware`：跨域处理（第三方 django-cors-headers）

#### 2.4 认证与权限

Django 内置认证系统（`django.contrib.auth`）包含两部分：

- 认证（Authentication）：确认用户身份，核心方法 `authenticate()`、`login()`、`logout()`
- 授权（Authorization）：确认用户权限，基于 User、Group、Permission 模型

```python
from django.contrib.auth.decorators import login_required, permission_required

@login_required  # 要求登录
def dashboard(request):
    pass

@permission_required('app.can_edit')  # 要求特定权限
def edit_post(request):
    pass
```

> 支持自定义用户模型（继承 `AbstractUser`）和自定义认证后端。

#### 2.5 CSRF 防护

- 通过 `CsrfViewMiddleware` 中间件实现
- 服务器生成 CSRF Token，嵌入表单或 Cookie 中
- 提交请求时携带 Token，服务器验证一致性，防止跨站请求伪造
- 前后端分离项目中，API 通常用 JWT 认证，可以豁免 CSRF

#### 2.6 文件上传

- 模型中使用 `FileField` 或 `ImageField`
- 配置 `MEDIA_ROOT`（存储路径）和 `MEDIA_URL`（访问路径）
- 上传文件通过 `request.FILES` 获取

### 3. Django REST Framework（DRF）

#### 3.1 序列化器（Serializer）

用于数据校验和格式转换（Python 对象 ↔ JSON）：

```python
from rest_framework import serializers

class UserSerializer(serializers.ModelSerializer):
    class Meta:
        model = User
        fields = ['id', 'username', 'email']
```

- `Serializer`：手动定义字段，灵活度高
- `ModelSerializer`：根据模型自动生成字段，减少重复代码

#### 3.2 视图类型

| 类型 | 说明 |
|------|------|
| APIView | 最基础的视图类，手动处理 GET/POST 等方法 |
| GenericAPIView | 提供通用的增删改查逻辑（配合 Mixin 使用） |
| ViewSet | 组合多种操作，一次定义自动生成多个路由 |
| ModelViewSet | 最强大，自动提供完整的 CRUD 接口 |

#### 3.3 DRF 认证与权限

```python
from rest_framework.permissions import IsAuthenticated
from rest_framework.authentication import TokenAuthentication

class UserView(APIView):
    authentication_classes = [TokenAuthentication]
    permission_classes = [IsAuthenticated]
```

常用权限类：
- `IsAuthenticated`：已登录用户
- `IsAdminUser`：管理员
- `AllowAny`：任何人
- 自定义权限：继承 `BasePermission`，重写 `has_permission()`

### 4. Flask

#### 4.1 核心特点

- 微框架，核心只有路由和模板引擎（Jinja2），其他功能通过扩展实现
- 灵活自由，不强制项目结构
- 适合小型项目、API 服务、快速原型开发

#### 4.2 基本用法

```python
from flask import Flask, request, jsonify

app = Flask(__name__)

@app.route('/api/users', methods=['GET'])
def get_users():
    return jsonify({'users': []})

@app.route('/api/users', methods=['POST'])
def create_user():
    data = request.get_json()
    return jsonify({'message': 'created'}), 201
```

#### 4.3 常用扩展

| 扩展 | 作用 |
|------|------|
| Flask-SQLAlchemy | ORM 数据库操作 |
| Flask-Migrate | 数据库迁移（基于 Alembic） |
| Flask-RESTful | 构建 RESTful API |
| Flask-JWT-Extended | JWT 认证 |
| Flask-CORS | 跨域处理 |
| Flask-Login | 用户登录管理 |
| Flask-WTF | 表单验证 |

#### 4.4 蓝图（Blueprint）

用于模块化组织大型项目，类似 Django 的 app：

```python
from flask import Blueprint

user_bp = Blueprint('user', __name__, url_prefix='/api/users')

@user_bp.route('/')
def get_users():
    return jsonify({'users': []})

# 在主应用中注册
app.register_blueprint(user_bp)
```

#### 4.5 Flask 上下文

| 上下文 | 说明 |
|--------|------|
| 应用上下文（`current_app`、`g`） | 存储应用级别的数据，如配置、数据库连接 |
| 请求上下文（`request`、`session`） | 存储当前请求的数据 |

> Flask 的上下文是面试常考点，理解"为什么 `request` 不需要传参就能在函数中使用"——因为 Flask 用线程局部变量（`LocalProxy`）实现了请求上下文。

### 5. FastAPI

#### 5.1 核心特点

- 原生异步支持（async/await）
- 基于 Python 类型提示自动校验请求参数
- 自动生成 Swagger UI 和 ReDoc API 文档
- 基于 Starlette（Web 框架）和 Pydantic（数据校验）
- 性能接近 Node.js 和 Go

#### 5.2 基本用法

```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class UserCreate(BaseModel):
    username: str
    email: str

@app.get("/api/users")
async def get_users():
    return {"users": []}

@app.post("/api/users", status_code=201)
async def create_user(user: UserCreate):
    return {"message": "created", "user": user.dict()}
```

#### 5.3 依赖注入

FastAPI 的核心设计模式，用于复用逻辑（数据库连接、认证、权限等）：

```python
from fastapi import Depends
from sqlalchemy.ext.asyncio import AsyncSession

async def get_db():
    async with SessionLocal() as session:
        yield session

@app.get("/api/users")
async def get_users(db: AsyncSession = Depends(get_db)):
    # db 由依赖注入自动提供
    users = await db.execute(select(User))
    return users.scalars().all()
```

#### 5.4 Pydantic 数据校验

```python
from pydantic import BaseModel, EmailStr, Field

class UserCreate(BaseModel):
    username: str = Field(..., min_length=2, max_length=50)
    email: EmailStr
    age: int = Field(..., ge=0, le=150)
```

- 自动校验请求数据，不合法直接返回 422 错误
- 自动生成 API 文档中的请求/响应模型

#### 5.5 异步数据库操作

```python
from sqlalchemy.ext.asyncio import create_async_engine, AsyncSession

# 异步引擎
engine = create_async_engine("postgresql+asyncpg://user:pass@localhost/db")

# 异步查询
async def get_user(db: AsyncSession, user_id: int):
    result = await db.execute(select(User).where(User.id == user_id))
    return result.scalar_one_or_none()
```

> FastAPI + SQLAlchemy async + asyncpg 是 Python 异步后端的主流技术栈。

### 6. 框架选择面试问答

**Q: Django 和 FastAPI 怎么选？**
A: 需要快速开发完整 Web 应用（带 Admin、模板、认证）选 Django；需要高性能异步 API 选 FastAPI。前后端分离项目推荐 FastAPI。

**Q: Flask 和 FastAPI 怎么选？**
A: 两者都是轻量级框架。Flask 生态成熟、社区资源多；FastAPI 性能更好、自带数据校验和文档生成。新项目推荐 FastAPI。

**Q: 什么是 WSGI 和 ASGI？**
A: WSGI（Web Server Gateway Interface）是 Python 同步 Web 应用的标准接口，Django 和 Flask 使用。ASGI（Asynchronous Server Gateway Interface）是异步版本，FastAPI 使用。ASGI 支持 WebSocket 和长连接。

**Q: 什么是 ORM 的懒加载？**
A: ORM 默认不会立即执行 SQL，而是在真正访问数据时才查询数据库。好处是避免不必要的查询，坏处是可能导致 N+1 问题。


---

## 九、Git 版本控制

### 1. 常用命令

| 命令 | 作用 |
|------|------|
| `git init` | 初始化仓库 |
| `git clone <url>` | 克隆远程仓库 |
| `git add .` | 暂存所有修改 |
| `git commit -m "msg"` | 提交到本地仓库 |
| `git push origin main` | 推送到远程仓库 |
| `git pull origin main` | 拉取远程最新代码并合并 |
| `git fetch` | 拉取远程代码但不合并 |
| `git branch` | 查看分支 |
| `git branch feature` | 创建分支 |
| `git checkout feature` | 切换分支 |
| `git checkout -b feature` | 创建并切换分支 |
| `git merge feature` | 合并分支到当前分支 |
| `git log --oneline` | 查看简洁提交历史 |
| `git stash` | 暂存当前修改（不提交） |
| `git stash pop` | 恢复暂存的修改 |
| `git reset --hard HEAD~1` | 回退到上一个提交（丢弃修改） |
| `git reset --soft HEAD~1` | 回退到上一个提交（保留修改在暂存区） |
| `git revert <commit>` | 撤销某次提交（生成新提交，不改历史） |
| `git diff` | 查看未暂存的修改 |

### 2. Git 工作区域

```
工作区（Working Directory）
    ↓ git add
暂存区（Staging Area）
    ↓ git commit
本地仓库（Local Repository）
    ↓ git push
远程仓库（Remote Repository）
```

### 3. 分支策略

| 分支 | 用途 |
|------|------|
| main / master | 主分支，保持稳定可发布状态 |
| develop | 开发分支，集成最新开发代码 |
| feature/xxx | 功能分支，开发新功能 |
| hotfix/xxx | 热修复分支，紧急修复线上 Bug |
| release/xxx | 发布分支，准备发布时的测试和修复 |

### 4. 冲突解决

当两个分支修改了同一文件的同一位置，合并时会产生冲突：

```
<<<<<<< HEAD
当前分支的代码
=======
要合并的分支的代码
>>>>>>> feature
```

解决步骤：手动编辑文件选择保留的代码 → `git add` → `git commit`。

### 5. Git 常见面试问答

**Q: git merge 和 git rebase 的区别？**
A: merge 会生成一个合并提交，保留完整的分支历史；rebase 会把当前分支的提交"移动"到目标分支之后，历史更线性但会改写提交记录。团队协作推荐 merge，个人分支整理可以用 rebase。

**Q: git reset 和 git revert 的区别？**
A: reset 直接回退到某个提交，会改写历史（适合本地未推送的提交）；revert 生成一个新提交来撤销某次修改，不改历史（适合已推送的提交）。

**Q: .gitignore 的作用？**
A: 指定不需要被 Git 跟踪的文件（如 `node_modules/`、`.env`、`__pycache__/`、`*.pyc`）。

---

## 十、Docker 容器化

### 1. 核心概念

| 概念 | 说明 |
|------|------|
| 镜像（Image） | 只读模板，包含运行应用所需的代码、依赖、环境配置 |
| 容器（Container） | 镜像的运行实例，可以启动、停止、删除 |
| Dockerfile | 构建镜像的脚本文件 |
| Docker Compose | 多容器编排工具，用 YAML 文件定义和管理多个服务 |
| 数据卷（Volume） | 持久化存储，容器删除后数据不丢失 |
| 仓库（Registry） | 存储和分发镜像的服务（如 Docker Hub） |

> 类比：镜像 = 类，容器 = 实例。一个镜像可以创建多个容器。

### 2. Dockerfile 常用指令

```dockerfile
FROM python:3.11-slim          # 基础镜像
WORKDIR /app                   # 设置工作目录
COPY requirements.txt .        # 复制文件到容器
RUN pip install -r requirements.txt  # 执行命令（构建时）
COPY . .                       # 复制项目代码
EXPOSE 8000                    # 声明端口（文档作用）
CMD ["uvicorn", "main:app", "--host", "0.0.0.0"]  # 容器启动命令
```

| 指令 | 作用 |
|------|------|
| `FROM` | 指定基础镜像 |
| `WORKDIR` | 设置工作目录 |
| `COPY` / `ADD` | 复制文件到镜像中 |
| `RUN` | 构建时执行命令（安装依赖等） |
| `CMD` | 容器启动时执行的默认命令 |
| `ENTRYPOINT` | 容器启动时执行的命令（不可被覆盖） |
| `ENV` | 设置环境变量 |
| `EXPOSE` | 声明容器监听的端口 |
| `VOLUME` | 声明数据卷挂载点 |

### 3. Docker Compose

用于定义和运行多容器应用：

```yaml
services:
  app:
    build: .
    ports:
      - "8000:8000"
    depends_on:
      - db
      - redis
    environment:
      - DATABASE_URL=postgresql://user:pass@db:5432/mydb

  db:
    image: postgres:16
    volumes:
      - postgres_data:/var/lib/postgresql/data
    environment:
      - POSTGRES_DB=mydb
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=pass

  redis:
    image: redis:7-alpine

volumes:
  postgres_data:
```

### 4. 常用命令

| 命令 | 作用 |
|------|------|
| `docker build -t myapp .` | 构建镜像 |
| `docker run -d -p 8000:8000 myapp` | 运行容器 |
| `docker ps` | 查看运行中的容器 |
| `docker ps -a` | 查看所有容器（含已停止） |
| `docker logs <container>` | 查看容器日志 |
| `docker exec -it <container> bash` | 进入容器内部 |
| `docker stop <container>` | 停止容器 |
| `docker rm <container>` | 删除容器 |
| `docker images` | 查看本地镜像 |
| `docker rmi <image>` | 删除镜像 |
| `docker-compose up -d` | 启动所有服务（后台） |
| `docker-compose down` | 停止并移除所有容器 |
| `docker-compose down -v` | 停止并移除容器和数据卷 |
| `docker-compose logs -f` | 实时查看日志 |
| `docker-compose restart <service>` | 重启某个服务 |

### 5. Docker 常见面试问答

**Q: 镜像和容器的区别？**
A: 镜像是只读模板，容器是镜像的运行实例。镜像类似类，容器类似对象。

**Q: Docker 和虚拟机的区别？**
A: Docker 容器共享宿主机内核，启动快、占用资源少；虚拟机有独立的操作系统，隔离性更强但开销大。

**Q: COPY 和 ADD 的区别？**
A: 都能复制文件，但 ADD 额外支持自动解压 tar 文件和从 URL 下载。一般推荐用 COPY，语义更明确。

**Q: CMD 和 ENTRYPOINT 的区别？**
A: CMD 可以被 `docker run` 后面的命令覆盖，ENTRYPOINT 不会被覆盖。通常 ENTRYPOINT 定义主命令，CMD 定义默认参数。

**Q: 如何减小镜像体积？**
A: 使用 slim/alpine 基础镜像、合并 RUN 指令减少层数、使用 .dockerignore 排除无用文件、多阶段构建。

---

## 十一、Linux 常用命令

### 1. 文件操作

| 命令 | 作用 | 示例 |
|------|------|------|
| `pwd` | 显示当前工作目录 | `pwd` |
| `ls` | 列出目录内容 | `ls -l`（详细）、`ls -a`（含隐藏文件） |
| `cd` | 切换目录 | `cd /home/user`、`cd ..`（上级目录） |
| `mkdir` | 创建目录 | `mkdir mydir`、`mkdir -p a/b/c`（递归创建） |
| `rmdir` | 删除空目录 | `rmdir mydir` |
| `rm` | 删除文件/目录 | `rm file.txt`、`rm -r mydir`（递归删除） |
| `cp` | 复制文件/目录 | `cp file1 file2`、`cp -r dir1 dir2` |
| `mv` | 移动/重命名 | `mv file1 file2`、`mv file.txt /tmp/` |
| `touch` | 创建空文件 | `touch file.txt` |

### 2. 文件查看

| 命令 | 作用 | 示例 |
|------|------|------|
| `cat` | 查看文件全部内容 | `cat file.txt` |
| `less` | 分页查看（支持上下翻页） | `less file.txt` |
| `head` | 查看前 N 行 | `head -n 10 file.txt` |
| `tail` | 查看后 N 行 | `tail -n 10 file.txt` |
| `tail -f` | 实时查看文件变化（看日志） | `tail -f /var/log/syslog` |
| `grep` | 搜索文件内容 | `grep "error" app.log` |
| `wc` | 统计行数/字数 | `wc -l file.txt`（行数） |

### 3. 文件查找

| 命令 | 作用 | 示例 |
|------|------|------|
| `find` | 按条件查找文件 | `find /home -name "*.txt"` |
| `locate` | 快速查找（依赖索引数据库） | `locate file.txt` |
| `which` | 查找命令所在路径 | `which python` |

### 4. 权限管理

```
权限格式：rwxr-xr-x
         │││││││││
         ├┤├┤├┤
         │ │ └── 其他用户：r-x (5)
         │ └──── 所属组：r-x (5)
         └────── 所有者：rwx (7)

r = 4（读）  w = 2（写）  x = 1（执行）
```

| 命令 | 作用 | 示例 |
|------|------|------|
| `chmod` | 修改文件权限 | `chmod 755 file.txt`、`chmod +x script.sh` |
| `chown` | 修改文件所有者 | `chown user:group file.txt` |

### 5. 压缩与解压

| 命令 | 作用 | 示例 |
|------|------|------|
| `tar -cvf` | 打包 | `tar -cvf archive.tar folder/` |
| `tar -xvf` | 解包 | `tar -xvf archive.tar` |
| `tar -czvf` | 打包并压缩（gzip） | `tar -czvf archive.tar.gz folder/` |
| `tar -xzvf` | 解压 gzip | `tar -xzvf archive.tar.gz` |
| `zip` | 压缩为 zip | `zip -r archive.zip folder/` |
| `unzip` | 解压 zip | `unzip archive.zip` |

### 6. 进程管理

| 命令 | 作用 | 示例 |
|------|------|------|
| `ps aux` | 查看所有进程 | `ps aux \| grep python` |
| `top` / `htop` | 实时查看系统资源和进程 | `top` |
| `kill` | 终止进程 | `kill <PID>`、`kill -9 <PID>`（强制） |
| `nohup` | 后台运行（不受终端关闭影响） | `nohup python app.py &` |
| `systemctl` | 管理系统服务 | `systemctl start nginx`、`systemctl status docker` |

### 7. 网络管理

| 命令 | 作用 | 示例 |
|------|------|------|
| `ping` | 测试网络连通性 | `ping www.baidu.com` |
| `curl` | 测试 HTTP 请求 | `curl http://example.com` |
| `ifconfig` / `ip addr` | 查看网卡信息 | `ip addr` |
| `netstat -tulnp` | 查看端口占用 | `netstat -tulnp` |
| `wget` | 下载文件 | `wget http://example.com/file.zip` |
| `ss -tulnp` | 查看端口占用（netstat 替代） | `ss -tulnp` |

### 8. 其他常用命令

| 命令 | 作用 | 示例 |
|------|------|------|
| `sudo` | 以管理员权限执行 | `sudo apt update` |
| `apt-get install` | 安装软件包（Debian/Ubuntu） | `sudo apt-get install nginx` |
| `df -h` | 查看磁盘使用情况 | `df -h` |
| `du -sh` | 查看目录大小 | `du -sh /var/log` |
| `free -h` | 查看内存使用情况 | `free -h` |
| `echo` | 输出文本 | `echo "hello"` |
| `env` | 查看环境变量 | `env` |
| `export` | 设置环境变量 | `export PATH=$PATH:/usr/local/bin` |

---

## 十二、数据库迁移

### 1. 什么是数据库迁移？

数据库迁移是用代码管理数据库表结构变更的方式，类似代码的版本控制。每次修改模型后生成迁移文件，执行迁移文件来更新数据库结构。

### 2. Alembic（SQLAlchemy 配套）

```bash
# 初始化 Alembic
alembic init migrations

# 自动生成迁移文件（检测模型变化）
alembic revision --autogenerate -m "add user table"

# 执行迁移（更新数据库）
alembic upgrade head

# 回退一个版本
alembic downgrade -1

# 查看当前版本
alembic current

# 查看迁移历史
alembic history
```

### 3. Django Migrations

```bash
# 生成迁移文件
python manage.py makemigrations

# 执行迁移
python manage.py migrate

# 查看迁移状态
python manage.py showmigrations

# 回退到指定迁移
python manage.py migrate app_name 0003
```

### 4. 面试要点

- 迁移文件应该提交到 Git，团队成员拉取后执行 `migrate` 即可同步表结构
- 不要手动修改数据库表结构，所有变更通过迁移管理
- 生产环境执行迁移前先备份数据库

---

## 十三、日志系统

### 1. Python logging 模块

```python
import logging

# 基本配置
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    handlers=[
        logging.FileHandler('app.log'),     # 写入文件
        logging.StreamHandler()              # 输出到控制台
    ]
)

logger = logging.getLogger(__name__)

logger.debug("调试信息")
logger.info("普通信息")
logger.warning("警告信息")
logger.error("错误信息")
logger.critical("严重错误")
```

### 2. 日志级别

| 级别 | 数值 | 用途 |
|------|------|------|
| DEBUG | 10 | 调试信息，开发环境使用 |
| INFO | 20 | 正常运行信息 |
| WARNING | 30 | 警告，不影响运行但需关注 |
| ERROR | 40 | 错误，功能受影响 |
| CRITICAL | 50 | 严重错误，程序可能崩溃 |

> 生产环境一般设置为 INFO 或 WARNING，避免 DEBUG 日志过多影响性能。

### 3. 面试要点

- 不要用 `print()` 调试，用 `logging`，支持级别控制、文件输出、格式化
- 日志应记录：请求路径、异常堆栈、关键业务操作
- 生产环境日志需要做日志轮转（`RotatingFileHandler`），防止日志文件过大

---

## 十四、测试（pytest）

### 1. 基本用法

```python
# test_example.py
def add(a, b):
    return a + b

def test_add():
    assert add(2, 3) == 5
    assert add(-1, 1) == 0
    assert add(0, 0) == 0
```

```bash
# 运行测试
pytest
pytest test_example.py
pytest -v  # 详细输出
```

### 2. 常用功能

**fixture（测试夹具）：** 用于准备测试数据或资源

```python
import pytest

@pytest.fixture
def sample_user():
    return {"name": "test", "email": "test@example.com"}

def test_user_name(sample_user):
    assert sample_user["name"] == "test"
```

**参数化测试：**

```python
@pytest.mark.parametrize("a, b, expected", [
    (2, 3, 5),
    (-1, 1, 0),
    (0, 0, 0),
])
def test_add(a, b, expected):
    assert add(a, b) == expected
```

**异常测试：**

```python
def test_division_by_zero():
    with pytest.raises(ZeroDivisionError):
        1 / 0
```

### 3. 面试要点

- 测试文件以 `test_` 开头，测试函数以 `test_` 开头
- `assert` 是核心断言方式
- fixture 用于复用测试准备逻辑
- 单元测试测函数逻辑，集成测试测接口和数据库交互

---

## 十五、CI/CD

### 1. 概念

| 术语 | 全称 | 含义 |
|------|------|------|
| CI | Continuous Integration（持续集成） | 代码提交后自动运行测试、代码检查，确保新代码不破坏已有功能 |
| CD | Continuous Delivery / Deployment（持续交付/部署） | 测试通过后自动部署到测试/生产环境 |

### 2. 典型流程

```
开发者提交代码（git push）
    ↓
CI 服务器自动触发
    ↓
拉取代码 → 安装依赖 → 运行测试 → 代码检查（lint）
    ↓
测试通过 → 构建 Docker 镜像 → 推送到镜像仓库
    ↓
CD 自动部署到服务器
```

### 3. 常用工具

| 工具 | 说明 |
|------|------|
| GitHub Actions | GitHub 内置，配置简单 |
| GitLab CI/CD | GitLab 内置 |
| Jenkins | 老牌 CI/CD 工具，功能强大但配置复杂 |

### 4. 面试要点

- CI 的核心价值：尽早发现问题，每次提交都自动测试
- CD 的核心价值：减少手动部署的风险和时间
- 了解基本流程即可，不需要深入配置细节

---

## 十六、Nginx 反向代理

### 1. 什么是 Nginx？

Nginx 是高性能的 Web 服务器和反向代理服务器，常用于：
- 反向代理：将请求转发给后端应用服务器
- 负载均衡：将请求分发到多台后端服务器
- 静态文件服务：直接返回 HTML、CSS、JS、图片等
- HTTPS 终端：处理 SSL/TLS 加密

### 2. 正向代理 vs 反向代理

| 对比 | 正向代理 | 反向代理 |
|------|---------|---------|
| 代理对象 | 客户端 | 服务器 |
| 用途 | 客户端通过代理访问外部资源（如 VPN） | 服务器通过代理接收客户端请求 |
| 客户端是否知道 | 知道代理的存在 | 不知道后端服务器的存在 |

### 3. 基本配置示例

```nginx
server {
    listen 80;
    server_name example.com;

    # 静态文件
    location /static/ {
        alias /var/www/static/;
    }

    # 反向代理到后端
    location /api/ {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

### 4. 负载均衡

```nginx
upstream backend {
    server 127.0.0.1:8001;
    server 127.0.0.1:8002;
    server 127.0.0.1:8003;
}

server {
    location /api/ {
        proxy_pass http://backend;
    }
}
```

常见负载均衡策略：
- 轮询（默认）：依次分配请求
- 权重（`weight`）：按权重比例分配
- IP 哈希（`ip_hash`）：同一 IP 固定分配到同一台服务器
- 最少连接（`least_conn`）：分配给当前连接数最少的服务器

### 5. 面试要点

- Nginx 处理静态文件比 Python 应用服务器快得多
- 反向代理隐藏了后端服务器的真实地址，提高安全性
- 负载均衡可以提高系统的并发处理能力和可用性
- 生产环境典型架构：客户端 → Nginx → Gunicorn/Uvicorn → FastAPI/Django
