# JPA

## 默认方法

- save 会根据Id 查询是否已存在决定使用insert还是update；
  - 若参数Entity部分属性为null则修改数据库数据字段为null，若想只修改部分参数，需要先查询数据将不需要修改字段值进行填充;
  - Jpa不能够将查询得到的实体更改主键值（id）后进行save()，只能够创建新的实体填充需要的参数去save();

## JPA 对象关联

### ORM 基础

- 实体类与表的映射
- `@Entity`
- `@Table`
- `@Id`
- `@Column`

- 主键策略
  - `GenerationType.IDENTITY`
  - `SEQUENCE`
  - `AUTO`

### 对象关联映射

- `@OneToOne`
- `@ManyToOne`
- `@OneToMany`
- `@ManyToMany`

### Fetch 策略

- `LAZY`
  - 查询时只加载主表
  - 若对象生命周期在事务内，访问子对象，可触发二次查询。
- `EAGER`
  - 查询时自动把关联对象查出来

### 级联操作 Cascade

- 决定对父实体的操作是否传播到子实体
- 级联不影响查询，只影响 `persist` , `merge` , `remove`
- 常见枚举
  - PERSIST：保存父对象时保存子对象。
  - MERGE：更新父对象时更新子对象。
  - REMOVE：删除父对象时删除子对象。
  - ALL：包含以上所有。

### 事务与关联的关系

- JPA 对象持久状态管理
  - 通过 `JpaRepository` （基于 EntityManager）查询得到的对象是持久化状态(Managed Entities)。
    - 持久化状态对象在当前方法体标记 `Transcation()` 时，修改他们的属性，在事务提交时 `JPA` 的脏检查 `dirty checking` 会自动检测到变化并执行 `UPDATE` ，不需要显式 `save` 。
  - 通过 `QueryDSL` 查询的得到的对象是 脱管状态（Detached Entities）。
    - 托管状态对象需要显式 `save` ，否则不会触发更新。
- 事务保证了 `Session/EntityManager` 的存活
  - 在事务内访问懒加载字段 → 会触发 SQL 查询。
  - 在事务外访问懒加载字段 → LazyInitializationException。
- save()实体时，需要实体间相互填充，反而表之间关联的属性并不是必须的;（在OneToMany具有同等效益）

### 常见问题

N+1 查询问题 查询主表执行1条sql，若存在10条数据，则会触发10次子查询，若数据量过大，则n+1。 瞬间爆发大量sql查询。

原因：LAZY 在循环访问时不断触发 SQL。

解决：@EntityGraph / JPQL fetch join。

事务外 LazyLoading 异常

原因：事务关闭，Session 已关闭。

解决：在 Service 层保证事务；或改 EAGER；或 DTO 转换提前加载。

Cascade REMOVE 导致误删

场景：删除用户时不小心删掉所有订单。

解决：谨慎使用 CascadeType.REMOVE，生产中多数只用 MERGE 和 PERSIST。

## Bug记录

### org.hibernate.LazyInitializationException异常解决办法

- @OneToMany()(mappedBy = "customized",cascade = CascadeType.ALL,fetch = FetchType.EAGER)  需要使用“customized” 对应映射表@ManyToOne注解标注的字段名； 重要：@OneToMany fetch需要设置为 EAGER （立刻加载）
- @ManyToOne(cascade = CascadeType.ALL, fetch = FetchType.LAZY) 需要配合 @JoinColumn(name = "customized_id", referencedColumnName = "id")限定联系表字段映射，'customized_id'对应联系表@OneToMany相关联的字段  重要：@ManyToOne fetch则设置为懒加载

### 方法抛出'java.lang.stackOverflowError"异常 无法对...Object.toString()求值

- bug产生原因，关联子查询，children 以及 parent，导致toString 输出parent字段时进入死循环，堆栈溢出。
- 修复 自定义toString()方法。

### 主附表关联添加，当附属表需要主表添加完成后生成的某个字段作为自身属性时

- @MapsId 可以完成在附属表sql执行前将主表添加完成后生成的字段值注入字表参数中。例主表Id = 附表Id，主表添加完成后生成Id值，注入到附表Object当中，继而添加附表数据。(@OneToOne  @OneToMany 两种关系中见)

### entity实体子类实体注入若是lazy，子类方法会抛出 'org.hibernate.LazyInitializationException' 异常。 无法对 com.wesoft.cloud.platform.commons.entity.UserObject.toString() 求值；但方法体若是标注@Transactional(readOnly=true)则Jpa会进行子类注入
