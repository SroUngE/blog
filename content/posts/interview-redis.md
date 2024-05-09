---
title: "Interview Redis"
description: "基于 ChatGPT3.5，回答常见的 Redis 面试题。"
date: 2024-01-25
tags: ["Redis"]
featuredImage: "https://s2.loli.net/2024/01/23/FdJWU6IpNZGfACv.png"
featuredImagePreview: "https://s2.loli.net/2024/01/23/FdJWU6IpNZGfACv.png"
draft: true
---

<!--more-->

### Redis 介绍

#### 介绍

Redis，英文全称是 **Remote Dictionary Server**（远程字典服务），是一个开源的内存 Key-Value 存储系统，常用于缓存、会话管理和消息队列。它支持多种数据结构如字符串、哈希、列表等，具有**快速读写**和**持久化**的特性，适合处理大规模的数据和高并发访问。

#### 特征

* 高性能（基于内存、IO 多路复用、良好的编码）
* key-value 型，value 支持多种灵活的数据结构支持
* 持久性，允许在需要时将数据保存到磁盘，保证数据不会因意外丢失
* 支持主从集群、分片集群



### Redis 的高性能

Redis具有高性能的原因主要包括以下几点：

1. **基于内存：** Redis将数据存储在内存中，相比于从磁盘读取数据，内存访问速度更快，提高了读写性能。
2. **单线程：** Redis采用单线程模型处理命令，每个命令具备原子性，通过避免线程切换的开销，简化了并发控制，从而提高了性能。尽管在6.0之后引入了多线程，但那只应用于网络传输，在核心的命令执行上还是单线程。
3. **非阻塞的 I/O 操作：** Redis使用了非阻塞的I/O操作，可以在等待数据从磁盘加载或网络传输时执行其他操作，充分利用了系统资源。
4. **高效的数据结构：** Redis支持多种数据结构，如哈希表、跳跃表等，这些数据结构的设计和实现都经过优化，提高了读写操作的效率。



### Redis 的数据结构类型

Redis支持多种数据结构类型，包括

1. String（字符串）：存储文本或二进制数据。value其实不仅是字符串，也可以是数字，JPEG图片或者序列化的对象。
2. List（列表）：双向链表，支持从两端添加和弹出元素，可用于实现队列、栈等数据结构。
3. Set（集合）：无序且唯一的元素集合，支持集合间的交、并、差运算。
4. zset（有序集合）：类似集合，但每个元素关联一个 score，可用于实现排行榜等功能。
5. Hash（哈希）：将多个键值对组织在一起，常用于存储对象。



### Redis SDS

C语言的字符串本质是字符数组：需要计算字符串长度，非二进制安全，不可修改。

Redis 构建了一种新的字符串结构，是二进制安全的，称为 简单动态字符串（Simple Dynamix String） 。它具备动态扩容的能力。

![image-20240324195733217](https://s2.loli.net/2024/03/24/uifrpXGoKFJ5BR7.png)



### Redis IntSet

IntSet 是 Redis中set集合的一种实现方式，基于整数数组来实现，具备长度可变、有序等特征。具备类型升级机制，节省内存空间。底层采用二分查找方式来查询。



### Redis Dict

redis 的键值映射是通过 Dict 来实现的。Dict 由三部分组成：哈希表（Dict Hash Table）、哈希节点（Dict Entry）、字典（Dict）。Dict 的HashTable是数组结合单向链表实现的，在每次新增键值对时都会检查负载因子，决定是否扩容。删除元素的时候，也会对负载因子做检查，决定是否收缩。Redi 的rehash过程是渐进式的，分多次完成。每次增删改的时候，只迁移了一个角标的数据。跳跃表是Redis特有的数据结构，就是在链表的基础上，增加多级索引提升查找效率。

![image-20240324201521223](https://s2.loli.net/2024/03/24/DGoiuYst4ATSLOe.png)



### 缓存穿透，缓存雪崩，缓存击穿

**缓存穿透（Cache Penetration）** 

缓存穿透是指请求一个根本不存在的数据，由于缓存没有命中，请求直接传递到数据库，使得系统每次都要去查询数据库，从而消耗资源。

**解决方法** 

- **缓存空对象：**可以在查询结果为空时，仍然将空结果进行缓存，但设置较短的过期时间，避免频繁查询数据库。
- **布隆过滤器：**过滤掉不存在的数据的请求。



**缓存雪崩（Cache Avalanche）** 

缓存雪崩指的是在同一时间有大量的缓存数据失效，导致大量的请求直接击穿到数据库，造成系统瞬时压力激增，可能导致系统崩溃。这通常是由于缓存数据设置了相同的过期时间，一旦过期，大量的请求落在同一时刻。

**解决方法** 

- 设置不同的过期时间，使缓存数据的失效时间分散开来，防止同时失效。
- 给缓存业务添加降级限流策略，给业务添加多级缓存。
- 针对Redis宕机的情况，利用Redis 集群提高服务的可用性。



**缓存击穿（Hotspot Invalid）**

缓存击穿指的是缓存某个热门或频繁请求的缓存键过期后，恰好此时有大量请求同时涌入，导致有大量请求直达数据库，增加了数据库的负载。

**解决方法** 

- 使用互斥锁（Mutex Lock）或分布式锁来保护查询数据库重建缓存数据的操作，可能会导致较长等待时间。
- 使用热点数据永不过期的策略，或者采用预先加载的方式，确保热点数据始终在缓存中。

![](https://s2.loli.net/2024/03/12/2QsJn8cGbPjeM73.png)



### Redis和数据库缓存一致性

在处理Redis和数据库（如MySQL）之间的数据一致性问题时，关于是否应该修改Redis中的数据，还是直接删除对应的数据并通过后续的读请求从数据库重新加载，以及写操作的顺序（先更新Redis还是先更新数据库），这取决于具体的应用场景和业务需求。

首先，关于是否修改Redis中的数据或直接删除并重新加载：

- **更新缓存**：当数据发生变化时，可以直接在Redis中更新这些数据。这样做的好处是可以减少从数据库加载数据的开销，提高性能。但这也增加了复杂性，因为需要确保Redis中的更新与数据库中的更新保持一致。
- **淘汰缓存**：另一种策略是当数据发生变化时，删除Redis中对应的数据。当后续的读请求到来时，如果Redis中没有找到所需数据，就会从数据库中加载并缓存到Redis中。这种方法的好处是简化了更新逻辑，但可能会增加读操作的延迟，因为需要从数据库重新加载数据。

接下来，关于写操作的顺序（先更新Redis还是先更新数据库）：

- **先更新数据库，再更新Redis**：这种顺序适用于需要保证数据一致性和可靠性的场景。首先确保数据在数据库中持久化，然后再更新Redis中的缓存数据。这样可以确保即使Redis中的数据丢失或不一致，也可以从数据库中恢复。但在高并发场景下，可能会增加数据库的压力。
- **先更新Redis，再更新数据库**：这种顺序适用于对数据实时性要求较高，但对数据一致性要求不严格的场景。首先更新Redis中的数据以提高读取性能，然后异步地更新数据库。这样可以降低对数据库的实时更新要求，但可能会牺牲一定的数据一致性。可以使用延迟双删：在更新数据库的前后都删除一下



### Redis 的过期策略

Redis 每个 Key 都会设置一个过期时间（TTL，Time To Live），主要的过期策略：

1. **定时删除（Timed Expiry）：** 定期抽样检测部分 Key 是否过期，过期则删除。这样，定时删除机制确保过期的 Key 在一定时间内被及时删除。

2. **惰性删除（Lazy Expiry）：** 在访问 Key 时才检查它是否过期，过期则删除。这样的机制在实际应用中能够更好地利用系统资源，因为不是所有 Key 都会被及时访问。



### Redis 的内存淘汰策略

当内存达到上限时，可以通过设置 `maxmemory-policy` 参数来指定Redis的内存淘汰策略。以下是几种具体的内存淘汰策略：

1. `noeviction`：默认策略，不淘汰任何key，当内存达到上限时，写入操作将会报错，不会删除任何数据。

2. `allkeys-lru`：Redis会从所有键中选择最近最少使用的键进行淘汰。

3. `allkeys-lfu`：Redis会从所有键中选择最不经常使用的键进行淘汰。

4. `allkeys-random`：Redis会从所有键中随机选择一个进行淘汰。

5. `volatile-lru`：Redis会从设置了过期时间的键中选择最近最少使用的键进行淘汰。

6. `volatile-lfu`：Redis会从设置了过期时间的键中选择最不经常使用的键进行淘汰。

7. `volatile-random`：Redis会从设置了过期时间的键中随机选择一个进行淘汰。

8. `volatile-ttl`：Redis会从设置了过期时间的键中，选择剩余时间最短的键进行淘汰。

这些策略可以根据系统的特定需求进行配置。例如，如果希望优先保留设置了过期时间的键，可以选择带有`volatile`前缀的策略。



### Redis 的常见应用场景

1. **缓存管理：** 使用Redis缓存热点数据，减轻数据库负担，提高响应速度。
2. **会话管理：** 替代分布式Session管理（不同服务器同个Session）存储用户会话数据，确保用户登录状态的一致性。
3. **消息队列：** 通过Redis的发布/订阅机制，实现异步消息通信，提高系统解耦性。
4. **分布式锁：** 利用Redis的原子性操作 `setnx` ，实现分布式环境下的锁机制，保证数据一致性。
5. **计数器：** Redis自带计数器功能，非常适合用于统计网站访数据，高效实现原子递增和递减操作。
6. **实时排行榜：** 利用有序集合 `zset` 存储分数和成员，实现实时排名功能，如用户积分排行。
7. **位操作：** 使用Redis的位操作可以轻松地表示用户在线状态。每一位可以代表一个用户的在线或离线状态，通过位运算实现高效的在线状态管理。



### Redis 实现 限流

从简单到复杂，有三种方案：

1. SETNX 实现固定窗口算法；
2. ZSET 实现滑动窗口算法；
3. List 实现令牌桶算法；

#### 使用SETNX（SET if Not Exists）实现固定窗口限流

**整体思路**：

1. 使用Redis的SETNX命令尝试设置一个具有过期时间的key。
2. 如果SETNX命令返回1（表示设置成功），则说明这是该时间窗口内的第一个请求，允许该请求。
3. 如果SETNX命令返回0（表示key已存在），则检查键的时间戳是否在窗口内，如果在窗口内，表示可以通过，否则不通过。

**基于Jedis的实现**：

```java
public class FixedWindowRateLimiter {
    private final Jedis jedis;
    private final String keyPrefix;
    private final int windowSizeInSeconds;

    public FixedWindowRateLimiter(Jedis jedis, String keyPrefix, int windowSizeInSeconds) {
        this.jedis = jedis;
        this.keyPrefix = keyPrefix;
        this.windowSizeInSeconds = windowSizeInSeconds;
    }

    /**
     * 1. 使用Redis的SETNX命令尝试设置一个具有过期时间的key。
     * 2. 如果SETNX命令返回1（表示设置成功），则说明这是该时间窗口内的第一个请求，允许该请求。
     * 3. 如果SETNX命令返回0（表示key已存在），则检查键的时间戳是否在窗口内，如果在窗口内，表示可以通过，否则不通过。
     */
    public boolean allowRequest(String identifier) {
        String key = keyPrefix + identifier;
        long currentTime = System.currentTimeMillis() / 1000;

        // Try to set the key if it doesn't exist
        long result = jedis.setnx(key, String.valueOf(currentTime));

        // If result is 1, it means the key was successfully set, and the request is allowed
        if (result == 1) {
            jedis.expire(key, windowSizeInSeconds); // Set expiration time
            return true;
        }

        // If result is 0, it means the key already exists
        // Check if the difference between current time and key's timestamp is less than window size
        String value = jedis.get(key);
        if (value != null) {
            long timestamp = Long.parseLong(value);
            if (currentTime - timestamp < windowSizeInSeconds) {
                return true;
            }
        }

        // If key doesn't exist or the difference is larger than window size, deny the request
        return false;
    }
}
```



#### 使用ZSET（Sorted Set）实现滑动窗口限流

**整体思路**：

1. 使用 ZADD 命令将当前时间戳作为分数，请求标识作为成员添加到有序集合中。
2. 使用 ZREMRANGEBYSCORE 命令移除时间窗口之外的成员。
3. 检查有序集合中的成员数量是否超过限流阈值。

**基于Jedis的实现**：

```java
public class SlidingWindowRateLimiter {
    private final Jedis jedis;
    private final String keyPrefix;
    private final int windowSizeInSeconds;
    private final int maxRequests;

    public SlidingWindowRateLimiter(Jedis jedis, String keyPrefix, int windowSizeInSeconds, int maxRequests) {
        this.jedis = jedis;
        this.keyPrefix = keyPrefix;
        this.windowSizeInSeconds = windowSizeInSeconds;
        this.maxRequests = maxRequests;
    }

    public boolean allowRequest(String identifier) {
        String key = keyPrefix + identifier;
        long currentTime = System.currentTimeMillis() / 1000;

        // Remove old entries from the sorted set
        long windowStart = currentTime - windowSizeInSeconds;
        jedis.zremrangeByScore(key, "-inf", String.valueOf(windowStart));

        // Count the number of requests in the window
        List<Tuple> window = jedis.zrangeWithScores(key, 0, -1);
        int requestsInWindow = window.size();

        // If the number of requests exceeds the maximum allowed, deny the request
        // Or, add current request timestamp to the sorted set
        if (requestsInWindow < maxRequests) {
            jedis.zadd(key, currentTime, String.valueOf(currentTime));
            return true;
        } else {
            return false;
        }
    }
}
```

### Lua脚本实现令牌桶算法

**整体思路**：

1. 从 Redis 中获取令牌桶的当前状态，包括令牌数量和最近一次取令牌的时间。
2. 计算当前时间与最近一次取令牌时间的时间差，以及根据填充速率计算出当前应该有的令牌数量。
3. 判断当前令牌数量是否满足本次请求所需的令牌数量，如果够，则减去请求消耗的令牌数量，并标记允许请求的数量为1。
4. 计算令牌桶的填满时间和对应的过期时间。
5. 如果令牌桶未过期，则更新令牌数量和最近一次取令牌时间，然后返回允许请求的数量和新的令牌数量。

这个脚本的目的是实现基于令牌桶算法的限流功能，确保系统在一定时间内只处理指定数量的请求。

**基于Lua的实现**：

```lua
local tokens_key = KEYS[1]          -- 令牌桶保存在redis中的 key
local timestamp_key = KEYS[2]       -- 最近一次拿令牌的时间的 key
local rate = tonumber(ARGV[1])      -- 每秒往桶里放的令牌数，即填充速率
local capacity = tonumber(ARGV[2])  -- 桶的最大容量
local now = tonumber(ARGV[3])       -- 当前时间，单位:秒
local requested = tonumber(ARGV[4]) -- 每个请求需要几个令牌，默认 1 个

local last_tokens = tonumber(redis.call("get", tokens_key)) or capacity -- 目前桶里的令牌数，如果还没有初始化，则令牌数按最大容量capacity计
local last_timestamp = tonumber(redis.call("get", timestamp_key)) or 0  -- 最后一次取令牌的时间，如果还没有初始化，计为 0
local delta = math.max(0, now - last_timestamp) -- 计算当前时间与最近一次取令牌时间的时间差
local filled_tokens = math.min(capacity, last_tokens + (delta * rate))  -- 根据填充速率计算出当前应该有的令牌数量
local new_tokens = filled_tokens
local allowed_num = 0
if (filled_tokens >= requested) then
    -- 判断当前令牌数量是否满足本次请求所需的令牌数量，如果够，则减去请求消耗的令牌数量，并标记允许请求的数量为1。
    new_tokens = filled_tokens - requested
    allowed_num = 1
end

local fill_time = capacity / rate   -- 计算需要多少时间可以填满桶
local ttl = math.floor(fill_time * 2)   -- key 的存活时间，如果超过了这个时间，桶就填充满了，限流数据就没有存在的必要了，过期后再遇到请求可以从头再计算
if ttl > 0 then
    -- 如果当前key还没有过期，则需要更新缓存，如果已经过了有效期的话就不需要更新数据了，直接让数据过期就可以
    redis.call("setex", tokens_key, ttl, new_tokens)    -- 更新令牌桶里的令牌数
    redis.call("setex", timestamp_key, ttl, now)    -- 更新最后一次执行时间
end

return { allowed_num, new_tokens }
```

https://zhuanlan.zhihu.com/p/643085747



### Redis 实现分布式锁

分布式锁是一种跨进程的、跨机器节点的互斥锁，其目标是在分布式系统中实现互斥访问共享资源的能力，以避免多个进程或线程同时修改相同的数据。它主要具有以下几个特性：

1. **互斥性**：这是锁的最基本要求。在任意时刻，只能有一个客户端获取到锁，确保同一时刻只有一个节点能够访问共享资源。
2. **可重入性**：同一个线程或进程在没有释放锁之前可以重复多次获得锁。
3. **可重试**：当某个客户端或线程尝试获取锁失败时，它可以等待一段时间后再次尝试获取，而不是直接放弃
4. **超时释放**：为了防止线程或进程因意外退出而没有正常释放锁，导致其他线程或进程无法正常获取到锁，分布式锁通常支持设置超时时间。当加锁时间超过设定的时间，锁会自动释放。

![](https://s2.loli.net/2024/03/12/fsJZjI1WvSnCAlX.png)

![](https://s2.loli.net/2024/03/17/DNxEOWTfpSZ2qGj.png)

实现思路：利用 `set nx ex` 获取锁，并设置过期时间，保存线程标识。释放锁时先判断线程标识是否一致，一致则删除锁。

**Lua脚本：**利用lua脚本实现指令的原子性。不可重入，不可尝试

**Reddison：**通过Hash结构记录每个key的获取次数来解决不可重入。

**可重入：**利用Hash结构记录线程id和重入次数

**可重试：**利用信号量和PubSub功能实现等待、唤醒、获取锁失败的重试机制

**自动续期**：如果持有锁的客户端因为网络问题或者GC等原因导致长时间未释放锁，则 Redisson 会自动给这把锁续期，避免锁超时失效。

**超时续约：**利用watchDog，每隔一段时间，重置超时时间

![](https://s2.loli.net/2024/03/20/iPMTqAW6UCg1poQ.png)

### Redis 的持久化机制

Redis有两种主要的持久化机制：RDB（Redis DataBase Backup）和AOF（Append Only File）。

**RDB持久化**：通过定期将内存中的数据快照保存到**磁盘**上的二进制文件 `dump.rdb`，它是Redis默认的持久化方式。这种方式适用于大规模的数据恢复场景，如备份和灾难恢复。你可以配置Redis定期创建RDB快照，也可以手动触发，Redis 停机前会自动执行一次。RDB没办法做到实时持久化/秒级持久化。

![image-20240322210023171](https://s2.loli.net/2024/03/22/DbNqCcxrGg8X42f.png)

![image-20240322210334593](https://s2.loli.net/2024/03/22/wAnWsVriyJpKP5C.png)

![image-20240322211135452](https://s2.loli.net/2024/03/22/SPI7hnbqoUQmyRs.png)

**RDB 方式 bgsave 的基本流程？**

- fork 主进程得到一个紫禁城，共享内存空间
- 子进程读取内存数据并写入新的 RDB 文件
- 用心RDB 文件替换旧的 RDB 文件

**RDB 的缺点？**

- RDB 执行间隔时间长，两次 RDB 之间写入数据有丢失的风险
- fork 子进程、压缩、写 RDB 文件都比较耗时



**AOF持久化**：将每个写操作追加到 AOF 文件中，以记录所有执行的命令，默认是不开启的。这种方式更加持久，因为即使在发生故障时，只要AOF文件没有损坏，Redis可以通过重新执行AOF文件中的命令来恢复数据。AOF记录的内容越多，文件越大，数据恢复变慢。

![image-20240322211835736](https://s2.loli.net/2024/03/22/i9WmQkbKhOp5ZaF.png)

![image-20240322212044871](https://s2.loli.net/2024/03/22/LAInXHgofu6S5Tb.png)

可以同时使用RDB和AOF，也可以选择只使用其中一种。需要注意的是，AOF和RDB可以同时启用，但在恢复时，AOF的优先级更高。如果启用了AOF，Redis将首先尝试使用AOF文件进行恢复，如果AOF文件不存在或不完整，则尝试使用RDB文件。



### 用户空间和内核空间

![image-20240324204000662](https://s2.loli.net/2024/03/24/iHuD3ZyojIchWAe.png)

IO 多路复用：是利用单个线程来同时监听多个FD，并在某个FD可读、可写时、得到通知，从而避免无效的等待，充分利用CPU资源。常见的实现方式：

select：

poll：

epoll：

select 和poll只会通知用户进程由FD就绪，但不确实具体是那个FD，需要用户进程逐个遍历确认。

epoll 则会在通知用户进程 FD 就绪的同时，把已经就绪的FD 写入用户空间

### Redis 网络模型

**Redis到底是单线程还是多线程?**

- 如果仅仅聊Redis的核心业务部分(命令处理)，答案是单线程
- 如果是聊整个Redis，那么答案就是多线程在Redis版本迭代过程中

在两个重要的时间节点上引入了多线程的支持：

- Redis v4.0:引入多线程异步处理一些耗时较长的任务，例如异步删除命令unlink

- Redis v6.0:在核心网络模型中引入 多线程，进一步提高对于多核CPU的利用率

**为什么Redis要选择单线程?**

- 抛开持久化不谈，Redis是**纯内存操作，执行速度非常快**（微秒级别），它的性能瓶颈是网络延迟而不是执行速度，因此多线程并不会带来巨大的性能提升。

- 多线程会导致过多的上下文切换，带来不必要的开销

- 引入多线程会面临线程安全问题，必然要引入线程锁这样的安全手段，实现复杂度增高，而且性能也会大打折扣
