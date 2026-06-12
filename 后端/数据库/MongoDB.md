# MongoDB

> MongoDB 是文档型数据库，适合存“结构灵活、嵌套明显、对象感强”的数据。

- MongoDB 和 MySQL 的区别？
  - MySQL 是关系型数据库，适合强事务、强结构、复杂关联查询。
  - MongoDB 是文档型数据库，适合结构灵活、嵌套数据、快速迭代和大规模文档存储。

## Spring Boot 集成 MongoDB

### 依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-mongodb</artifactId>
</dependency>
```

### 配置

```yamlspring:
  data:
    mongodb:
      host: localhost
      port: 27017
      database: test
      username: root
      password: root
```

### 实体类

```java
@Document(collection = "users")
public class User {
    @Id
    private String id;
    private String name;
    private int age;
    private Map<String, Object> specs;
    // getters and setters
}
```

### Repository 接口

```java
public interface UserRepository extends MongoRepository<User, String> {
    List<User> findByName(String name);
}
```

### 使用

```java
@Service
public class UserService {
    @Autowired
    private UserRepository userRepository;

    public List<User> getUsersByName(String name) {
        return userRepository.findByName(name);
    }

    public User saveUser() {
        User user = new User();
        user.setName("Alice");
        user.setAge(30);
        user.setSpecs(Map.of("hobby", "reading", "city", "New York"));
        return userRepository.save(user);
    }

    public List<user> queryUser(UserRequest request) {
        // 构建查询条件
        Query query = new Query();
        if (request.getName() != null) {
            query.addCriteria(Criteria.where("name").is(request.getName()));
        }
        if (request.getAge() != null) {
            query.addCriteria(Criteria.where("age").is(request.getAge()));
        }
        // 执行查询
        return mongoTemplate.find(query, User.class);
    }
}
``` 