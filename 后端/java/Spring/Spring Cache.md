# Spring Cache

* 缓存实现
  * ConcurrentMap（默认）
  * Caffeine
  * Redis
  * Ehcache
  * Memcached
  * 等等

* 常用注解
  * @EnableCaching
    * 用于标记spring启动类，启用缓存功能。
  * @Cacheable
    * 用于标记方法，缓存结果

    ```java
    @Cacheable(
      cacheNames = "proxy-site-privilege-tree-by-employer-id-cache", // 缓存名称
      key = "#p0+'-'+#p1", // 缓存数据标记 key
      unless = "#result==null" // 剔除不需要缓存的结果
      ) 
    ````

  * @CachePut
    * 更新缓存

    ```java
    @CachePut(
        cacheNames = "proxy-site-privilege-tree-by-employer-id-cache",
        key = "#employerId+'-'+#tag" //更新 当前缓存名称下key的对应缓存
    )
    ```

  * @CacheEvict
    * 删除缓存

    ```java
    @CacheEvict(
        cacheNames = "proxy-site-privilege-tree-by-employer-id-cache", // 所需删除的缓存名称
        key = "#employerId+'-'+#tag" // 仅删除当前缓存名称对应key的缓存
    )

    @CacheEvict(
        cacheNames = "proxy-site-privilege-tree-by-employer-id-cache",
        allEntries = true // 删除当前缓存名称下的所有缓存
    )
    ```
