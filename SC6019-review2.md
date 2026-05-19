这是一份基于您提供的PPT幻灯片整理的**非常详细的中英对照课件文档**。文档包含了所有的课程内容、知识点、核心概念解析以及图表和案例中的细节整理，排版采用Markdown格式，非常适合用作复习笔记和学习参考。

---

# SC6019 – Blockchain Privacy & Scalability
# 区块链隐私与可扩展性

## Lecture 08: Layer 1 Solutions - On-Chain Scalability
## 第八讲：第一层解决方案 - 链上可扩展性
**Scaling Computation – Sharding & Blockchain Interoperability**
**扩展计算：分片技术与区块链互操作性**

---

### 1. Fundamental Problem in Blockchain Scalability (区块链可扩展性的根本问题)
**English:**
The fundamental bottleneck in traditional blockchain architecture is that **"every node processes all TXs, stores all data, plus total ordering of TXs is required."** 
**What if we split the TX processing into smaller groups?**
*   **Sharding Idea:** Horizontally scale by partitioning transaction processing into *shards*, and parallelizing their execution.

**Chinese:**
传统区块链架构的根本瓶颈在于：**“每个节点都需要处理所有的交易，存储所有的数据，并且还需要对所有交易进行全局排序。”**
**如果我们将交易处理过程拆分成更小的组会怎样？**
*   **分片的核心理念：** 通过将交易处理划分到不同的**分片 (shards)** 中来实现水平扩展，并让它们并行执行。

*(注：非分片网络中所有节点相互连接处理同一套数据；在传统分片架构中，节点被划分到诸如分片A、分片B中，各自处理系统的一部分任务，从而提升整体网络效率。)*

---

### 2. Sharding: The Concept (分片的概念与传统应用)
**English:**
Sharding is widely used in traditional databases like **MongoDB**.
*   **Why use it?** Huge data volume, High Read/Write request loads.
*   **Benefits:** Better parallelization => Higher throughput. Smaller search space per shard => faster response.
*   **Shard key:** An index field that determines which shard a piece of data/request should go to. For example, key space for 'x' can be partitioned into Chunk 1, Chunk 2, etc.
*   **Global Distributed Systems:** Like Google Spanner or TiDB, which utilize partitions, placement drivers, and replicas (using Paxos for consensus) to manage large-scale distributed data.

**Chinese:**
分片技术在 **MongoDB** 等传统数据库中被广泛应用。
*   **应用场景：** 海量数据，极高的读/写请求负载。
*   **优势：** 更好的并行化 => 更高的吞吐量。每个分片内的数据搜索空间更小 => 更快的响应速度。
*   **分片键 (Shard key)：** 一种索引字段，用于决定某条数据或某个请求应该被路由到哪一个分片。例如，可以将变量'x'的键空间划分为区块1、区块2等。
*   **全局分布式系统应用：** 例如 Google Spanner 或 TiDB 等系统，利用数据分区(partitions)、调度管理(placement driver)和副本机制(基于Paxos共识)来管理大规模分布式数据。

---

### 3. Blockchain Sharding: Challenge (区块链分片面临的挑战)
**English:**
*   **Traditional Sharding:** Relies on trusted infrastructure, deals mainly with crash-failures, and operates with permissioned identities.
*   **Blockchain Sharding:** Operates on an adversarial network, must tolerate Byzantine-failures, and deal with Sybil identities (fake/multiple node identities).

**Chinese:**
*   **传统分片：** 依赖可信的基础设施，主要处理崩溃故障(Crash-failure)，且节点拥有许可身份。
*   **区块链分片：** 运行在对抗性网络环境中，必须容忍拜占庭故障(Byzantine-failure)，并且需要应对女巫身份(Sybil identity，即恶意伪造多个节点身份的攻击)。

---

### 4. Strawman Sharding (基础分片/草人模型)
**English:**
**Idea:** Each shard has its own validators and manages its chain separately.
*   **✗ Problems:** 
    1.  Dividing into $s$ shards **reduces security $s$ folds** if there is no random validator assignment (an attacker only needs to compromise one specific small shard).
    2.  Cannot send transactions across shards (Isolated states).
*   **💡 Solution:** Randomly assign validators to shards.

**Chinese:**
**思路：** 每个分片拥有自己的验证者，并独立管理自己的区块链。
*   **✗ 存在的问题：**
    1.  如果没有随机的验证者分配机制，划分为 $s$ 个分片会**使安全性降低 $s$ 倍**（攻击者只需要集中算力攻击某一个较小的分片即可破坏系统）。
    2.  无法跨分片发送交易（状态相互隔离）。
*   **💡 解决方案：** 将验证者随机分配到各个分片中。

---

### 5. Random Validator Assignment (随机分配验证者的作用)
**English:**
Assume a total network has 30% malicious nodes.
*   **Math:** The probability of a specific shard having more than 30% malicious nodes (if randomly sampled) is approximately **35%**. 
    *(Calculated using combinatorial probability: $Pr[P_2 > 30\%] = 1 - \sum_{i=0}^{3} Pr[m=i] \approx 35\%$)*
*   **✓ Good:** Random assignment doesn't reduce total network security much.
*   **✗ Bad:** A shard is still a smaller target. It could potentially be compromised *after* the random assignment if adversaries collude.
*   **💡 Solution:** Randomly **shuffle/rotate** the validator occasionally. (This requires a random **beacon chain**!)

**Chinese:**
假设整个网络中有30%的恶意节点。
*   **数学概率：** 如果随机抽样，某一个特定分片中恶意节点超过30%的概率约为 **35%**。
*   **✓ 优点：** 随机分配不会大幅降低系统的整体安全性。
*   **✗ 缺点：** 分片仍然是一个较小的攻击目标。在随机分配**之后**，恶意节点可能通过串谋攻破该分片。
*   **💡 最终解决方案：** 偶尔对验证者进行**随机洗牌/轮换**。（这需要引入一条提供随机性的**信标链/Beacon chain**！）

---

### 6. Obtaining Unbiased Randomness (如何获取无偏的随机性？)
**English:**
*   **RandHound/RandHerd:** "Large-scale distributed protocols... which provide publicly-verifiable, unpredictable, and unbiasable randomness against Byzantine adversaries." (Participants jointly generate).
*   **Verifiable Random Function (VRF):** "A public-key version of a keyed cryptographic hash. Only the holder of the private key can compute the hash, but anyone with the public key can verify the correctness of the hash."

**Chinese:**
*   **RandHound/RandHerd 协议：** “一种大规模分布式协议……它能提供公开可验证的、不可预测的且无偏的随机性，以对抗拜占庭对手。”（由网络参与者共同生成随机数）。
*   **可验证随机函数 (VRF)：** “带有密钥的密码学哈希的公钥版本。只有私钥持有者能够计算出哈希值，但任何拥有公钥的人都能验证该哈希的正确性。”

---

### 7. Cross-Shard Transactions (如何促进跨分片交易？)
**English:**
The beacon chain needs updated states => Validators on each shard periodically send/commit its new block (a.k.a. **"chain-link"**) to the beacon chain (Hub-spoke model).
**Question: What to commit? Entire block or just the block header?**
1.  **Chain-link = entire block:** Means **transaction sharding only**. Global states are kept in the beacon chain, cross-shard transactions are *synchronously* executed in the beacon chain.
2.  **Chain-link = block header:** Means **transaction & state sharding**. Data (all blockchain states) are also sharded. There is no global state. Cross-shard transactions take effect *asynchronously* and locally inside a shard with the help of relaying via the beacon chain.

**Chinese:**
信标链需要获得更新的状态 => 每个分片上的验证者定期向信标链发送/提交其新生成的区块（即所谓的 **“交叉链接 / chain-link”**）（采用中心辐射模型 Hub-spoke）。
**问题：应该提交什么？是整个区块还是仅仅是区块头？**
1.  **提交整个区块 (entire block)：** 这意味着**仅进行交易分片**。全局状态保存在信标链中，跨分片交易在信标链中被**同步**执行。
2.  **提交区块头 (block header)：** 这意味着进行了**交易和状态分片**。数据（即所有的区块链状态）也被分片了，不存在全局状态。跨分片交易在信标链的“中继”帮助下，在分片内部**异步**且局部地生效。

---

### 8. Asynchronous Cross-Shard Transaction (异步跨分片交易)
**English:**
*   Because it's asynchronous, it needs **atomicity** (i.e., the "train-hotel problem").
*   **Atomicity:** All-or-nothing guarantee. Either both the train ticket and hotel reservation are successful, or none are. (If one fails, the other must be reverted).
*(Details of how to achieve atomicity are omitted due to complexity).*

**Chinese:**
*   由于跨分片是异步进行的，因此需要保证**原子性 (Atomicity)**（例如经典的“火车票-酒店预订问题”）。
*   **原子性：** 一种“全有或全无”的保证。要么火车票和酒店预订双双成功，要么都不成功（如果其中一个失败，另一个必须回滚）。
*（由于实现过于复杂，本节省略了实现原子性的具体技术细节）。*

---

### 9. Sharding in a Nutshell (分片架构全貌总结)
**English:**
*   **Beacon Chain:** Provides randomness, facilitates cross-shard tx, aggregates cross-links into finalized blocks. 
    *   *Note:* Content in cross-links is treated as raw data blobs and merkelized *without validation* by the beacon chain.
*   **Shard Chain:** Runs efficient BFT consensus. (Usually at most a few thousands validators in BFT!)
*   **Shard Key:** For incoming tx, it is usually the last few digits of the user's address (e.g., `0x...01` in shard 1, `0x...09` in shard 9).
*   **Where do different sharding protocols differ?**
    1. Sybil-resistant validator selection method?
    2. Random validator shuffling method?
    3. Cross-link content: block header or entire block?

**Chinese:**
*   **信标链 (Beacon Chain)：** 提供随机性，促进/协调跨分片交易，将交叉链接聚合到最终确定的区块中。
    *   *注意：* 信标链将交叉链接中的内容仅仅视为原始数据块(raw data blob)并对其生成默克尔树(merkelized)，而**不会对其进行验证**。
*   **分片链 (Shard Chain)：** 运行高效的BFT（拜占庭容错）共识机制。（通常BFT共识中最多只能容纳几千个验证者！）
*   **分片键 (Shard Key)：** 进入网络的交易通常根据用户地址的最后几位数字进行分片（例如，地址以`0x...01`结尾的进入分片1，以`0x...09`结尾的进入分片9）。
*   **不同分片协议的核心区别在于哪些地方？**
    1. 抗女巫攻击的验证者选择机制是什么？
    2. 随机验证者洗牌/轮换的具体方法是什么？
    3. 交叉链接包含的内容是什么：仅区块头还是整个区块？

---

### 10. Sharding Examples (分片协议案例)

#### A. Elastico [Luu et.al] / Zilliqa Sharding
**English:**
*   **Identity Establishment (Sybil-resistance):** PoW (Proof of Work).
*   **Assigning Shard Committee:** Uses RANDAO.
*   **Intra-committee Consensus:** PBFT.
*   **Cross-link:** Commits the **entire block** (Transaction sharding only).
*   **Steps:** 1. Identity Est. -> 2. Assign Committee -> 3. Consensus -> 4. Broadcast Final block -> 5. Generate Next Epoch Randomness.

**Chinese:**
*   **身份建立 (抗女巫攻击机制)：** 采用 PoW (工作量证明)。
*   **分配分片委员会 (随机性)：** 采用 RANDAO 机制。
*   **委员会内部共识：** 采用 PBFT (实用拜占庭容错)。
*   **交叉链接：** 提交**整个区块**（因此仅做交易分片，无状态分片）。
*   **运行步骤：** 1. 建立身份 -> 2. 分配分片委员会 -> 3. 内部共识 -> 4. 广播最终共识(整个区块) -> 5. 生成下一个周期的随机数。

#### B. NearProtocol Sharding
**English:**
*   **Sybil-resistance:** Threshold PoS + TxFlow (easier for low resource nodes).
*   **Randomness:** VRF based on stake.
*   **Cross-link:** Only **block header** (Transaction & state sharding).

**Chinese:**
*   **抗女巫攻击机制：** 阈值 PoS (权益证明) + TxFlow（对低资源节点更友好）。
*   **随机性机制：** 基于权益的 VRF (可验证随机函数)。
*   **交叉链接：** 仅提交**区块头**（因此实现了交易与状态的双重分片）。

#### C. ETH2.0 Sharding
**English:**
*   **Main Chain:** Provides staking (Bounded PoS, security deposit in Management Contract).
*   **Beacon Chain:** Provides random numbers (VRF), uses Casper FFG PoS (Friendly Finality Gadget).
*   **Shard Chain:** Provides data. Proposer = shard validator; Attester = "validity inspector".
*   **VM:** Provides state execution result. Cross-link is only block header. Notaries sign on checkpointed blocks.

**Chinese:**
*   **主链：** 提供质押功能（有界PoS，分片管理合约中的安全押金）。
*   **信标链：** 提供随机数 (VRF)，采用 Casper FFG PoS 共识。
*   **分片链：** 提供数据。区块提议者(Proposer)为分片验证者；证明者(Attester)扮演“有效性检查员”的角色。
*   **虚拟机(VM)：** 提供状态执行结果。交叉链接仅包含区块头。公证人(Notaries)在检查点区块上进行签名。

---

### 11. Pause & Ponder: Deep Dive Questions (思考与讨论)
**English & Chinese Context:**

*   **Q: What computation do validators on the beacon chain do? (信标链上的验证者执行哪些计算？)**
    *   *Ans / 答:* Validate signatures on the shard blocks approved by the correct shard committee + process cross-shard transactions + aggregate/merkelize all raw blocks into a global state tree. 
    *(验证由正确的分片委员会批准的分片区块上的签名 + 处理跨分片交易 + 将所有原始区块聚合并构建全局状态树的默克尔树。)*

*   **Q: Does a validator on the beacon chain require more hardware resources than that of a shard chain? (信标链验证者是否比分片链验证者需要更多的硬件资源？)**
    *   *Ans / 答:* Yes. With $s$ shards, beacon chain validators store all raw blocks from all shards, requiring higher communication bandwidth and $s$ times the storage. However, they don't *re-process* these shard blocks, so they don't necessarily require more powerful computation. (Shard committees do the heavy lifting of executing/validating; beacon chain does coordination/verification). 
    *(是的。对于 $s$ 个分片，信标链验证者需存储所有分片的原始区块，需要更高带宽和 $s$ 倍存储空间。但他们无需重新处理这些区块，因此不需要更强的算力。分片委员会做执行验证的脏活累活，信标链只做协调和证明验证。)*

*   **Q: Why are state validity and data availability guaranteed? (为何能保证状态有效性和数据可用性？)**
    *   *Ans / 答:* **Data availability:** All states are globally aggregated at the beacon chain level. **State validity:** PBFT at the shard chain ensures correctness; even if a shard is corrupted, users can challenge malicious validators by providing proof of state transition at the beacon chain. 
    *(**数据可用性：**所有状态在信标链上全局聚合。**状态有效性：**分片层的PBFT确保正确性；即使某分片被攻破，用户仍能在信标链提供状态转换证明来挑战恶意节点。)*

*   **Q: Can we add an infinite amount of shards to the system? (我们可以无限量地增加分片数量吗？)**
    *   *Ans / 答:* No. The beacon chain is a blockchain itself and is computationally bounded by its maintaining nodes. However, there is a multiplicative effect: if each node is 4X as fast, shard chain throughput is 4X, and the system can accommodate ~4X shards. Total improvement = ~16X. Hence the name **"quadratic sharding"**. 
    *(不行。信标链本身受限于其节点的计算能力。但存在乘数效应：如果节点性能提升4倍，分片吞吐量提升4倍，且能容纳的分片数也提升约4倍，总提升达16倍。这被称为**“二次方分片”**。)*

---

### 12. Ensuring State Validity (确保状态有效性的方案)
**English:**
*   **Approach 1: Interactive proof of computation (aka "Fisherman" or "fraud proof")**
    *   **Idea:** Before a predefined challenge period elapses, anyone can question the computation integrity at a certain locality. The one that computes the result must provide evidence in time, otherwise, failure to respond is used as proof of fraud. 
    *   *(Requires submitting Valid signatures & Merkle paths of states like UTXO to prove validity during a challenge).*
*   **Approach 2: Non-interactive cryptographic proof of computation (e.g., zk-SNARK)**
    *   **Idea:** Generate a mathematical proof showing: "The result of $f_1( f_2( f_3( \dots f_k(x) \dots ))) \text{ is } y$, and here's a publicly-verifiable proof $\pi$."

**Chinese:**
*   **方法一：交互式计算证明（又名“渔夫/Fisherman”机制 或 “欺诈证明”）**
    *   **核心思想：** 在预设的挑战期结束之前，任何人都可以对某个局部的计算完整性提出质疑。执行计算的节点必须在规定时间内提供证据，否则未能响应将被直接视为欺诈。
    *   *(细节：在挑战时，节点需提供有效的签名以及UTXO等状态的Merkle路径以证明其响应令人信服)。*
*   **方法二：非交互式密码学计算证明（例如：zk-SNARK）**
    *   **核心思想：** 直接提供数学证明。声明：“$f_1( f_2( f_3( \dots f_k(x) \dots )))$ 的结果是 $y$，并且提供一份公开可验证的密码学证明 $\pi$。”

---

### 13. Blockchain Interoperability ≈ State Sharding (区块链互操作性 ≈ 状态分片)
**English:**
Scaling through interoperability closely mirrors the architecture of State Sharding. 
*   **State Sharding Architecture:** Beacon chain <-> Shard chain (All run BFT/PoS variants. Uses Cross-shard txn).
*   **Cosmos Ecosystem:** **Cosmos Hub** <-> **Cosmos Zone**. (All use Tendermint Consensus or bridge to it. Uses IBC - Inter-blockchain Communication).
*   **Polkadot Ecosystem:** **Relay Chain** <-> **Para Chain** (Pluggable consensus. Uses Cross-chain txn).

**Chinese:**
通过跨链互操作性来实现的扩展，在架构上与“状态分片”高度相似。
*   **状态分片架构：** 信标链 <-> 分片链（都运行BFT或PoS变体共识。使用跨分片交易进行通信）。
*   **Cosmos 生态系统：** **Cosmos枢纽 (Cosmos Hub)** <-> **Cosmos分区 (Cosmos Zone)**。（所有Zone使用Tendermint共识或通过网桥连接到它。使用 **IBC 跨链通信协议**）。
*   **Polkadot 生态系统：** **中继链 (Relay Chain)** <-> **平行链 (Para Chain)**。（支持可插拔共识。使用跨链交易）。

---

### 14. Deep Dive into Cosmos (深入解析 Cosmos)
**English:**
*   **Architecture:** Multiple independent "Zones" connected to a central "Hub". Within each Zone (which is an app-specific blockchain), nodes hold the same data.
*   **Tendermint Consensus:** 
    *   A BFT-based state machine: `Propose` -> `Prevote` -> `Precommit` -> `Commit` (New block committed). 
    *   Requires +2/3 prevotes/precommits to proceed to the next step.
*   **Deterministic vs. Probabilistic Finality:**
    *   *Probabilistic (e.g., Bitcoin):* Based on the longest chain. The probability of a chain being canonical is high but never 100%.
    *   *Deterministic (Cosmos/Tendermint):* The canonical chain is 100% final the moment a block is committed.
*   **Easier Light Clients:**
    *   Light clients only store block headers. Every time a header is downloaded, it verifies validity by calculating votes from the known validator set. It keeps track of the validator set to know when it is outdated.
*   **Ethereum vs. Cosmos Paradigm:**
    *   *Ethereum:* Multiple applications run on ONE shared Ethereum blockchain (miners execute all apps).
    *   *Cosmos:* Each application gets its own Application-specific blockchain with its own validators.
*   **Hubs and Zones via IBC (Inter-Blockchain Communication):**
    *   Zones track block headers of the Hub, and the Hub tracks block headers of each Zone.
    *   *Example of cross-chain transfer:* Zone 1 locks up 10 coins designated for Zone 2 -> Generates a Proof of these locked coins -> Sends proof to Hub via IBC -> Hub relays proof to Zone 2 via IBC -> Zone 2 mints/credits the tokens.

**Chinese:**
*   **架构设计：** 多个独立的“分区 (Zones)”连接到一个中心的“枢纽 (Hub)”。每个分区本身是一条特定于应用的区块链，网络内节点同步相同的数据。
*   **Tendermint 共识机制：**
    *   基于BFT的状态机流程：`提议(Propose)` -> `预投票(Prevote)` -> `预提交(Precommit)` -> `提交(Commit，生成新区块)`。
    *   必须收集到超过 2/3 节点权重的预投票或预提交，才能进入下一阶段。
*   **确定性终局性 vs 概率性终局性：**
    *   *概率性终局性（如比特币）：* 基于最长链原则。区块成为规范链的概率很高，但永远达不到100%（总是存在理论上的重组分叉风险）。
    *   *确定性终局性（Cosmos/Tendermint）：* 一旦区块被提交，它100%就是规范链，不可回滚。
*   **更简单的轻客户端 (Easier Light Clients)：**
    *   Cosmos轻客户端只需存储区块头。每次下载区块头时，只需通过计算已知验证者集合的投票签名即可验证有效性。它内置逻辑来跟踪验证者集合的变更，以便随时更新。
*   **以太坊架构 vs Cosmos架构：**
    *   *以太坊：* 所有的应用程序（智能合约）都运行在**同一个**共享的底层区块链上，底层矿工处理所有应用的计算。
    *   *Cosmos：* 每个应用程序都可以拥有**专属的**、特定于应用程序的独立区块链和专属的验证者节点。
*   **枢纽与分区及其 IBC 跨链通信流程：**
    *   分区(Zone)会持续跟踪枢纽(Hub)的区块头，反之枢纽也会跟踪各个分区的区块头。
    *   *跨链资产转移示例：* Zone 1 锁定指定的 10 个代币 -> 生成代币已被锁定的证明 (Proof) -> 通过 IBC 协议将证明发送给 Hub -> Hub 通过 IBC 将证明中继给 Zone 2 -> Zone 2 验证后，在自己的链上铸造/释放对应的代币。

---
*(课件整理完毕 / End of Document)*