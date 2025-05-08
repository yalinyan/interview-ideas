以下是 **LeetCode 上所有系统设计相关的题目**，按类别和难度整理，包含题目编号和核心考察点：

---

### **一、LeetCode 原生系统设计题**
#### **1. 基础系统设计**
| 题号 | 题目名称                     | 难度   | 核心考察点                     |
|------|------------------------------|--------|------------------------------|
| 146  | [LRU 缓存](https://leetcode.com/problems/lru-cache/) | Hard   | 哈希表 + 双向链表、缓存淘汰策略 |
| 460  | [LFU 缓存](https://leetcode.com/problems/lfu-cache/) | Hard   | 多层哈希表 + 频率统计          |
| 355  | [设计推特](https://leetcode.com/problems/design-twitter/) | Medium | 社交媒体 Feed 流设计           |
| 642  | [设计搜索自动补全系统](https://leetcode.com/problems/design-search-autocomplete-system/) | Hard   | Trie 树 + 热度排序             |
| 588  | [设计内存文件系统](https://leetcode.com/problems/design-in-memory-file-system/) | Hard   | 目录树结构、文件操作模拟       |
| 635  | [设计日志存储系统](https://leetcode.com/problems/design-log-storage-system/) | Medium | 时间序列数据存储、范围查询     |
| 1603 | [设计停车场系统](https://leetcode.com/problems/design-parking-system/) | Easy   | 面向对象设计、资源分配         |
| 1472 | [设计浏览器历史记录](https://leetcode.com/problems/design-browser-history/) | Medium | 双向链表、前进后退逻辑         |
| 1396 | [设计地铁系统](https://leetcode.com/problems/design-underground-system/) | Medium | 数据聚合统计、面向对象设计     |
| 901  | [股票价格跨度](https://leetcode.com/problems/online-stock-span/) | Medium | 单调栈、实时数据流处理         |

#### **2. 并发/多线程设计**
| 题号 | 题目名称                     | 难度   | 核心考察点                     |
|------|------------------------------|--------|------------------------------|
| 1115 | [交替打印 FooBar](https://leetcode.com/problems/print-foobar-alternately/) | Medium | 线程同步（信号量/锁）         |
| 1116 | [打印零与奇偶数](https://leetcode.com/problems/print-zero-even-odd/) | Medium | 多线程协作                   |
| 1188 | [设计阻塞队列](https://leetcode.com/problems/design-bounded-blocking-queue/) | Medium | 生产者-消费者模型             |
| 1226 | [哲学家进餐问题](https://leetcode.com/problems/the-dining-philosophers/) | Medium | 死锁预防、资源分配策略       |

---

### **二、可扩展为系统设计的算法题**
以下题目虽然是算法题，但涉及系统设计核心思想：
| 题号 | 题目名称                     | 关联系统设计场景               |
|------|------------------------------|------------------------------|
| 208  | [实现 Trie 树](https://leetcode.com/problems/implement-trie-prefix-tree/) | 搜索引擎自动补全               |
| 706  | [设计哈希映射](https://leetcode.com/problems/design-hashmap/) | 分布式哈希表设计               |
| 622  | [设计循环队列](https://leetcode.com/problems/design-circular-queue/) | 消息队列缓冲设计               |
| 239  | [滑动窗口最大值](https://leetcode.com/problems/sliding-window-maximum/) | 实时监控系统（如流量控制）     |
| 23   | [合并K个排序链表](https://leetcode.com/problems/merge-k-sorted-lists/) | 分布式归并排序（如 MapReduce） |

---

### **三、数据库设计题**
| 题号 | 题目名称                     | 关联系统设计场景               |
|------|------------------------------|------------------------------|
| 176  | [第二高的薪水](https://leetcode.com/problems/second-highest-salary/) | 数据库索引优化                 |
| 185  | [部门工资最高的员工](https://leetcode.com/problems/department-top-three-salaries/) | 复杂 SQL 查询（如 HR 系统）    |
| 534  | [游戏玩法分析 III](https://leetcode.com/problems/game-play-analysis-iii/) | 用户行为数据分析系统           |

---

### **四、LeetCode 缺少但面试高频的系统设计题**
LeetCode 系统设计题较少，以下场景需结合其他资源练习：
1. **短链系统**（类似 LeetCode 534 的扩展）
2. **分布式聊天系统**（消息队列、在线状态）
3. **电商秒杀系统**（高并发、库存一致性）
4. **分布式ID生成器**（雪花算法、UUID）
5. **推荐系统**（协同过滤、实时排序）

---

### **如何高效练习？**
1. **LeetCode 题目**：优先完成 **146 (LRU)、460 (LFU)、355 (推特)、642 (自动补全)**。
2. **扩展练习**：
   - 将算法题（如 Trie 树、哈希表）扩展到分布式场景。
   - 用面向对象思想实现题目（如停车场系统）。
3. **系统设计四步法**：
   ```text
   1. 明确需求（QPS、数据量）
   2. 画架构图（服务、存储、缓存）
   3. 分模块设计（API、数据库表）
   4. 解决瓶颈（扩容、一致性）
   ```

