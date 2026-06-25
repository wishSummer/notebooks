# MongoDB

> MongoDB 是文档型数据库，适合存“结构灵活、嵌套明显、对象感强”的数据。

- MongoDB 和 MySQL 的区别？
  - MySQL 是关系型数据库，适合强事务、强结构、复杂关联查询。
  - MongoDB 是文档型数据库，适合结构灵活、嵌套数据、快速迭代和大规模文档存储。
- MongoDB 逻辑结构： database -> collection -> document

## Mysql 和 MongoDB 的区别

|   对比点  |	MySQL   |	MongoDB |
|   ----    |   ----    |   ----    |
|   数据模型	|   表、行、列	|   集合、文档  |
|   结构	|   强 schema	|   弱 schema / 灵活 schema |
|   关联	|   join 强	|   join 弱，更多靠嵌入或引用   |
|   事务	|   强项	|   支持事务，但不是主要优势    |
|   适合	|   结构稳定、关系清晰  |	结构灵活、对象明显  |
|   Java 映射	|   Entity -> Table |	Document -> Collection  |

##  数据结构设计（嵌入|引用）

- 嵌入 ： 将关联数据作为子数据放入当前数据集合中
    ```json
    {
      "_id": "a1001",
      "title": "MongoDB 入门",
      "content": "正文内容",
      "author": {
        "id": "u1001",
        "name": "张三",
        "avatar": "https://example.com/a.png"
      }
    }
    ```
- 引用 ： 将关联数据的id放入当前数据集合中
    ```json
    {
      "_id": "a1001",
      "title": "MongoDB 入门",
      "content": "正文内容",
      "authorId": "u1001"
    }

    {
      "_id": "u1001",
      "name": "张三",
      "avatar": "https://example.com/a.png"
    }
    ```

- 如何选择使用嵌入、引用设计数据格式
  - 嵌入
    - 子数据经常与父数据绑定查询
    - 子数据生命周期依赖父数据
    - 子数据数量有限
    - 子数据不会独立于父数据更新
  - 引用
    - 子数据被多个父数据引用
    - 子数据独立于父数据更新
    - 子数据无限量增长
    - 子数据独立于父数据被查询

## 索引

### 单字段索引

```java
@Indexed(unique = true)
private Long productId;
```

```sql
db.product_details.createIndex({ category: 1 })
```

### 复合索引

```java
@Document("product_details")
@CompoundIndex(name = "idx_category_brand", def = "{'category': 1, 'brand': 1}")
public class ProductDetail {
    @Id
    private String id;

    private String category;
    private String brand;
}
```

```sql
db.product_details.createIndex({ category: 1, brand: 1 })
```

### 唯一索引

```java
@Indexed(unique = true)
private Long productId;
```

### TTL 索引

- 文档自动过期删除
- TTL 不是毫秒级删除， MongoDB 后台线程周期性清理

```java
// TTL 索引需要基于日期字段进行判断。 例 expireAt = "2026-06-17T12:00:00"， 表示 到达日期后的 3600毫秒后过期删除
@Indexed(expireAfterSeconds = 3600)
private LocalDateTime expireAt;
```

### 文本索引

- 文本检索 实际场景还是使用elasticSearch。

```sql
db.articles.createIndex({ title: "text", content: "text" })
```

### 地理位置索引

### 多键索引

- 数组字段索引：多键索引
- MongoDB 会为数组中的每个元素建立索引，这叫多键索引。

```sql
{
  "productId": 1001,
  "tags": ["Java", "MongoDB", "NoSQL"]
}

db.articles.createIndex({ tags: 1 })

```

### 查看是否走索引

```sql
db.product_details.find({ productId: 1001 }).explain("executionStats")
```

## Spring Boot 集成 MongoDB

### 依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-mongodb</artifactId>
</dependency>
```

### 配置

```yaml
spring:
  data:
    mongodb:
      uri: mongodb://username:password@localhost:27017/demo
```

### 实体类

- @Document("product_details")：表示这个类映射 MongoDB 的 product_details 集合
- @Id：表示 MongoDB 文档主键，对应 _id
  - 在新增时可以不设置标记 @Id的字段，MongoDB 会自行填充。
  - 也可以自行填充 id 值
- @Field("product_id") ： 指定mongoDB 内存储的字段名
- @Indexed(unique = true) : 设置索引。

```java
@Document("product_details")
public class ProductDetail {
    @Id
    private String id;

    @Field("product_id")
    @Indexed(unique = true)
    private Long productId;
    private String name;
    private String category;

    private Map<String, Object> specs;
    private List<String> images;
    private String detailHtml;

    private LocalDateTime createdAt;
    private LocalDateTime updatedAt;
}
```

### Repository 接口

```java
public interface ProductDetailRepository
        extends MongoRepository<ProductDetail, String> {

    Optional<ProductDetail> findByProductId(Long productId);

    List<ProductDetail> findByCategory(String category);

    Page<ProductDetail> findByCategory(String category, Pageable pageable);
}
```

### service

```java
@Service
public class ProductDetailService {
    private final ProductDetailRepository repository;

    public ProductDetailService(ProductDetailRepository repository) {
        this.repository = repository;
    }

    public ProductDetail save(ProductDetail detail) {
        detail.setCreatedAt(LocalDateTime.now());
        detail.setUpdatedAt(LocalDateTime.now());
        return repository.save(detail);
    }

    public Optional<ProductDetail> getByProductId(Long productId) {
        return repository.findByProductId(productId);
    }

    public List<ProductDetail> listByCategory(String category) {
        return repository.findByCategory(category);
    }

    // 分页
    Page<ProductDetail> result = repository.findByCategory(
      "keyboard",PageRequest.of(0, 10, Sort.by(Sort.Direction.DESC, "createdAt"))
    );
    

    // 分页 复杂查询
    public List<ProductDedtail> query(ProductDetailRequest request){
        Query query = new Query();

        query.addCriteria(Criteria.where("category").is(request.keyboard));

        query.with(Sort.by(Sort.Direction.DESC, "createdAt"));
        query.skip(0);
        query.limit(10);

        List<ProductDetail> list = mongoTemplate.find(
            query,
            ProductDetail.class
        );

        return list;
    }

    // 更新某个字段
    public void update(ProductDetailRequest request){
      Query query = Query.query(
        Criteria.where("productId").is(productId)
      );

      Update update = new Update()
          .set("detailHtml", detailHtml)
          .set("updatedAt", LocalDateTime.now());
 
    // set 直接覆盖
    // push：直接追加，允许重复
    // addToSet：不存在才追加，自动去重
    // pull：从数组中删除匹配值
    //   Update update = new Update()
    //       .pull("images", imageUrl)
    //       .set("updatedAt", LocalDateTime.now());


      mongoTemplate.updateFirst(
          query,
          update,
          ProductDetail.class
      );
    }

    // 查询
    public void deleteByProductId(Long productId) {
      Query query = Query.query(
          Criteria.where("productId").is(productId)
      );

      mongoTemplate.remove(query, ProductDetail.class);
  }
}
``` 