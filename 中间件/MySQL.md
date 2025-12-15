# Mysql 常用命令

    # 查看连接池连接
    SHOW PROCESSLIST;

    // 清理闲置连接
    select concat('KILL ',id,';') from information\_schema.processlist where Command = 'Sleep';
    kill (连接Id);

    // 查看链接
    select p.db,count(id) from information\_schema.processlist  p where Command = 'Sleep' GROUP BY db;

    // 查看最大连接数
    show global variables like '%max\_connect\_errors%';

    //查询锁状态
    show status like ‘%lock%’; 

    // sql优化，查看sql语句执行策略 
    explain (selection statement)

    // 重新统计索引信息
    analyze table tablename
    ```

# Mysql 索引优化
## Explain
> 查看优化器关于sql语句执行信息。
> Explain 执行计划包含字段信息分别是: `id`、`select_type`、`table`、`partitions`、`type`、`possible_keys`、`key、key_len`、`ref、rows`、`filtered`、`Extra` 。

### `id`
> `id` 表示查询中执行select子句或者操作表的顺序，id的值越大，代表优先级越高，越先执行。

| 值        | 含义    | 实战理解         |
| -------- | ----- | ------------ |
| 相同 id    | 同一层查询 | 多表 join 同级执行 |
| id 大 → 小 | 嵌套优先  | 子查询先执行       |


* `id相同` 可以理解id相同表为一组，具有同样的优先级，执行顺序由上而下，具体顺序由优化器决定。
* `id不同` 如果我们的 SQL 中存在子查询，那么 id的序号会递增，id值越大优先级越高，越先被执行 。当查询依次嵌套，发现最里层的子查询 id最大，最先执行。
* `以上两种情况同时存在`  优化器会将相同id划分为一组，同组的从上往下顺序执行，不同组 id值越大，优先级越高，越先执行。

### `select_type`
> `select_type`：表示 `select` 查询的类型，主要是用于区分各种复杂的查询，例如：普通查询、联合查询、子查询等。

| 类型                 | 含义            | 是否常见   |
| ------------------ | ------------- | ------ |
| SIMPLE             | 无子查询          | ⭐⭐⭐⭐⭐  |
| PRIMARY            | 外层查询          | ⭐⭐⭐⭐   |
| SUBQUERY           | 子查询           | ⭐⭐⭐    |
| DERIVED            | 派生表（FROM 子查询） | ⭐⭐     |
| UNION              | UNION 中的查询    | ⭐      |
| DEPENDENT SUBQUERY | 相关子查询         | ❌ 性能风险 |


1. `SIMPLE`：表示最简单的 select 查询语句，也就是在查询中不包含子查询或者 union交并差集等操作。
2. `PRIMARY`：当查询语句中包含任何复杂的子部分，最外层查询则被标记为PRIMARY。
3. `DERIVED`：表示包含在from子句中的子查询的select，在我们的 from 列表中包含的子查询会被标记为derived 。
4. `SUBQUERY`：当 select 或 where 列表中包含了子查询，该子查询被标记为：SUBQUERY 。
5. `UNION`：如果union后边又出现的select 语句，则会被标记为union；若 union 包含在 from 子句的子查询中，外层 select 将被标记为 derived。
6. `UNION RESULT`：代表从union的临时表中读取数据，例如table列的<union1,4>表示用第一个和第四个select的结果进行union操作。

### `table`
> 表示查询的表名，但并不一定是真实存在的表。当查询列有别名显示别名，或者为临时表，例如上边的DERIVED、 <union1,4>等。

| 值            | 含义       |
| ------------ | -------- |
| 表名           | 真实表      |
| `<derivedX>` | 派生表      |
| `<unionX>`   | UNION 结果 |


### `partitions`
> 查询时匹配到的分区信息，对于没有进行分区的表值为NULL，当查询的是分区表时，partitions显示分区表命中的分区情况。

### `type`
> 用于标记查询使用了何种查询类型，它在 SQL优化中是一个非常重要的指标，以下性能从好到坏依次是：
> `system`  > `const` > `eq_ref` > `ref`  > `ref_or_null` > `index_merge` > `unique_subquery` > `index_subquery` > `range` > `index` > `ALL`

| 类型     | 性能    | 说明        |
| ------ | ----- | --------- |
| system | ⭐⭐⭐⭐⭐ | 仅一行       |
| const  | ⭐⭐⭐⭐⭐ | 主键/唯一键等值  |
| eq_ref | ⭐⭐⭐⭐  | JOIN 唯一命中 |
| ref    | ⭐⭐⭐   | 非唯一索引等值   |
| range  | ⭐⭐    | 范围扫描      |
| index  | ⭐     | 全索引扫描     |
| ALL    | ❌     | 全表扫描      |


1. `system`： 当表仅有一行记录时(系统表)，数据量很少，往往不需要进行磁盘IO，速度非常快。
2. `const`：表示查询时命中 primary key 主键或者 unique 唯一索引，或者被连接的部分是一个常量(const)值。这类扫描效率极高，返回数据量少，速度非常快。
3. `eq_ref`：查询时命中主键primary key 或者 unique key索引， type 就是 eq_ref。
4. `ref`：区别于eq_ref ，ref表示使用非唯一性索引，会找到很多个符合条件的行。
5. `ref_or_null`：这种连接类型类似于 ref，区别在于 MySQL会额外搜索包含NULL值的行。
6. `index_merge`：使用了索引合并优化方法，查询使用了两个以上的索引。
7. `unique_subquery`：替换IN子查询，子查询返回不重复的集合(当IN 子查询时，子查询范围结果为唯一字段)。
8. `index_subquery`：区别于unique_subquery，用于非唯一索引，可以返回重复值。
9. `range`：使用索引选择行，仅检索给定范围内的行。简单点说就是针对一个有索引的字段，给定范围检索数据。在where语句中使用 bettween...and、<、>、<=、in 等条件查询 type 都是 range。
10. `index`：Index 与ALL 其实都是读全表，区别在于index是遍历索引树读取，而ALL是从硬盘中读取。
11. `ALL`：将遍历全表以找到匹配的行，性能最差。

### `possible_keys`
> `possible_keys`：表示在MySQL中通过哪些索引，能让我们在表中找到想要的记录，一旦查询涉及到的某个字段上存在索引，则索引将被列出，但这个索引并不定一会是最终查询数据时所被用到的索引。列出可能使用的索引。

| 含义   | 实战解读    |
| ---- | ------- |
| 非空   | 优化器候选   |
| NULL | 没任何索引可用 |


### `key` 
> `key`：区别于possible_keys，key是查询中实际使用到的索引，若没有使用索引，显示为NULL。

| 情况   | 说明     |
| ---- | ------ |
| NULL | 未用索引   |
| 有值   | 当前执行路径 |

### `key_len`
> `key_len`：表示查询用到的索引长度（字节数），原则上长度越短越好 。
> 是否被范围截断，是否没用到后面的列。

| 特点     | 说明       |
| ------ | -------- |
| 越大     | 用到的索引列越多 |
| 小于索引总长 | 只用到左前缀   |

### `ref`
> `ref`：常见的有：`const`，`func`，`null`，`字段名`。

| 值       | 含义      |
| ------- | ------- |
| const   | 常量      |
| tbl.col | 与另一表列比较 |
| func    | 表达式     |


1. `const`: 当使用常量等值查询。
2. `字段名`: 当关联查询时，会显示相应关联表的关联字段。
3. `func`: 如果查询条件使用了表达式、函数，或者条件列发生内部隐式转换，可能显示为func。
4. `null`: 其他情况为null。
   
### `rows`
> `rows`：展示表的的统计信息和索引使用情况，估算检索数据所需要扫描的行数，可以直观看到当前sql性能，当前值越小越好。

| 含义   | 实战解读  |
| ---- | ----- |
| 数字越大 | 越慢    |
| 估算值  | 不等于真实 |

### `filtered`
> `filtered` 这个是一个百分比的值，表示符合条件记录数的百分比。简单点说，这个字段表示存储引擎返回的数据在经过过滤后，剩下满足条件的记录数量的比例。

| 值   | 含义    |
| --- | ----- |
| 100 | 几乎不过滤 |
| 很小  | 过滤效果好 |


### `Extra`
> `Extra` ：不适合在其他列中显示的信息，Explain 中的很多额外的信息会在 Extra 字段显示。

| Extra                         | 含义      | 是否危险 |
| ----------------------------- | ------- | ---- |
| Using index                   | 覆盖索引    | ✅ 好  |
| Using where                   | 需回表/过滤  | ⚠️   |
| Using index condition         | ICP     | ✅    |
| Using temporary               | 临时表     | ❌    |
| Using filesort                | 额外排序    | ❌    |
| Backward index scan           | 反向索引扫描  | ✅    |
| Range checked for each record | 每行判断    | ❌    |
| Using join buffer             | JOIN 缓冲 | ⚠️   |


1. `Using index`：查询操作使用了`覆盖索引`。（覆盖索引：查询结果字段，索引中已经包含，故当查询检索索引后无需回表查找元数据，索引值已经满足查询需求),使用到覆盖索引查询速度会非常快，SQl优化中理想的状态。
2. `Using where`：查询时未找到可用的索引，进而通过where条件过滤获取所需数据，但要需要注意的是并不是所有带有where语句的查询都会显示Using where。
3. `Using temporary`：表示查询后结果需要使用临时表来存储，一般在排序或者分组查询时用到。
4. `Using filesort`：表示无法利用索引完成的排序操作，也就是ORDER BY的字段没有索引，通常这样的SQL都是需要优化的。(添加索引可以规定排序方式 `desc` 或者 `aes`，默认`aes`)
5. `Using join buffer`：在我们联表查询的时候，如果表的连接条件没有用到索引，需要有一个连接缓冲区来存储中间结果。
6. `Impossible where`：表示当前where语句没有得到符合条件的行。
7. `No tables used`：我们的查询语句中没有FROM子句，或者有 FROM DUAL子句。

### 快速判断 SQL 是否“健康”

- type ≤ range
- rows 很小
- Extra 无 temporary/filesort
- 能看到 Backward index scan

## MySQL 优化策略

### 索引设计逻辑
> 索引不是为了“命中字段”，而是为了“减少扫描 + 提前停止扫描”
> 不能提前停的索引，都是假索引
> 索引是给 `SQL` 用的，不是给表用的
> 索引的本质，是用“有序扫描”换掉“无序排序”，并尽可能早地停下来。

- 索引是给 `SQL` 用的，不是给表用的,一条索引服务一类 SQL
- `MySQL` 只会连续使用索引的「左前缀」
    ```
    idx(a,b,c)

    where a and b and c [√]

    where a [√]

    where a and c [×]

    where b [×]

    where c [×]

    ```

- 等值在前，范围在后，排序在最后（经典规则）
    | 条件类型 | 例子                |
    | ---- | ----------------- |
    | 等值   | `=`、`IN`          |
    | 范围   | `>`、`<`、`BETWEEN` |
    | 其它   | `LIKE '%x%'`、函数   |

- 范围条件一出现，后续列就“失去定位能力”
  - 在 MySQL 的 B+Tree 索引里，一旦某一列用了“范围条件”，后面的列就不能再帮你“缩小扫描范围”，只能决定“扫出来的结果是不是已经排好序”。
  - B+Tree 结构
    - 假设你有索引：
        ```java
        (site_id, record_at, id)
        ```
    - 在 B+Tree 里的真实顺序是：
        ```javaScript
            site_id = A
              record_at = 2024-01-01
                id = 1
                id = 2
              record_at = 2024-01-02
                id = 3
            site_id = B
              record_at = 2024-01-01
                id = 4
        ```
- 能提前停的索引，才是好索引（最重要）
  - 能够扫描最少的列得到需要的数据。
- 日志 / 明细表：索引数量 ≤ 3
  - 写多、读取多的表每多一个索引，写入成本*2，`buffer pool` 压力* 2
- 若查询需要获取最新数据
  - 可以尝试使用排序字段作为复合索引第一个字段
    | 业务含义      | 索引最左列            |
    | --------- | ---------------- |
    | 最新插入      | `id DESC`        |
    | 最新时间（高离散） | `record_at DESC` |
    | 单主体查询     | `site_id`        |


### SQL设计逻辑

- 配合索引顺序，使用条件语句。
- 等值在前，范围在后，排序在最后（经典规则）
- 在符合索引的情况下，优先使用能够缩小数据量的条件语句
- 排序规则和索引排序一致