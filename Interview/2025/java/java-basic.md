# 中级Java后端开发面试指南

## 核心Java知识

### 面试题1：Java集合框架
**问题**：请解释HashMap的工作原理，包括put和get方法的实现细节？

**答案**：
HashMap基于哈希表实现，使用数组+链表/红黑树结构：
1. put方法：
   - 计算key的hash值
   - 根据hash值确定数组下标
   - 如果该位置为空，直接插入
   - 如果不为空，遍历链表/红黑树：
     - 如果找到相同key，替换value
     - 否则添加到链表末尾
   - 当链表长度超过8且数组长度≥64时，链表转为红黑树
   - 如果size超过threshold(容量×负载因子)，进行扩容

2. get方法：
   - 计算key的hash值
   - 根据hash值定位数组下标
   - 遍历该位置的链表/红黑树
   - 通过equals方法比较key，找到对应节点
   - 返回节点的value

### 面试题2：多线程与并发
**问题**：请解释synchronized和ReentrantLock的区别？

**答案**：
1. 实现机制：
   - synchronized是JVM层面的锁，通过monitor实现
   - ReentrantLock是JDK层面的锁，通过AQS实现

2. 功能区别：
   - ReentrantLock可以尝试非阻塞获取锁(tryLock)
   - ReentrantLock可中断(lockInterruptibly)
   - ReentrantLock可实现公平锁
   - ReentrantLock可绑定多个条件(Condition)

3. 性能：
   - Java 6后synchronized优化后性能接近
   - 高竞争下ReentrantLock可能更好

4. 使用：
   - synchronized更简单，自动释放锁
   - ReentrantLock需要手动释放，更灵活

## Java虚拟机(JVM)

### 面试题3：JVM内存模型
**问题**：请描述JVM内存结构及各部分的作用？

**答案**：
1. 程序计数器：线程私有，记录当前线程执行的字节码行号
2. 虚拟机栈：线程私有，存储栈帧(局部变量表、操作数栈、动态链接、方法出口)
3. 本地方法栈：为Native方法服务
4. 堆：线程共享，存放对象实例和数组，GC主要区域
5. 方法区(元空间)：存储类信息、常量、静态变量、即时编译器编译后的代码
6. 运行时常量池：方法区的一部分，存放编译期生成的各种字面量和符号引用

### 面试题4：垃圾回收
**问题**：请解释G1垃圾收集器的工作原理及其优势？

**答案**：
G1(Garbage-First)是一种面向服务端应用的垃圾收集器：
1. 分区化：将堆划分为多个大小相等的Region(1MB~32MB)
2. 分代收集：逻辑上保留分代概念，物理上不再隔离
3. 工作流程：
   - 初始标记(STW)
   - 并发标记
   - 最终标记(STW)
   - 筛选回收(Evacuation, STW)

优势：
- 可预测的停顿时间模型
- 高吞吐量
- 对大堆处理更高效
- 整体基于标记-整理，局部基于复制算法
- 可有效避免内存碎片

## 框架相关

### 面试题5：Spring框架
**问题**：Spring Bean的生命周期是怎样的？

**答案**：
1. 实例化：通过构造函数或工厂方法创建Bean实例
2. 属性赋值：填充属性(依赖注入)
3. 初始化：
   - 调用Aware接口方法(BeanNameAware, BeanFactoryAware等)
   - 执行BeanPostProcessor的postProcessBeforeInitialization
   - 调用InitializingBean的afterPropertiesSet
   - 调用自定义init-method
   - 执行BeanPostProcessor的postProcessAfterInitialization
4. 使用：Bean已就绪，可被应用使用
5. 销毁：
   - 调用DisposableBean的destroy
   - 调用自定义destroy-method

### 面试题6：Spring Boot
**问题**：Spring Boot自动配置是如何工作的？

**答案**：
Spring Boot自动配置基于以下机制：
1. @EnableAutoConfiguration注解触发自动配置
2. spring-boot-autoconfigure.jar中的META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports文件
3. 条件注解控制配置类的生效条件，如：
   - @ConditionalOnClass
   - @ConditionalOnMissingBean
   - @ConditionalOnProperty
4. 自动配置类通常使用@Configuration注解，并定义@Bean方法
5. 通过spring.factories(Spring Boot 2.7之前)或AutoConfiguration.imports(2.7+)注册自动配置类

## 数据库与ORM

### 面试题7：MyBatis/Hibernate
**问题**：MyBatis中#{}和${}的区别是什么？

**答案**：
1. #{}：
   - 使用预编译语句，参数占位符?
   - 自动进行类型处理和防SQL注入
   - 更安全，推荐使用
   - 例：`WHERE name = #{name}` → `WHERE name = ?`

2. ${}：
   - 直接字符串替换，不做处理
   - 有SQL注入风险
   - 适用于动态表名、列名等无法使用参数化的情况
   - 例：`ORDER BY ${columnName}` → `ORDER BY user_name`

### 面试题8：事务管理
**问题**：Spring事务传播行为有哪些？请解释PROPAGATION_REQUIRES_NEW的行为？

**答案**：
Spring定义了7种传播行为：
1. REQUIRED(默认)：如果当前存在事务，则加入该事务；否则新建事务
2. SUPPORTS：如果当前存在事务，则加入；否则以非事务方式执行
3. MANDATORY：必须存在事务，否则抛出异常
4. REQUIRES_NEW：总是新建事务，如果当前存在事务，则挂起当前事务
5. NOT_SUPPORTED：以非事务方式执行，如果当前存在事务，则挂起
6. NEVER：以非事务方式执行，如果当前存在事务，则抛出异常
7. NESTED：如果当前存在事务，则在嵌套事务内执行；否则新建事务

PROPAGATION_REQUIRES_NEW行为：
- 总是启动新事务
- 如果已有事务存在，将该事务挂起
- 新事务与原有事务完全独立，各自提交或回滚
- 适用于需要独立记录日志等场景

## 分布式与微服务

### 面试题9：分布式系统
**问题**：分布式系统中如何保证数据一致性？

**答案**：
保证分布式系统数据一致性的常见方法：
1. 强一致性方案：
   - 两阶段提交(2PC)：协调者协调参与者完成事务
   - 三阶段提交(3PC)：解决2PC阻塞问题，增加预提交阶段
   - TCC(Try-Confirm-Cancel)：业务层面实现补偿

2. 最终一致性方案：
   - 消息队列：通过可靠消息实现异步数据同步
   - 事件溯源：记录状态变化事件，通过重放恢复状态
   - 补偿事务：定时任务检查并修复不一致数据

3. 其他技术：
   - 分布式锁：控制并发访问
   - 版本号/时间戳：检测数据冲突
   - CRDTs(无冲突复制数据类型)：支持多副本并发修改

### 面试题10：微服务
**问题**：Spring Cloud中服务发现是如何工作的？

**答案**：
Spring Cloud服务发现通常通过Eureka实现：
1. 服务注册：
   - 服务启动时向Eureka Server注册自己的元数据(IP、端口、健康检查URL等)
   - 默认每30秒发送心跳续约

2. 服务发现：
   - 客户端通过Eureka Client查询服务注册表
   - Eureka Server返回可用服务实例列表
   - 客户端缓存注册表信息，定期更新(默认30秒)

3. 服务剔除：
   - 如果服务实例90秒未发送心跳，Eureka Server将其标记为下线
   - 通过自我保护机制防止网络分区时过度剔除

4. 负载均衡：
   - Ribbon客户端从Eureka获取服务列表
   - 根据负载均衡策略(轮询、随机等)选择实例

## 系统设计与性能优化

### 面试题11：缓存
**问题**：如何解决缓存穿透、缓存击穿和缓存雪崩问题？

**答案**：
1. 缓存穿透(查询不存在数据)：
   - 布隆过滤器拦截无效请求
   - 缓存空对象(设置较短过期时间)
   - 接口层增加参数校验

2. 缓存击穿(热点key突然失效)：
   - 设置热点数据永不过期
   - 互斥锁：第一个请求重建缓存，其他请求等待
   - 逻辑过期：异步更新缓存

3. 缓存雪崩(大量key同时失效)：
   - 分散缓存过期时间(基础时间+随机值)
   - 多级缓存(本地缓存+分布式缓存)
   - 熔断降级机制
   - 集群部署提高可用性

### 面试题12：高并发
**问题**：如何设计一个秒杀系统？

**答案**：
秒杀系统设计要点：
1. 架构设计：
   - 前后端分离
   - 动静分离(静态资源CDN加速)
   - 微服务化(独立秒杀服务)

2. 流量控制：
   - 限流(令牌桶、漏桶算法)
   - 队列削峰(消息队列缓冲请求)
   - 验证码/答题延缓请求

3. 数据层优化：
   - 库存预热(Redis缓存)
   - 异步扣减库存(消息队列+数据库)
   - 分布式锁控制并发

4. 降级方案：
   - 熔断机制
   - 兜底数据
   - 服务降级(关闭非核心功能)

5. 监控与恢复：
   - 实时监控系统指标
   - 快速故障转移
   - 预案演练

## 编码题示例

### 面试题13：多线程编程
**问题**：实现一个线程安全的阻塞队列

```java
public class BlockingQueue<T> {
    private final Queue<T> queue = new LinkedList<>();
    private final int capacity;
    private final Lock lock = new ReentrantLock();
    private final Condition notEmpty = lock.newCondition();
    private final Condition notFull = lock.newCondition();

    public BlockingQueue(int capacity) {
        this.capacity = capacity;
    }

    public void put(T element) throws InterruptedException {
        lock.lock();
        try {
            while (queue.size() == capacity) {
                notFull.await();
            }
            queue.add(element);
            notEmpty.signal();
        } finally {
            lock.unlock();
        }
    }

    public T take() throws InterruptedException {
        lock.lock();
        try {
            while (queue.isEmpty()) {
                notEmpty.await();
            }
            T item = queue.remove();
            notFull.signal();
            return item;
        } finally {
            lock.unlock();
        }
    }
}
```

### 面试题14：算法与数据结构
**问题**：实现LRU缓存

```java
public class LRUCache<K, V> {
    private final int capacity;
    private final Map<K, Node<K, V>> map;
    private final DoublyLinkedList<K, V> list;

    public LRUCache(int capacity) {
        this.capacity = capacity;
        this.map = new HashMap<>();
        this.list = new DoublyLinkedList<>();
    }

    public V get(K key) {
        if (!map.containsKey(key)) {
            return null;
        }
        Node<K, V> node = map.get(key);
        list.moveToHead(node);
        return node.value;
    }

    public void put(K key, V value) {
        if (map.containsKey(key)) {
            Node<K, V> node = map.get(key);
            node.value = value;
            list.moveToHead(node);
            return;
        }
        
        if (map.size() == capacity) {
            Node<K, V> tail = list.removeTail();
            map.remove(tail.key);
        }
        
        Node<K, V> newNode = new Node<>(key, value);
        map.put(key, newNode);
        list.addToHead(newNode);
    }

    private static class Node<K, V> {
        K key;
        V value;
        Node<K, V> prev;
        Node<K, V> next;
        
        public Node(K key, V value) {
            this.key = key;
            this.value = value;
        }
    }

    private static class DoublyLinkedList<K, V> {
        private Node<K, V> head;
        private Node<K, V> tail;
        
        public void addToHead(Node<K, V> node) {
            if (head == null) {
                head = tail = node;
            } else {
                node.next = head;
                head.prev = node;
                head = node;
            }
        }
        
        public void moveToHead(Node<K, V> node) {
            if (node == head) return;
            if (node == tail) {
                tail = tail.prev;
                tail.next = null;
            } else {
                node.prev.next = node.next;
                node.next.prev = node.prev;
            }
            node.prev = null;
            node.next = head;
            head.prev = node;
            head = node;
        }
        
        public Node<K, V> removeTail() {
            if (tail == null) return null;
            Node<K, V> removed = tail;
            if (head == tail) {
                head = tail = null;
            } else {
                tail = tail.prev;
                tail.next = null;
            }
            return removed;
        }
    }
}
```

以上问题和答案涵盖了中级Java后端开发需要掌握的核心知识点，可根据候选人实际经验和岗位要求适当调整难度和深度。