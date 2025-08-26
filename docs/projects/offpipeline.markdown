# OffPipeline.mis管理端

## 1.整体架构

```mermaid
graph TB
    subgraph "用户层"
        Web[Web前端界面]
        API[REST API客户端]
        RPC[Thrift RPC客户端]
    end
    
    subgraph "offpipeline.mis 系统"
        subgraph "Interface Layer - 用户接口层 (mis-iface)"
            Controller[控制器层]
            RestAPI[REST API]
            ThriftService[Thrift服务]
        end
        
        subgraph "Application Layer - 应用服务层 (mis-app)"
            AppService[应用服务]
            DTO[数据传输对象]
            Converter[转换器]
            InfraAPI[基础设施接口]
        end
        
        subgraph "Domain Layer - 领域层 (mis-domain)"
            Entity[领域实体]
            ValueObject[值对象]
            DomainService[领域服务]
            Repository[仓储接口]
            DomainEnum[领域枚举]
        end
        
        subgraph "Infrastructure Layer - 基础设施层 (mis-infra)"
            RepoImpl[仓储实现]
            DAL[数据访问层]
            External[外部服务集成]
            Config[配置管理]
        end
    end
    
    subgraph "外部系统"
        subgraph "美团内部服务"
            MTMSP[MTMSP任务调度平台]
            SSO[SSO单点登录]
            MafKa[MafKa消息队列]
            DaXiang[大象消息平台]
            Cellar[Cellar缓存]
        end
        
        subgraph "第三方服务"
            S3[AWS S3存储]
            K8S[Kubernetes]
            MySQL[MySQL数据库]
        end
    end
    
    Web --> Controller
    API --> RestAPI
    RPC --> ThriftService
    
    Controller --> AppService
    RestAPI --> AppService
    ThriftService --> AppService
    
    AppService --> Entity
    AppService --> DomainService
    AppService --> InfraAPI
    
    Entity --> Repository
    DomainService --> Repository
    
    InfraAPI --> RepoImpl
    RepoImpl --> DAL
    External --> MTMSP
    External --> SSO
    External --> MafKa
    External --> DaXiang
    External --> Cellar
    External --> S3
    External --> K8S
    DAL --> MySQL
```

## 2.核心业务流程
```mermaid
graph TB
    subgraph "🔄 端到端机器学习流水线"
        direction LR
        
        subgraph "📊 数据层"
            direction LR
            DS[数据源] --> SG[样本生成] --> FE[特征工程] --> DV[数据验证]
        end
        
        subgraph "🧪 实验层"
            direction LR
            EG[实验组] --> EV[实验版本] --> ET[任务执行] --> ER[结果评估]
        end
        
        subgraph "🤖 模型层"
            direction LR
            MC[模型配置] --> MT[模型训练] --> ME[模型评估] --> MO[模型优化]
        end
        
        subgraph "🚀 部署层"
            direction LR
            MS[模型同步] --> OD[在线部署] --> AB[A/B测试] --> PR[生产发布]
        end
    end
    
    DS --> EG
    SG --> EV
    FE --> ET
    DV --> ER
    
    EG --> MC
    EV --> MT
    ET --> ME
    ER --> MO
    
    MC --> MS
    MT --> OD
    ME --> AB
    MO --> PR
    
    style DS fill:#81d4fa
    style EG fill:#ce93d8
    style MC fill:#a5d6a7
    style MS fill:#ffcc80
```

## 3.数据流向图
```mermaid
graph TB
    subgraph "外部数据源"
        A1[用户请求数据]
        A2[训练任务数据]
        A3[模型文件数据]
        A4[特征数据]
    end
    
    subgraph "Interface Layer 数据入口"
        B1[REST API请求]
        B2[Thrift RPC调用]
        B3[消息队列消费]
    end
    
    subgraph "Application Layer 数据处理"
        C1[DTO数据转换]
        C2[业务逻辑编排]
        C3[状态机控制]
        C4[异步任务调度]
    end
    
    subgraph "Domain Layer 领域数据"
        D1[实验实体数据]
        D2[样本数据模型]
        D3[模型状态数据]
        D4[工作流定义数据]
    end
    
    subgraph "Infrastructure Layer 数据持久化"
        E1[(MySQL数据库)]
        E2[S3对象存储]
        E3[Redis缓存]
        E4[MafKa消息队列]
    end
    
    subgraph "外部系统数据交互"
        F1[MTMSP任务调度]
        F2[Velenmis模型服务]
        F3[大象消息平台]
        F4[SSO用户数据]
    end
    
    A1 --> B1
    A2 --> B3
    A3 --> B2
    A4 --> B1
    
    B1 --> C1
    B2 --> C1
    B3 --> C1
    
    C1 --> C2
    C2 --> C3
    C3 --> C4
    
    C2 --> D1
    C2 --> D2
    C2 --> D3
    C4 --> D4
    
    D1 --> E1
    D2 --> E2
    D3 --> E3
    D4 --> E4
    
    C4 --> F1
    C4 --> F2
    C4 --> F3
    C2 --> F4
    
    E4 --> C3
    F1 --> E4
    F2 --> E4
    
    style A1 fill:#e3f2fd
    style C2 fill:#f3e5f5
    style D1 fill:#e8f5e8
    style E1 fill:#fff8e1
```

## 4.技术栈组成
```mermaid
graph TB
    subgraph "核心技术"
        SB[Spring Boot]
        MB[MyBatis]
        JAVA[Java 8]
    end
    
    subgraph "美团生态"
        XF[XFrame]
        MS[MTMSP]
        MK[Mafka]
    end
    
    subgraph "数据存储"
        MYSQL[(MySQL)]
        REDIS[(Redis)]
        HIVE[(Hive)]
    end
    
    SB --> XF
    MB --> MYSQL
    XF --> MS
    XF --> MK
    MS --> REDIS
    
    style XF fill:#ffe6e6
    style MS fill:#ffe6e6
    style MK fill:#ffe6e6
```
# 缓存
## 本地缓存 Guava Cache
###  缓存简介
Guava Cache 是 Google Guava 库提供的一个本地内存缓存解决方案，设计精巧、性能优秀，广泛应用于Java应用程序中。

###  核心特性
#### 1. 高性能设计
- ConcurrentHashMap 基础：底层基于分段锁实现
- O(1) 访问复杂度：读写操作都是常数时间
- 异步清理：过期条目异步清理，不阻塞正常访问
- 内存友好：自动垃圾回收，避免内存泄漏

#### 2. 丰富的过期策略
```java
// 基于时间的过期
CacheBuilder.newBuilder()
    .expireAfterWrite(10, TimeUnit.MINUTES)     // 写入后过期
    .expireAfterAccess(5, TimeUnit.MINUTES)     // 访问后过期
    .refreshAfterWrite(1, TimeUnit.MINUTES)     // 写入后刷新
```

#### 3. 多种淘汰策略
```java
CacheBuilder.newBuilder()
    .maximumSize(1000)                          // 基于条目数量
    .maximumWeight(10000)                       // 基于权重
    .weigher((key, value) -> value.toString().length()) // 自定义权重计算
```

#### 4. 自动加载机制
```java
LoadingCache<String, String> cache = CacheBuilder.newBuilder()
    .build(new CacheLoader<String, String>() {
        @Override
        public String load(String key) throws Exception {
            return loadFromDatabase(key);  // 缓存未命中时自动加载
        }
    });
```

###  核心API详解

#### 1. 基础缓存操作
```java
// 在SquirrelUtil中的实际使用
private final Cache<String, Object> squirrelCache = CacheBuilder.newBuilder()
        .expireAfterWrite(10, java.util.concurrent.TimeUnit.MINUTES)
        .build();

// 读取操作
Object value = squirrelCache.getIfPresent(key);     // 非阻塞获取
if (value == null) {
    value = loadFromRemote(key);
    squirrelCache.put(key, value);                  // 存入缓存
}

// 写入操作  
squirrelCache.put(key, value);                      // 直接存入
squirrelCache.invalidate(key);                      // 移除特定key
squirrelCache.invalidateAll();                      // 清空所有缓存
```

#### 2. 批量操作
```java
// 批量获取
Map<String, Object> values = cache.getAllPresent(keys);

// 批量写入
Map<String, Object> map = ImmutableMap.of("key1", "value1", "key2", "value2");
cache.putAll(map);

// 批量失效
cache.invalidateAll(keys);
```

#### 3. 统计信息
```java
Cache<String, Object> cache = CacheBuilder.newBuilder()
    .recordStats()  // 开启统计
    .build();

// 获取统计信息
CacheStats stats = cache.stats();
System.out.println("命中率: " + stats.hitRate());
System.out.println("请求总数: " + stats.requestCount());
System.out.println("加载时间: " + stats.averageLoadTime());
```

### 详细配置选项
#### 1. 容量限制配置
```java
CacheBuilder.newBuilder()
    // 基于条目数量限制
    .maximumSize(10000)
    
    // 基于权重限制（更灵活）
    .maximumWeight(100000)
    .weigher(new Weigher<String, Object>() {
        @Override
        public int weigh(String key, Object value) {
            return key.length() + value.toString().length();
        }
    })
```

#### 2. 过期时间配置
```java
CacheBuilder.newBuilder()
    // 写入后过期（适合静态数据）
    .expireAfterWrite(Duration.ofMinutes(10))
    
    // 访问后过期（适合热点数据）  
    .expireAfterAccess(Duration.ofMinutes(5))
    
    // 写入后刷新（后台异步刷新，不阻塞访问）
    .refreshAfterWrite(Duration.ofMinutes(1))
```

#### 3. 并发配置
```java
CacheBuilder.newBuilder()
    .concurrencyLevel(16)  // 并发级别，默认4，影响内部分段数
    .initialCapacity(100)  // 初始容量，避免扩容开销
```

#### 4. 监听器机制
```java
CacheBuilder.newBuilder()
    .removalListener(new RemovalListener<String, Object>() {
        @Override
        public void onRemoval(RemovalNotification<String, Object> notification) {
            String cause = notification.getCause().name();
            logger.info("缓存移除: key={}, cause={}", notification.getKey(), cause);
        }
    })
```

### 内部实现原理
#### 1. 分段锁架构
```java
// Guava Cache 内部结构（简化版）
class LocalCache<K, V> {
    final Segment<K, V>[] segments;  // 默认16个段
    
    static class Segment<K, V> {
        final ReentrantLock lock = new ReentrantLock();  // 每段一个锁
        final AtomicReferenceArray<ReferenceEntry<K, V>> table;  // 哈希表
        
        V get(K key) {
            // 读操作大多数情况下无需加锁
            // 只在必要时获取读锁
        }
        
        V put(K key, V value) {
            lock.lock();  // 写操作需要加锁
            try {
                // 插入逻辑
            } finally {
                lock.unlock();
            }
        }
    }
}
```

#### 2. LRU链表实现
```java
// 每个Segment内部维护访问顺序链表
class Segment<K, V> {
    final Queue<ReferenceEntry<K, V>> accessQueue;  // 访问队列
    final Queue<ReferenceEntry<K, V>> writeQueue;   // 写入队列
    
    void recordAccess(ReferenceEntry<K, V> entry) {
        // 将访问的条目移到队列尾部
        accessQueue.add(entry);
    }
    
    void evictEntries() {
        // 从队列头部移除最久未访问的条目
        while (needsEviction()) {
            ReferenceEntry<K, V> entry = accessQueue.remove();
            removeEntry(entry);
        }
    }
}
```

#### 3. 时间轮过期机制
```java
// 基于时间轮的高效过期检查
class TimerWheel {
    private final long[] buckets = new long[256];  // 时间轮桶
    
    boolean isExpired(ReferenceEntry<K, V> entry, long now) {
        long expireTime = entry.getWriteTime() + expireAfterWriteNanos;
        return now >= expireTime;
    }
    
    void cleanUp() {
        // 异步清理过期条目，不阻塞正常访问
        executor.execute(() -> {
            expireEntries();
            evictEntries();
        });
    }
}
```

### 性能特点分析
#### 1. 读性能
```java
// 读操作性能特点
public V get(K key) {
    int hash = hash(key);
    Segment<K, V> segment = segmentFor(hash);
    
    // 大多数情况下无锁读取
    ReferenceEntry<K, V> entry = segment.getEntry(key, hash);
    if (entry != null && !entry.isExpired()) {
        return entry.getValue();  // 纳秒级返回
    }
    
    return null;
}

// 性能数据（参考）
// - 缓存命中：10-50纳秒
// - 缓存未命中：50-100纳秒  
// - 并发读取：几乎无锁竞争
```

#### 2. 写性能
```java
// 写操作需要获取段锁
public V put(K key, V value) {
    int hash = hash(key);
    Segment<K, V> segment = segmentFor(hash);
    
    segment.lock();  // 获取段锁
    try {
        // 插入或更新条目
        return segment.put(key, hash, value);  // 微秒级操作
    } finally {
        segment.unlock();
    }
}

// 写性能特点：
// - 单次写入：1-10微秒
// - 并发写入：16个段并行，高吞吐
// - 扩容开销：渐进式rehash，平摊O(1)
```

#### 3. 内存效率
```java
// 内存使用优化
class ReferenceEntry<K, V> {
    final K key;
    volatile V value;
    final int hash;           // 缓存hash值，避免重复计算
    ReferenceEntry<K, V> next; // 链表指针，解决hash冲突
    
    // 时间戳字段（按需使用）
    volatile long accessTime;  // 访问时间
    volatile long writeTime;   // 写入时间
}

// 内存开销：
// - 每个条目额外开销：32-48字节
// - 相比ConcurrentHashMap增加约50%内存使用
// - 通过WeakReference支持自动GC
```

### 适用场景
#### 1. 理想场景

```java

// ✅ 适合的使用场景

// 1. 热点数据缓存
Cache<String, UserProfile> userCache = CacheBuilder.newBuilder()
    .maximumSize(10000)
    .expireAfterWrite(30, TimeUnit.MINUTES)
    .build();

// 2. 计算结果缓存  
Cache<String, ExpensiveResult> computeCache = CacheBuilder.newBuilder()
    .maximumSize(1000)
    .build();

// 3. 配置数据缓存（如SquirrelUtil中的使用）
Cache<String, Map<String, Object>> configCache = CacheBuilder.newBuilder()
    .expireAfterWrite(10, TimeUnit.MINUTES)
    .build();

// 4. API调用结果缓存
Cache<String, ApiResponse> apiCache = CacheBuilder.newBuilder()
    .expireAfterAccess(5, TimeUnit.MINUTES)
    .refreshAfterWrite(1, TimeUnit.MINUTES)
    .build();
```

#### 2. 不适合的场景

```java

// ❌ 不适合的使用场景

// 1. 大数据集缓存（占用太多JVM内存）
// Cache<String, LargeObject> largeCache; // 不推荐

// 2. 持久化存储需求
// Cache不是持久化存储，重启后数据丢失

// 3. 分布式缓存需求  
// Guava Cache只是本地缓存，无法跨JVM共享

// 4. 强一致性要求
// 无法保证多实例间的缓存一致性
```
### 最佳实践
#### 1. 合理配置缓存大小

```java

Apply
// 根据应用内存情况配置
Cache<String, Object> cache = CacheBuilder.newBuilder()
    .maximumSize(Runtime.getRuntime().maxMemory() / 1024 / 100)  // 约占用1%堆内存
    .build();
```

#### 2. 选择合适的过期策略

```java
// 不同场景的过期策略选择
CacheBuilder.newBuilder()
    .expireAfterWrite(Duration.ofMinutes(10))    // 静态数据：写入后过期
    .expireAfterAccess(Duration.ofMinutes(5))    // 热点数据：访问后过期  
    .refreshAfterWrite(Duration.ofMinutes(1))    // 实时数据：后台刷新
```

#### 3. 监控缓存效果

```java
@Component
public class CacheMonitor {
    
    @Scheduled(fixedRate = 60000)  // 每分钟输出统计
    public void logCacheStats() {
        CacheStats stats = cache.stats();
        
        logger.info("缓存统计: 命中率={:.2f}%, 请求数={}, 加载次数={}, 平均加载时间={:.2f}ms",
            stats.hitRate() * 100,
            stats.requestCount(), 
            stats.loadCount(),
            stats.averageLoadTime() / 1_000_000.0);
        
        // 命中率过低时告警
        if (stats.hitRate() < 0.8) {
            logger.warn("缓存命中率过低: {:.2f}%", stats.hitRate() * 100);
        }
    }
}
```

#### 4. 避免缓存穿透

```java
// 使用Loading Cache避免缓存穿透
LoadingCache<String, Optional<User>> userCache = CacheBuilder.newBuilder()
    .build(new CacheLoader<String, Optional<User>>() {
        @Override
        public Optional<User> load(String userId) {
            User user = userService.findById(userId);
            return Optional.ofNullable(user);  // 即使为null也缓存，避免穿透
        }
    });

// 使用时
Optional<User> userOpt = userCache.get(userId);
if (userOpt.isPresent()) {
    return userOpt.get();
}
```

### ✅ 总结
Guava Cache 的核心优势：
🚀 高性能：基于ConcurrentHashMap，读写都很快
🛠️ 功能丰富：过期、淘汰、统计、监听等功能齐全
💡 使用简单：API设计优雅，学习成本低
🔒 线程安全：分段锁机制，支持高并发
🎯 内存友好：自动淘汰和垃圾回收
📊 监控友好：内置统计信息，便于调优
在 SquirrelUtil 中的应用非常合适：缓存特征配置数据，提供10分钟TTL，在本地缓存和远程Squirrel之间提供高效的中间层！

