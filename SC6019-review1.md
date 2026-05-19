# SC6019 Blockchain - Lecture 07: Privacy & Scalability  
# SC6019 区块链 - 第 7 讲：隐私与可扩展性

> Source / 来源：`SC6019 - Lecture 07.pdf`  
> Lecturer / 授课教师：Neil HAN  
> Theme / 主题：Blockchain Scalability, Scalability Issues, Blockchain Trilemma, Scaling Landscape, Case Studies  
> 主题中文：区块链可扩展性、扩容问题、区块链三难困境、扩容方案版图、案例分析

---

## 0. How to Use This Note / 使用说明

| English | 中文 |
|---|---|
| This document converts the full PDF lecture into a detailed bilingual Markdown course note. It preserves all slide-level content, reorganizes it into lecture-style notes, and adds explanatory context for concepts that are only shown visually in the slides. | 本文档将 PDF 课件完整整理成中英对照的 Markdown 课程笔记。它保留每页幻灯片的核心内容，并将其重组为更适合复习的课程讲义，同时补充了幻灯片中只通过图示表达的背景解释。 |
| Page numbers refer to the original PDF pages. | 页码均对应原始 PDF 的页数。 |
| The lecture focuses on why blockchain systems are hard to scale, why naive throughput improvements are insufficient, and how scalability interacts with decentralization, security, composability, and computation integrity. | 本讲重点解释为什么区块链系统难以扩展，为什么单纯提高 TPS 不足以解决问题，以及可扩展性如何与去中心化、安全性、可组合性和计算完整性相互制约。 |

---

## 1. Lecture Roadmap / 课程路线图

### 1.1 Position of Lecture 07 in the Second Half / 第 7 讲在后半学期中的位置

| Week | English Topic | 中文主题 | Notes / 说明 |
|---:|---|---|---|
| 7 | Introduction to Scalability Issues in Blockchain; Understanding the Blockchain Trilemma; Projects | 区块链可扩展性问题导论；理解区块链三难困境；项目 | This lecture establishes the conceptual foundation for all following scaling lectures. / 本讲为后续所有扩容主题建立概念基础。 |
| 8 | Layer 1 Solutions: On-Chain Scalability | Layer 1 解决方案：链上扩容 | How base-layer protocols attempt to increase capacity. / 基础链协议如何提升容量。 |
| 9 | Layer 2 Solutions: Off-Chain Scalability | Layer 2 解决方案：链下扩容 | Off-chain execution and settlement models. / 链下执行与链上结算模型。 |
| 10 | Rollups: Optimistic and Zero Knowledge Rollups | Rollups：乐观 Rollup 与零知识 Rollup | Specialized L2 techniques. / 专门的二层扩容技术。 |
| 11 | Data Availability Scaling, EVM Scaling and Consensus Scaling | 数据可用性扩容、EVM 扩容与共识扩容 | Scaling bottlenecks beyond execution. / 执行之外的扩容瓶颈。 |
| 12 | The Scaling to Your Own Blockchain; Quiz | 扩展到自己的区块链；测验 | Application-specific chains and final assessment. / 应用链与最终评估。 |

**Key takeaway / 核心结论：**

| English | 中文 |
|---|---|
| Lecture 07 is not primarily about one specific scaling technology. It introduces the problem space: what “scalability” means, why it is hard, and which trade-offs dominate blockchain system design. | 第 7 讲并不是直接讲某一种具体扩容技术，而是建立问题空间：什么是“可扩展性”、为什么它困难，以及区块链系统设计中哪些权衡最关键。 |

---

## 2. References / 参考书目

| English | 中文 |
|---|---|
| **Mastering Blockchain - Third Edition**, by Imran Bashir. | **《Mastering Blockchain》第三版**，作者 Imran Bashir。 |
| **Mastering Ethereum, 2nd Edition**, by Carlo Parisi, Alessandro Mazza, Niccolo Pozzolini. | **《Mastering Ethereum》第二版**，作者 Carlo Parisi、Alessandro Mazza、Niccolo Pozzolini。 |

**Study suggestion / 学习建议：**

| English | 中文 |
|---|---|
| Use the references for background reading, but this lecture’s key value is in connecting scalability definitions to practical blockchain architecture constraints. | 参考书适合补背景知识，但本讲最重要的是把“可扩展性”的定义与真实区块链架构约束连接起来。 |

---

## 3. Introduction to Scalability Issues in Blockchain / 区块链可扩展性问题导论

### 3.1 Problems and Challenges / 问题与挑战

| English | 中文 |
|---|---|
| The lecture begins with the broad problem: blockchain scalability. | 本讲从一个总问题开始：区块链可扩展性。 |
| Public blockchains are designed to be open, verifiable, and decentralized, but these design goals impose major performance costs. | 公链的设计目标是开放、可验证和去中心化，但这些目标会带来显著的性能成本。 |
| Unlike centralized systems, a blockchain cannot simply rely on a trusted server to compute quickly and update state privately. | 与中心化系统不同，区块链不能简单依赖一个可信服务器快速计算并私下更新状态。 |

---

## 4. Transaction Speed Comparison / 交易速度对比

### 4.1 Visual Point from Page 5 / 第 5 页图示重点

The slide compares cryptocurrency transaction speeds with Visa and PayPal.

该页用图示比较了加密货币系统与 Visa、PayPal 的交易处理速度。

| System / 系统 | Approximate TPS shown in slide / 幻灯片中显示的大致 TPS | Interpretation / 含义 |
|---|---:|---|
| Visa | 24,000 | A highly optimized centralized payment network. / 高度优化的中心化支付网络。 |
| Ripple | 1,500 | Much higher than early public chains, but architecture differs. / 高于早期公链，但架构不同。 |
| PayPal | 193 | Centralized digital payment service. / 中心化数字支付服务。 |
| Litecoin | 56 | Public cryptocurrency network. / 公共加密货币网络。 |
| Dash | 48 | Public cryptocurrency network. / 公共加密货币网络。 |
| Ethereum | 20 | General-purpose smart contract blockchain. / 通用智能合约区块链。 |
| Bitcoin | 7 | Highly decentralized and conservative base-layer design. / 高去中心化、保守的基础层设计。 |

### 4.2 Meaning of the Comparison / 对比的含义

| English | 中文 |
|---|---|
| The comparison is not only about raw speed. It motivates the central question: why are public blockchains so much slower than centralized payment systems? | 这个对比不只是为了说明速度差异，而是引出核心问题：为什么公链比中心化支付系统慢这么多？ |
| The answer is not simply “bad engineering.” Public blockchains intentionally require many nodes to verify data independently. | 答案并不是简单的“工程做得差”。公链有意要求大量节点独立验证数据。 |
| The speed gap reflects a deeper trade-off between throughput, decentralization, and trust minimization. | 速度差距反映了吞吐量、去中心化与最小信任之间更深层的权衡。 |

---

## 5. Gas Cost and Cost Surge / Gas 成本与费用激增

### 5.1 Gas and Gas Price / Gas 与 Gas 价格

| English | 中文 |
|---|---|
| In Ethereum, transactions consume **gas** because smart contract execution uses computation and storage resources. | 在以太坊中，交易会消耗 **Gas**，因为智能合约执行会使用计算和存储资源。 |
| The slide shows examples such as `3 gas`, `40 gas`, and `21,000 gas`, where 21,000 gas is the standard cost of a basic ETH transfer. | 幻灯片展示了如 `3 gas`、`40 gas`、`21,000 gas` 等例子，其中 21,000 gas 是基础 ETH 转账的标准成本。 |
| The gas price determines how much a user pays per unit of gas. | Gas 价格决定用户每单位 Gas 需要支付多少费用。 |
| Example shown: `10 GWEI x 21,000 gas = 2.1 MGWEI = 0.000021 ETH`. | 幻灯片示例：`10 GWEI x 21,000 gas = 2.1 MGWEI = 0.000021 ETH`。 |

### 5.2 Why Costs Surge / 为什么费用会激增

| English | 中文 |
|---|---|
| Transaction cost increases when network activity increases. | 当网络活动增加时，交易成本会上升。 |
| The mempool contains pending transactions waiting to be included in blocks. | Mempool 中包含等待被打包进区块的待处理交易。 |
| When demand exceeds available block space, users compete by offering higher fees. | 当需求超过可用区块空间时，用户会通过提高费用来竞争。 |
| This creates a fee market: transactions paying higher fees are usually prioritized. | 这形成了费用市场：支付更高手续费的交易通常会被优先处理。 |
| Scalability is therefore also an economic issue: low capacity produces high congestion costs. | 因此，可扩展性也是经济问题：容量不足会导致拥堵费用上升。 |

### 5.3 Important Formula / 重要公式

```text
Transaction fee = Gas used x Gas price
交易费用 = 使用的 Gas 数量 x Gas 单价
```

| English | 中文 |
|---|---|
| Improving scalability can reduce congestion pressure, but the relationship between capacity and fees depends on demand, blockspace markets, and protocol design. | 提升可扩展性可以降低拥堵压力，但容量与费用之间的关系取决于需求、区块空间市场和协议设计。 |

---

## 6. CryptoKitties Architecture / CryptoKitties 架构

### 6.1 Slide Architecture / 幻灯片中的架构

The CryptoKitties example illustrates how a popular blockchain application can congest a public chain.

CryptoKitties 的例子说明了一个热门区块链应用如何造成公链拥堵。

| Component / 组件 | English Description | 中文说明 |
|---|---|---|
| Browser | User interacts through a web browser. | 用户通过浏览器访问应用。 |
| Web server / Front-end | Front-end resources such as `assets`, `css`, and `js`. | Web 服务器提供前端资源，如 `assets`、`css`、`js`。 |
| Ethereum smart contracts | Game logic and ownership-related logic run on Ethereum. | 游戏逻辑与资产所有权相关逻辑在以太坊智能合约上运行。 |
| Ethereum blockchain | Stores transactions and contract state. | 存储交易和合约状态。 |

### 6.2 Why This Matters / 为什么这个案例重要

| English | 中文 |
|---|---|
| CryptoKitties became a classic example of application-driven congestion. | CryptoKitties 是应用驱动型拥堵的经典案例。 |
| Even a seemingly simple game can generate many transactions: breeding, buying, selling, transferring, and updating ownership state. | 即使是一个看似简单的游戏，也可能产生大量交易：繁殖、购买、出售、转移和更新所有权状态。 |
| Since Ethereum required nodes to process these transactions globally, application popularity directly translated into network congestion. | 由于以太坊要求节点全局处理这些交易，应用热度会直接转化为网络拥堵。 |

---

## 7. What Is “Scalability”? / 什么是“可扩展性”？

### 7.1 Ambiguity of the Term / 概念的多义性

The lecture asks whether scalability means any of the following:

本讲提出：可扩展性到底是否意味着以下内容？

| Candidate Meaning / 可能含义 | English Explanation | 中文解释 |
|---|---|---|
| Higher throughput | More transactions per second, usually measured as TPS. | 更高吞吐量，即每秒处理更多交易，通常用 TPS 衡量。 |
| Shorter confirmation/finalization latency | Transactions become confirmed or finalized faster. | 更短确认或最终确定延迟，即交易更快被确认或最终确定。 |
| More transactions with same hardware | The same hardware validates more transactions; better cost efficiency. | 同样硬件验证更多交易，即单位成本下效率更高。 |
| Non-degrading performance with more transactions | Performance remains stable when transaction volume grows. | 交易量上升时性能不明显下降。 |

### 7.2 Why the Definition Matters / 为什么定义很重要

| English | 中文 |
|---|---|
| Different definitions lead to different engineering choices. | 不同定义会导向不同工程方案。 |
| If scalability only means TPS, one may increase block size or reduce block time, but this may harm decentralization or security. | 如果可扩展性只等于 TPS，就可能通过增大区块或缩短出块时间来扩容，但这可能损害去中心化或安全性。 |
| If scalability means maintaining decentralization and security while serving more users, the problem becomes much harder. | 如果可扩展性意味着在服务更多用户的同时保持去中心化和安全性，那么问题就困难得多。 |

---

## 8. Formal Definition of Scalability / 可扩展性的形式化定义

### 8.1 Reference to Mark D. Hill / 对 Mark D. Hill 论文的引用

| English | 中文 |
|---|---|
| The slide references Mark D. Hill’s work, “What is Scalability?”, noting that scalability is frequently claimed but has no universally accepted definition. | 幻灯片引用 Mark D. Hill 的 “What is Scalability?”，指出“可扩展性”经常被声称，但并没有一个被普遍接受的定义。 |
| The abstract shown emphasizes that common usage of scalability is often intuitive rather than rigorous. | 该页展示的摘要强调，可扩展性的常见用法往往是直觉性的，而不是严格定义的。 |

### 8.2 Relevance to Blockchain / 与区块链的关系

| English | 中文 |
|---|---|
| Blockchain scalability is especially hard to define because performance cannot be separated from trust assumptions. | 区块链可扩展性尤其难定义，因为性能不能与信任假设分离。 |
| A centralized database can scale by adding servers, but a public blockchain must preserve verifiability and adversarial robustness. | 中心化数据库可以通过增加服务器扩展，但公链必须保持可验证性和对抗性安全。 |

---

## 9. Attempt 1: Scalability Equals Throughput / 尝试一：可扩展性等于吞吐量

### 9.1 Abstraction / 抽象模型

| English | 中文 |
|---|---|
| Blockchain is abstracted as a service. | 将区块链抽象为一种服务。 |
| Transactions are abstracted as computational tasks of various difficulties. | 将交易抽象为不同难度的计算任务。 |
| Users submit requests; the blockchain processes and returns results. | 用户提交请求，区块链处理并返回结果。 |

### 9.2 Throughput Formula / 吞吐量公式

```text
Throughput = Transactions per block / Block interval
吞吐量 = 每个区块中的交易数 / 出块间隔
```

### 9.3 Naive Scaling Methods / 朴素扩容方法

| Method / 方法 | English | 中文 | Main Risk / 主要风险 |
|---|---|---|---|
| Increase transactions per block | Increase block size or block gas limit so each block contains more transactions. | 增大区块大小或区块 Gas 上限，使每个区块容纳更多交易。 | Higher hardware and bandwidth requirements. / 更高硬件与带宽要求。 |
| Reduce block interval | Produce blocks faster. | 缩短出块间隔，提高出块频率。 | More forks and less stable confirmation. / 更多分叉，确认更不稳定。 |

### 9.4 Why Naive Throughput Scaling Is Not Enough / 为什么朴素 TPS 扩容不够

| English | 中文 |
|---|---|
| Increasing block size raises hardware requirements. If only powerful server farms or ASIC-like specialized participants can keep up, decentralization decreases. | 增大区块会提高硬件要求。如果只有强大的服务器农场或类似 ASIC 的专业参与者才能跟上，去中心化程度会下降。 |
| Shorter block intervals create faster block generation, but also more competing blocks. This increases forks and weakens stable block confirmation. | 缩短出块间隔会加快出块速度，但也会产生更多竞争区块，从而增加分叉并削弱区块确认稳定性。 |
| Therefore, blockchain scalability must consider decentralization and security, not just throughput. | 因此，区块链可扩展性必须同时考虑去中心化和安全性，而不能只看吞吐量。 |

---

## 10. Attempt 2: Wikipedia-Style Scalability / 尝试二：Wikipedia 式可扩展性定义

### 10.1 General Definition / 通用定义

| English | 中文 |
|---|---|
| Scalability is generally described as the property of a system to handle growing work by adding resources. | 一般来说，可扩展性是指系统通过增加资源来处理不断增长工作量的能力。 |

### 10.2 Vertical Scaling / 垂直扩展

| English | 中文 |
|---|---|
| Vertical scaling means adding resources to a single node, such as stronger CPU, more memory, faster storage, or higher bandwidth. | 垂直扩展指给单个节点增加资源，例如更强 CPU、更多内存、更快存储或更高带宽。 |
| In blockchain, vertical scaling can make nodes expensive to run. | 在区块链中，垂直扩展会使运行节点的成本变高。 |
| This leads again to centralization because fewer participants can afford to validate. | 这会再次导致中心化，因为能负担验证成本的参与者变少。 |

### 10.3 Horizontal Scaling / 水平扩展

| English | 中文 |
|---|---|
| Horizontal scaling means adding new nodes to the system. | 水平扩展指向系统中增加更多节点。 |
| In ordinary distributed systems, more nodes can increase capacity. | 在普通分布式系统中，更多节点可能提升容量。 |
| In blockchain, however, adding more validating nodes may slow consensus and increase communication overhead. | 但在区块链中，增加更多验证节点可能会降低共识速度并提高通信开销。 |
| If every node still processes everything, more nodes do not automatically increase throughput. | 如果每个节点仍然处理所有内容，那么节点数量增加并不会自动提高吞吐量。 |

---

## 11. Fundamental Problems in Blockchain Scalability / 区块链可扩展性的根本问题

### 11.1 Core Bottlenecks / 核心瓶颈

The slide identifies three fundamental problems:

幻灯片指出三个根本问题：

| English Term | 中文术语 | Explanation / 解释 |
|---|---|---|
| Replicated computation | 重复计算 | Every full node re-executes or verifies the same transactions. / 每个全节点都重新执行或验证同一批交易。 |
| Replicated storage | 重复存储 | Every full node stores the same blockchain data and state. / 每个全节点都存储同样的链上数据和状态。 |
| Consensus overhead | 共识开销 | Nodes must agree on transaction ordering and state updates. / 节点必须就交易排序和状态更新达成一致。 |

### 11.2 One-Sentence Summary from Slide / 幻灯片原意概括

| English | 中文 |
|---|---|
| “Every node processes all transactions, stores all data, and total ordering of transactions is required.” | “每个节点处理所有交易、存储所有数据，并且需要对交易进行全序排序。” |

### 11.3 Why This Is Fundamental / 为什么这是根本问题

| English | 中文 |
|---|---|
| In a traditional centralized system, work can be partitioned across machines relatively freely. | 在传统中心化系统中，工作可以较自由地分配到不同机器上。 |
| In a public blockchain, replication is a security feature: independent verification reduces trust in any single party. | 在公链中，重复处理本身是安全特性：独立验证降低对单一主体的信任。 |
| But the same replication also limits performance, because all nodes repeat work rather than dividing work. | 但这种重复也限制了性能，因为所有节点都在重复工作，而不是分工处理。 |

---

## 12. Fundamental Challenges for Scalable Blockchain / 可扩展区块链的根本挑战

### 12.1 Desired Direction / 期望方向

| English | 中文 |
|---|---|
| The slide suggests a more scalable direction: every node processes only partial transactions and stores only partial data, with more efficient consensus. | 幻灯片提出更可扩展的方向：每个节点只处理部分交易、存储部分数据，并使用更高效的共识。 |
| This is the intuition behind many scaling approaches such as sharding, rollups, modular blockchains, and data availability sampling. | 这也是分片、Rollup、模块化区块链和数据可用性采样等扩容方案背后的直觉。 |

### 12.2 Three Critical Challenges / 三个关键挑战

| Challenge | 中文 | Explanation / 解释 |
|---|---|---|
| State validity | 状态有效性 | How can the system know that state transitions are correct if not every node executes every transaction? / 如果不是每个节点都执行每笔交易，系统如何知道状态转换是正确的？ |
| Data availability | 数据可用性 | How can nodes verify or reconstruct state if transaction data is missing or withheld? / 如果交易数据缺失或被隐藏，节点如何验证或重建状态？ |
| Byzantine adversary | 拜占庭对手 | How can the system remain safe when some participants are malicious, dishonest, or unavailable? / 当部分参与者恶意、不诚实或不可用时，系统如何保持安全？ |

### 12.3 Key Insight / 核心洞察

| English | 中文 |
|---|---|
| Scaling blockchain means reducing redundant work without losing the ability to detect invalid behavior. | 区块链扩容的本质，是减少重复工作，同时不丧失发现无效行为的能力。 |

---

## 13. Understanding the Blockchain Trilemma / 理解区块链三难困境

### 13.1 Transition / 过渡

| English | 中文 |
|---|---|
| After defining scalability challenges, the lecture moves to the blockchain trilemma. | 在定义可扩展性挑战之后，本讲进入区块链三难困境。 |
| The trilemma frames the tension among scalability, decentralization, and security. | 三难困境刻画了可扩展性、去中心化和安全性之间的张力。 |

---

## 14. System Design - CAP Problem / 系统设计：CAP 问题

### 14.1 CAP Concepts / CAP 概念

| CAP Term | 中文 | Definition / 定义 |
|---|---|---|
| Consistency | 一致性 | Every read receives the most recent write or an error. / 每次读取都能得到最新写入结果，或者返回错误。 |
| Availability | 可用性 | Every request receives a non-error response. / 每个请求都能收到非错误响应。 |
| Partition tolerance | 分区容错性 | The system continues to operate despite arbitrary messages being dropped or delayed between nodes. / 即使节点之间任意消息被丢弃或延迟，系统仍能继续运行。 |

### 14.2 CAP Theorem / CAP 定理

| English | 中文 |
|---|---|
| CAP theorem implies that when a network partition occurs, a distributed system must choose between consistency and availability. | CAP 定理说明，当网络分区发生时，分布式系统必须在一致性和可用性之间做选择。 |
| Blockchain systems are also distributed systems, but with adversarial assumptions and economic incentives layered on top. | 区块链系统也是分布式系统，但还叠加了对抗性假设和经济激励。 |

### 14.3 Why CAP Is Introduced Here / 为什么这里引入 CAP

| English | 中文 |
|---|---|
| CAP is used as an analogy: distributed systems often face unavoidable trade-offs. | CAP 在这里作为类比：分布式系统经常面对不可避免的权衡。 |
| Blockchain trilemma is not identical to CAP, but it plays a similar conceptual role in blockchain design. | 区块链三难困境并不等同于 CAP，但在区块链设计中具有类似的概念作用。 |

---

## 15. The Scalability Trilemma / 可扩展性三难困境

### 15.1 Three Corners / 三个顶点

| Corner | 中文 | Core Question / 核心问题 |
|---|---|---|
| Scalable | 可扩展 | Can the system process high transaction volume and support many users? / 系统能否处理高交易量并支持大量用户？ |
| Decentralized | 去中心化 | How many nodes exist, and are they concentrated in a center of power? / 有多少节点？它们是否集中在少数权力中心？ |
| Secure | 安全 | How resistant is the system to attacks? / 系统对攻击的抵抗力有多强？ |

### 15.2 Visual Examples from the Slide / 幻灯片图示中的例子

| Position in Triangle | Examples Shown | Interpretation / 解释 |
|---|---|---|
| Traditional chains near secure and decentralized | Bitcoin, Ethereum | Prioritize security and decentralization, but limited throughput. / 优先安全与去中心化，但吞吐量有限。 |
| More scalable side of triangle | Solana, Binance Chain | Pursue higher throughput, often with stronger hardware or validator constraints. / 追求更高吞吐量，通常伴随更强硬件或验证者约束。 |

### 15.3 Main Message / 主要信息

| English | 中文 |
|---|---|
| It is difficult to maximize scalability, decentralization, and security simultaneously. | 很难同时最大化可扩展性、去中心化和安全性。 |
| A design that improves one dimension often weakens another. | 改善某一个维度的设计往往会削弱另一个维度。 |

---

## 16. Trilemma Definitions / 三难困境定义

| Concept | English Definition | 中文定义 |
|---|---|---|
| Decentralization | The process of distributing control and decision-making from a centralized entity to a distributed network. | 将控制权和决策权从中心化实体转移到分布式网络的过程。 |
| Security | Measures and protocols used to prevent unauthorized access and attacks on the network. | 防止未经授权访问和网络攻击的措施与协议。 |
| Scalability | The ability of a blockchain to handle growth in transactions, size, and usage without losing performance. | 区块链在交易数量、系统规模和使用量增长时仍保持性能的能力。 |

---

## 17. Trilemma Deep Dive / 三难困境深入分析

### 17.1 Decentralization / 去中心化

| English | 中文 |
|---|---|
| Benefits include trustless operation, fault tolerance, and community control. | 优点包括无需信任的运行方式、容错性和社区控制。 |
| Challenges include wealth concentration and mining or validator power concentration. | 挑战包括财富集中以及矿工或验证者权力集中。 |
| Decentralization is not binary. A system can be more or less decentralized across many dimensions: node count, client diversity, governance, token distribution, and infrastructure dependence. | 去中心化不是非黑即白。一个系统可以在节点数量、客户端多样性、治理、代币分布和基础设施依赖等多个维度上表现出不同程度的去中心化。 |

### 17.2 Security / 安全性

| English | 中文 |
|---|---|
| Security depends on cryptographic techniques and consensus mechanisms. | 安全性依赖密码学技术和共识机制。 |
| Common threats include 51% attacks, double spending, and network attacks. | 常见威胁包括 51% 攻击、双花攻击和网络攻击。 |
| Security in blockchain is adversarial security: the system must remain correct even when some participants actively try to break it. | 区块链安全是对抗性安全：即使部分参与者主动试图破坏系统，系统也必须保持正确。 |

### 17.3 Scalability / 可扩展性

| English | 中文 |
|---|---|
| Current limitations include block size, block interval, and transaction throughput. | 当前限制包括区块大小、出块间隔和交易吞吐量。 |
| User experience is directly affected through transaction cost and speed. | 用户体验会通过交易费用和速度直接受到影响。 |
| Scalability is therefore both a technical and product-level issue. | 因此，可扩展性既是技术问题，也是产品体验问题。 |

---

## 18. Architecture of Centralised Exchange / 中心化交易所架构

### 18.1 Components Shown in Slide / 幻灯片中的组件

The centralized exchange diagram contains the following elements:

中心化交易所架构图包含以下元素：

| Component | 中文 | Role / 作用 |
|---|---|---|
| User | 用户 | Deposits money, buys crypto, sells crypto. / 存钱、买币、卖币。 |
| User bank account | 用户银行账户 | Source or destination for fiat transfers. / 法币转入或转出的来源与目的地。 |
| Victim/user bank account in diagram | 图中的银行账户 | Represents external banking interaction. / 表示外部银行系统交互。 |
| Payment gateway | 支付网关 | Requests or processes fiat payment. / 请求或处理法币支付。 |
| User wallets | 用户钱包 | Internal accounting representation of user balances. / 交易所内部账户余额表示。 |
| Indexing / trade engine | 索引或交易引擎 | Updates entries in database and matches trades. / 更新数据库记录并撮合交易。 |
| Database | 数据库 | Stores all transactions and internal exchange state. / 存储所有交易与内部状态。 |
| Bitcoin/Ethereum/Ripple wallets | 链上钱包 | Exchange-controlled blockchain wallets. / 交易所控制的链上钱包。 |
| P2P transactions | 点对点交易 | On-chain transfers when funds move externally. / 资金转出时发生链上转账。 |

### 18.2 Architectural Meaning / 架构含义

| English | 中文 |
|---|---|
| A centralized exchange can be fast because most actions happen in an internal database. | 中心化交易所可以很快，因为大多数操作发生在内部数据库中。 |
| Users trust the exchange to maintain correct balances and custody assets. | 用户信任交易所正确维护余额并托管资产。 |
| Blockchain settlement may occur only when users deposit or withdraw. | 链上结算可能只在用户充值或提现时发生。 |
| This design is scalable but trust-dependent. | 这种设计可扩展，但依赖信任。 |

---

## 19. The Blockchain Scaling Landscape / 区块链扩容版图

### 19.1 Two Main Ways to Scale / 两种主要扩容方式

The slide divides scaling strategies into two broad groups:

幻灯片将扩容策略分为两大类：

| Category | 中文 | Description / 说明 |
|---|---|---|
| Increase Layer 1 function | 增强 Layer 1 功能 | On-chain block size, parallel processing, sharding, etc. / 链上区块大小、并行处理、分片等。 |
| Modular Layer 1 functions | 模块化 Layer 1 功能 | Move execution off-chain or separate functions such as settlement, execution, and data availability. / 将执行移到链下，或将结算、执行、数据可用性等功能拆分。 |

### 19.2 Dimensions in the Landscape / 版图中的维度

| Dimension | 中文 | Explanation / 解释 |
|---|---|---|
| Execution | 执行 | Where transactions and smart contracts are executed. / 交易和智能合约在哪里执行。 |
| Settlement | 结算 | Where final state commitment and dispute resolution occur. / 最终状态承诺和争议解决在哪里发生。 |
| Consensus | 共识 | How nodes agree on ordering and validity. / 节点如何就排序和有效性达成一致。 |
| Data availability | 数据可用性 | Where transaction data is published so others can verify state. / 交易数据在哪里发布，以便其他人验证状态。 |

### 19.3 Examples and Categories Mentioned in Slide / 幻灯片中出现的例子与类别

| Type / 类型 | Examples Shown / 例子 | Meaning / 含义 |
|---|---|---|
| Monolithic chains | Ethereum, Solana, Aptos, Sui, Near, ShareduM, Quai | A single chain or base protocol handles multiple functions together. / 单一链或基础协议同时承担多个功能。 |
| Sidechains | Polygon, Ronin | Separate chains connected to a main ecosystem but with their own security assumptions. / 与主生态连接但具有自身安全假设的独立链。 |
| Application-specific chains | Cosmos, Osmosis, Sei, Kinto, Polkadot, Acala | Chains specialized for particular applications or ecosystems. / 面向特定应用或生态的专用链。 |
| State channels | Raiden, Connext, Celer | Off-chain interactions with on-chain settlement. / 链下交互，链上结算。 |
| Plasma | OMG, Gluon | Earlier off-chain scaling framework. / 较早期的链下扩容框架。 |
| Sovereign rollups | Fuel v2, Ethereum, Celestia, Polygon Avail | Rollups with more independent execution/settlement assumptions. / 具有更独立执行或结算假设的 Rollup。 |
| Smart contract rollups | Optimistic rollups, ZK rollups, Arbitrum, Optimism, Scroll, StarkNet, zkSync | Rollups settle on a base chain using fraud proofs or validity proofs. / Rollup 通过欺诈证明或有效性证明在基础链结算。 |
| Enshrined rollups | zkEVM, etc. | Rollup-like functions more deeply integrated into the base protocol. / 更深度集成进基础协议的 Rollup 类功能。 |
| Settlement rollups | Smart-contract rollups, settlement rollup, shared sequencing examples | Systems specialized around settlement layers. / 围绕结算层专门化的系统。 |
| Validium-like | Arbitrum Nova, Immutable X, rhino.fi | Execution with off-chain data availability assumptions. / 执行在链下，同时数据可用性也可能在链下。 |

### 19.4 Notes from Bottom of Landscape Slide / 版图底部注释整理

| English Note | 中文整理 |
|---|---|
| Vertical scaling solutions can be applied on top of horizontal ones. | 垂直扩容方案可以叠加在水平扩容方案之上。 |
| EVM-compatible L1s such as Avalanche, Fantom, and BNB Smart Chain may be viewed as monolithic chains or sidechains depending on assumptions. | Avalanche、Fantom、BNB Smart Chain 等 EVM 兼容 L1 可根据安全与架构假设被视为单体链或侧链。 |
| Celestia solves the proof/celestia-related data availability problem, while Avail can be used as an execution or data availability chain. | Celestia 主要解决数据可用性相关问题；Avail 可作为执行或数据可用性相关链使用。 |
| Fuel v2 can operate in multiple modes, including optimistic chains and sovereign rollups. | Fuel v2 可运行在多种模式下，例如乐观链和主权 Rollup。 |
| Layer 1 functions can be validium-like if consensus and data availability are handled by Celestia. | 如果共识和数据可用性交给 Celestia，某些 Layer 1 功能可以呈现 Validium-like 结构。 |
| Arbitrum Nova is technically not a Validium because it is a particular architecture similar to Validium. | Arbitrum Nova 技术上不一定严格等同于 Validium，但架构上具有 Validium-like 特征。 |

### 19.5 Key Insight / 核心洞察

| English | 中文 |
|---|---|
| Modern blockchain scaling is increasingly modular: execution, settlement, consensus, and data availability can be separated. | 现代区块链扩容越来越模块化：执行、结算、共识、数据可用性可以被拆分。 |
| This decomposition is the foundation for rollups, data availability layers, and app-specific chains. | 这种拆分是 Rollup、数据可用性层和应用链的基础。 |

---

## 20. Axie Infinity / Axie Infinity 案例

### 20.1 Why Axie Infinity Appears Here / 为什么这里出现 Axie Infinity

| English | 中文 |
|---|---|
| Axie Infinity is used as an example of a blockchain application with large user demand and game-like interactions. | Axie Infinity 被用作区块链应用案例，它具有大量用户需求和游戏化交互。 |
| Such applications often need higher throughput and lower fees than a congested base chain can provide. | 这类应用通常需要比拥堵基础链更高的吞吐量和更低费用。 |
| This demand motivates application-specific scaling approaches such as sidechains. | 这种需求推动了侧链等面向应用的扩容方式。 |

---

## 21. The Ronin Sidechain / Ronin 侧链

### 21.1 Diagram Explanation / 图示解释

The Ronin sidechain slide shows a validator compromise scenario:

Ronin 侧链图示展示了验证者被攻破的场景：

| Step | English | 中文 |
|---:|---|---|
| 1 | Hacker exploits validators. | 黑客利用或攻破验证者。 |
| 2 | Hacker compromises another validator. | 黑客继续攻破另一个验证者。 |
| 3 | Only 5 validators are needed to approve a fraudulent transaction. | 只需要 5 个验证者批准即可通过欺诈交易。 |

### 21.2 Lesson / 教训

| English | 中文 |
|---|---|
| Sidechains can improve performance, but they may rely on a smaller validator set or different security model. | 侧链可以提升性能，但可能依赖更小的验证者集合或不同安全模型。 |
| If validator control is concentrated or the threshold is too low, attacks become easier. | 如果验证者控制权集中或阈值太低，攻击会更容易发生。 |
| This directly illustrates the scalability trilemma: scaling by reducing validator overhead may weaken security and decentralization. | 这直接体现三难困境：通过降低验证者开销来扩容，可能削弱安全性和去中心化。 |

---

## 22. Blockchain Paradox / 区块链悖论

### 22.1 Statement from Slide / 幻灯片表述

| English | 中文 |
|---|---|
| Bitcoin was launched in 2008, Ethereum in 2015. | Bitcoin 于 2008 年推出，Ethereum 于 2015 年推出。 |
| Why are they still slow compared with Visa? | 为什么它们相比 Visa 仍然这么慢？ |
| The slide answers: “It’s all about computational integrity.” | 幻灯片回答：关键在于“计算完整性”。 |
| Quote shown: “Integrity means doing the right thing, even when no one is watching.” - C.S. Lewis | 幻灯片引用：“Integrity means doing the right thing, even when no one is watching.” —— C.S. Lewis。可理解为：完整性意味着即使没人监督，也做正确的事。 |

### 22.2 Computational Integrity / 计算完整性

| English | 中文 |
|---|---|
| Computational integrity means that computation results should be correct and verifiable. | 计算完整性意味着计算结果应当正确且可验证。 |
| Blockchain is slow partly because it does not merely compute; it computes in a way that allows untrusted parties to verify correctness. | 区块链慢的一部分原因在于它不只是计算，而是以一种让不可信参与方也能验证正确性的方式计算。 |
| Centralized systems optimize for trusted efficiency; blockchains optimize for verifiable correctness under distrust. | 中心化系统优化的是可信环境下的效率；区块链优化的是不信任环境下的可验证正确性。 |

---

## 23. Current State of Affairs / 当前状态

### 23.1 Two Models Shown / 图示中的两种模型

| Model | English | 中文 |
|---|---|---|
| Trust Big Computer | Fast, exclusive. | 信任大型计算机：快速，但排他。 |
| Verify all transactions, do not trust | Slow, inclusive. | 验证所有交易而不信任：慢，但包容。 |

### 23.2 Interpretation / 解读

| English | 中文 |
|---|---|
| A centralized service can be fast because users trust the server to compute correctly. | 中心化服务可以很快，因为用户信任服务器正确计算。 |
| A decentralized blockchain is inclusive because many participants can independently verify the system. | 去中心化区块链具有包容性，因为许多参与者可以独立验证系统。 |
| But inclusiveness and verification create overhead: communication, repeated execution, storage, and consensus. | 但包容性和可验证性带来开销：通信、重复执行、存储和共识。 |

---

## 24. Increasing Scale / 提升规模

### 24.1 Trade-off Illustrated / 图示表达的权衡

| English | 中文 |
|---|---|
| The lecture contrasts two scaling directions: trusting large computers or expanding verification networks. | 本讲对比两种扩容方向：信任大型计算机，或扩大验证网络。 |
| Trusting powerful machines can increase speed but makes participation exclusive. | 信任强大机器可以提升速度，但会使参与门槛变高。 |
| Requiring many participants to verify everything is inclusive but slow. | 要求大量参与者验证所有内容更包容，但速度慢。 |

### 24.2 Hardware Recommendation Slide / 硬件要求页面

Page 27 includes a social-media style screenshot and hardware recommendations, implying that some high-performance blockchain designs may require stronger validator hardware.

第 27 页包含类似社交媒体截图和硬件推荐，暗示某些高性能区块链设计可能要求更强验证者硬件。

| Hardware Item | English Content Shown | 中文整理 |
|---|---|---|
| CPU | 12 cores / 24 threads or more; 2.8GHz or faster; AVX2 instruction support; AVX512 helpful. | CPU：至少 12 核 / 24 线程，2.8GHz 或更快；支持 AVX2，AVX512 有帮助。 |
| RAM | 128GB or more. | 内存：128GB 或更多。 |
| Motherboard | High capacity suggested. | 主板：建议具备较强扩展能力。 |

### 24.3 Lesson / 课程含义

| English | 中文 |
|---|---|
| Scaling by increasing hardware requirements can shift the system toward centralization. | 通过提高硬件要求来扩容，可能使系统走向中心化。 |
| If ordinary users cannot run validators or full nodes, the network becomes less independently verifiable. | 如果普通用户无法运行验证者或全节点，网络的独立可验证性会下降。 |

---

## 25. The Composability-Performance Trilemma / 可组合性-性能三难困境

### 25.1 Three Corners / 三个顶点

| Corner | 中文 | Meaning / 含义 |
|---|---|---|
| Performance | 性能 | High throughput, low latency, efficient execution. / 高吞吐、低延迟、高效执行。 |
| Composability | 可组合性 | Applications can interact synchronously and reuse each other’s liquidity, logic, and state. / 应用可以同步交互并复用彼此的流动性、逻辑和状态。 |
| Decentralization | 去中心化 | Broad participation and low dependence on centralized infrastructure. / 广泛参与，低中心化基础设施依赖。 |

### 25.2 Examples in the Diagram / 图中例子

| Region | Examples | Interpretation / 解释 |
|---|---|---|
| High composability ecosystem | Uniswap, Compound, Aave, Curve | DeFi protocols benefit from shared state and synchronous composability. / DeFi 协议受益于共享状态和同步可组合性。 |
| Higher performance but weaker decentralization/composability trade-off | BSC, Polygon, xDai | Often improve performance through alternative chain assumptions. / 通常通过替代链假设提升性能。 |
| High performance specialized systems | dYdX, DeversiFi, Sorare | More specialized or app-specific performance designs. / 更专门化或应用特定的性能设计。 |
| Layer 2 examples | StarkWare, ZK rollups; Optimism, Arbitrum | Rollups attempt to improve performance while inheriting some base-layer security. / Rollup 试图在继承部分基础层安全性的同时提升性能。 |

### 25.3 Why Composability Matters / 为什么可组合性重要

| English | 中文 |
|---|---|
| Ethereum DeFi is powerful partly because protocols share one execution environment. | 以太坊 DeFi 强大的一部分原因是协议共享同一个执行环境。 |
| A trade on Uniswap can interact with lending on Aave or Compound in the same transaction. | Uniswap 的交易可以在同一笔交易中与 Aave 或 Compound 的借贷交互。 |
| If applications are split across many chains or rollups, performance may improve, but synchronous composability becomes harder. | 如果应用被拆到许多链或 Rollup 上，性能可能提升，但同步可组合性会变困难。 |

---

## 26. Architecture of Decentralised Exchange / 去中心化交易所架构

### 26.1 Components Shown / 图示组件

| Component | 中文 | Function / 功能 |
|---|---|---|
| Alice and Bob | 用户 Alice 和 Bob | Users sign orders. / 用户签署订单。 |
| Exchange | 交易所前端或入口 | Receives signed user orders. / 接收用户签名订单。 |
| Matching engine | 撮合引擎 | Matches buy and sell orders. / 撮合买卖订单。 |
| Exchange back-end | 交易所后端 | Produces transaction legs or settlement statements. / 生成交易结算相关数据。 |
| StarkWare / SHARP-like prover | 证明系统 | Verifies or aggregates state transition proofs. / 验证或聚合状态转换证明。 |
| Ethereum blockchain | 以太坊区块链 | Hosts exchange smart contracts and verifier smart contracts. / 部署交易所智能合约和验证器智能合约。 |
| Exchange smart contract | 交易所智能合约 | Applies state updates. / 应用状态更新。 |
| Verifier smart contract | 验证器智能合约 | Verifies cryptographic proof or state transition validity. / 验证密码学证明或状态转换有效性。 |
| State update | 状态更新 | Updates final state on-chain. / 在链上更新最终状态。 |

### 26.2 Architecture Meaning / 架构含义

| English | 中文 |
|---|---|
| The architecture shows a hybrid model: execution or matching may happen off-chain, while verification and settlement happen on Ethereum. | 该架构展示了一种混合模型：执行或撮合可在链下发生，而验证与结算在以太坊上完成。 |
| This is the direction of many Layer 2 systems: reduce on-chain computation while preserving on-chain verifiability. | 这是许多 Layer 2 系统的方向：减少链上计算，同时保持链上可验证性。 |
| The goal is not to trust the exchange fully, but to verify the correctness of its updates through smart contracts or proofs. | 目标不是完全信任交易所，而是通过智能合约或证明验证其状态更新的正确性。 |

---

## 27. Projects / 项目

| English | 中文 |
|---|---|
| The lecture ends by transitioning to projects related to blockchain scalability. | 本讲最后过渡到与区块链可扩展性相关的项目。 |
| A good project should connect technical scaling ideas with real trade-offs in decentralization, security, data availability, and user experience. | 好的项目应把扩容技术思想与去中心化、安全性、数据可用性和用户体验中的真实权衡结合起来。 |

---

# 28. Consolidated Knowledge Map / 综合知识地图

## 28.1 Core Problem Chain / 核心问题链

```text
High demand for transactions
        ↓
Limited blockspace and replicated validation
        ↓
Congestion, high gas fees, slow confirmation
        ↓
Pressure to scale
        ↓
Naive scaling raises hardware or consensus risks
        ↓
Trade-offs among scalability, decentralization, security, composability
        ↓
Need modular scaling, L2, rollups, data availability scaling, app-specific chains
```

```text
交易需求上升
        ↓
区块空间有限 + 重复验证
        ↓
拥堵、Gas 费上升、确认变慢
        ↓
产生扩容压力
        ↓
朴素扩容会带来硬件要求或共识风险
        ↓
可扩展性、去中心化、安全性、可组合性之间产生权衡
        ↓
需要模块化扩容、Layer 2、Rollup、数据可用性扩容、应用链等方案
```

---

## 28.2 Key Terms / 关键术语表

| Term | 中文 | Detailed Meaning / 详细含义 |
|---|---|---|
| TPS | 每秒交易数 | A throughput metric, but insufficient alone to define blockchain scalability. / 吞吐量指标，但不足以单独定义区块链可扩展性。 |
| Finality | 最终确定性 | Point at which a transaction is considered irreversible or economically final. / 交易被认为不可逆或经济上最终确定的时刻。 |
| Gas | Gas | Unit measuring computational and storage cost in Ethereum. / 以太坊中衡量计算和存储成本的单位。 |
| Mempool | 内存池 | Pool of pending transactions waiting to be included in blocks. / 等待被打包进区块的待处理交易集合。 |
| Replicated computation | 重复计算 | Multiple nodes process the same transaction to independently verify correctness. / 多个节点处理同一交易以独立验证正确性。 |
| Replicated storage | 重复存储 | Many nodes store the same data and state. / 多个节点存储同样的数据和状态。 |
| Consensus overhead | 共识开销 | Communication and coordination cost required for agreement. / 达成一致所需的通信与协调成本。 |
| Data availability | 数据可用性 | Guarantee that transaction data needed for verification is publicly available. / 保证验证所需交易数据公开可得。 |
| State validity | 状态有效性 | Assurance that state transitions follow protocol rules. / 状态转换符合协议规则的保证。 |
| Byzantine adversary | 拜占庭对手 | Malicious or faulty participants that may deviate arbitrarily. / 可能任意偏离协议的恶意或故障参与者。 |
| Blockchain trilemma | 区块链三难困境 | Trade-off among scalability, decentralization, and security. / 可扩展性、去中心化和安全性之间的权衡。 |
| Composability | 可组合性 | Ability of applications/protocols to interoperate synchronously and reuse state/liquidity. / 应用或协议同步互操作并复用状态与流动性的能力。 |
| Rollup | Rollup | L2 approach that executes transactions off-chain and posts commitments/proofs to L1. / 在链下执行交易，并向 L1 提交承诺或证明的二层方案。 |
| Sidechain | 侧链 | Independent chain connected to another ecosystem but with its own security model. / 与某生态连接但拥有自身安全模型的独立链。 |
| Validium | Validium | Scaling design using validity proofs but off-chain data availability. / 使用有效性证明但数据可用性在链下的扩容设计。 |

---

# 29. Exam and Discussion Questions / 考试与讨论题

## 29.1 Conceptual Questions / 概念题

1. **English:** Why is TPS alone an insufficient definition of blockchain scalability?  
   **中文：** 为什么单独用 TPS 定义区块链可扩展性是不充分的？

2. **English:** Explain how increasing block size can harm decentralization.  
   **中文：** 解释为什么增大区块大小可能损害去中心化。

3. **English:** Why can shorter block intervals increase fork rates and reduce confirmation stability?  
   **中文：** 为什么缩短出块间隔可能增加分叉率并降低确认稳定性？

4. **English:** What are replicated computation, replicated storage, and consensus overhead? Why are they fundamental to blockchain scalability problems?  
   **中文：** 什么是重复计算、重复存储和共识开销？为什么它们是区块链可扩展性问题的根源？

5. **English:** Compare vertical scaling and horizontal scaling in blockchain systems.  
   **中文：** 比较区块链系统中的垂直扩展和水平扩展。

6. **English:** What is the blockchain trilemma? Give one example of a design trade-off.  
   **中文：** 什么是区块链三难困境？举一个设计权衡的例子。

7. **English:** Why does computational integrity make blockchains slower than centralized payment systems?  
   **中文：** 为什么计算完整性会使区块链比中心化支付系统更慢？

8. **English:** What role does data availability play in scalable blockchain design?  
   **中文：** 数据可用性在可扩展区块链设计中起什么作用？

9. **English:** Why can sidechains increase performance but weaken security assumptions? Use Ronin as an example.  
   **中文：** 为什么侧链可以提升性能但可能削弱安全假设？请以 Ronin 为例。

10. **English:** Explain the composability-performance trilemma in DeFi.  
    **中文：** 解释 DeFi 中的可组合性-性能三难困境。

---

## 29.2 Short Answer Templates / 简答题模板

### Question: Why is blockchain slow compared with Visa?  
### 问题：为什么区块链相比 Visa 慢？

| English Answer Template | 中文答案模板 |
|---|---|
| Public blockchains are slower because they prioritize computational integrity, decentralization, and adversarial verification. Every full node may need to verify transactions, store data, and participate in consensus. Visa is a centralized payment network that can rely on trusted infrastructure and internal databases, so it avoids most replicated verification overhead. | 公链较慢是因为它优先保证计算完整性、去中心化和对抗性验证。每个全节点可能都需要验证交易、存储数据并参与共识。Visa 是中心化支付网络，可以依赖可信基础设施和内部数据库，因此避免了大量重复验证开销。 |

### Question: What is the core scalability bottleneck?  
### 问题：可扩展性的核心瓶颈是什么？

| English Answer Template | 中文答案模板 |
|---|---|
| The core bottleneck is that blockchains replicate computation, storage, and consensus across many nodes. This improves trustlessness and security, but it means nodes repeat work instead of dividing it efficiently. | 核心瓶颈在于区块链在许多节点之间重复计算、重复存储并执行共识。这提升了无需信任和安全性，但意味着节点在重复工作，而不是高效分工。 |

### Question: What is the goal of modern scaling?  
### 问题：现代扩容的目标是什么？

| English Answer Template | 中文答案模板 |
|---|---|
| Modern scaling aims to reduce unnecessary redundancy while preserving verifiability. Techniques such as rollups, sharding, data availability layers, and application-specific chains try to process or store only partial data while still allowing the network to detect invalid state transitions. | 现代扩容的目标是在保留可验证性的同时减少不必要的重复。Rollup、分片、数据可用性层和应用链等技术尝试只处理或存储部分数据，同时仍允许网络发现无效状态转换。 |

---

# 30. Slide-by-Slide Index / 逐页索引

| PDF Page | English Slide Topic | 中文主题 | Key Points / 重点 |
|---:|---|---|---|
| 1 | Title | 标题页 | SC6019 Blockchain; Privacy & Scalability; Lecture 07; Neil HAN. |
| 2 | About the 2nd Half | 后半学期安排 | Weeks 7-12 roadmap from scalability introduction to projects and quiz. |
| 3 | References | 参考文献 | Mastering Blockchain; Mastering Ethereum. |
| 4 | Introduction | 导论 | Problems, challenges, blockchain scalability. |
| 5 | Transaction Speeds | 交易速度对比 | Visa much faster than Bitcoin/Ethereum; motivates scalability problem. |
| 6 | Gas Cost and Cost Surge | Gas 成本与费用激增 | Gas calculation, mempool congestion, fee surge. |
| 7 | CryptoKitties Architecture | CryptoKitties 架构 | Browser, frontend, smart contracts, Ethereum blockchain congestion. |
| 8 | What is Scalability? | 什么是可扩展性 | TPS, latency, hardware efficiency, non-degrading performance. |
| 9 | Formal Definition | 形式化定义 | Scalability lacks universally accepted definition. |
| 10 | Attempt 1: Throughput | 尝试一：吞吐量 | Blockchain as service; transactions as computational tasks. |
| 11 | Naive TPS Scaling | 朴素 TPS 扩容 | Throughput formula; block size and block interval risks. |
| 12 | Wikipedia Scalability | Wikipedia 式定义 | Vertical and horizontal scaling problems in blockchain. |
| 13 | Fundamental Problems | 根本问题 | Replicated computation, replicated storage, consensus overhead. |
| 14 | Fundamental Challenges | 根本挑战 | Partial processing/storage requires state validity, data availability, Byzantine resilience. |
| 15 | Blockchain Trilemma Section | 三难困境章节 | Transition slide. |
| 16 | CAP Problem | CAP 问题 | Consistency, availability, partition tolerance. |
| 17 | Scalability Trilemma | 可扩展性三难 | Scalable, decentralized, secure triangle. |
| 18 | Trilemma Definitions | 三难定义 | Decentralization, security, scalability. |
| 19 | Trilemma Deep Dive | 三难深入 | Benefits/challenges/threats/current limitations. |
| 20 | Centralised Exchange Architecture | 中心化交易所架构 | Fast internal database; trust-dependent custody. |
| 21 | Scaling Landscape | 扩容版图 | Monolithic chains, sidechains, rollups, DA, modular architecture. |
| 22 | Axie Infinity | Axie Infinity | Game/application demand motivating scaling. |
| 23 | Ronin Sidechain | Ronin 侧链 | Validator exploit, security trade-off. |
| 24 | Blockchain Paradox | 区块链悖论 | Slow because of computational integrity. |
| 25 | Current State | 当前状态 | Trust big computer vs verify all transactions. |
| 26 | Increase Scale | 提升规模 | Fast/exclusive vs slow/inclusive trade-off. |
| 27 | Increase Scale & Hardware | 提升规模与硬件 | High hardware requirements can centralize validation. |
| 28 | Composability-Performance Trilemma | 可组合性-性能三难 | Performance, composability, decentralization. |
| 29 | Decentralised Exchange Architecture | 去中心化交易所架构 | Off-chain matching with on-chain verification/settlement. |
| 30 | Projects | 项目 | Transition to scalability projects. |

---

# 31. One-Page Final Summary / 一页终极总结

| English | 中文 |
|---|---|
| Blockchain scalability is difficult because blockchains are not merely databases. They are adversarial, decentralized verification systems. | 区块链可扩展性困难，是因为区块链不只是数据库，而是对抗性环境下的去中心化验证系统。 |
| A centralized exchange or payment system can be fast because users trust the operator and computation is performed internally. | 中心化交易所或支付系统可以很快，因为用户信任运营方，计算在内部完成。 |
| Public blockchains deliberately replicate computation, storage, and consensus across many nodes to achieve trust minimization. | 公链故意在许多节点间重复计算、重复存储和执行共识，以实现最小信任。 |
| Naive throughput improvements, such as larger blocks or shorter block times, can harm decentralization and security. | 朴素吞吐量提升，如增大区块或缩短出块时间，可能损害去中心化和安全性。 |
| The blockchain trilemma states that scalability, decentralization, and security are difficult to maximize simultaneously. | 区块链三难困境指出，可扩展性、去中心化和安全性很难同时最大化。 |
| Modern scaling attempts to reduce redundant work while preserving verifiability, using techniques such as sidechains, rollups, modular architectures, and data availability layers. | 现代扩容试图在保留可验证性的同时减少重复工作，使用侧链、Rollup、模块化架构和数据可用性层等技术。 |
| The deeper reason blockchains are slow is computational integrity: doing computation in a way that others can verify without trusting the executor. | 区块链慢的更深层原因是计算完整性：以他人无需信任执行者也能验证的方式进行计算。 |
| The future of scalability is not just “more TPS”; it is designing systems that balance performance, security, decentralization, data availability, and composability. | 可扩展性的未来不只是“更高 TPS”，而是设计能够平衡性能、安全性、去中心化、数据可用性和可组合性的系统。 |

