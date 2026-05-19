# SC6019 - Blockchain Privacy & Scalability - Lecture 11  
# Data Availability, EVM Scaling & Consensus Scaling  
# SC6019 区块链隐私与可扩展性 - 第 11 讲：数据可用性、EVM 扩展与共识扩展

> Source / 来源：`SC6019 - Lecture 11.pdf`，共 91 页。  
> Lecturer / 授课教师：Neil HAN。  
> Topic / 主题：Blockchain Scalability - Scaling Data, EVM Scaling, Scaling Consensus, and recap of previous scalability topics.

---

## 0. Lecture Roadmap / 本讲路线图

### English
Lecture 11 is a broad scalability lecture. It connects three layers of scalability work:

1. **Scaling Data - Data Availability (DA)**: how light clients and rollups can be confident that block data is actually published, not merely committed by a header.
2. **EVM Scaling**: why the standard EVM executes transactions sequentially, and how parallel execution, access lists, Block-STM, GPU acceleration, and modular sequencers try to increase throughput.
3. **Scaling Consensus**: how non-Nakamoto consensus, especially BFT, HotStuff, and Sync HotStuff, can improve latency, communication structure, and fault tolerance.
4. **Final recap**: blockchain trilemma, sharding, interoperability, Layer 2, and OP Rollup vs ZK Rollup.

### 中文
第 11 讲是一次综合性的区块链可扩展性课程，重点把三个层面的扩展问题串起来：

1. **数据扩展 - 数据可用性（Data Availability, DA）**：轻客户端、Rollup、分片系统如何确认区块数据真的被发布，而不是只看到一个区块头或状态根。
2. **EVM 扩展**：标准 EVM 为什么是顺序执行的；并行执行、访问列表、Block-STM、GPU 加速、模块化排序器如何提升吞吐。
3. **共识扩展**：非 Nakamoto 共识如何通过 BFT、HotStuff、Sync HotStuff 改善延迟、通信复杂度与容错能力。
4. **总复习**：区块链三难困境、分片、互操作、Layer 2、OP Rollup 与 ZK Rollup。

---

## 1. Fundamental Challenge / 可扩展区块链的根本挑战

### English
The lecture repeats the core scalability challenge:

> Every node should process only partial transactions, store only partial data, and use more efficient consensus.

But this creates three hard requirements:

| Requirement | Meaning | Why it matters |
|---|---|---|
| **State validity** | The new state must follow correctly from previous state and transactions. | Prevents invalid state transitions, fake balances, or unauthorized updates. |
| **Data availability** | The data needed to verify state transitions must be accessible. | Without data, validators or light clients cannot independently check correctness. |
| **Byzantine adversary resistance** | The protocol must tolerate malicious, colluding, or non-responsive participants. | Blockchain systems operate in adversarial environments. |

### 中文
本讲再次强调可扩展区块链的核心问题：

> 不是让每个节点处理所有交易、存储所有数据，而是让每个节点只处理部分交易、存储部分数据，并使用更高效的共识。

但这样会带来三个必须同时解决的难题：

| 要求 | 含义 | 为什么重要 |
|---|---|---|
| **状态有效性 State validity** | 新状态必须由旧状态和交易正确推导出来。 | 防止非法状态转换、凭空增发余额、未授权修改。 |
| **数据可用性 Data availability** | 验证状态转换所需的数据必须能够被获取。 | 如果数据不可见，轻节点或验证者无法独立验证区块。 |
| **拜占庭敌手 Byzantine adversary** | 协议必须能抵抗恶意、串谋、离线、拒绝响应的参与者。 | 区块链默认运行在对抗环境中。 |

**Page reference / 页码：** p.3, p.83。

---

# Part I - Scaling Data: Data Availability  
# 第一部分：数据扩展 - 数据可用性

---

## 2. Why Data Availability is a Big Issue in Sharding / 为什么 DA 是分片中的大问题

### English
The NEAR Protocol sharding example illustrates why DA becomes hard once transaction and state data are split across shards.

In the diagram on p.4:

- The **Beacon Chain** provides randomness, facilitates cross-shard transactions, and aggregates cross-links into finalized blocks.
- Each **Shard Chain** runs efficient BFT consensus.
- Validator selection can use VRF based on stake.
- The cross-link may include only the **block header**, not the full block data.
- Transaction and state sharding reduce per-node workload, but also create the DA problem: if only headers are finalized, who ensures that the underlying shard data is available?

### 中文
第 4 页用 NEAR Protocol 的分片架构说明：一旦交易和状态被切分到不同分片，数据可用性就会成为大问题。

图中结构包括：

- **Beacon Chain 信标链**：提供随机性，辅助跨分片交易，把 cross-link 聚合进最终确认区块。
- **Shard Chain 分片链**：各自运行高效 BFT 共识。
- 验证者选择可以基于权益和 VRF。
- cross-link 可能只包含**区块头**，而不是完整区块数据。
- 交易分片和状态分片降低了单节点负载，但也产生 DA 问题：如果信标链只看到区块头，谁保证区块体数据真的公开可得？

**Key insight / 核心理解：**

> Header availability is not data availability. A header can be finalized while the underlying data is hidden.
>
> 区块头可见不等于数据可用。区块头可以被最终确认，但区块体数据仍可能被隐藏。

---

## 3. Data Availability and Data Availability Proof / 数据可用性与数据可用性证明

### English
**Data Availability (DA)** ensures that all data necessary for verifying transactions is accessible to network participants. It enables independent verification of ledger correctness.

**Data Availability Proof (DAP)** uses cryptographic methods to verify that published data is actually accessible, helping maintain integrity and transparency.

DA is not the same as state validity:

- **State validity** asks: “Is the computation correct?”
- **Data availability** asks: “Can we obtain the data needed to check the computation?”

### 中文
**数据可用性（DA）**指的是：验证网络交易所需的全部数据，对参与者而言必须是可访问的。这样任何参与者才能独立验证账本条目的正确性。

**数据可用性证明（DAP）**则使用密码学机制，证明数据已经发布并且能够被访问，从而维护系统完整性与透明性。

DA 与状态有效性不同：

- **状态有效性**问的是：“计算结果对不对？”
- **数据可用性**问的是：“用于检查计算结果的数据能不能拿到？”

**Page reference / 页码：** p.5。

---

## 4. Full Nodes vs Light Clients / 全节点与轻客户端

### English
The lecture compares full nodes and light clients.

**Full nodes:**

- Download and verify the entire blockchain.
- Validate every transaction.
- Broadcast block headers to light clients.
- May participate in consensus.

**Light clients:**

- Receive block headers.
- Use Merkle proofs to verify inclusion.
- Do not download or verify all transaction data.

The DA problem appears when full nodes or validators are dishonest or collude. If light clients only receive headers, they may accept a block whose underlying data is unavailable or whose transactions are invalid.

### 中文
课程区分了全节点与轻客户端。

**全节点：**

- 下载并验证整条区块链。
- 验证每笔交易。
- 向轻客户端广播区块头。
- 可能参与共识。

**轻客户端：**

- 接收区块头。
- 使用 Merkle proof 验证某笔交易或数据是否被包含。
- 不下载、也不验证所有交易数据。

问题在于：如果全节点或验证者不诚实、发生串谋，而轻客户端只看到区块头，那么轻客户端可能接受一个底层数据不可用或包含无效交易的区块。

**Visual notes / 图示说明：**

- p.6 展示全节点与轻客户端网络结构。
- p.7 展示轻客户端依赖区块头和 Merkle proof。
- p.8 展示恶意验证者可能只传播 header，使轻客户端无法检查完整数据。

---

## 5. Fraud Proof / 欺诈证明

### English
A **fraud proof** contains the transaction and Merkle tree data needed to convince a light client that a block is invalid.

The important implication:

> If fraud proofs are possible, light clients can theoretically obtain near-full-node-equivalent assurance that accepted blocks are not fraudulent.

However, fraud proofs require that the data needed to construct the proof is actually available. This links fraud proofs directly to the DA problem.

### 中文
**欺诈证明（fraud proof）**包含交易和相关 Merkle 树数据，用于说服轻客户端某个区块是无效的。

关键意义是：

> 如果欺诈证明机制可行，轻客户端理论上可以获得接近全节点级别的安全保证。

但是欺诈证明必须依赖可获取的数据。若数据被隐藏，诚实节点可能无法构造欺诈证明。因此 fraud proof 与 DA 问题紧密绑定。

**Page reference / 页码：** p.9。

---

## 6. Naive Solution and Why It Fails / 朴素方案及其失败原因

### English
The naive idea is for a light client to ask a full node for data chunk `mi` and its proof. But an adversary may withhold some chunk `mj`. To be safe, the light client may eventually need to request all chunks, effectively becoming a full node.

This does not scale:

- It loses the benefit of being a light client.
- It creates high bandwidth requirements.
- It harms decentralization.

The lecture’s conclusion:

> To scale, chunks should be dependent, and data should be recoverable from fewer chunks.

This motivates **erasure coding**.

### 中文
一个朴素方案是：轻客户端向全节点请求某个数据片段 `mi` 和证明。但攻击者可能隐藏另一个片段 `mj`。为了安全，轻客户端最终可能不得不请求全部片段，这就变成了全节点。

这个方案无法扩展：

- 失去轻客户端的低资源优势。
- 带宽需求过高。
- 破坏去中心化。

课程给出的结论是：

> 要扩展，数据片段之间应该存在依赖关系，并且应该能用较少片段恢复完整数据。

这引出了**纠删码 Erasure Coding**。

**Page reference / 页码：** p.10。

---

## 7. Erasure Coding and Reed-Solomon Codes / 纠删码与 Reed-Solomon 编码

### English
The lecture introduces erasure coding using Reed-Solomon (RS) codes.

Core idea:

- Start with data chunks such as `m1, m2, m3, m4`.
- Generate redundant coded chunks such as `m5, m6, m7, m8`.
- In RS coding, any 50% of symbols can allow full recovery.
- The construction relies on finite-field linear algebra.

For RS codes:

- Construct a polynomial `p(x) = a0 + a1x + a2x^2 + a3x^3`.
- Coefficients `ai` come from a finite field.
- Evaluate the polynomial at points such as `(1, p(1)), ..., (4, p(4))`.
- This forms a Vandermonde system `Va = p`.
- Recover original coefficients by `V^-1 p = a`.

### 中文
课程用 Reed-Solomon（RS）码介绍纠删码。

核心思想：

- 原始数据片段如 `m1, m2, m3, m4`。
- 生成冗余编码片段如 `m5, m6, m7, m8`。
- 在 RS 编码中，任意 50% 的符号都可以恢复完整数据。
- 其数学基础是有限域线性代数。

RS 码构造方式：

- 构造多项式 `p(x) = a0 + a1x + a2x^2 + a3x^3`。
- 系数 `ai` 来自有限域。
- 在多个点上求值，例如 `(1, p(1)), ..., (4, p(4))`。
- 形成 Vandermonde 矩阵方程 `Va = p`。
- 通过 `V^-1 p = a` 恢复原始数据。

**Study note / 学习提示：**

RS 编码的重点不是记公式，而是理解“冗余编码使随机抽样变得有效”：攻击者如果想隐藏数据，就必须隐藏大量编码片段，否则其他节点可以恢复原数据。

**Page reference / 页码：** p.11-p.12。

---

## 8. Merkle Tree of Erasure-Coded Blocks / 纠删编码区块的 Merkle 树

### English
The lecture then combines erasure coding with Merkle trees.

The block is encoded into symbols `m1 ... m8`, each symbol has a hash `h1 ... h8`, and these hashes are aggregated into a Merkle root.

Why this helps:

- A light client can randomly sample some encoded chunks.
- If an adversary reveals some chunks, but hides too many others, the hiding becomes detectable.
- With coding, an adversary must hide a large fraction of symbols to make the block unrecoverable.

The p.14 example shows light clients asking for random pieces such as `m2`, `m4`, and `p3`. Once too many samples are requested, the adversary can no longer answer consistently and must hide many remaining pieces, which is easily detectable.

### 中文
课程随后把纠删码与 Merkle 树结合起来。

区块被编码为 `m1 ... m8` 等符号，每个符号有对应哈希 `h1 ... h8`，再聚合成 Merkle root。

这样做的好处：

- 轻客户端可以随机抽样部分编码片段。
- 如果攻击者能回答一部分请求，却隐藏过多剩余片段，则隐藏行为会暴露。
- 使用编码后，攻击者必须隐藏大量符号，才能让数据无法恢复。

第 14 页示例中，轻客户端随机请求 `m2`、`m4`、`p3` 等片段。当抽样足够多时，攻击者无法继续一致地回答，只能隐藏剩余大量片段，这很容易被检测到。

---

## 9. Coding vs No Coding / 编码与不编码的区别

### English
Without coding:

- If the adversary hides one data symbol, light clients must sample many pieces to catch it.
- Sampling complexity is roughly linear in the number of data symbols.

With coding:

- If the adversary wants to make data unrecoverable, it must hide many coded symbols.
- Random sampling catches hiding with high probability using only O(1) samples.

### 中文
不使用编码时：

- 攻击者只要隐藏一个数据片段，轻客户端就必须抽样很多次才可能发现。
- 抽样复杂度接近数据片段数量的线性级别。

使用编码后：

- 攻击者若想让数据无法恢复，必须隐藏大量编码片段。
- 轻客户端只需 O(1) 次随机抽样，就能以高概率发现隐藏行为。

**Page reference / 页码：** p.15。

---

## 10. Incorrect-Coding Attack / 错误编码攻击

### English
Erasure coding creates another problem: the adversary may encode data incorrectly and publish almost everything.

This is called an **incorrect-coding attack**.

Problem:

- Light nodes can sample chunks and Merkle proofs.
- But checking whether the entire coding is correct may require reproducing the coding process.
- The proof size can become as large as the block size.

For 1D Reed-Solomon:

| Scheme | Number of roots | Sampling + Merkle proofs | Incorrect-coding proof size | Encoding/decoding complexity |
|---|---:|---:|---:|---:|
| 1D-RS | 1 | O(1) | O(K) | O(K^2) |

where `K` is the block size in number of symbols.

### 中文
纠删码又带来另一个问题：攻击者可以错误编码，然后发布几乎全部数据。

这叫做**错误编码攻击（incorrect-coding attack）**。

问题在于：

- 轻节点可以抽样数据片段和 Merkle proof。
- 但要验证整个编码是否正确，可能需要重新执行编码。
- 错误编码证明的大小可能达到区块大小。

对于 1D Reed-Solomon：

| 方案 | Merkle root 数量 | 抽样和 Merkle proof | 错误编码证明大小 | 编码/解码复杂度 |
|---|---:|---:|---:|---:|
| 1D-RS | 1 | O(1) | O(K) | O(K^2) |

其中 `K` 是区块被切分成的符号数量。

**Page reference / 页码：** p.16-p.17。

---

## 11. 2D Reed-Solomon: Divide and Conquer / 二维 RS：分而治之

### English
To reduce incorrect-coding proof size, the lecture introduces 2D-RS.

Core idea:

1. Turn `K` symbols into a `sqrt(K) x sqrt(K)` square.
2. Apply 1D-RS to each row and each column.
3. Use multiple row roots and column roots.
4. To prove incorrect coding, one can send the chunks in the problematic row or column.

The lecture notes that 2D-RS uses more roots than 1D-RS, but reduces incorrect-coding proof size.

### 中文
为了降低错误编码证明大小，课程介绍 2D-RS。

核心思想：

1. 把 `K` 个符号排成 `sqrt(K) x sqrt(K)` 的矩阵。
2. 对每一行和每一列分别应用 1D-RS。
3. 使用多个行 root 和列 root。
4. 若出现错误编码，只需发送出问题的行或列数据作为证明。

课程指出：2D-RS 比 1D-RS 使用更多 root，但显著降低错误编码证明大小。

**Page reference / 页码：** p.18。

---

## 12. Can We Do Better? LDPC and Coded Merkle Tree / 能不能更好？LDPC 与编码 Merkle 树

### English
The lecture asks whether one can do better than 2D-RS. It introduces the idea of low-density codes:

- Instead of each coded symbol being a linear combination of all chunks, make it a linear combination of only a subset.
- This makes it easier to point out invalid coding using fewer chunks.
- LDPC codes are used in 4G.
- A good LDPC class can have equation size 6, and any 87.6% of symbols allows full recovery through iterative decoding.

However, a new attack appears:

> Incorrect coding + hiding hashes.

If not all lower-layer hashes are available, a light node may reconstruct a value but cannot verify it against the missing hash.

This motivates **Coded Merkle Tree (CMT)**.

### 中文
课程进一步问：能不能比 2D-RS 更好？于是引入低密度编码的想法：

- 不让每个编码符号都成为全部数据片段的线性组合，而是只依赖一小部分片段。
- 这样更容易用少量片段指出错误编码。
- LDPC 码被用于 4G。
- 一类好的 LDPC 码可以做到方程大小为 6，并且任意 87.6% 符号可通过迭代解码恢复。

但新的攻击出现了：

> 错误编码 + 隐藏哈希。

如果底层哈希不可用，轻节点即使恢复出某个值，也无法验证该值是否匹配缺失的哈希。

这就引出了**编码 Merkle 树（Coded Merkle Tree, CMT）**。

**Page reference / 页码：** p.19-p.20。

---

## 13. Coded Merkle Tree (CMT) / 编码 Merkle 树

### English
Coded Merkle Tree solves DA attacks by maintaining availability layer by layer.

Key ideas:

- Partition the block.
- Erasure-encode data at each layer.
- Hash encoded values.
- Repeat encoding and hashing layer by layer.
- Ensure DA at every layer.

The lecture mentions:

- 4-ary Merkle tree.
- 8 hashes with 8 coded hashes.
- Light node sampling automatically samples every layer.
- Honest full nodes can decode and audit layer by layer.
- If incorrect coding is detected, the block is rejected.

Performance table from p.24:

| Scheme | Roots | Samples + Merkle proofs | Incorrect-coding proof size | Encode/decode complexity |
|---|---:|---:|---:|---:|
| 1D-RS | 1 | O(1) | O(K) | O(K^2) |
| 2D-RS | O(sqrt(K)) or many roots | O(1) | O(sqrt(K)) | improved vs 1D |
| CMT | 1 | O(1) | 6 symbols | O(K) |

For `K = 65536`, encoded to `65536 * 4` coded symbols:

| Scheme | Roots | Samples for 99% confidence | Incorrect-coding proof size | Encode/decode complexity |
|---|---:|---:|---:|---:|
| 1D-RS | 1 | 7 | 65536 | 13B ops |
| 2D-RS | 1025 | 15 | 256 | 50M ops |
| CMT | 1 | 32 | 6 | 1.2M ops |

### 中文
编码 Merkle 树通过“逐层维护数据可用性”解决 DA 攻击。

关键思想：

- 对区块进行切分。
- 每层都做纠删编码。
- 对编码后的值做哈希。
- 一层层重复编码与哈希。
- 保证每一层的数据可用性。

课程中提到：

- 使用 4 叉 Merkle 树。
- 8 个原始哈希配 8 个编码哈希。
- 轻节点抽样时会自动覆盖每一层。
- 诚实全节点可以逐层解码并审计。
- 如果检测到错误编码，就拒绝区块。

第 24 页性能对比：

| 方案 | Root 数量 | 抽样和 Merkle proof | 错误编码证明大小 | 编码/解码复杂度 |
|---|---:|---:|---:|---:|
| 1D-RS | 1 | O(1) | O(K) | O(K^2) |
| 2D-RS | 多个 root | O(1) | O(sqrt(K)) | 相比 1D 改善 |
| CMT | 1 | O(1) | 6 个符号 | O(K) |

当 `K = 65536`，并编码为 `65536 * 4` 个符号时：

| 方案 | Root 数量 | 99% 置信度所需抽样数 | 错误编码证明大小 | 编码/解码复杂度 |
|---|---:|---:|---:|---:|
| 1D-RS | 1 | 7 | 65536 | 13B ops |
| 2D-RS | 1025 | 15 | 256 | 50M ops |
| CMT | 1 | 32 | 6 | 1.2M ops |

**Page reference / 页码：** p.21-p.24。

---

## 14. Data Availability for Rollups / Rollup 的数据可用性

### English
Rollups use Layer 1 for DA differently depending on whether they are optimistic or validity-proof based.

### Optimistic Rollup
Workflow shown on p.25:

1. Users submit transaction messages and signatures.
2. The rollup applies transactions and computes a new state root.
3. A new rollup block is produced.
4. Transaction data and signatures are committed to L1.
5. If someone challenges the transition, the rollup contract verifies `verify_state_transition(pre_state, block, witness)`.
6. A valid challenge rewards the challenger and slashes the validator.

Dispute resolution may be:

- One-round, where L1 verifies the disputed transition directly.
- Multi-round, such as Arbitrum, where asserter and challenger narrow down the dispute interactively to minimize final L1 work.

### zkRollup
Workflow shown on p.26:

1. Users submit transaction messages and signatures.
2. The rollup applies transactions and computes old/new state roots.
3. A new rollup block is formed.
4. A validity proof `π` is generated and committed.

Important distinction:

- Optimistic rollups rely on fraud proofs after a challenge.
- zkRollups rely on validity proofs before acceptance.
- Both still need DA for users to reconstruct state and exit.

### 中文
Rollup 对 L1 数据可用性的依赖，会因 OP Rollup 与 ZK Rollup 而不同。

### Optimistic Rollup
第 25 页流程：

1. 用户提交交易消息和签名。
2. Rollup 执行交易并计算新的状态根。
3. 生成新的 Rollup 区块。
4. 将交易数据和签名提交到 L1。
5. 如果有人挑战状态转换，Rollup 合约验证 `verify_state_transition(pre_state, block, witness)`。
6. 挑战成功则奖励挑战者，并惩罚/削减验证者。

争议解决可以是：

- 单轮：L1 直接验证争议状态转换。
- 多轮：例如 Arbitrum，通过 asserter 和 challenger 在合约上交互式缩小争议范围，减少最终由 L1 执行的工作量。

### zkRollup
第 26 页流程：

1. 用户提交交易消息和签名。
2. Rollup 执行交易并计算旧/新状态根。
3. 形成新的 Rollup 区块。
4. 生成有效性证明 `π` 并提交。

关键区别：

- OP Rollup 依赖挑战后的欺诈证明。
- ZK Rollup 依赖提交时的有效性证明。
- 两者仍然都需要 DA，用户才能重建状态、退出系统或独立验证。

---

## 15. Celestia Case Study / Celestia 案例研究

### English
Celestia is presented as a modular blockchain network for DA.

Core design:

- Separate consensus and execution layers.
- Provide a DA layer for rollups or other execution environments.
- Achieve scalability and interoperability by specializing in data publication and sampling.

How it works:

- Allows **Data Availability Sampling (DAS)**.
- Uses **Namespaced Merkle Trees (NMTs)**.
- DAS lets light nodes verify data availability without downloading an entire block.
- NMTs let execution and settlement layers download only transactions relevant to their namespace.

### 中文
Celestia 被作为模块化 DA 区块链案例。

核心设计：

- 分离共识层与执行层。
- 为 Rollup 或其他执行环境提供 DA 层。
- 通过专注数据发布与抽样，提高可扩展性和互操作性。

工作机制：

- 支持**数据可用性抽样（DAS）**。
- 使用**命名空间 Merkle 树（NMT）**。
- DAS 让轻节点无需下载完整区块即可验证 DA。
- NMT 让执行层或结算层只下载与自己命名空间相关的交易。

**Page reference / 页码：** p.27-p.28。

---

## 16. Celestia DAS and NMT / Celestia 的 DAS 与 NMT

### English
Celestia’s DAS uses:

- Block headers containing commitments.
- 2D Reed-Solomon encoding.
- Sampling from a `2k x 2k` extended data square.
- Light nodes randomly choose coordinates, query full nodes, and verify Merkle proofs.
- For block data size `n^2` bytes, a light node downloads `O(n)` bytes.

Namespaced Merkle Trees:

- Leaves are grouped by namespace.
- Each internal node commits not only to hashes, but also to namespace ranges.
- This enables selective proof and retrieval for particular applications or rollups.

### 中文
Celestia 的 DAS 使用：

- 包含承诺的区块头。
- 二维 Reed-Solomon 编码。
- 从 `2k x 2k` 扩展数据矩阵中抽样。
- 轻节点随机选择坐标，向全节点请求数据与 Merkle proof。
- 若区块数据大小为 `n^2` 字节，轻节点只需下载 `O(n)` 字节。

命名空间 Merkle 树：

- 叶子按 namespace 分组。
- 内部节点不仅承诺哈希，也承诺命名空间范围。
- 这使应用或 Rollup 可以只检索与自己相关的数据。

**Visual notes / 图示说明：**

- p.29 展示二维 RS 编码和轻节点随机抽样。
- p.30 展示 NMT：不同 namespace 的数据被组织在同一 Merkle 树中，并通过 namespace 范围支持选择性证明。

---

## 17. Celestia Architecture / Celestia 架构

### English
The Celestia DA layer is a PoS blockchain.

- `celestia-app` is built on top of `celestia-core`.
- `celestia-core` is a modified version of Tendermint.
- It enables erasure coding of block data.
- It replaces Tendermint’s standard Merkle tree with a Namespaced Merkle Tree.
- `celestia-core` connects to the application layer through ABCI++.

Transaction lifecycle:

- A PayForBlobs transaction enters Celestia App.
- The state machine processes e-transactions and blobs.
- The proposed block contains a header with commitment and a data section containing e-Txs and blobs.
- The DA network serves extended headers and data chunks to light nodes.

### 中文
Celestia DA 层本身是一条 PoS 区块链。

- `celestia-app` 构建在 `celestia-core` 之上。
- `celestia-core` 是修改版 Tendermint。
- 它支持区块数据纠删编码。
- 它用 Namespaced Merkle Tree 替代 Tendermint 原本的 Merkle tree。
- `celestia-core` 通过 ABCI++ 与应用层连接。

交易生命周期：

- PayForBlobs 交易进入 Celestia App。
- 状态机处理 e-transactions 与 blobs。
- 提议区块包含带 commitment 的 header，以及包含 e-Txs 和 blobs 的 data 部分。
- DA 网络向轻节点提供 extended headers 和 data chunks。

**Page reference / 页码：** p.31-p.33。

---

## 18. EigenDA / EigenLayer 的 EigenDA

### English
The lecture also shows EigenDA from EigenLayer.

Architecture on p.34:

- A sequencer orders transactions.
- Data blobs are sent to a disperser.
- The disperser performs KZG proof generation and dispersal.
- EigenDA nodes store chunks and return signatures.
- Aggregated signatures are sent to an EigenDA contract on Ethereum.
- Rollups can use this as an external DA layer.

### 中文
课程也展示了 EigenLayer 的 EigenDA。

第 34 页架构：

- Sequencer 对交易排序。
- 数据 blob 被发送给 disperser。
- Disperser 执行 KZG 证明生成和数据分发。
- EigenDA 节点存储 chunk 并返回签名。
- 聚合签名被发送到以太坊上的 EigenDA 合约。
- Rollup 可以将 EigenDA 用作外部 DA 层。

---

## 19. Why DA Scaling Matters Now / 为什么当前 DA 扩展很重要

### English
The p.35 table compares Ethereum L1 execution, Ethereum blobs, and Celestia.

| Feature | Ethereum L1 Execution | Ethereum Blobs (EIP-4844) | Celestia |
|---|---|---|---|
| DA method | Full node download | Blobs + KZG commitments | Erasure coding + sampling |
| Light client support | Weak | Stronger via KZG proofs | Strong via DAS |
| Scalable for rollups | No | Yes | Yes |
| Long-term storage | Permanent | Temporary, around 18 days | Depends on application |

The key note: Ethereum light clients cannot verify data availability under ordinary L1 execution.

### 中文
第 35 页比较了 Ethereum L1 执行、Ethereum Blobs（EIP-4844）和 Celestia。

| 特性 | Ethereum L1 Execution | Ethereum Blobs (EIP-4844) | Celestia |
|---|---|---|---|
| DA 方法 | 全节点下载 | Blobs + KZG commitments | 纠删编码 + 抽样 |
| 轻客户端支持 | 弱 | 通过 KZG proofs 更强 | 通过 DAS 很强 |
| 是否适合 Rollup 扩展 | 否 | 是 | 是 |
| 长期存储 | 永久 | 临时，约 18 天 | 取决于应用 |

关键备注：在普通以太坊 L1 执行下，以太坊轻客户端无法验证数据可用性。

---

# Part II - EVM Scaling  
# 第二部分：EVM 扩展

---

## 20. Ethereum World State / 以太坊世界状态

### English
The Ethereum world state is a mapping between addresses and account states.

Important details:

- Addresses are 160-bit identifiers.
- States are not stored directly “on the blockchain” as raw history.
- The mapping is maintained in a trie database.
- A root hash identifies the world state.
- Account state is a collection of code and data.

### 中文
以太坊世界状态是地址到账号状态的映射。

重要细节：

- 地址是 160-bit 标识符。
- 状态不是以原始形式直接“存储在区块链上”。
- 映射由 trie 数据库维护。
- root hash 标识当前世界状态。
- 账号状态可以理解为 code + data 的集合。

**Page reference / 页码：** p.37。

---

## 21. State-Transition Chain / 状态转换链

### English
Ethereum is a state machine.

- It starts from a genesis state `S0`.
- Each block contains transactions.
- Transactions update the previous world state into the next world state.
- A transaction is an atomic digitally signed instruction on the world state.

Formula-like view:

```text
S_n + Tx -> S_{n+1}
```

### 中文
以太坊可以看作一个状态机。

- 从创世状态 `S0` 开始。
- 每个区块包含若干交易。
- 交易把前一个世界状态更新为下一个世界状态。
- 交易是对世界状态的一条原子性的、数字签名的指令。

类公式表示：

```text
S_n + Tx -> S_{n+1}
```

**Page reference / 页码：** p.38-p.39。

---

## 22. Ethereum Virtual Machine / 以太坊虚拟机

### English
The EVM executes contract bytecode based on:

- The current account state.
- The input data or init data specified in the transaction.

During execution:

- Nonce gets updated.
- Balance may update.
- Storage gets updated.
- Code remains the same.

### 中文
EVM 根据以下内容执行合约字节码：

- 当前账号状态。
- 交易指定的 input data 或 init data。

执行过程中：

- nonce 会更新。
- balance 可能更新。
- storage 会更新。
- code 通常保持不变。

**Page reference / 页码：** p.40。

---

## 23. EVM Architecture and Instruction Set / EVM 架构与指令集

### English
EVM is a stack-based machine with internal memory.

Components:

| Component | Role |
|---|---|
| Stack | Temporary computation stack, 256-bit entries, up to 1024 elements. |
| Memory | Volatile execution memory, byte-addressed. |
| Storage | Persistent contract storage, part of world state. |
| Virtual ROM | Contract code; immutable during execution. |
| Program Counter | Tracks current opcode location. |
| Arguments | Input data supplied by the transaction. |

Opcode categories:

- Arithmetic and logic operations.
- Stack, memory, and storage access.
- Control-flow operations.
- Environmental enquiries.
- System operations such as logging and calls.

### 中文
EVM 是一个基于栈的虚拟机，并带有内部内存。

组件：

| 组件 | 作用 |
|---|---|
| Stack 栈 | 临时计算栈，256-bit 项，最多 1024 个元素。 |
| Memory 内存 | 易失性执行内存，按字节寻址。 |
| Storage 存储 | 合约持久化存储，是世界状态的一部分。 |
| Virtual ROM | 合约代码，执行期间不可变。 |
| Program Counter | 指向当前执行 opcode 的位置。 |
| Arguments | 交易提供的输入数据。 |

Opcode 类型：

- 算术与逻辑操作。
- 栈、内存、存储访问。
- 控制流操作。
- 执行环境查询。
- 日志、调用等系统操作。

**Page reference / 页码：** p.41-p.42。

---

## 24. EVM Compilers / EVM 编译器

### English
Smart contracts are usually written in high-level languages and compiled into EVM bytecode.

- **Solidity**: high-level object-oriented language; statically typed; C++-like; supports inheritance, libraries, and custom types.
- **Vyper**: Pythonic high-level language; strongly typed; deliberately fewer features; no inheritance or overloading.
- Runtime systems include Ethereum nodes such as Geth and OpenEthereum.
- Hardware platforms include x86 and ARM processors.

### 中文
智能合约通常用高级语言编写，再编译成 EVM 字节码。

- **Solidity**：高级面向对象语言；静态类型；类似 C++；支持继承、库、自定义类型。
- **Vyper**：Python 风格高级语言；强类型；刻意减少特性；不支持继承和重载。
- 运行时系统包括 Geth、OpenEthereum 等以太坊节点。
- 硬件平台包括 x86、ARM 等处理器。

**Page reference / 页码：** p.43。

---

## 25. Sequential EVM Workflow / 顺序 EVM 工作流

### English
The standard EVM workflow is sequential:

1. Select transactions.
2. Execute opcode by opcode.
3. Store updates in stateDB.

Transactions update the world state sequentially. This guarantees deterministic execution, but creates performance bottlenecks.

Limitations:

- Slow and inefficient.
- Performance degrades when the network is busy.

### 中文
标准 EVM 的工作流是顺序的：

1. 选择交易。
2. 按 opcode 执行。
3. 将更新写入 stateDB。

交易按顺序更新世界状态。这保证了确定性执行，但也造成性能瓶颈。

局限：

- 慢且低效。
- 网络繁忙时性能下降明显。

**Page reference / 页码：** p.44-p.45。

---

## 26. Parallel Execution / 并行执行

### English
Parallel execution tries to run independent transactions on multiple EVM instances or threads.

Main challenge: conflicts.

Questions:

- Which transactions can run in parallel?
- How to schedule transactions?
- How to detect and manage dependencies?
- How to merge states after parallel execution?

### 中文
并行执行试图让互不依赖的交易在多个 EVM 实例或多个线程中同时执行。

核心挑战是冲突。

需要解决的问题：

- 哪些交易可以并行执行？
- 如何调度交易？
- 如何检测和管理依赖？
- 并行执行后如何合并状态？

**Page reference / 页码：** p.46。

---

## 27. Account-Based Access Lists / 基于账号访问列表的并行执行

### English
One approach is to require each transaction to declare which accounts it will read or write.

Rules:

- Scheduler runs non-conflicting transactions in parallel.
- If two transactions touch the same account, they are executed sequentially.
- Solana uses pre-declared access lists to schedule transactions in parallel.

Pros:

- Clear scheduling.
- Easier conflict detection.
- Better deterministic execution.

Cons:

- Requires accurate access declaration.
- May be conservative if access lists are too broad.
- Harder for dynamically accessed state.

### 中文
一种方法是要求每笔交易声明自己会读写哪些账号。

规则：

- 调度器并行运行不冲突的交易。
- 如果两笔交易访问同一个账号，则顺序执行。
- Solana 使用预声明访问列表进行并行调度。

优点：

- 调度清晰。
- 冲突检测容易。
- 更容易保证确定性执行。

缺点：

- 需要准确声明访问集合。
- 如果访问列表过宽，调度会过于保守。
- 动态访问状态时更困难。

**Page reference / 页码：** p.47。

---

## 28. Block-STM / 软件事务内存 Block-STM

### English
Block-STM uses optimistic parallelization and Software Transactional Memory.

Methods:

- Execute transactions optimistically in parallel.
- Detect conflicts dynamically.
- Re-execute conflicting transactions.
- Resolve conflicts and merge state.

Advantages:

- Scales efficiently with more workload.
- Detects conflicts dynamically instead of requiring all accesses upfront.

Used in:

- Layer 1: Aptos, Polygon PoS, Monad, Sei V2.
- Layer 2: Reddio.

### 中文
Block-STM 使用乐观并行化和软件事务内存。

方法：

- 乐观地并行执行交易。
- 动态检测冲突。
- 对冲突交易重新执行。
- 解决冲突并合并状态。

优点：

- 工作负载越大，扩展效果越明显。
- 不要求事先完全声明访问集合，而是动态检测冲突。

应用：

- Layer 1：Aptos、Polygon PoS、Monad、Sei V2。
- Layer 2：Reddio。

**Page reference / 页码：** p.48。

---

## 29. Why Parallel Execution Matters / 为什么并行执行重要

### English
Parallel execution is described as a next-generation Layer 2 scalability solution.

Use cases requiring high throughput and low latency:

- DEX with order book.
- GameFi.
- Low-latency DePIN.
- Micropayments.

The key idea is that future blockchain applications look more like high-performance online systems, not just simple token transfers.

### 中文
课程把并行执行视为下一代 Layer 2 扩展方案之一。

需要高吞吐和低延迟的场景包括：

- 订单簿型 DEX。
- GameFi。
- 低延迟 DePIN。
- 微支付。

关键理解：未来链上应用会越来越接近高性能在线系统，而不只是简单转账。

**Page reference / 页码：** p.49。

---

## 30. Reddio Architecture and Approach / Reddio 架构与方法

### English
Reddio is presented as an example of parallel EVM scaling.

Architecture:

- Solidity code is compiled into EVM code.
- Runtime system: Reddio Node / Parallel Geth.
- Hardware platform: processors and CUDA-compatible GPUs.
- Ethereum remains the settlement/base layer.

Approach:

- GPU acceleration.
- Decentralized GPU network.
- Decentralized ZKP network.
- Scaling with modular sequencer SDK.

The p.51 visual suggests large speedups from Parallel EVM and GPU/CUDA acceleration, and a Reddio L2 network plus Reddio ZK network.

### 中文
Reddio 被作为并行 EVM 扩展案例。

架构：

- Solidity 代码被编译为 EVM code。
- 运行时系统：Reddio Node / Parallel Geth。
- 硬件平台：处理器和 CUDA 兼容 GPU。
- Ethereum 作为结算/基础层。

方法：

- GPU 加速。
- 去中心化 GPU 网络。
- 去中心化 ZKP 网络。
- 通过模块化 sequencer SDK 扩展。

第 51 页图中展示 Parallel EVM 与 GPU/CUDA 加速带来的吞吐提升，以及 Reddio L2 网络和 Reddio ZK 网络。

**Page reference / 页码：** p.50-p.51。

---

# Part III - Scaling Consensus  
# 第三部分：共识扩展

---

## 31. From Nakamoto Consensus to Non-Nakamoto Consensus / 从 Nakamoto 共识到非 Nakamoto 共识

### English
The lecture begins the consensus section by noting:

- Bitcoin and its variants use Nakamoto consensus: Proof-of-Work and longest-chain-wins.
- The focus here is non-Nakamoto consensus.
- The lecture surveys decades of distributed computing research.
- It focuses on a recent protocol: Sync HotStuff.

### 中文
共识部分从以下背景开始：

- Bitcoin 及其变体使用 Nakamoto 共识：PoW + 最长链胜出。
- 本节关注非 Nakamoto 共识。
- 课程回顾分布式计算 40 年研究。
- 重点介绍较新的协议：Sync HotStuff。

**Page reference / 页码：** p.52-p.53。

---

## 32. Consensus as Replicated Service / 作为复制服务的共识

### English
Consensus is one of the most fundamental problems in distributed computing, dating back to Pease-Shostak-Lamport 1980.

Target application: replicated service.

The goal is to provide an abstraction of a single non-faulty server, even though the system consists of multiple nodes.

Two properties:

| Property | Meaning |
|---|---|
| Safety / consistency | All nodes commit the same sequence of values. |
| Liveness | Nodes keep committing new values. |

### 中文
共识是分布式计算中最基础的问题之一，可追溯到 Pease-Shostak-Lamport 1980。

目标应用是复制服务。

共识的目标是在多个节点组成的系统中，提供一个“单一无故障服务器”的抽象。

两个性质：

| 性质 | 含义 |
|---|---|
| Safety / consistency 安全性/一致性 | 所有节点提交相同的值序列。 |
| Liveness 活性 | 系统持续提交新的值。 |

**Page reference / 页码：** p.54。

---

## 33. Many Faces of Consensus / 共识问题的多种模型

### English
Consensus changes significantly depending on the model.

Key model questions:

- How is the problem defined? Replication, broadcast, agreement, etc.
- Is every pair of nodes connected? Static or dynamic network?
- Is there a bound on message delay? Synchrony, asynchrony, partial synchrony.
- What faults exist? Crash faults, Byzantine faults, etc.

Small model changes can lead to very different protocols, impossibility results, and lower bounds.

### 中文
共识问题会随着模型假设变化而显著变化。

关键模型问题：

- 问题如何定义？复制、广播、协议一致等。
- 节点之间是否两两连接？网络是静态还是动态？
- 消息延迟是否有上界？同步、异步、部分同步。
- 故障类型是什么？崩溃故障、拜占庭故障等。

微小的模型变化可能导致完全不同的协议、不可能性结果和下界。

**Page reference / 页码：** p.55-p.56。

---

## 34. Fault Tolerance Bounds / 容错边界

### English
The lecture summarizes classic fault tolerance bounds for replication.

Let:

- `n` = total number of nodes.
- `f` = faulty nodes.

| Fault model | Synchrony | Partial synchrony |
|---|---:|---:|
| Crash faults | f < n | f < n / 2 |
| Byzantine faults | f < n / 2 | f < n / 3 |

Examples:

- Primary-backup in synchronous crash-fault settings.
- Paxos for partial synchrony with crash faults.
- PBFT / HotStuff for Byzantine partial synchrony.
- Deterministic asynchrony has very restrictive guarantees.

### 中文
课程总结了复制系统中的经典容错边界。

设：

- `n` = 节点总数。
- `f` = 故障节点数。

| 故障模型 | 同步网络 | 部分同步网络 |
|---|---:|---:|
| 崩溃故障 | f < n | f < n / 2 |
| 拜占庭故障 | f < n / 2 | f < n / 3 |

示例：

- 同步崩溃故障模型中的 primary-backup。
- 部分同步崩溃故障模型中的 Paxos。
- 部分同步拜占庭模型中的 PBFT / HotStuff。
- 确定性异步模型的保证非常受限。

**Page reference / 页码：** p.57。

---

## 35. Consensus Algorithms: Proof-Based vs Voting-Based / 共识算法：基于证明与基于投票

### English
The lecture distinguishes two broad families:

**Proof-based consensus**

- Uses proof to validate transactions and create blocks.
- Examples include PoW and PoS variants.

**Voting-based consensus**

- Validators vote to reach agreement on ledger state.
- Usually faster and consumes less power.
- Often permissioned or limited to a validator set.

### 中文
课程区分两大类共识：

**基于证明的共识**

- 使用某种 proof 来验证交易并创建新区块。
- 例子包括 PoW、PoS 变体。

**基于投票的共识**

- 验证者通过投票对账本状态达成一致。
- 通常更快、能耗更低。
- 往往是 permissioned，或限制在特定验证者集合内。

**Page reference / 页码：** p.58。

---

## 36. Byzantine Fault Tolerance / 拜占庭容错 BFT

### English
BFT addresses the Byzantine Generals’ Problem, where some participants may lie, equivocate, or coordinate maliciously.

A common BFT intuition:

- About two-thirds of nodes must agree.
- The system assumes a higher level of trust among a validator set than open PoW systems.
- BFT protocols aim for fast finality.

The p.60 diagram shows BFT voting rounds: request, pre-prepare, prepare, commit, reply.

### 中文
BFT 解决拜占庭将军问题：部分参与者可能撒谎、发送矛盾消息、恶意协作。

常见 BFT 直觉：

- 大约需要三分之二节点同意。
- 相比开放 PoW 系统，BFT 通常假设一个较明确的验证者集合。
- BFT 协议通常追求快速最终性。

第 60 页图展示 BFT 投票轮次：request、pre-prepare、prepare、commit、reply。

**Page reference / 页码：** p.59-p.61。

---

## 37. HotStuff / HotStuff 共识

### English
HotStuff:

- Proposed by VMware Research.
- Used by Libra.
- Based on PBFT.
- Operates under partial synchrony.
- Moves from mesh communication to star communication.
- PBFT-like two rounds become three phases with view change.

HotStuff phases:

1. **Prepare**: leader sends prepare message.
2. **Pre-commit**: prepare messages form a prepare quorum certificate.
3. **Commit**: precommitQC.
4. **Decide**: commitQC, decide message sent, next view starts.

### 中文
HotStuff：

- 由 VMware Research 提出。
- 被 Libra 使用。
- 基于 PBFT。
- 运行在部分同步模型下。
- 将 mesh 通信结构转为 star 通信结构。
- PBFT 类两轮交换在加入 view change 后变为三阶段。

HotStuff 阶段：

1. **Prepare**：leader 发送 prepare 消息。
2. **Pre-commit**：prepare 消息形成 prepare quorum certificate。
3. **Commit**：形成 precommitQC。
4. **Decide**：形成 commitQC，发送 decide 消息，进入下一 view。

**Page reference / 页码：** p.62-p.63。

---

## 38. Sync HotStuff Model / Sync HotStuff 模型

### English
Sync HotStuff is a protocol for synchronous state machine replication.

Model assumptions:

- Consensus can be framed as replication, broadcast, or agreement.
- Network connectivity can vary by model.
- Synchrony means there is a known message delay bound `Δ`.
- Faults are Byzantine with `n = 2f + 1`.
- Public key setup is assumed.
- All messages are signed.

### 中文
Sync HotStuff 是同步状态机复制协议。

模型假设：

- 共识可以被表述为复制、广播或协议一致问题。
- 网络连接性取决于模型设定。
- 同步模型意味着存在已知消息延迟上界 `Δ`。
- 拜占庭故障模型中 `n = 2f + 1`。
- 假设存在公钥基础设施。
- 所有消息都带签名。

**Page reference / 页码：** p.64。

---

## 39. Sync HotStuff Basic Flow / Sync HotStuff 基本流程

### English
Basic protocol:

1. Leader proposes value `x` and signs it.
2. Upon receiving the first proposal, each node forwards and votes for it.
3. If nothing bad happens within `2Δ`, nodes commit `x`.
4. `f + 1` signed votes form a certificate.
5. The process repeats view by view.
6. If the leader misbehaves, a view change occurs.

The `2Δ` waiting time allows equivocation evidence to propagate before a node commits.

### 中文
基本协议：

1. Leader 提议值 `x` 并签名。
2. 节点收到第一个 proposal 后，转发并投票。
3. 如果 `2Δ` 内没有坏事发生，则提交 `x`。
4. `f + 1` 个签名投票形成 certificate。
5. 协议逐个 view 重复执行。
6. 如果 leader 作恶，则触发 view change。

等待 `2Δ` 的作用是：在节点提交之前，让 equivocation 证据有足够时间传播。

**Page reference / 页码：** p.65-p.66。

---

## 40. Bad Things: Equivocation and Blame / 坏事：双重提议与 blame

### English
Bad things include:

1. **Leader equivocation**: the leader signs and sends conflicting proposals.
2. **Lack of progress**: leader does not make progress.

Rules:

- Leader equivocation stops a commit.
- Equivocation triggers view change.
- Nodes forward signed equivocating proposals to all.
- Lack of progress triggers timeout/blame.
- `f + 1` blames trigger view change.
- On view change, nodes quit the view, wait `Δ`, then enter a new view.

### 中文
坏事包括：

1. **Leader equivocation**：leader 签名并发送相互冲突的 proposal。
2. **缺乏进展**：leader 不推进协议。

规则：

- Leader equivocation 会阻止提交。
- Equivocation 会触发 view change。
- 节点把签名的冲突 proposal 证据转发给所有人。
- 缺乏进展会触发 timeout/blame。
- `f + 1` 个 blame 会触发 view change。
- view change 时，节点退出当前 view，等待 `Δ`，再进入新 view。

**Page reference / 页码：** p.67-p.71。

---

## 41. The Attack and Commit-Lock / 攻击与 Commit-Lock

### English
The lecture asks: are we done? No.

Attack:

- A malicious leader may make one honest node commit `x` in a view.
- In the next view, another leader proposes `y`.
- Without additional mechanism, safety can break.

Solution: **Commit-Lock**.

Rules:

- If one honest node commits `x`, all honest nodes lock on `x`.
- A lock means the node will not vote for a different value unless convinced.
- Upon entering a new view, nodes lock on a value certified by `f + 1` votes from the latest view.
- A node may change its mind only if another value is certified in an equal or higher view.
- If one honest node commits `x`, `x` remains the unique highest certified value forever.
- Therefore no honest node votes for `x'`, preserving safety.

### 中文
课程问：到这里协议完成了吗？还没有。

攻击：

- 恶意 leader 可以让某一个诚实节点在某个 view 中提交 `x`。
- 下一 view 中，另一个 leader 提议 `y`。
- 如果没有额外机制，安全性可能被破坏。

解决方案：**Commit-Lock**。

规则：

- 如果一个诚实节点提交了 `x`，所有诚实节点都应锁定 `x`。
- lock 表示节点不会投票给不同值，除非被新的证据说服。
- 进入新 view 时，节点根据最新 view 中由 `f + 1` 票认证的值上锁。
- 节点只有在看到同 view 或更高 view 中另一个值被认证时，才会改变锁定。
- 如果一个诚实节点提交 `x`，`x` 将永远保持唯一最高认证值。
- 因此没有诚实节点会投给 `x'`，安全性成立。

**Page reference / 页码：** p.72-p.75。

---

## 42. Status Step and Full Protocol / 状态汇报步骤与完整协议

### English
To complete the protocol, Sync HotStuff adds a status step:

- Propose, forward/vote, commit after `2Δ`.
- Lack of progress leads to blame.
- Equivocation or `f + 1` blames leads to view change.
- Nodes quit view, wait `Δ`, and enter new view.
- Nodes report lock status to the new leader.
- The new leader re-proposes the highest certified value.

This ensures an honest leader can always make progress.

Final properties:

- Tolerates 49% corruption of replicas.
- This improves over the 33% tolerance typical of asynchronous/partial synchronous BFT.
- Low latency: `2Δ`.
- Supports pipelined execution.

### 中文
为了完成协议，Sync HotStuff 加入 status step：

- 提议、转发/投票，经过 `2Δ` 后提交。
- 缺乏进展则发送 blame。
- Equivocation 或 `f + 1` 个 blame 触发 view change。
- 节点退出 view，等待 `Δ`，进入新 view。
- 节点向新 leader 汇报 lock status。
- 新 leader 重新提议最高认证值。

这样可以保证诚实 leader 总能推动进展。

最终性质：

- 可容忍 49% 副本被腐化。
- 相比异步/部分同步 BFT 常见的 33% 容错更强。
- 低延迟：`2Δ`。
- 支持流水线执行。

**Page reference / 页码：** p.76-p.78。

---

## 43. Consensus Algorithm Comparison / 共识算法对比

### English
The p.79 table compares consensus families including PoW, PoS, DPoS, PoET, Ripple, Tendermint, PBFT variants, and Federated BFT.

Important comparison dimensions:

- Blockchain type: open, permissioned, or both.
- Energy saving.
- Tolerated adversarial power.
- Example systems.
- Transaction finality.
- Transaction rate.
- Whether token is needed.
- Cost of participation.
- Network scalability.
- Trust model.

Typical observations:

- PoW is open and permissionless, but energy saving is poor and finality is probabilistic.
- Voting-based BFT protocols can provide immediate finality, but are often permissioned or semi-trusted.
- PoS and DPoS try to improve energy efficiency while retaining some openness.

### 中文
第 79 页表格比较了 PoW、PoS、DPoS、PoET、Ripple、Tendermint、PBFT variants、Federated BFT 等共识。

比较维度包括：

- 区块链类型：开放、许可、或两者。
- 是否节能。
- 可容忍的敌手控制比例。
- 示例系统。
- 交易最终性。
- 交易速率。
- 是否需要 token。
- 参与成本。
- 网络可扩展性。
- 信任模型。

典型观察：

- PoW 开放且无需许可，但能耗高，最终性是概率性的。
- 投票型 BFT 协议可以提供即时最终性，但通常是许可制或半信任模型。
- PoS 和 DPoS 试图在开放性与能源效率之间折中。

---

## 44. Noteworthy Novel Consensus and Network Layer / 值得关注的新型共识与网络层扩展

### English
The lecture lists several directions.

Scaling Nakamoto consensus:

- Bitcoin-NG.
- Conflux.

Proof of Stake and leader-based:

- Casper FFG.
- Algorand.
- HotStuff BFT.
- Streamlet.

Leaderless:

- Avalanche.

Network-layer faster block propagation:

- BloXroute.

### 中文
课程列举了若干方向。

扩展 Nakamoto 共识：

- Bitcoin-NG。
- Conflux。

PoS 与 leader-based 共识：

- Casper FFG。
- Algorand。
- HotStuff BFT。
- Streamlet。

Leaderless 共识：

- Avalanche。

网络层更快区块传播：

- BloXroute。

**Page reference / 页码：** p.80-p.81。

---

# Part IV - Recap of Previous Scalability Topics  
# 第四部分：前面可扩展性主题回顾

---

## 45. Blockchain Trilemma / 区块链三难困境

### English
The final part revisits the blockchain trilemma:

- Scalability.
- Security.
- Decentralization.

Most scalability designs improve one or two dimensions at the cost of another. This lecture’s DA, EVM, and consensus techniques are all attempts to reduce those tradeoffs.

### 中文
最后部分回顾区块链三难困境：

- 可扩展性。
- 安全性。
- 去中心化。

大多数扩展设计都是在其中一两个维度上优化，同时牺牲另一个维度。本讲的数据可用性、EVM 扩展、共识扩展技术，本质上都在尝试缓解这些权衡。

**Page reference / 页码：** p.82-p.85。

---

## 46. Sharding in a Nutshell / 分片概览

### English
The recap slide describes sharding as:

- Beacon chain provides randomness, supports cross-shard transactions, and aggregates cross-links.
- Shard chains run efficient BFT consensus.
- Different sharding protocols differ on:
  - Sybil-resistant validator selection.
  - Random validator shuffling.
  - Cross-link content: block header or entire block.
  - Incoming transaction shard key, often based on address suffix.
- Beacon chain may treat cross-link contents as raw data blobs and Merkleize them without validating the shard block.

### 中文
复习页将分片总结为：

- Beacon chain 提供随机性、支持跨分片交易、聚合 cross-link。
- Shard chain 各自运行高效 BFT 共识。
- 不同分片协议的差异包括：
  - 如何抗 Sybil 地选择验证者。
  - 如何随机打乱验证者。
  - cross-link 包含区块头还是完整区块。
  - 交易 shard key，通常可能根据地址后几位决定。
- Beacon chain 可能把 cross-link 内容当作 raw data blob，只做 Merkle 化而不验证分片区块内容。

**Page reference / 页码：** p.86-p.87。

---

## 47. Blockchain Interoperability as State Sharding / 区块链互操作近似状态分片

### English
The lecture compares state sharding with Cosmos and Polkadot.

| Concept | State Sharding | Cosmos | Polkadot |
|---|---|---|---|
| Coordinator | Beacon chain | Cosmos Hub | Relay Chain |
| Sub-chain | Shard chain | Cosmos Zone | Parachain |
| Consensus | Shards run BFT or PoS variants | Tendermint or bridged consensus | Pluggable consensus |
| Cross-domain transaction | Cross-shard transaction | IBC | Cross-chain transaction |

### 中文
课程把状态分片与 Cosmos、Polkadot 做类比。

| 概念 | 状态分片 | Cosmos | Polkadot |
|---|---|---|---|
| 协调层 | Beacon chain | Cosmos Hub | Relay Chain |
| 子链 | Shard chain | Cosmos Zone | Parachain |
| 共识 | 各分片运行 BFT 或 PoS 变体 | Tendermint 或桥接到 Tendermint | 可插拔共识 |
| 跨域交易 | 跨分片交易 | IBC | 跨链交易 |

**Page reference / 页码：** p.88。

---

## 48. Layer 2 Landscape / Layer 2 方案图谱

### English
The lecture recaps off-chain scalability.

Layer 2 off-chain solutions include:

1. **Channel-based networks**
   - Payment Channel.
   - Spilman Channel.
   - DMC.
   - Lightning Network.
   - State Channel.

2. **Commit-chains**
   - Plasma.
   - Minimal Viable Plasma.
   - Plasma Cash.
   - Rollup.
   - Optimistic Rollup.
   - ZK Rollup.

### 中文
课程回顾 Layer 2 离链扩展方案。

Layer 2 off-chain solutions 包括：

1. **基于通道的网络**
   - Payment Channel。
   - Spilman Channel。
   - DMC。
   - Lightning Network。
   - State Channel。

2. **Commit-chain 类方案**
   - Plasma。
   - Minimal Viable Plasma。
   - Plasma Cash。
   - Rollup。
   - Optimistic Rollup。
   - ZK Rollup。

**Page reference / 页码：** p.89-p.90。

---

## 49. OP Rollup vs ZK Rollup / OP Rollup 与 ZK Rollup 对比

### English
The final diagram compares Optimistic Rollup and ZK Rollup.

Optimistic Rollup path:

- L2 optimistic rollup posts transaction calldata to Ethereum.
- Fraud proof is used during challenge.
- An OVM-like execution environment may be involved.
- Correctness is enforced by the possibility of challenge.

ZK Rollup path:

- L2 zkRollup posts transaction calldata and validity proof.
- ZK proving generates proof of correct transition.
- Correctness is enforced cryptographically at commit time.

Comparison:

| Dimension | OP Rollup | ZK Rollup |
|---|---|---|
| Correctness mechanism | Fraud proof | Validity proof / ZK proof |
| Assumption | At least one honest challenger monitors | Proof system soundness |
| Finality experience | Delayed by challenge window | Faster after proof verification |
| Computation cost | Lower proving cost, more dispute complexity | Higher proving cost, lower dispute need |
| Data availability | Still required | Still required |

### 中文
最后一页图比较 OP Rollup 与 ZK Rollup。

Optimistic Rollup 路径：

- L2 optimistic rollup 向 Ethereum 发布交易 calldata。
- 出现挑战时使用 fraud proof。
- 可能涉及 OVM 类执行环境。
- 正确性由“可挑战性”保障。

ZK Rollup 路径：

- L2 zkRollup 发布交易 calldata 和有效性证明。
- ZK proving 生成状态转换正确性的证明。
- 正确性在提交时由密码学证明保障。

对比：

| 维度 | OP Rollup | ZK Rollup |
|---|---|---|
| 正确性机制 | 欺诈证明 | 有效性证明 / ZK proof |
| 假设 | 至少一个诚实挑战者监控 | 证明系统可靠 |
| 最终性体验 | 受挑战期影响 | 证明验证后更快 |
| 计算成本 | 证明成本低，但争议处理复杂 | 证明成本高，但无需常规争议 |
| 数据可用性 | 仍然需要 | 仍然需要 |

**Page reference / 页码：** p.91。

---

# 50. Bilingual Glossary / 中英术语表

| English | 中文 | Explanation / 解释 |
|---|---|---|
| Data Availability | 数据可用性 | 验证区块或状态转换所需数据能够被节点获得。 |
| Data Availability Sampling | 数据可用性抽样 | 轻节点随机抽样数据片段，以高概率确认数据可用。 |
| Data Availability Proof | 数据可用性证明 | 证明数据已经发布且可访问的密码学机制。 |
| Fraud Proof | 欺诈证明 | 证明某个状态转换或区块无效的证据。 |
| Light Client | 轻客户端 | 只下载区块头和证明，不下载完整区块数据的客户端。 |
| Full Node | 全节点 | 下载并验证完整区块链数据的节点。 |
| Erasure Coding | 纠删码 | 通过冗余编码，让部分数据缺失时仍可恢复。 |
| Reed-Solomon Code | Reed-Solomon 编码 | 基于有限域多项式的经典纠删码。 |
| Merkle Tree | 默克尔树 | 用哈希树承诺大量数据，支持高效包含证明。 |
| Namespaced Merkle Tree | 命名空间 Merkle 树 | 支持按 namespace 选择性证明和检索的 Merkle 树。 |
| Coded Merkle Tree | 编码 Merkle 树 | 对 Merkle tree 的每层都维护数据可用性的结构。 |
| Rollup | Rollup 扩展方案 | 在 L2 执行交易，并把数据/证明提交到 L1 的方案。 |
| Optimistic Rollup | 乐观 Rollup | 默认状态转换正确，错误时通过 fraud proof 挑战。 |
| ZK Rollup | 零知识 Rollup | 通过有效性证明证明状态转换正确。 |
| EVM | 以太坊虚拟机 | 执行以太坊智能合约字节码的虚拟机。 |
| World State | 世界状态 | 以太坊中地址到账户状态的全局映射。 |
| Account State | 账户状态 | 包含 nonce、balance、storageRoot、codeHash 等信息。 |
| Access List | 访问列表 | 交易预声明读写的账户或状态集合。 |
| Block-STM | 软件事务内存并行执行 | 乐观并行执行并动态检测冲突的执行模型。 |
| Consensus | 共识 | 多节点对相同值序列达成一致的协议问题。 |
| BFT | 拜占庭容错 | 在部分节点恶意的情况下仍能保持正确性的容错机制。 |
| HotStuff | HotStuff 共识 | 基于 PBFT 的现代 BFT 协议。 |
| Sync HotStuff | 同步 HotStuff | 同步模型下的 HotStuff 变体，可容忍接近 49% 腐化。 |
| View Change | 视图切换 | leader 异常或无进展时切换到新 leader 的过程。 |
| Certificate | 证书 | 由足够多签名投票组成的证明。 |
| Commit-Lock | 提交锁 | 防止不同诚实节点提交冲突值的安全机制。 |

---

# 51. Exam-Oriented Summary / 考试复习总结

## 51.1 Core Questions / 核心问题

### English
1. Why is data availability separate from state validity?
2. Why do light clients need fraud proofs and DA sampling?
3. Why does naive chunk requesting turn light clients into full nodes?
4. How does erasure coding help DA sampling?
5. What is an incorrect-coding attack?
6. How do 2D-RS and CMT reduce proof size?
7. Why do rollups still need DA even if they have fraud proofs or ZK proofs?
8. Why is standard EVM sequential?
9. How do access lists and Block-STM support parallel execution?
10. What are safety and liveness in consensus?
11. Why does partial synchrony BFT usually tolerate only `f < n/3` Byzantine faults?
12. How does Sync HotStuff use `2Δ`, blame, view change, and commit-lock?

### 中文
1. 为什么数据可用性与状态有效性是两个不同问题？
2. 为什么轻客户端需要欺诈证明和 DA 抽样？
3. 为什么朴素的数据片段请求会让轻客户端退化成全节点？
4. 纠删码如何帮助 DA 抽样？
5. 什么是错误编码攻击？
6. 2D-RS 和 CMT 如何降低证明大小？
7. 为什么 Rollup 即使有 fraud proof 或 ZK proof，仍然需要 DA？
8. 标准 EVM 为什么是顺序执行？
9. access list 和 Block-STM 如何支持并行执行？
10. 共识中的 safety 和 liveness 是什么？
11. 为什么部分同步 BFT 通常只能容忍 `f < n/3` 拜占庭故障？
12. Sync HotStuff 如何使用 `2Δ`、blame、view change 和 commit-lock？

---

## 51.2 One-Page Mental Model / 一页心智模型

### English
Blockchain scalability can be decomposed into three bottlenecks:

1. **Data bottleneck**: Can nodes obtain the data needed to verify blocks?  
   Solution direction: DA sampling, erasure coding, Celestia, EigenDA, blobs.

2. **Execution bottleneck**: Can nodes execute more transactions faster?  
   Solution direction: parallel EVM, access lists, Block-STM, GPU acceleration.

3. **Consensus bottleneck**: Can nodes agree faster with lower communication and strong fault tolerance?  
   Solution direction: BFT, HotStuff, Sync HotStuff, faster propagation.

### 中文
区块链可扩展性可以拆成三个瓶颈：

1. **数据瓶颈**：节点能否获得验证区块所需数据？  
   解决方向：DA 抽样、纠删码、Celestia、EigenDA、blobs。

2. **执行瓶颈**：节点能否更快执行更多交易？  
   解决方向：并行 EVM、访问列表、Block-STM、GPU 加速。

3. **共识瓶颈**：节点能否用更低通信成本、更低延迟、更强容错达成一致？  
   解决方向：BFT、HotStuff、Sync HotStuff、更快区块传播。

---

# 52. Page-by-Page Detailed Outline / 逐页详细提纲

| Page | English outline | 中文提纲 |
|---:|---|---|
| 1 | Title: SC6019 Blockchain Privacy & Scalability Lecture 11. | 标题页：SC6019 区块链隐私与可扩展性第 11 讲。 |
| 2 | Scaling Data - Data Availability. | 数据扩展：数据可用性。 |
| 3 | Fundamental challenges: partial transaction processing, partial storage, efficient consensus; state validity, DA, Byzantine adversary. | 根本挑战：部分处理、部分存储、高效共识；状态有效性、DA、拜占庭敌手。 |
| 4 | NEAR sharding shows DA problem: beacon chain, shard chain, VRF, cross-link as header only. | NEAR 分片展示 DA 问题：信标链、分片链、VRF、cross-link 可能只有 header。 |
| 5 | Defines DA and DAP. | 定义数据可用性与数据可用性证明。 |
| 6 | Full nodes download and verify full chain. | 全节点下载并验证完整链。 |
| 7 | Light clients use headers and Merkle proofs. | 轻客户端使用区块头和 Merkle proof。 |
| 8 | Dishonest colluding validators can fool light clients if only headers are distributed. | 恶意串谋验证者可能只传播 header 误导轻客户端。 |
| 9 | Fraud proofs can give light clients near full-node assurance if data is available. | 若数据可得，欺诈证明可让轻客户端接近全节点安全。 |
| 10 | Naive chunk requests do not scale; need recoverability from fewer chunks. | 朴素请求片段不可扩展；需要少量片段恢复完整数据。 |
| 11 | Erasure coding and RS: any 50% symbols allow recovery. | 纠删码和 RS：任意 50% 符号可恢复。 |
| 12 | RS construction via polynomials and Vandermonde matrix. | RS 基于多项式和 Vandermonde 矩阵构造。 |
| 13 | Merkle tree over erasure-coded block. | 对纠删编码区块构建 Merkle 树。 |
| 14 | Random sampling makes hiding many symbols detectable. | 随机抽样让大规模隐藏容易被发现。 |
| 15 | Coding vs no coding: O(1) samples vs linear samples. | 编码 vs 不编码：O(1) 抽样 vs 线性抽样。 |
| 16 | Incorrect-coding attack; proof may be block-sized. | 错误编码攻击；证明可能和区块一样大。 |
| 17 | Cost summary for 1D-RS. | 1D-RS 成本总结。 |
| 18 | 2D-RS divide-and-conquer reduces proof size. | 2D-RS 分治降低证明大小。 |
| 19 | LDPC idea and ideal target. | LDPC 思路与理想目标。 |
| 20 | Incorrect coding plus hidden hashes attack. | 错误编码加隐藏哈希攻击。 |
| 21 | Coded Merkle Tree maintains DA layer by layer. | 编码 Merkle 树逐层维护 DA。 |
| 22 | Light node sampling automatically samples every layer. | 轻节点抽样自动覆盖每层。 |
| 23 | Honest full node performs layer-by-layer recovery and rejects incorrect coding. | 诚实全节点逐层恢复并拒绝错误编码。 |
| 24 | Performance comparison: 1D-RS, 2D-RS, CMT. | 性能比较：1D-RS、2D-RS、CMT。 |
| 25 | DA for Optimistic Rollup and challenge workflow. | OP Rollup 的 DA 与挑战流程。 |
| 26 | DA for zkRollup and validity proof workflow. | zkRollup 的 DA 与有效性证明流程。 |
| 27 | Celestia as modular DA blockchain. | Celestia 作为模块化 DA 区块链。 |
| 28 | Celestia uses DAS and NMT. | Celestia 使用 DAS 和 NMT。 |
| 29 | Celestia DAS uses 2D-RS and light-node random sampling. | Celestia DAS 使用 2D-RS 和轻节点随机抽样。 |
| 30 | Namespaced Merkle Tree visual. | 命名空间 Merkle 树图示。 |
| 31 | Celestia DA layer as PoS blockchain built on modified Tendermint. | Celestia DA 层是基于修改版 Tendermint 的 PoS 链。 |
| 32 | Lifecycle of a celestia-app transaction. | celestia-app 交易生命周期。 |
| 33 | Checking data availability through DA network and light nodes. | 通过 DA 网络和轻节点检查数据可用性。 |
| 34 | EigenDA architecture from EigenLayer. | EigenLayer 的 EigenDA 架构。 |
| 35 | DA scaling comparison: Ethereum L1, EIP-4844 blobs, Celestia. | DA 扩展比较：Ethereum L1、EIP-4844 blobs、Celestia。 |
| 36 | EVM Scaling section. | EVM 扩展章节。 |
| 37 | Ethereum world state. | 以太坊世界状态。 |
| 38 | State-transition chain from genesis state through blocks. | 从创世状态经区块推进的状态转换链。 |
| 39 | Transactions as atomic signed instructions. | 交易是原子数字签名指令。 |
| 40 | EVM executes contract bytecode and updates account state. | EVM 执行合约字节码并更新账号状态。 |
| 41 | EVM stack machine architecture. | EVM 栈式机器架构。 |
| 42 | EVM opcode categories and execution environment access. | EVM opcode 类型与执行环境访问。 |
| 43 | Solidity/Vyper compile to EVM bytecode. | Solidity/Vyper 编译到 EVM 字节码。 |
| 44 | Sequential EVM workflow. | 顺序 EVM 工作流。 |
| 45 | Sequential EVM limitations. | 顺序 EVM 局限。 |
| 46 | Parallel execution and conflict management. | 并行执行与冲突管理。 |
| 47 | Account-based access lists; Solana example. | 基于账号访问列表；Solana 示例。 |
| 48 | Block-STM optimistic parallelization. | Block-STM 乐观并行化。 |
| 49 | Parallel execution matters for DEX, GameFi, DePIN, micropayments. | 并行执行对 DEX、GameFi、DePIN、微支付重要。 |
| 50 | Reddio architecture with Parallel Geth and CUDA GPUs. | Reddio 架构：Parallel Geth 与 CUDA GPU。 |
| 51 | Reddio approach: GPU, decentralized GPU/ZKP networks, modular sequencer SDK. | Reddio 方法：GPU、去中心化 GPU/ZKP 网络、模块化排序器 SDK。 |
| 52 | Scaling Consensus section. | 共识扩展章节。 |
| 53 | From Nakamoto consensus to non-Nakamoto consensus and Sync HotStuff. | 从 Nakamoto 共识到非 Nakamoto 共识和 Sync HotStuff。 |
| 54 | Consensus as replicated service with safety and liveness. | 共识作为复制服务，关注安全性与活性。 |
| 55 | Consensus model dimensions. | 共识模型维度。 |
| 56 | 40 years of distributed consensus research. | 40 年分布式共识研究。 |
| 57 | Fault tolerance bounds table. | 容错边界表。 |
| 58 | Proof-based vs voting-based consensus. | 基于证明 vs 基于投票共识。 |
| 59 | Byzantine Fault Tolerance. | 拜占庭容错。 |
| 60 | BFT voting rounds. | BFT 投票轮次。 |
| 61 | BFT-based consensus architecture. | BFT 共识架构图。 |
| 62 | HotStuff overview. | HotStuff 概览。 |
| 63 | HotStuff phases. | HotStuff 阶段。 |
| 64 | Sync HotStuff model. | Sync HotStuff 模型。 |
| 65 | Sync HotStuff proposal and 2Δ commit. | Sync HotStuff 提议和 2Δ 提交。 |
| 66 | Certificates and view change. | 证书与 view change。 |
| 67 | Leader equivocation stops commit. | Leader 双重提议阻止提交。 |
| 68 | Why 2Δ matters. | 为什么需要 2Δ。 |
| 69 | Equivocation proof triggers view change. | 双重提议证据触发 view change。 |
| 70 | Timeout/blame and view change rules. | 超时/blame 与 view change 规则。 |
| 71 | Interim protocol and attack question. | 阶段性协议与攻击问题。 |
| 72 | Malicious leader can make one honest node commit. | 恶意 leader 可让一个诚实节点提交。 |
| 73 | Next view may propose a different value. | 下一 view 可能提议不同值。 |
| 74 | Commit-lock introduced. | 引入 commit-lock。 |
| 75 | Lock change rule and safety argument. | 锁改变规则与安全性论证。 |
| 76 | Status step: report locks and re-propose highest certified. | Status step：汇报锁并重提最高认证值。 |
| 77 | Sync HotStuff full protocol. | Sync HotStuff 完整协议。 |
| 78 | Sync HotStuff properties: 49% corruption, 2Δ latency, pipelining. | Sync HotStuff 性质：49% 腐化容忍、2Δ 延迟、流水线。 |
| 79 | Consensus algorithm comparison table. | 共识算法对比表。 |
| 80 | Novel consensus list: Bitcoin-NG, Conflux, Casper, Algorand, HotStuff, Streamlet, Avalanche. | 新型共识列表。 |
| 81 | Network layer: faster block propagation via BloXroute. | 网络层：BloXroute 加速区块传播。 |
| 82 | Introduction to scalability issues recap. | 可扩展性问题回顾。 |
| 83 | Fundamental challenges repeated. | 根本挑战重复。 |
| 84 | Blockchain trilemma recap. | 区块链三难困境回顾。 |
| 85 | Scalability trilemma diagram. | 可扩展性三难图示。 |
| 86 | Layer 1 scaling: sharding and interoperability. | Layer 1 扩展：分片和互操作。 |
| 87 | Sharding in a nutshell. | 分片概览。 |
| 88 | Interoperability as state sharding. | 互操作近似状态分片。 |
| 89 | Layer 2 off-chain scalability recap. | Layer 2 离链扩展回顾。 |
| 90 | Promising Layer 2 solutions taxonomy. | Layer 2 方案分类。 |
| 91 | OP Rollup vs ZK Rollup diagram. | OP Rollup 与 ZK Rollup 对比图。 |

---

# 53. Final High-Density Cheat Sheet / 最终高密度速记

### English
- DA is about whether verifying data is accessible; validity is about whether computation is correct.
- Fraud proofs need DA; without data, fraud cannot be proven.
- Erasure coding turns data hiding into a high-probability detectable event.
- RS coding uses finite-field polynomial reconstruction.
- 1D-RS has compact roots but large incorrect-coding proofs.
- 2D-RS reduces proof size by row/column decomposition.
- CMT maintains DA at every Merkle layer and gives small incorrect-coding proofs.
- Rollups post data to L1 or DA layer so users can reconstruct state.
- EVM is sequential because transactions mutate shared global state.
- Parallel EVM needs conflict detection, scheduling, and deterministic state merging.
- Access lists are pessimistic/static; Block-STM is optimistic/dynamic.
- Consensus safety means no two honest nodes commit conflicting sequences; liveness means progress.
- BFT partial synchrony usually needs `n > 3f`; Sync HotStuff under synchrony uses `n = 2f + 1`.
- Sync HotStuff uses signed proposals, forwarding/voting, `2Δ` waiting, blame, view change, commit-lock, and status reports.

### 中文
- DA 关心验证数据能否获得；validity 关心计算是否正确。
- 欺诈证明依赖 DA；没有数据，就无法证明欺诈。
- 纠删码让隐藏数据变成高概率可检测事件。
- RS 编码使用有限域多项式重构。
- 1D-RS root 少，但错误编码证明大。
- 2D-RS 通过行/列分解降低证明大小。
- CMT 在 Merkle 每一层维护 DA，并提供很小的错误编码证明。
- Rollup 把数据发布到 L1 或 DA 层，使用户能重建状态。
- EVM 顺序执行，是因为交易会修改共享全局状态。
- 并行 EVM 需要冲突检测、调度和确定性状态合并。
- Access list 是偏静态/悲观的；Block-STM 是乐观/动态的。
- 共识 safety 表示诚实节点不会提交冲突序列；liveness 表示系统持续前进。
- 部分同步 BFT 通常需要 `n > 3f`；同步模型下 Sync HotStuff 使用 `n = 2f + 1`。
- Sync HotStuff 使用签名提议、转发/投票、`2Δ` 等待、blame、view change、commit-lock 和 status report。

