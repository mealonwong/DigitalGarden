---
{"dg-publish":true,"permalink":"/redis/redis-interview/"}
---

### 缓存穿透
```
缓存穿透指的是部分请求中的数据，在Redis中不存在，而且在MySQL中同样不存在这部分数据，导致这部分请求将始终穿透Redis打到MySQL上，会导致MySQL压力增加
```

解决办法：
1. 过滤非法请求，将明显不符合规范的请求过滤掉；
2. 对于不存在的这部分数据在Redis中以空值进行存储，但需要注意在MySQL数据更新时将Redis数据更新，保证数据一致性；
3. 使用bloom过滤器，在查询之前判断该请求的数据是否存在，不存在则不用请求MySQL

### 缓存雪崩
```
缓存雪崩指的是同一时间大面积key同时失效，导致此时请求大面积都打到了MySQL上，直接增加了数据库的负载，也有可能是由于Redis服务器宕机导致的，这种情况就需要高可用的Redis集群架构
```

解决办法：
1. 缓存时间随机分布，避免大量key同时过期
2. 引用二级缓存，即本地缓存
3. 预加载缓存，缓存即将过期时提前加载新的缓存数据
4. 限流和熔断，确保不会有大量请求直接打到数据库

### 缓存击穿
```
缓存击穿指的是在高并发环境下，某个热点key突然失效，导致大量请求同时打到数据库，导致数据库负载骤增。
```
解决办法：
1. 使用互斥锁，即在缓存失效时，只允许一个线程去重加载缓存数据，其余线程等待，避免大量请求同时打到数据库
2. 热点数据永不过期，对于某些可提前预知的热点数据设置永不过期
3. 使用本地缓存，减轻数据库压力
4. 监控告警，熔断限流

### Redis热点key
```
Redis热点key（Hotspot Key）是指在Redis缓存中经常被访问的某个特定的缓存键。这些热点key可能是数据、计数器、频繁访问的用户信息等等，而访问热点key的频率远高于其他缓存数据。处理Redis热点key是很重要的，因为它们可能成为系统的性能瓶颈，导致Redis服务器压力过大，从而影响系统的稳定性和性能。
```

以下是一些处理Redis热点key的常见策略：

1. **分片**：将相同类型的热点key分散到多个Redis实例或集群中，以减轻单个实例的压力。这样可以提高并发处理能力。
2. **LRU淘汰**：使用LRU（最近最少使用）或其他缓存淘汰算法，定期或根据策略删除一些不常访问的热点key，以便腾出内存空间。
3. **缓存预热**：在系统启动或低峰期，提前加载热点数据到Redis缓存中，以避免在高峰期间突然的大量请求。
4. **二级缓存**：使用二级缓存，例如本地缓存（如Guava Cache）来减轻Redis的压力。在本地缓存中，可以缓存一部分热点数据，以加快访问速度。
5. **限流**：通过限制对热点key的访问频率，控制同时对同一key的请求数量。
6. **使用ZSET来分散请求**：如果有一个热点key用于存储某种排名，可以使用ZSET（有序集合）来分散对该排名的读取请求。
7. **数据分片**：对于某些数据，可以将其按一定规则分片存储在多个key中，从而分散访问热点。
8. **定时刷新**：周期性地更新或刷新热点key中的数据，以减轻热点key压力。

在处理Redis热点key时，需要根据具体应用场景和数据特点来选择合适的策略。一种通用的策略可能无法适应所有情况，因此需要根据需求和性能分析来选择和调整策略。

## Redis大key

Redis大key指的是占用内存较大的key，但并不是指key很大，而是该key指向的value很大，导致内存的过度使用，也有可能是存储了大量的元素在集合（set/zset/hash）中。

可能导致以下问题：
1. **内存消耗**：大key会占用大量的内存，这可能导致Redis服务器的内存不足。
    
2. **性能下降**：操作大key可能会导致性能下降，因为Redis是单线程的，处理大key可能需要更长的时间。
    
3. **备份和持久化问题**：在备份和持久化时，大key可能导致备份文件变得庞大，增加了备份和恢复的时间。
    
4. **网络传输延迟**：大key的读写操作可能会导致网络传输延迟，尤其是在集群部署中。

解决策略：

1. **拆分大key**：如果可能的话，将大key拆分成多个小key，以减小单个键的大小。
    
2. **优化数据结构**：考虑使用适当的数据结构，以减小数据的大小。例如，使用压缩数据结构或将一些数据存储在多个小键中。
    
3. **限制数据大小**：在应用程序中限制数据写入Redis的大小。可以设置最大值，以防止数据过大。
    
4. **监控和清理**：定期监控Redis中的大key，然后执行清理操作，删除不再需要的大key。
    
5. **使用分布式缓存**：考虑使用多个Redis实例或分布式缓存系统，以分担大key的内存压力。


### Redis&MySQL数据一致性

延时双删：
在数据更新时，首先删除Redis中的缓存数据，然后更新MySQL中的数据，数据更新后延时一段时间后再次将Redis中的缓存数据删除；
为了防止删除失败，还可以将删除Redis缓存的请求作为消息发送出去或者多次进行重试操作；
其次可以通过监听binlog的变更，然后删除Redis中的缓存数据并进行重载。