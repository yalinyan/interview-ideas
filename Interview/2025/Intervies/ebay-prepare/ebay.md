# eBay高频系统设计面试题及答案

## 1. 设计一个在线拍卖系统

**题目描述**：
设计一个支持实时竞价的在线拍卖系统，需要处理高并发竞价、倒计时、最高价显示等功能。

**核心组件设计**：

1. **服务架构**：
   - 前端服务：处理用户界面和WebSocket实时通信
   - 竞价服务：处理竞价逻辑和验证
   - 商品服务：管理拍卖商品信息
   - 用户服务：处理用户账户和认证
   - 通知服务：发送出价通知和拍卖结果

2. **数据模型**：
```java
class Auction {
    String id;
    String itemId;
    double currentPrice;
    String currentWinnerId;
    Date startTime;
    Date endTime;
    AuctionStatus status; // 进行中/已结束/已取消
}

class Bid {
    String id;
    String auctionId;
    String userId;
    double amount;
    Date bidTime;
}
```

3. **竞价流程**：
   - 使用WebSocket保持客户端与服务器的实时连接
   - 竞价服务验证新出价必须高于当前价+最小增幅
   - 采用乐观锁处理并发竞价：
```java
public boolean placeBid(String auctionId, String userId, double amount) {
    Auction auction = auctionRepository.findById(auctionId);
    if (auction.status != ACTIVE) return false;
    
    // 乐观锁实现
    int version = auction.getVersion();
    if (amount <= auction.currentPrice + auction.minIncrement) {
        return false;
    }
    
    auction.setCurrentPrice(amount);
    auction.setCurrentWinnerId(userId);
    auction.setVersion(version + 1);
    
    try {
        auctionRepository.update(auction);
        // 保存竞价记录并通知所有参与者
        return true;
    } catch (OptimisticLockingFailureException ex) {
        // 重试或返回失败
        return false;
    }
}
```

4. **扩展性考虑**：
   - 使用Redis缓存热门拍卖数据
   - 竞价服务采用分片处理不同拍卖
   - 使用Kafka处理竞价事件流

## 2. 设计商品推荐系统

**题目描述**：
为eBay设计一个个性化商品推荐系统，需要考虑用户历史行为、实时行为、商品相似性等因素。

**解决方案**：

1. **数据收集层**：
   - 用户显式数据：收藏、购买、评价
   - 用户隐式数据：浏览时长、搜索记录、点击流
   - 商品数据：类别、标签、价格区间

2. **推荐算法组合**：
```
推荐结果 = α×协同过滤 + β×内容相似度 + γ×实时行为 + δ×热门商品
```
(其中α,β,γ,δ为可调权重)

3. **架构设计**：
   - 离线计算：每晚用MapReduce计算用户相似度矩阵
   - 近线计算：每小时用Spark更新用户兴趣模型
   - 在线服务：实时响应推荐请求

4. **伪代码实现**：
```java
public List<Item> getRecommendations(String userId) {
    // 从缓存获取实时推荐
    List<Item> realtimeRecs = realtimeEngine.getRecs(userId);
    
    // 获取离线推荐
    List<Item> cfRecs = cfModel.getRecs(userId);
    List<Item> contentRecs = contentModel.getRecs(userId);
    
    // 融合推荐结果
    return hybridMerge(
        realtimeRecs,  // 权重0.4
        cfRecs,        // 权重0.3 
        contentRecs,    // 权重0.2
        hotItems       // 权重0.1
    );
}
```

5. **性能优化**：
   - 为活跃用户预计算推荐结果
   - 使用FAISS进行相似商品向量快速检索
   - 实施降级策略：当个性化推荐不可用时返回热门商品

## 3. 设计分布式键值存储系统

**题目描述**：
设计一个类似Redis的分布式键值存储系统，要求支持高可用、持久化和水平扩展。

**设计方案**：

1. **数据分片**：
   - 采用一致性哈希将键分布到不同节点
   - 虚拟节点解决数据倾斜问题

2. **复制与一致性**：
   - 每个分片有3个副本(1主2从)
   - 使用Raft协议保证副本一致性
   - 读写配置：
     - 写：必须成功写入所有副本
     - 读：可以从任意副本读取

3. **存储引擎**：
   - 内存中使用跳表(sorted set)和哈希表
   - 持久化采用WAL(Write-Ahead Log) + SSTable
   - 定期压缩合并SSTable文件

4. **故障处理**：
```java
public void handleNodeFailure(Node failedNode) {
    // 1. 检测故障
    if (heartbeatMonitor.isNodeDown(failedNode)) {
        // 2. 重新分配哈希环上的分区
        HashRing.reassignPartitions(failedNode);
        
        // 3. 从其他副本恢复数据
        for (Partition p : failedNode.partitions) {
            Node newPrimary = p.replicas.get(0);
            newPrimary.promoteToLeader(p);
        }
    }
}
```

5. **扩展性设计**：
   - 客户端缓存常用键值对
   - 支持动态添加/移除节点
   - 采用gossip协议传播集群状态

## 系统设计回答技巧

1. **问题澄清**：
   - 明确需求：QPS、数据规模、延迟要求
   - 询问假设：是否需要考虑国际化？数据一致性级别？

2. **设计步骤**：
   ```
   1. 估算规模 → 2. 设计接口 → 3. 数据模型 → 
   4. 高层设计 → 5. 详细组件 → 6. 识别瓶颈
   ```

3. **eBay特别关注点**：
   - 强调拍卖场景的并发控制
   - 考虑全球用户的数据分区
   - 讨论如何防止虚假竞价等欺诈行为

4. **常见优化方向**：
   - 读写分离
   - 多级缓存(本地+分布式)
   - 异步处理非关键路径
   - 优雅降级策略

建议针对eBay的业务特点，在系统设计中特别关注拍卖机制、竞价公平性、商品搜索相关性等电商特有问题的解决方案。

# 直播业务系统设计高频题目及答案

## 1. 设计一个直播平台（类似Twitch或抖音直播）

### 题目描述
设计一个支持百万级用户同时在线观看的直播平台，需要包含主播开播、观众观看、弹幕互动、礼物打赏等功能。

### 核心设计方案

**架构图**：
```
[主播端] → [推流服务器] → [转码集群] → [CDN边缘节点] ← [观众端]
                ↓               ↓
           [消息队列]       [存储服务]
                ↓
         [互动服务集群] → [数据库]
```

**关键组件**：

1. **视频流处理**：
   - 使用RTMP协议接收主播推流
   - 转码集群将流转为不同分辨率（1080p/720p/480p）
   - 通过CDN分发视频流（HLS或DASH协议）

2. **弹幕系统设计**：
```java
// 弹幕消息结构
class Danmu {
    String liveId;    // 直播间ID
    String userId;    // 用户ID
    String content;   // 弹幕内容
    long timestamp;   // 发送时间
    String color;     // 弹幕颜色
}

// 弹幕分发服务
public class DanmuService {
    private Map<String, List<WebSocketSession>> liveRoomConnections;
    
    public void sendDanmu(String liveId, Danmu danmu) {
        // 1. 存储到消息队列
        kafkaTemplate.send("danmu-topic", liveId, danmu);
        
        // 2. 实时广播
        List<WebSocketSession> sessions = liveRoomConnections.get(liveId);
        if (sessions != null) {
            for (WebSocketSession session : sessions) {
                session.sendMessage(new TextMessage(toJson(danmu)));
            }
        }
    }
}
```

3. **高并发优化**：
   - 弹幕消息使用Redis Pub/Sub进行房间级广播
   - 热门直播间弹幕采用合并批处理减少网络开销
   - 非活跃观众延迟1-3秒接收以平滑流量峰值

## 2. 设计直播电商系统（类似淘宝直播）

### 题目描述
设计一个支持商品实时讲解、在线下单的直播电商系统，需要处理高并发观看和瞬时抢购场景。

### 解决方案

**核心流程**：
```
1. 主播添加商品 → 2. 讲解商品 → 3. 观众点击购买 → 
4. 生成订单 → 5. 库存扣减 → 6. 订单支付
```

**关键设计**：

1. **商品库存管理**：
```java
// 使用Redis+Lua保证原子性扣减
String luaScript = 
  "if redis.call('exists', KEYS[1]) == 1 then\n" +
  "    local stock = tonumber(redis.call('get', KEYS[1]))\n" +
  "    if stock >= tonumber(ARGV[1]) then\n" +
  "        return redis.call('decrby', KEYS[1], ARGV[1])\n" +
  "    end\n" +
  "end\n" +
  "return -1";

public boolean deductStock(String itemId, int num) {
    Long result = redisTemplate.execute(
        new DefaultRedisScript<>(luaScript, Long.class),
        Collections.singletonList("stock:" + itemId),
        String.valueOf(num));
    return result != null && result >= 0;
}
```

2. **秒杀系统设计**：
   - 分层过滤：CDN → 缓存 → 数据库
   - 预扣库存：活动开始前预热库存到Redis
   - 请求限流：令牌桶算法控制下单QPS

3. **订单系统**：
   - 使用RocketMQ处理订单创建异步流程
   - 采用本地消息表保证最终一致性

## 3. 设计直播连麦系统（多人视频互动）

### 题目描述
设计一个支持主播与观众实时视频连麦的系统，要求延迟低于500ms。

### 技术方案

**架构设计**：
```
[连麦观众] → [SFU服务器] ← [主播]
    ↑               ↓
[WebRTC]      [混流服务]
    ↓               ↓
[观众端] ← [CDN分发]
```

**关键技术点**：

1. **网络传输优化**：
   - 使用WebRTC协议实现P2P传输
   - 采用QUIC协议改善弱网环境表现
   - 智能选路：根据网络质量动态选择传输路径

2. **混流服务**：
```java
// 伪代码：混流参数配置
class MixStreamConfig {
    List<StreamSource> inputs;  // 输入流
    OutputResolution output;    // 输出分辨率
    LayoutTemplate layout;      // 画面布局模板
    BitrateConfig bitrate;      // 码率配置
}

public void startMixStream(String liveId, MixStreamConfig config) {
    // 1. 从源站拉取各连麦者视频流
    // 2. 按照模板进行画面合成
    // 3. 将混流推送到CDN
}
```

3. **QoS保障**：
   - 动态码率调整（DASH）
   - 前向纠错（FEC）
   - 网络拥塞检测与自动降级

## 4. 直播数据分析系统

### 题目描述
设计一个实时分析直播观看数据的系统，需要支持：
- 实时在线人数统计
- 用户行为分析（停留时长、互动频率）
- 热门时段预测

### 设计方案

**数据处理流水线**：
```
[客户端SDK] → [日志收集] → [Flink实时计算] → [OLAP存储]
                                   ↓
                            [实时仪表盘]
```

**关键实现**：

1. **实时计数服务**：
```java
// 使用Flink实现滑动窗口统计
DataStream<UserAction> actions = ...;
actions
    .keyBy(action -> action.getRoomId())
    .window(SlidingEventTimeWindows.of(Size.minutes(5), Slide.seconds(10)))
    .aggregate(new OnlineUserCounter())
    .addSink(new RedisSink());

// 自定义聚合函数
class OnlineUserCounter implements AggregateFunction<UserAction, Set<String>, Integer> {
    public Set<String> createAccumulator() {
        return new HashSet<>();
    }
    public Set<String> add(UserAction action, Set<String> set) {
        set.add(action.getUserId());
        return set;
    }
    public Integer getResult(Set<String> set) {
        return set.size();
    }
    // ... merge方法
}
```

2. **存储方案**：
   - 实时数据：Redis HyperLogLog统计UV
   - 短期存储：Elasticsearch做行为分析
   - 长期存储：数据仓库（Hive）用于离线分析

## 系统设计考察要点

1. **直播特有挑战**：
   - 高带宽成本优化
   - 内容审核实时性要求
   - 突发流量处理（开播/下播时刻）

2. **电商直播特殊需求**：
   - 商品-直播关联管理
   - 即时购买体验优化
   - 防刷单机制

3. **回答技巧**：
   - 明确延迟要求（互动直播 vs 普通观看）
   - 讨论不同规模下的架构演进
   - 强调成本控制策略（如P2P-CDN混合方案）

建议准备时重点关注视频流处理、实时互动、高并发优化等直播特有问题的解决方案，并结合具体业务场景（如电商、游戏、教育等）进行差异化设计。







1. LeetCode 429: N-ary Tree Level Order Traversal
Problem: Given an N-ary tree, return the level order traversal of its nodes' values.
Similarity: This problem involves traversing a tree structure level by level, which is similar to printing or processing the tree structure in your code.
2. LeetCode 105: Construct Binary Tree from Preorder and Inorder Traversal
Problem: Given preorder and inorder traversal of a binary tree, construct the binary tree.
Similarity: This problem involves constructing a tree from given data, similar to how you are building a tree from the CSV file.
3. LeetCode 589: N-ary Tree Preorder Traversal
Problem: Given the root of an N-ary tree, return the preorder traversal of its nodes' values.
Similarity: This problem involves traversing an N-ary tree, which is similar to your printTree method.
4. LeetCode 590: N-ary Tree Postorder Traversal
Problem: Given the root of an N-ary tree, return the postorder traversal of its nodes' values.
Similarity: This problem also involves traversing an N-ary tree, but in postorder, which is another way to process hierarchical data.
5. LeetCode 652: Find Duplicate Subtrees
Problem: Given the root of a binary tree, return all duplicate subtrees.
Similarity: This problem involves identifying and handling shared or duplicate structures in a tree, which is conceptually similar to optimizing your tree to avoid redundant nodes.
6. LeetCode 863: All Nodes Distance K in Binary Tree
Problem: Given the root of a binary tree, a target node, and an integer K, return all nodes that are K distance from the target node.
Similarity: This problem involves traversing a tree and processing nodes based on their relationships, similar to how you traverse and process nodes in your tree.
7. LeetCode 1514: Path with Maximum Probability
Problem: Given a graph, find the path with the maximum probability from a start node to an end node.
Similarity: While this is a graph problem, it shares similarities with your tree traversal, as both involve navigating hierarchical or connected structures.