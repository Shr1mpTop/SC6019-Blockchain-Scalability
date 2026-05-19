# SC6019 Blockchain — Lecture 09  
# Privacy & Scalability：Layer 2 Solutions — Off-Chain Scalability  
# 中英对照超详细课件笔记 / Detailed Bilingual Lecture Notes

> Source / 来源：`SC6019 - Lecture 09.pdf`  
> Course / 课程：SC6019 – Blockchain Privacy & Scalability  
> Lecture / 讲次：Lecture 09  
> Instructor / 授课教师：Neil HAN  
> Theme / 主题：Layer 2 Solutions: Off-Chain Scalability，Scaling Computation – Layer 2

---

## 0. Lecture Map / 本讲结构总览

### English
This lecture studies **Layer 2 scaling**, especially **off-chain scalability**. The central idea is to move most transactions and state transitions away from Layer 1, while keeping Layer 1 as a final court for deposits, withdrawals, fraud proofs, disputes, and settlement.

The lecture covers:

1. The fundamental scalability problem of blockchains.
2. Why Layer 2 is attractive.
3. Channel-based networks:
   - Generic channel workflow
   - Payment channels
   - Spilman Channel
   - Duplex Micropayment Channel
   - Lightning Network
   - HTLC routing
   - State channels
   - Virtual channels
   - Sprite channels
   - Generalized state channels
4. Summary of channel-based networks.
5. Commit-chains:
   - Generic commit-chain workflow
   - Difference between commit-chain and sharding
6. Plasma:
   - Minimal Viable Plasma
   - Plasma MVP Q&A
   - Confirmed message problem
   - Plasma Cash
7. Historical motivation toward Rollups.

### 中文
本讲讨论的是**区块链 Layer 2 扩容方案**，尤其是**链下扩容 / Off-chain Scalability**。核心思想是：不要让所有交易都直接在 Layer 1 上执行，而是把大量交易和状态更新放到链下，只在必要时把结果、承诺、争议证据或提款请求提交到 Layer 1。

本讲覆盖：

1. 区块链可扩展性的根本问题。
2. 为什么 Layer 2 很有吸引力。
3. 基于通道的网络：
   - 通用通道工作流
   - 支付通道
   - Spilman 单向支付通道
   - Duplex Micropayment Channel
   - Lightning Network 闪电网络
   - HTLC 多跳路由
   - 状态通道
   - 虚拟通道
   - Sprite Channel
   - 广义状态通道
4. 通道型网络总结。
5. Commit-chain：
   - 通用 Commit-chain 工作流
   - Commit-chain 与 Sharding 的区别
6. Plasma：
   - Minimal Viable Plasma
   - Plasma MVP 问答
   - Confirmed Message 问题
   - Plasma Cash
7. Plasma 对 Rollup 的历史启发。

---

## 1. Layer 2 Solutions: Off-Chain Scalability  
## 1. Layer 2 方案：链下可扩展性

### English
The title slide identifies the lecture as **Layer 2 Solutions: Off-Chain Scalability**. The goal is to scale computation by moving most activity away from the base chain while preserving blockchain security properties as much as possible.

### 中文
本讲主题是 **Layer 2 Solutions: Off-Chain Scalability**，即通过链下方式扩展区块链计算能力。它不是简单地提高 Layer 1 的吞吐量，而是把大量计算、交易、交互迁移到链下，只在必要时回到主链结算。

---

## 2. Fundamental Problem in Blockchain Scalability  
## 2. 区块链可扩展性的根本问题

### Slide Core / 幻灯片核心内容
> “every node process all TXs, store all data, plus total ordering of TXs is required”

### English Explanation
The fundamental scalability bottleneck of a traditional blockchain is that **every full node repeats the same work**:

1. Every node processes all transactions.
2. Every node stores all data.
3. The system requires a total ordering of transactions.
4. Consensus is needed to agree on this ordering.

This leads to enormous replicated computation and replicated storage. Even if a transaction concerns only two users, the whole network must usually validate and store it.

Layer 2 asks the key question:

> What if we push transactions and state updates off-chain and only broadcast computation results on-chain?

This means Layer 1 becomes more like a **supreme court**:
- It does not handle every local dispute or every small transaction.
- It adjudicates only when there is a dispute.
- It provides final settlement, data commitments, deposits, exits, and fraud resolution.

Layer 2 protocols become the **local courts**:
- They process frequent interactions cheaply.
- They only escalate to Layer 1 when needed.

### 中文解释
传统区块链的扩容瓶颈在于：

1. 每个全节点都处理所有交易。
2. 每个全节点都存储所有数据。
3. 所有交易都需要形成全局总排序。
4. 网络需要通过共识对这个排序达成一致。

这造成了大量的**重复计算**和**重复存储**。哪怕一笔交易只涉及 Alice 和 Bob，整个网络也要验证和存储。

Layer 2 的关键问题是：

> 如果我们把交易和状态更新放到链下，只把计算结果或争议结果广播到链上，会怎样？

这意味着 Layer 1 更像一个**最高法院**：
- 不处理每一个小额交易或局部交互。
- 只在争议、提款、存款、欺诈证明等关键时刻介入。
- 提供最终结算与安全兜底。

Layer 2 则像**地方法院**：
- 负责高频、低成本的日常交互。
- 只有出现纠纷时才上诉到 Layer 1。

### Layer 2 Categories / Layer 2 分类
The lecture highlights two Layer 2 approaches:

| English | 中文 |
|---|---|
| Channel-based approach | 通道型方案 |
| Commit-chain approach | Commit-chain / 承诺链方案 |

It also contrasts with Layer 1 scaling:

| Layer 1 Scaling | 中文 |
|---|---|
| Sharding | 分片 |
| Blockchain Interoperability | 区块链互操作性 |
| Faster client implementation | 更快的客户端实现 |
| Compact transaction/state encoding | 紧凑交易/状态编码 |
| Parallelization | 并行化 |

---

## 3. Why Layer 2 Is Exciting  
## 3. 为什么 Layer 2 很有吸引力

### Slide Core / 幻灯片核心内容
The slide shows a simple Layer 2 pattern:

1. **1 transaction to join/start the off-chain protocol**
2. **10k+ transactions authorized/processed off-chain**
3. **1 transaction to leave/close the off-chain protocol**

### English Explanation
Layer 2 is powerful because it **amortizes Layer 1 cost**. Instead of putting every transaction on-chain, a user may only need:

- one on-chain transaction to enter,
- many thousands of off-chain updates,
- one on-chain transaction to exit.

This reduces:
- replicated computation,
- Layer 1 gas cost,
- Layer 1 latency,
- congestion on the base chain.

At the same time, a good Layer 2 protocol tries to retain **non-custodial security**. This means users do not simply hand funds to a centralized exchange. Instead, they should still be able to recover or withdraw funds through Layer 1 if the Layer 2 operator or counterparty misbehaves.

### 中文解释
Layer 2 的吸引力在于它可以**摊薄 Layer 1 成本**。用户不再需要每笔交易都上链，而是：

- 用 1 笔链上交易进入协议；
- 在链下完成 10,000+ 笔授权或状态更新；
- 用 1 笔链上交易退出或关闭协议。

这样可以显著减少：

- Layer 1 重复计算；
- Layer 1 gas 成本；
- Layer 1 确认延迟；
- 主链拥堵。

但 Layer 2 不应该变成简单的中心化托管系统。优秀的 Layer 2 要尽量保留**非托管安全性 / non-custodial security**：即使对方或运营者作恶，用户仍然可以通过 Layer 1 取回资金或发起争议。

### Important Open Questions / 关键问题
The slide also emphasizes several unresolved dimensions:

| English Question | 中文问题 |
|---|---|
| What are the safety and liveness properties? | 安全性和活性如何保证？ |
| How is state validity guaranteed? | 状态有效性如何保证？ |
| How is data availability guaranteed? | 数据可用性如何保证？ |

These three questions are the backbone of Layer 2 analysis.

---

## 4. Promising Layer 2 Solutions  
## 4. 主要 Layer 2 方案图谱

### English
The lecture classifies off-chain solutions into two major families:

1. **Channel-based networks**
   - Payment Channel
   - Spilman Channel
   - Duplex Micropayment Channel
   - Lightning Network
   - State Channel

2. **Commit-chains**
   - Plasma
   - Minimal Viable Plasma
   - Plasma Cash
   - Rollup
   - Optimistic Rollup
   - ZK Rollup

### 中文
课件将链下扩容方案分为两大类：

1. **通道型网络 / Channel-based Networks**
   - 支付通道
   - Spilman Channel
   - Duplex Micropayment Channel
   - Lightning Network
   - 状态通道

2. **Commit-chain / 承诺链**
   - Plasma
   - Minimal Viable Plasma
   - Plasma Cash
   - Rollup
   - Optimistic Rollup
   - ZK Rollup

### Conceptual Contrast / 概念对比

| Dimension | Channel-based | Commit-chain |
|---|---|---|
| Main idea | Participants update state off-chain by mutual signatures | A separate chain periodically commits to Layer 1 |
| 中文 | 参与者通过链下签名更新状态 | 独立链定期向 Layer 1 提交承诺 |
| Best for | Repeated interactions among fixed participants | More open transaction systems |
| Dispute style | Latest signed state, challenge-response | Fraud proof, exit game, validity/data commitment |
| Data responsibility | Users usually store their own relevant states | Depends: Plasma off-chain data; Rollup on-chain data |

---

# Part I — Channel-Based Networks  
# 第一部分：通道型网络

---

## 5. Generic Channel Workflow  
## 5. 通用通道工作流

### Slide Core / 幻灯片核心内容
The generic channel workflow contains:

1. Setup Channel
2. 1000+ off-chain transactions
3. Trigger Dispute
4. Dispute Period
5. Resolve Dispute

### English Explanation
A channel is opened by a **fixed group of participants**. The opening step initializes the channel state, such as deposit balances.

After setup, participants repeatedly perform off-chain updates. Each update is valid only if all required parties agree and sign the new state.

If everyone cooperates:
- no intermediate state goes on-chain;
- the channel can process many updates cheaply;
- the final agreed state is eventually settled.

If someone becomes unresponsive or malicious:
- an honest party can trigger an on-chain dispute;
- evidence is submitted during a dispute period;
- the contract resolves to the latest valid state or forces progression on-chain.

### 中文解释
通道通常由一个**固定参与者集合**打开。打开通道时，参与者初始化状态，例如双方各自存入多少资金。

之后，参与者在链下反复更新状态。每次状态更新都需要相关参与方签名确认。

如果大家都合作：
- 中间状态不需要上链；
- 大量更新可以低成本完成；
- 最终状态再上链结算即可。

如果有人不响应或作恶：
- 诚实方可以在链上触发争议；
- 在争议期内提交证据；
- 合约根据最新有效状态解决争议，或者强制把后续步骤搬到链上执行。

### Key Terms / 核心术语

| English | 中文 | Explanation |
|---|---|---|
| Setup Channel | 建立通道 | Deposit and initialize state |
| Off-chain progression | 链下推进 | Parties sign new state repeatedly |
| On-chain dispute | 链上争议 | Escalation to Layer 1 |
| Challenge-response | 挑战-响应 | One party claims, others challenge with evidence |
| Dispute period | 争议期 | Time window for submitting evidence |
| Latest signed state | 最新签名状态 | The most recent mutually authorized state |

---

## 6. Payment Channels  
## 6. 支付通道

### English
Payment channels are mostly associated with Bitcoin-style payment systems. The lecture introduces several historical designs:

1. **Spilman Channel**
   - One-way / unidirectional payment
   - Uses incentives to ensure the receiver broadcasts the best state

2. **Duplex Micropayment Channel**
   - Bi-directional payment
   - Uses time locks
   - Has expiry time

3. **Lightning Network**
   - Bi-directional
   - No fixed expiry for the channel
   - Uses revocation to replace old states
   - Uses HTLC to connect channels into a network

### 中文
支付通道主要与 Bitcoin 风格的支付系统相关。课件介绍了几个历史方案：

1. **Spilman Channel**
   - 单向支付
   - 通过激励保证收款方广播对自己最有利的最新状态

2. **Duplex Micropayment Channel / DMC**
   - 双向支付
   - 使用时间锁
   - 通道存在过期时间

3. **Lightning Network / 闪电网络**
   - 双向支付
   - 通道可以长期保持开放
   - 使用撤销机制替换旧状态
   - 使用 HTLC 将多个通道连接成支付网络

---

## 7. Spilman Payment Channel  
## 7. Spilman 单向支付通道

### 7.1 Setup / 建立通道

### English
In a Spilman payment channel, Alice deposits 1 BTC into a script. The deposit can be redeemed under one of two conditions:

1. A spending transaction authorized by both Alice and Bob.
2. After a timeout `t`, Alice receives a refund.

The output can be represented as:

- Alice gets `X BTC`
- Bob gets `1 - X BTC`

with signatures from Alice and Bob.

### 中文
在 Spilman 支付通道中，Alice 将 1 BTC 存入一个脚本。这个存款可以通过两种条件之一赎回：

1. Alice 和 Bob 共同授权的一笔花费交易；
2. 超过超时时间 `t` 后，Alice 自动退款。

状态可表示为：

- Alice 得到 `X BTC`
- Bob 得到 `1 - X BTC`

该交易需要 Alice 和 Bob 的签名。

---

### 7.2 Off-chain Updates / 链下更新

### English
Alice can repeatedly send Bob signed transactions with increasing payments to Bob:

- State 1: Alice 0.8 BTC, Bob 0.2 BTC
- State 2: Alice 0.7 BTC, Bob 0.3 BTC
- State 3: Alice 0.4 BTC, Bob 0.6 BTC

Bob has an incentive to broadcast the latest state because it pays him the most.

### 中文
Alice 可以不断给 Bob 发送新的签名交易，Bob 获得的金额逐渐增加：

- 状态 1：Alice 0.8 BTC，Bob 0.2 BTC
- 状态 2：Alice 0.7 BTC，Bob 0.3 BTC
- 状态 3：Alice 0.4 BTC，Bob 0.6 BTC

Bob 有动力广播最新状态，因为最新状态给他最多钱。

---

### 7.3 Security Logic / 安全逻辑

### English
Spilman works because it is **unidirectional**. Alice only pays Bob, and Bob's balance monotonically increases. Therefore, Bob never benefits from broadcasting an older state.

Questions raised by the lecture:

1. Can Alice broadcast a historical balance?
2. What if Bob is unresponsive?

Answers:

1. Alice cannot steal from Bob by broadcasting an old state because Bob is the one who closes with the best signed transaction.
2. If Bob is unresponsive, Alice can eventually recover funds after timeout `t`.

### 中文
Spilman 的安全性依赖于它是**单向支付**。Alice 只向 Bob 支付，Bob 的余额单调增加。因此 Bob 广播旧状态没有好处。

课件提出两个问题：

1. Alice 能不能广播历史余额？
2. Bob 不响应怎么办？

答案是：

1. Alice 不能通过旧状态偷 Bob 的钱，因为最终由 Bob 广播对自己最有利的签名状态。
2. 如果 Bob 不响应，Alice 可以等到超时时间 `t` 之后拿回退款。

### Key Limitation / 核心限制
Spilman is simple but only supports one-way payment. It cannot efficiently support arbitrary bidirectional payment.

Spilman 很简单，但只支持单向支付，不能很好支持双向支付。

---

## 8. Duplex Micropayment Channel / DMC  
## 8. 双向微支付通道

### English
The lecture asks: what if we want bidirectional payment?

DMC's core idea is:

1. Use a pair of unidirectional channels.
2. Use a validation tree.
3. Replace old states by time locks.
4. Newer payments have decreasing expiry time, ensuring the latest state is redeemed first.

The lecture notes that DMC is mostly of historical significance and that in practice:

> Lightning Network >> DMC

### 中文
课件提出：如果我们想要双向支付怎么办？

DMC 的核心思想是：

1. 使用一对单向通道；
2. 使用验证树；
3. 用时间锁替换旧状态；
4. 新状态设置更短的过期时间，确保最新状态先被赎回。

课件指出 DMC 的细节较复杂，主要具有历史意义，实际应用中：

> Lightning Network 远比 DMC 更重要。

---

## 9. Replace by Time Locks  
## 9. 通过时间锁替换状态

### English
The time-lock replacement idea is:

- Older states have longer time locks.
- Newer states have shorter time locks.
- Because newer states expire earlier, they can be redeemed first.
- This prevents older states from overriding newer states.

Example:

| State | Timelock | Meaning |
|---|---|---|
| Initial refund | After 20 blocks | Alice can refund after 20 blocks |
| Later state | After 19 blocks | Newer state can be redeemed earlier |
| Even later state | After 18 blocks | Even newer state wins first |

For same-direction payments, decreasing time locks are not necessary because incentives already make the receiver choose the best state.

### 中文
时间锁替换的思路是：

- 旧状态有更长时间锁；
- 新状态有更短时间锁；
- 新状态更早到期，因此可以先被赎回；
- 这样旧状态不能覆盖新状态。

例如：

| 状态 | 时间锁 | 含义 |
|---|---|---|
| 初始退款 | 20 个区块后 | Alice 20 区块后可退款 |
| 较新状态 | 19 个区块后 | 新状态更早可执行 |
| 更新状态 | 18 个区块后 | 更新状态先执行 |

对于同一方向的付款，不需要递减时间锁，因为激励已经足够：收款方自然会选择让自己收到最多钱的状态。

---

## 10. Lightning Network  
## 10. 闪电网络

### 10.1 High-level Properties / 高层特性

### English
The Lightning Network, proposed by Poon and Dryja, offers:

1. **Instant finalization**
   - No need to wait for block confirmations for each payment.

2. **Low cost**
   - If parties cooperate, only two on-chain transactions are needed:
     - open channel
     - close channel

3. **Scalability**
   - One on-chain update can aggregate thousands of off-chain updates.

4. **Some privacy**
   - Intermediate transactions are not visible to the whole world.

5. **State replacement by revocation**
   - Avoids restrictive expiry time from DMC.
   - Enables bidirectional payment.

6. **HTLC**
   - Hash Time Lock Contracts connect channels into a payment network.

### 中文
闪电网络由 Poon 和 Dryja 提出，具有以下特点：

1. **即时最终性**
   - 每次支付不需要等待区块确认。

2. **低成本**
   - 如果双方合作，只需要两笔链上交易：
     - 打开通道；
     - 关闭通道。

3. **高可扩展性**
   - 一笔链上状态可以聚合数千笔链下更新。

4. **一定程度的隐私**
   - 中间交易不会暴露给全网。

5. **通过撤销机制替换状态**
   - 避免 DMC 中过于僵硬的过期时间；
   - 支持双向支付。

6. **HTLC**
   - 哈希时间锁合约将多个通道连接成支付网络。

---

## 11. Lightning Payment: Core Idea  
## 11. 闪电网络支付核心思想

### English
For every state, each party signs a transaction that **only the counterparty can broadcast**. When closing the channel, one of these transactions can be broadcast.

A payment update requires two steps:

1. Both parties authorize the new state.
2. Both parties revoke the old state.

If Alice broadcasts a revoked historical state, Bob can present revocation evidence and take Alice's deposit as punishment.

### 中文
对于每一个状态，双方都会签署一个**只能由对方广播**的交易。关闭通道时，某一方可以广播自己持有的有效交易。

一次支付更新需要两步：

1. 双方授权新状态；
2. 双方撤销旧状态。

如果 Alice 广播已撤销的历史状态，Bob 可以提交撤销证据，罚没 Alice 的押金。

### Why Revocation Matters / 为什么撤销机制重要
In a bidirectional channel, either party might benefit from an old state. Therefore, incentives alone are insufficient. Revocation creates a punishment mechanism:

- honest closing: get the latest balance;
- dishonest old-state closing: lose all channel funds.

在双向通道中，某一方可能从旧状态中获利。因此仅靠激励不够。撤销机制提供惩罚：

- 诚实关闭：按最新余额结算；
- 恶意用旧状态关闭：失去全部通道资金。

---

## 12. Lightning Payment Under the Hood  
## 12. 闪电网络底层机制

### English
The lecture uses an example with Alice and Bob depositing:

- Alice: 3 BTC
- Bob: 7 BTC

The channel creates scripts where funds can be claimed in two ways:

1. The intended party can claim after dispute period `t`.
2. The counterparty can claim immediately if they reveal a revocation secret.

A revocation is implemented by revealing the preimage of a hash digest:

- `H(s1) = h1`
- revealing `s1` lets the counterparty punish misuse of the revoked state.

### 中文
课件使用了一个例子：

- Alice 存入 3 BTC；
- Bob 存入 7 BTC。

通道创建脚本，使资金可以通过两种方式领取：

1. 正常方在争议期 `t` 后领取；
2. 如果对方能提供撤销秘密，可以立即惩罚性领取。

撤销机制通过哈希原像实现：

- `H(s1) = h1`
- 一旦透露 `s1`，对方就可以在你使用旧状态时惩罚你。

---

### State Validity / 状态有效性
**English:** State validity is ensured through unanimous consent, multi-signatures, and revocation. The latest state is the only safe state to broadcast.

**中文：** 状态有效性通过全员同意、多签名和撤销机制保证。只有最新状态才是安全可广播的状态。

### Data Availability / 数据可用性
**English:** Data availability is entirely the users' responsibility. Users must store all historical revocation secrets and the latest state.

**中文：** 数据可用性完全由用户自己负责。用户必须保存所有历史撤销秘密和最新状态。

### Safety / 安全性
**English:** The counterparty cannot steal funds if the user keeps the required evidence and reacts during the dispute window.

**中文：** 只要用户保存证据并在争议期内及时响应，对方不能偷钱。

### Liveness / 活性
**English:** A user can force the counterparty to respond or close the channel by triggering an on-chain dispute with the latest state.

**中文：** 用户可以通过提交最新状态触发链上争议，迫使对方响应或关闭通道。

---

## 13. Pause & Ponder: Failure Cases in Lightning  
## 13. 思考：闪电网络中的失败场景

### Case 1: What if malicious Bob refuses to send State 2A?  
### 情况 1：恶意 Bob 拒绝发送 State 2A 怎么办？

### English
If Bob refuses to send State 2A, Alice should not send the revocation secret for the old state. Alice should also stop signing further updates.

If Bob closes with State 2B, Alice is fine because she agreed to it.  
If Bob closes with State 1B, Alice is also fine because she never received State 2A and therefore never accepted State 2.

### 中文
如果 Bob 拒绝发送 State 2A，Alice 不应该发送旧状态的撤销秘密，也不应该继续签署后续更新。

如果 Bob 用 State 2B 关闭通道，Alice 可以接受，因为她同意过。  
如果 Bob 用 State 1B 关闭通道，Alice 也可以接受，因为她没有收到 State 2A，因此并未真正接受 State 2。

---

### Case 2: What if malicious Alice refuses to revoke State 1A?  
### 情况 2：恶意 Alice 拒绝撤销 State 1A 怎么办？

### English
Bob should not accept State 2 until he receives the revocation for State 1A.

### 中文
Bob 在收到 State 1A 的撤销秘密之前，不应该接受 State 2。

### Design Lesson / 设计启示
State update in payment channels must be **atomic enough**: parties should not consider a new state finalized until both authorization and revocation steps are complete.

支付通道中的状态更新必须具有足够的“原子性”：只有当新状态授权和旧状态撤销都完成后，新状态才算真正生效。

---

## 14. Routing Through Existing Channels  
## 14. 通过已有通道路由付款

### English
Creating a dedicated channel with every possible counterparty is expensive and capital intensive. The lecture asks:

> Can we route payment through existing channels without creating a new channel on-chain?

This motivates multi-hop payments using HTLC.

### 中文
如果每次付款都要和对方新建一个通道，成本很高，而且需要大量资本锁定。课件提出：

> 能不能通过已有通道路由付款，而不是每对用户都新建链上通道？

这引出了基于 HTLC 的多跳支付。

---

## 15. Conditional Transfer  
## 15. 条件转账

### English
A conditional transfer can be expressed as:

1. Bob randomly chooses `r`.
2. Bob computes `h = H(r)`.
3. Alice promises:
   > Bob, if you can reveal the preimage of `h` before time `t`, then you get 1 coin.

If Alice refuses to pay after Bob reveals `r`, Bob can broadcast:
- Alice's conditional transfer statement,
- the preimage `r`,
to force execution on-chain.

### 中文
条件转账可以表示为：

1. Bob 随机选择 `r`；
2. Bob 计算 `h = H(r)`；
3. Alice 承诺：
   > Bob，如果你能在时间 `t` 前揭示 `h` 的原像，那么你可以获得 1 coin。

如果 Bob 揭示 `r` 后 Alice 拒绝付款，Bob 可以把：

- Alice 的条件转账声明；
- 原像 `r`；

广播到链上，强制执行。

### Key Idea / 核心思想
A hash preimage acts like a cryptographic receipt. Whoever knows `r` can satisfy the condition.

哈希原像就像一个密码学收据。谁知道 `r`，谁就能满足付款条件。

---

## 16. Hash Time Lock Contract / HTLC  
## 16. 哈希时间锁合约

### 16.1 Setup / 设置流程

### English
Suppose Alice wants to pay Dave 1 BTC through intermediaries Bob and Charlie.

The setup goes from receiver to sender:

1. Dave chooses secret `r`.
2. Dave computes `h = H(r)`.
3. Conditional transfers are set up along the path.
4. Each hop commits to pay if the preimage `r` is revealed before a deadline.

The lecture emphasizes a **decremental timelock from sender to receiver**:

- The upstream party must have more time to claim than the downstream party.
- This creates a safety buffer for each intermediary.

### 中文
假设 Alice 想通过 Bob 和 Charlie 向 Dave 支付 1 BTC。

设置过程从收款方 Dave 向付款方 Alice 反向构造：

1. Dave 选择秘密 `r`；
2. Dave 计算 `h = H(r)`；
3. 沿路径设置条件转账；
4. 每一跳都承诺：如果对方在截止时间前揭示 `r`，则付款。

课件强调了**从发送方到接收方递减的时间锁**：

- 上游节点必须比下游节点有更长时间去索赔；
- 这样中间节点才有缓冲时间保证自己不会亏钱。

---

### 16.2 Cooperative HTLC Transfer / 合作式 HTLC 转账

### English
The HTLC transfer happens hop by hop:

1. Dave reveals `r` to Charlie and receives payment.
2. Charlie uses `r` to claim from Bob.
3. Bob uses `r` to claim from Alice.

The same preimage `r` unlocks payment across the entire route.

### 中文
HTLC 转账逐跳发生：

1. Dave 向 Charlie 揭示 `r` 并收款；
2. Charlie 用 `r` 向 Bob 索赔；
3. Bob 用 `r` 向 Alice 索赔。

同一个原像 `r` 解锁整条路径上的付款。

---

### 16.3 Dispute Case / 争议情况

### English
Question:

> What if Bob refuses to pay Charlie after Charlie reveals the preimage?

Charlie can go on-chain and submit:
- the conditional transfer,
- the preimage `r`,
- signature/evidence.

Then Bob is forced to pay Charlie. Once `r` is on-chain, everyone can see it. Bob then has an incentive to claim from Alice.

### 中文
问题：

> 如果 Charlie 向 Bob 揭示了原像，但 Bob 拒绝付款怎么办？

Charlie 可以上链提交：

- 条件转账；
- 原像 `r`；
- 签名和证据。

这样 Bob 会被强制付款给 Charlie。一旦 `r` 被发布到链上，所有人都能看到。于是 Bob 也有动力用 `r` 向 Alice 索赔。

---

### 16.4 Why Timelock Ordering Matters / 为什么时间锁顺序重要

### English
If the timelocks are wrong, intermediaries can lose money.

The lecture gives an important warning:

- If `t1 > t2`, then Dave could reveal `r` between `t1` and `t2`, causing Charlie to lose money.
- If `t2 < t1` but only by a very small margin, Charlie may still lose money by not reacting quickly enough.

Therefore, HTLC routing requires carefully spaced timelocks.

### 中文
如果时间锁设置错误，中间人可能亏钱。

课件提醒：

- 如果 `t1 > t2`，Dave 可能在 `(t1, t2)` 之间揭示 `r`，导致 Charlie 来不及索赔而亏钱；
- 如果 `t2 < t1` 但间隔太小，Charlie 也可能因为反应不够快而亏钱。

因此 HTLC 路由需要合理设置时间锁间隔。

---

## 17. Lightning Network Analysis  
## 17. 闪电网络分析

### Good / 优点

| English | 中文 |
|---|---|
| No need to set up a new channel for each pair-wise transfer | 不需要每对用户都新建通道 |
| Routing marketplace: anyone can provide collateral and join | 形成路由市场，任何人都可提供抵押加入 |
| Optimistically lightning fast | 在合作情况下速度极快 |

### Bad / 缺点

| English | 中文 |
|---|---|
| Free option to create and hold conditional transfers, wasting liquidity | 可以免费创建并持有条件转账，浪费流动性 |
| Heavy collateral for multi-hop transfer | 多跳转账需要大量抵押 |
| 4 hops means 4x collateral | 4 跳意味着约 4 倍抵押锁定 |
| Long worst-case resolution time | 最坏情况下争议解决时间长 |

### 中文补充
闪电网络在合作情况下非常高效，但它的资本效率是一个核心问题。每一跳都需要有足够余额来转发付款，因此多跳支付会锁住大量流动性。此外，如果某一跳进入争议，整个路径可能面临较长等待时间。

---

## 18. State Channel  
## 18. 状态通道

### English
A state channel generalizes payment channels. Instead of only supporting balances, it supports arbitrary application states.

Ethereum's expressive contract language makes this possible.

Core idea:

1. Arbitrary state can be represented inside a channel.
2. Each new state has an incremental counter.
3. Every new state must be signed by all participants.
4. The counter prevents replay attacks.
5. Storage is `O(1)` because users do not need to keep all historical states, only the latest valid state and evidence.

### 中文
状态通道是支付通道的泛化。它不只支持余额更新，还支持任意应用状态。

Ethereum 的智能合约语言使这种设计成为可能。

核心思想：

1. 通道中可以表示任意状态；
2. 每个新状态都有递增计数器；
3. 每个状态都需要所有参与者签名；
4. 计数器防止重放攻击；
5. 存储复杂度为 `O(1)`，因为用户不需要保存所有历史状态，只需要保存最新状态和必要证据。

---

## 19. Application-Specific State Channel  
## 19. 应用专用状态通道

### 19.1 Chess Game Example / 国际象棋例子

### English
The lecture uses a chess game:

- Alice deposits 5 ETH.
- Bob deposits 5 ETH.
- Winner takes 10 ETH.
- More than 100 moves happen off-chain.
- Each new state includes:
  - game state `S_i`,
  - counter `i`,
  - signatures.

### 中文
课件用国际象棋作为例子：

- Alice 存入 5 ETH；
- Bob 存入 5 ETH；
- 胜者获得 10 ETH；
- 100 多步棋都在链下完成；
- 每个新状态包含：
  - 游戏状态 `S_i`；
  - 计数器 `i`；
  - 双方签名。

---

### 19.2 Old State Attack / 旧状态攻击

### English
Suppose Alice regrets state `S_i` and tries to close the game with an older state `S_{i-5}`.

Bob can provide evidence:

- `S_i` is signed by both parties.
- `i > i - 5`.

Therefore, Alice is cheating and may forfeit her deposit.

### 中文
假设 Alice 后悔 `S_i`，试图用旧状态 `S_{i-5}` 关闭游戏。

Bob 可以提交证据：

- `S_i` 由双方签名；
- `i > i - 5`。

因此 Alice 使用旧状态属于作弊，可能被罚没押金。

---

### 19.3 Unresponsive Counterparty / 不响应对手

### English
Suppose Alice sees that Bob checkmated her in `S_{i+1}` and becomes unresponsive.

Bob can force the game to continue on-chain by broadcasting:

- the latest mutually signed state,
- his new move.

### 中文
假设 Alice 看到 Bob 在 `S_{i+1}` 中将死自己，于是不再响应。

Bob 可以通过链上提交：

- 最新的双方签名状态；
- 自己的新一步棋；

强制游戏继续执行。

### Design Pattern / 设计模式
State channels must support two failure modes:

1. Closing with old state.
2. Refusing to progress.

状态通道必须处理两种失败场景：

1. 用旧状态关闭；
2. 拒绝继续推进。

---

## 20. Virtual Channel  
## 20. 虚拟通道

### English
Virtual channels are associated with work by Dziembowski et al.

Advantage over HTLC routing:

- Intermediaries are involved only when opening and closing the virtual channel.
- Intermediaries do not need to cooperate during state updates in between.

This reduces interactive complexity during normal operation.

### 中文
虚拟通道与 Dziembowski 等人的工作相关。

相对于 Lightning 中的 HTLC 路由，虚拟通道的优势是：

- 中间节点只在打开和关闭虚拟通道时参与；
- 中间状态更新期间不需要中间节点持续配合。

这降低了正常运行时的交互复杂度。

---

## 21. Sprite Channel  
## 21. Sprite 通道

### English
Sprite Channel, associated with Miller et al., uses smart contracts to:

1. Reduce worst-case collateral lock-up time.
2. Allow partial withdrawal and deposit.

Compared with Lightning, Sprite aims to improve capital efficiency and reduce worst-case waiting time in linked payments.

### 中文
Sprite Channel 与 Miller 等人的研究相关，核心思想是使用智能合约来：

1. 减少最坏情况下的抵押锁定时间；
2. 支持部分提款和存款。

相比 Lightning，Sprite 的目标是提升资本效率，降低多跳支付中最坏情况下的等待时间。

---

## 22. Generalized State Channel  
## 22. 广义状态通道

### 22.1 Motivation / 动机

### English
Question:

> What if each channel is not application-specific?

A generalized state channel allows arbitrary contracts to be instantiated and executed off-chain.

### 中文
问题：

> 如果每个通道不再局限于某个特定应用，会怎样？

广义状态通道允许任意合约在链下实例化和执行。

---

### 22.2 Core Idea / 核心思想

### English
The core idea includes:

1. Instantiate a new contract off-chain.
2. Use `CREATE2` or a contract registry mechanism so the contract can be deterministically addressed or referred to.
3. Commit to contract code.
4. Execute contract functions via signed transactions and signed post-computation states.

### 中文
核心思想包括：

1. 在链下实例化新合约；
2. 使用 `CREATE2` 或合约注册机制，使合约可以被确定性寻址或引用；
3. 对合约代码做承诺；
4. 通过签名交易和签名后的计算状态执行合约函数。

---

### 22.3 Dispute Handling / 争议处理

### English
If there is a dispute or misbehavior:

1. Forcefully deploy the contract on-chain with the latest signed states; or
2. Collapse from a virtual channel to a higher-level channel and play out the dispute there.

### 中文
如果出现争议或作恶：

1. 强制把合约和最新签名状态部署到链上；
2. 或从虚拟通道“坍缩”到上一层通道，在上层通道中继续处理争议。

---

## 23. Learn More About GSC  
## 23. 广义状态通道拓展阅读

### English
The lecture lists several resources:

- Perun Network
- StateChannels Team, formerly Counterfactual
- Connext Network
- Kristina's presentation from TU Darmstadt
- Perun 2.0 non-technical introduction
- Formalized Generalized State Channel with UC proof
- Informal early proposal

### 中文
课件列出了一些拓展阅读资源：

- Perun Network
- StateChannels Team，前身是 Counterfactual
- Connext Network
- TU Darmstadt 的 Kristina 演讲
- Perun 2.0 非技术介绍
- 带 UC 形式化证明的广义状态通道论文
- 早期非正式提案

---

## 24. Channel-Based Network Summary  
## 24. 通道型网络总结

### Good / 优点

| English | 中文 |
|---|---|
| Instant finality | 即时最终性 |
| Low cost in most cases | 大多数情况下成本低 |
| Private: intermediate updates only known within the channel | 隐私较好：中间更新只在通道内可见 |
| Best for fixed participants with high transaction volume | 适合固定参与者之间的高频交易 |

### Bad / 缺点

| English | 中文 |
|---|---|
| High collateral lockups for routing hubs | 路由中心需要大量资金锁定 |
| Capital inefficiency | 资本效率低 |
| Potential point of failure | 路由中心可能成为故障点 |
| Always-online assumption | 用户需要保持在线 |
| Users must submit fraud proof before dispute period elapses | 用户需在争议期内提交欺诈证明 |
| Users are responsible for their own data availability | 用户自己负责相关数据可用性 |
| Multi-hop and virtual channel disputes are complex | 多跳和虚拟通道争议解决较复杂 |

### 中文重点
通道型网络的核心优势是：在合作情况下极快、便宜、隐私好。  
核心弱点是：资金锁定、在线要求、数据保存责任和争议复杂性。

---

# Part II — Commit-Chains and Plasma  
# 第二部分：Commit-chain 与 Plasma

---

## 25. Channel Hub, Sidechain and Commit-chain  
## 25. 通道枢纽、侧链与 Commit-chain

### English
The lecture notes that channel network routing complexity may push the topology into a **hub-and-spoke** structure.

This creates a tension:

- Instant finality is great.
- But to be a hub, one needs high collateral.
- This may compromise the permissionless property of the network.

The lecture asks:

> Can we relax finality to eventual finality to make Layer 2 more permissionless?

This motivates commit-chains.

### 中文
课件指出，通道网络中的路由复杂性可能导致网络形成**中心-辐射 / hub-and-spoke** 结构。

这带来矛盾：

- 即时最终性很棒；
- 但成为路由中心需要大量抵押；
- 这可能削弱网络的无许可性。

课件提出：

> 能不能放宽“即时最终性”，接受“最终最终性 / eventual finality”，从而让 Layer 2 更无许可？

这引出了 Commit-chain。

---

### Sidechain Contrast / 与侧链对比

### English
A sidechain often uses a trusted federation to facilitate a two-way peg into or out of another blockchain.

The lecture asks:

> Can we have a non-custodial sidechain?

Commit-chain is one attempt to answer that question.

### 中文
侧链通常使用受信任联盟来实现与另一条链之间的双向锚定。

课件提出：

> 能不能有一种非托管的侧链？

Commit-chain 可以被看作对这个问题的一种回答。

---

## 26. Generic Commit-chain Workflow  
## 26. 通用 Commit-chain 工作流

### English
A commit-chain involves:

1. Layer 1, such as Ethereum.
2. Layer 2 commit-chain.
3. A commit-chain manager contract or EthBridge contract.
4. Deposits from Layer 1 to Layer 2.
5. Transactions inside the commit-chain.
6. Periodic commitments to Layer 1.
7. Withdrawals back to Layer 1.
8. Fraud proofs if something invalid happens.

### 中文
Commit-chain 包含：

1. Layer 1，例如 Ethereum；
2. Layer 2 commit-chain；
3. Commit-chain 管理合约或 EthBridge 合约；
4. 从 Layer 1 存款到 Layer 2；
5. 在 commit-chain 内部交易；
6. 定期向 Layer 1 提交承诺；
7. 从 Layer 2 提款回 Layer 1；
8. 如果出现无效行为，则提交欺诈证明。

---

### What to Commit? / 提交什么？

| Commitment | English | 中文 |
|---|---|---|
| State root only | Data is off-chain; roughly Plasma | 只提交状态根，数据链下，大致对应 Plasma |
| State root + all transactions | Data is on-chain; Rollup | 提交状态根和所有交易数据，对应 Rollup |

### Who Can Be a Validator? / 谁可以当验证者？

### English
Usually, anyone who puts up enough security deposit in the EthBridge contract can become a validator. The validator set may range from a single validator to more than 100 validators.

### 中文
通常，任何人在 EthBridge 合约中放入足够安全押金，就可以成为验证者。验证者数量可以从 1 个到 100+ 个不等。

---

## 27. Commit-chain vs. Sharding  
## 27. Commit-chain 与分片的区别

### English
The lecture states that the architecture and approaches for ensuring state validity and data availability are similar, but the trust and security structures differ.

### Sharding
Sharding requires **tight coupling**:

- shard chains and beacon chain share security;
- validator pool is shared;
- committees are randomly sampled and shuffled;
- the system tries hard to ensure validity and availability of every part.

### Commit-chain
Commit-chain is more **decoupled** from Layer 1 validity:

- assets can move through a two-way peg;
- but the commit-chain may have only a few validators or even one validator;
- two commit-chains can be drastically different;
- local faults are tolerated, but their consequences should be limited.

### 中文
课件指出，Commit-chain 与 Sharding 在状态有效性和数据可用性的架构上有相似之处，但安全结构不同。

### 分片
分片需要**紧耦合 / tight coupling**：

- 分片链和信标链共享安全性；
- 验证者池共享；
- 委员会随机抽样和轮换；
- 系统努力保证每个部分的有效性和可用性。

### Commit-chain
Commit-chain 与 Layer 1 的有效性更**解耦**：

- 资产可以通过双向锚定转移；
- 但 commit-chain 可能只有少数验证者，甚至只有一个；
- 两条 commit-chain 可以完全不同；
- 它接受局部故障，但尽量限制故障影响范围。

### Key Contrast / 核心对比

| System | English | 中文 |
|---|---|---|
| Sharding | Try very hard to ensure total validity/availability of every part of the system | 尽力保证系统每个部分的完整有效性和可用性 |
| Plasma | Accept local faults but try to limit their consequences | 接受局部故障，但限制其影响 |

---

## 28. Scaling Computation – Plasma  
## 28. Plasma：计算扩容

### English
The lecture treats Plasma as historically important. It was once promising but is now mostly abandoned because it introduced more new problems than it solved.

However, Plasma is still useful for understanding the motivation behind Rollups.

### 中文
课件将 Plasma 视为一个具有历史意义的方案。它曾经非常有希望，但后来基本被放弃，因为它引入的问题比解决的问题还多。

不过，Plasma 对理解 Rollup 的动机非常重要。

---

## 29. History of Plasma  
## 29. Plasma 的历史

### English
The history slide shows Plasma's early excitement and later difficulties:

- Plasma: Scalable Autonomous Smart Contracts
- Minimal Viable Plasma
- Why smart contracts are not feasible on Plasma
- Mass exits for Plasma MVP
- Plasma Cash
- Denomination payments in Plasma Cash
- Less per-user data checking

### 中文
历史页面展示了 Plasma 从兴起到遇到困难的过程：

- Plasma：可扩展自治智能合约；
- Minimal Viable Plasma；
- 为什么智能合约不适合 Plasma；
- Plasma MVP 中的大规模退出问题；
- Plasma Cash；
- Plasma Cash 中的面额支付；
- 降低每个用户需要检查的数据量。

### Key Takeaway / 关键结论
Plasma's research history is a sequence of attempts to fix data availability, mass exit, and usability issues.

Plasma 的研究史可以理解为不断尝试修复数据可用性、大规模退出和可用性问题的过程。

---

## 30. Minimal Viable Plasma / Plasma MVP  
## 30. 最小可行 Plasma

### English
Minimal Viable Plasma uses:

- A Plasma contract on Ethereum as the bridge.
- A Plasma chain maintained by Plasma operators or validators.
- A UTXO model for simplicity.

### 中文
Minimal Viable Plasma 使用：

- Ethereum 上的 Plasma 合约作为桥；
- 由 Plasma operators / validators 维护的 Plasma 链；
- 为简化设计而采用 UTXO 模型。

---

### Plasma MVP Workflow / Plasma MVP 工作流

### English
1. User deposits into the Plasma contract on Ethereum.
2. The user's balance is added in the Plasma chain.
3. Many transactions are processed by Plasma validators.
4. Transactions are included in Plasma blocks.
5. Plasma block headers are committed to the Plasma contract.
6. Transactions inside committed Plasma blocks are considered confirmed.
7. To exit, a user provides proof of UTXOs to the Plasma contract.
8. After a challenge period, if no successful dispute occurs, the contract releases funds.

### 中文
1. 用户向 Ethereum 上的 Plasma 合约存款；
2. 用户余额被添加到 Plasma 链；
3. Plasma 验证者处理大量交易；
4. 交易被打包进 Plasma 区块；
5. Plasma 区块头被提交到 Plasma 合约；
6. 区块内交易被视为确认；
7. 用户退出时向 Plasma 合约提供 UTXO 证明；
8. 经过挑战期后，如果没有成功争议，合约释放资金。

---

### User Responsibility / 用户责任

### English
Users must frequently check the Plasma chain. If invalid blocks are detected, users must exit.

### 中文
用户必须频繁检查 Plasma 链。如果发现无效区块，用户需要及时退出。

---

## 31. Plasma MVP Q&A — Contract Storage  
## 31. Plasma MVP 问答：合约存储什么？

### Question / 问题
What is stored in the Plasma Contract?

Plasma 合约中存储什么？

### English Answer
The Plasma Contract stores:

1. Identities of Plasma operators.
2. Security deposits of Plasma operators.
3. All Plasma block headers.
4. Logic for user deposits.
5. Logic for withdrawals.
6. Logic for disputes.

### 中文答案
Plasma 合约存储：

1. Plasma operators 的身份；
2. Plasma operators 的安全押金；
3. 所有 Plasma 区块头；
4. 用户存款逻辑；
5. 用户提款逻辑；
6. 争议处理逻辑。

---

## 32. Plasma MVP Q&A — Timely Top-up  
## 32. Plasma MVP 问答：如何保证存款及时到账？

### Question / 问题
How can a user enforce timely balance top-up in Plasma chain after deposit?

用户存款后如何保证 Plasma 链上余额及时增加？

### English Answer
When a user deposits to the Plasma contract, the contract immediately adds one block header containing the root of a transaction tree with a single transaction. This transaction mints a UTXO to the user.

Since the Plasma contract represents the canonical history, the Plasma operator must include this single-transaction block, effectively topping up the user's balance.

### 中文答案
当用户向 Plasma 合约存款时，合约会立即添加一个区块头。该区块头包含一棵只有一笔交易的交易树根，这笔交易给用户铸造一个 UTXO。

由于 Plasma 合约代表规范历史，Plasma operator 必须包含这个单交易区块，从而实现用户余额到账。

---

## 33. Plasma MVP Q&A — Payment Lifecycle  
## 33. Plasma MVP 问答：支付生命周期

### Question / 问题
What is the lifecycle of a payment in Plasma chain?

Plasma 链上一笔支付的生命周期是什么？

### English Answer
1. Alice creates and signs a transaction.
2. Alice sends it to the Plasma operator.
3. The Plasma operator verifies it.
4. The operator includes it in the next block with other transactions.
5. The operator commits the new block header to the Plasma contract.
6. Alice's transaction can now be considered confirmed.
7. Alice still needs to send a “confirm message” to Bob.
8. Bob uses this confirm message to safely accept the payment and continue transferring it.

### 中文答案
1. Alice 创建并签署一笔交易；
2. Alice 将交易发送给 Plasma operator；
3. Plasma operator 验证交易；
4. operator 将其与其他交易打包进下一个区块；
5. operator 将新区块头提交到 Plasma 合约；
6. Alice 的交易可被视为确认；
7. Alice 还需要向 Bob 发送一个 “confirm message”；
8. Bob 依靠这个确认消息安全接受付款，并继续转给别人。

### Why This Is Awkward / 为什么很别扭
The confirm message exists because of Plasma's data availability and block withholding problems. Without it, Bob may accept a payment whose history or availability is unsafe.

确认消息之所以存在，是因为 Plasma 存在数据可用性和区块 withholding 问题。没有确认消息，Bob 可能接受一个历史不完整或不可用的付款。

---

## 34. Plasma MVP Q&A — Security  
## 34. Plasma MVP 问答：安全性

### Question / 问题
Why is Plasma secure? Can the operator steal money?

Plasma 为什么安全？operator 能偷钱吗？

### English Answer
If the Plasma operator commits an invalid block by minting money for itself and withdrawing it, users should notice the bad behavior and all try to exit.

The Plasma contract processes withdrawals in chronological order of UTXOs. By the time the operator's withdrawal claim is processed, the contract should already be drained by legitimate users, leaving no balance for the operator's fraudulent exit.

### 中文答案
如果 Plasma operator 提交无效区块，例如给自己凭空铸币并提款，用户应该发现这个坏行为并尝试退出。

Plasma 合约按 UTXO 的时间顺序处理提款。当 operator 的提款请求轮到处理时，合约中的资金应该已经被合法用户取走，因此 operator 的欺诈提款拿不到钱。

---

## 35. Plasma MVP Q&A — User Over-withdrawal  
## 35. Plasma MVP 问答：用户能多提现吗？

### Question / 问题
Can a user Bob withdraw more than he owns?

Bob 能不能提取超过自己拥有的钱？

### English Answer
When Bob exits, he submits:

1. A claimed UTXO.
2. A Merkle proof that the UTXO was included in a transaction tree.
3. A security deposit for the withdrawal.

This starts a challenge period, for example one week.

During the challenge period, others can submit fraud proofs, such as:

- a transaction signed by Bob that already spends the claimed UTXO;
- the corresponding Merkle proof.

If the challenge succeeds:
- Bob's security deposit goes to the challenger.

If no challenge succeeds:
- Bob gets his deposit refunded;
- Bob's UTXO is withdrawn.

### 中文答案
Bob 退出时需要提交：

1. 一个声称属于自己的 UTXO；
2. 该 UTXO 被包含在某个交易树中的 Merkle proof；
3. 提款安全押金。

这会启动一个挑战期，例如一周。

在挑战期内，其他人可以提交欺诈证明，例如：

- Bob 已经签署过花费该 UTXO 的交易；
- 对应的 Merkle proof。

如果挑战成功：
- Bob 的安全押金给挑战者。

如果没有挑战成功：
- Bob 拿回安全押金；
- Bob 的 UTXO 被提现。

---

## 36. Plasma MVP's Awkward Confirmed Message  
## 36. Plasma MVP 尴尬的 Confirmed Message

### English
The lecture explains two reasons behind the confirmed message problem:

1. Coins in Plasma MVP are fungible.
2. Exit priority depends on the time when exits are requested.

The operator can perform **block withholding**, meaning:

- the operator intentionally does not publish a block;
- or delays its publication;
- users then try to exit based on incomplete or incorrect information.

### 中文
课件解释 Confirmed Message 问题的两个原因：

1. Plasma MVP 中的币是可替代的 / fungible；
2. 退出处理顺序取决于退出请求的时间。

operator 可以执行 **block withholding / 区块扣留**：

- 故意不发布某个区块；
- 或延迟发布；
- 用户基于不完整或错误的信息尝试退出。

### Why It Matters / 为什么重要
If users do not know whether a payment was included, spent, or withheld, they cannot safely decide what to accept or exit. This creates complicated payment acceptance rules.

如果用户不知道某笔付款是否被包含、是否已花费、是否被扣留，就无法安全判断是否接受付款或退出。这使 Plasma 支付规则变得复杂。

---

## 37. Plasma Cash  
## 37. Plasma Cash

### English
Plasma Cash attempts to fix some Plasma MVP issues by making each coin **non-fungible** with a unique serial number.

### Good / 优点

| English | 中文 |
|---|---|
| Localizes potential damage | 将潜在损害局部化 |
| Avoids mass exit | 避免大规模退出 |
| Reduces data checking for each user | 减少每个用户需要检查的数据 |
| Users only track coins they own | 用户只需跟踪自己拥有的币 |
| Avoids confirmed message issue | 避免 confirmed message 问题 |

### Bad / 缺点

| English | 中文 |
|---|---|
| No arbitrary denomination | 不支持任意面额 |
| Ever-growing proof size for a coin | 每个币的证明大小持续增长 |
| Block withholding still exists | 区块扣留问题仍然存在 |

### 中文总结
Plasma Cash 通过把每个 coin 变成带唯一序列号的 NFT 式资产，降低了用户需要检查的数据范围。但它牺牲了面额灵活性，并且仍然没有彻底解决数据可用性问题。

---

## 38. Plasma Links  
## 38. Plasma 拓展链接

### English
The lecture lists several learning resources:

- Learn Plasma education site.
- Minimal Viable Plasma proposal.
- Karl's presentation.
- Plasma Cash research thread.
- Karl's simpler Plasma Cash specification.

### 中文
课件列出了一些 Plasma 学习资源：

- Learn Plasma 教育网站；
- Minimal Viable Plasma 提案；
- Karl 的演讲；
- Plasma Cash 研究讨论；
- Karl 的简化 Plasma Cash 规范。

---

# 39. Cross-Topic Synthesis  
# 39. 全讲综合理解

## 39.1 Layer 2 as “Local Court”  
## 39.1 Layer 2 作为“地方法院”

### English
A useful metaphor from the lecture is:

- Layer 1 is the supreme court.
- Layer 2 is the local court.

Layer 2 handles routine interactions. Layer 1 handles deposits, withdrawals, fraud proofs, and disputes.

### 中文
本讲一个重要比喻是：

- Layer 1 是最高法院；
- Layer 2 是地方法院。

Layer 2 处理日常交互；Layer 1 处理存款、提款、欺诈证明和争议。

---

## 39.2 State Validity vs Data Availability  
## 39.2 状态有效性 vs 数据可用性

### English
Layer 2 systems must solve two separate problems:

1. **State validity**
   - Is the new state correct?
   - Did all required parties authorize it?
   - Can invalid transitions be challenged?

2. **Data availability**
   - Can users access the data they need?
   - Can they prove fraud or exit safely?
   - Who stores the relevant data?

### 中文
Layer 2 必须解决两个不同问题：

1. **状态有效性**
   - 新状态是否正确？
   - 是否由必要参与方授权？
   - 无效状态能否被挑战？

2. **数据可用性**
   - 用户能否获得所需数据？
   - 能否证明欺诈或安全退出？
   - 相关数据由谁存储？

---

## 39.3 Channel vs Plasma vs Rollup  
## 39.3 通道、Plasma、Rollup 对比

| Dimension | Channel | Plasma | Rollup |
|---|---|---|---|
| 中文 | 通道 | Plasma | Rollup |
| Main idea | Off-chain signed state among fixed participants | Commit state roots to L1, data mostly off-chain | Commit state roots and transaction data to L1 |
| 主要思想 | 固定参与者链下签名状态 | 向 L1 提交状态根，数据多在链下 | 向 L1 提交状态根和交易数据 |
| Best for | Fixed high-frequency interactions | Historically attempted scalable payments | General L2 execution |
| Data availability | User responsibility | Major problem | Solved by putting data on-chain |
| Dispute | Latest state / challenge | Exit games / fraud proofs | Fraud proofs or validity proofs |
| Current relevance | Still useful for payments and specific apps | Mostly historical | Dominant modern direction |

---

# 40. Exam-Oriented Knowledge Points  
# 40. 考试导向知识点

## 40.1 Must-Know Definitions / 必背定义

### Layer 2
**English:** A protocol built on top of Layer 1 that processes transactions or state updates off-chain while relying on Layer 1 for settlement and dispute resolution.  
**中文：** 构建在 Layer 1 之上的协议，将交易或状态更新放到链下处理，同时依赖 Layer 1 进行结算和争议解决。

### Channel
**English:** A fixed participant off-chain protocol where parties update state by signatures and only use Layer 1 for setup, closure, or disputes.  
**中文：** 固定参与者之间的链下协议，各方通过签名更新状态，只在建立、关闭或争议时使用 Layer 1。

### Payment Channel
**English:** A channel specialized for repeated payments.  
**中文：** 专门用于高频支付的通道。

### State Channel
**English:** A generalized channel that supports arbitrary application states.  
**中文：** 支持任意应用状态的泛化通道。

### HTLC
**English:** A conditional payment contract where funds can be claimed by revealing a hash preimage before a timeout.  
**中文：** 哈希时间锁合约，允许在超时前揭示哈希原像来领取资金。

### Commit-chain
**English:** A Layer 2 chain that periodically commits state or transaction data to Layer 1 and uses Layer 1 for deposits, exits, and fraud proofs.  
**中文：** 定期向 Layer 1 提交状态或交易数据的 Layer 2 链，并通过 Layer 1 处理存款、退出和欺诈证明。

### Plasma
**English:** A commit-chain design that commits state roots to Layer 1 while keeping transaction data off-chain, relying on exits and fraud proofs.  
**中文：** 一种 Commit-chain 设计，只向 Layer 1 提交状态根，交易数据链下保存，依赖退出机制和欺诈证明保障安全。

### Plasma Cash
**English:** A Plasma variant where each coin is non-fungible with a unique serial number to localize data checking and damage.  
**中文：** Plasma 变体，将每个 coin 变成带唯一序列号的非同质化资产，以局部化数据检查和损害范围。

---

## 40.2 Important “Why” Questions / 重要原因类问题

### Why does Layer 2 reduce cost?
Because most transactions are processed off-chain and only setup, closure, commitments, or disputes are placed on-chain.

为什么 Layer 2 降低成本？  
因为大多数交易在链下处理，只有建立、关闭、承诺或争议需要上链。

### Why does Lightning need revocation?
Because in a bidirectional channel, an old state may benefit one party. Revocation punishes broadcasting old states.

为什么 Lightning 需要撤销机制？  
因为双向通道中旧状态可能让某一方获利，撤销机制用于惩罚广播旧状态的行为。

### Why does HTLC need timelocks?
Because intermediaries need time buffers to claim funds from upstream after paying downstream.

为什么 HTLC 需要时间锁？  
因为中间节点在向下游付款后，需要有足够时间从上游索赔。

### Why is Plasma difficult?
Because data availability, block withholding, mass exits, and complex exit games make safe usage hard.

为什么 Plasma 困难？  
因为数据可用性、区块扣留、大规模退出和复杂退出博弈使其难以安全使用。

---

# 41. Possible Exam Questions and Answers  
# 41. 可能考试题与参考答案

## Q1. Explain the fundamental scalability problem in blockchains.
## 问题 1：解释区块链可扩展性的根本问题。

### Answer / 答案
Traditional blockchains require every node to process every transaction, store all data, and agree on a total ordering of transactions. This creates replicated computation, replicated storage, and consensus overhead. As transaction volume grows, the system cannot scale linearly because all nodes repeat the same work.

传统区块链要求每个节点处理所有交易、存储所有数据，并对交易总排序达成共识。这造成重复计算、重复存储和共识开销。随着交易量增长，系统无法线性扩展，因为所有节点都在重复同样工作。

---

## Q2. Compare Spilman Channel and Lightning Network.
## 问题 2：比较 Spilman Channel 和 Lightning Network。

### Answer / 答案
Spilman Channel is unidirectional. Alice can repeatedly send Bob signed transactions that increase Bob's balance. Bob has an incentive to broadcast the latest state. It uses timeout refunds for Alice.

Lightning Network is bidirectional. Since either party may benefit from an old state, it uses revocation secrets to punish old-state broadcasts. It also uses HTLCs to route payments across multiple channels.

Spilman Channel 是单向的。Alice 不断向 Bob 发送让 Bob 余额增加的签名交易，Bob 有动力广播最新状态。Alice 通过超时退款保护自己。

Lightning Network 是双向的。由于双方都可能从旧状态获利，它使用撤销秘密惩罚旧状态广播。同时，它使用 HTLC 通过多个通道路由支付。

---

## Q3. What are the roles of hash preimage and timelock in HTLC?
## 问题 3：HTLC 中哈希原像和时间锁的作用是什么？

### Answer / 答案
The hash preimage acts as the condition for claiming payment. If a party reveals `r` such that `H(r) = h`, they can claim the funds. The timelock ensures that the claim must happen before a deadline. In multi-hop payments, timelocks are staggered to give intermediaries enough time to claim from upstream after paying downstream.

哈希原像是领取付款的条件。如果某方揭示 `r` 且满足 `H(r) = h`，即可领取资金。时间锁保证索赔必须在截止时间前完成。在多跳支付中，时间锁需要错开，以便中间节点在向下游付款后有足够时间向上游索赔。

---

## Q4. Why is data availability a major problem in Plasma?
## 问题 4：为什么数据可用性是 Plasma 的主要问题？

### Answer / 答案
Plasma keeps transaction data off-chain and only commits block headers or state roots to Layer 1. If an operator withholds blocks, users may not know whether their coins were spent, included, or invalidated. Without data, users cannot safely challenge fraud or exit. This leads to confirmed messages, mass exit risks, and complex exit games.

Plasma 将交易数据放在链下，只向 Layer 1 提交区块头或状态根。如果 operator 扣留区块，用户可能不知道自己的币是否已被花费、是否已被包含、是否已失效。没有数据，用户就无法安全挑战欺诈或退出。这导致 confirmed message、大规模退出风险和复杂退出博弈。

---

## Q5. Compare commit-chain and sharding.
## 问题 5：比较 Commit-chain 和 Sharding。

### Answer / 答案
Both commit-chain and sharding use similar ideas for distributing execution and committing results. However, sharding is tightly coupled with Layer 1 or the beacon chain and shares the same validator pool and security assumptions. Commit-chain is more decoupled; it may have its own validator set, possibly only a few validators, and only connects to Layer 1 through deposits, withdrawals, and commitments.

Commit-chain 和 Sharding 都涉及分布式执行和提交结果。但 Sharding 与 Layer 1 或信标链紧耦合，共享验证者池和安全假设。Commit-chain 更加解耦，它可以有自己的验证者集合，甚至只有少数验证者，只通过存款、提款和承诺与 Layer 1 连接。

---

# 42. One-Page Review Sheet  
# 42. 一页复习速记

## English
- Blockchain scalability bottleneck: every node processes every transaction and stores all data.
- Layer 2 reduces replicated computation by moving most updates off-chain.
- Channels require fixed participants and signed state updates.
- Spilman is one-way and incentive-based.
- DMC uses time locks but is mostly historical.
- Lightning uses revocation for bidirectional channels and HTLC for routing.
- HTLC uses hash preimages and timelocks.
- State channels generalize payment channels to arbitrary state.
- Virtual channels reduce intermediary involvement during updates.
- Sprite channels reduce worst-case collateral lock time.
- Generalized state channels instantiate contracts off-chain.
- Commit-chains periodically commit to Layer 1.
- Plasma commits state roots but keeps data off-chain.
- Plasma suffers from data availability and block withholding.
- Plasma Cash makes coins non-fungible to localize checking.
- Rollups can be seen as learning from Plasma by putting data on-chain.

## 中文
- 区块链扩容瓶颈：每个节点处理所有交易并存储所有数据。
- Layer 2 通过把大多数更新放到链下，减少重复计算。
- 通道需要固定参与者和签名状态更新。
- Spilman 是单向且基于激励的支付通道。
- DMC 使用时间锁，但主要具有历史意义。
- Lightning 用撤销机制支持双向通道，用 HTLC 支持路由。
- HTLC 使用哈希原像和时间锁。
- 状态通道将支付通道泛化到任意状态。
- 虚拟通道减少中间节点在状态更新期间的参与。
- Sprite 通道降低最坏情况下的抵押锁定时间。
- 广义状态通道支持链下实例化合约。
- Commit-chain 定期向 Layer 1 提交承诺。
- Plasma 提交状态根，但数据放在链下。
- Plasma 的主要问题是数据可用性和区块扣留。
- Plasma Cash 让 coin 非同质化，以局部化检查范围。
- Rollup 可以理解为吸取 Plasma 教训后，将数据放到链上的方案。

---

# 43. Glossary / 术语表

| English | 中文 | Meaning / 含义 |
|---|---|---|
| Layer 1 | 一层链 | Base blockchain such as Ethereum |
| Layer 2 | 二层协议 | Protocol built on top of Layer 1 |
| Off-chain | 链下 | Outside the base blockchain |
| On-chain | 链上 | Directly recorded on blockchain |
| Channel | 通道 | Off-chain state update mechanism |
| Payment Channel | 支付通道 | Channel for repeated payments |
| State Channel | 状态通道 | Channel for arbitrary application state |
| Dispute Period | 争议期 | Time for submitting evidence |
| Fraud Proof | 欺诈证明 | Evidence showing invalid behavior |
| Revocation | 撤销 | Punishment mechanism for old states |
| Hash Preimage | 哈希原像 | Secret input to a hash |
| HTLC | 哈希时间锁合约 | Conditional payment using hash and timeout |
| Commit-chain | 承诺链 | L2 chain periodically committing to L1 |
| Plasma | Plasma | State-root-only commit-chain with off-chain data |
| Plasma Cash | Plasma Cash | Non-fungible coin version of Plasma |
| Block Withholding | 区块扣留 | Operator hides or delays block data |
| Mass Exit | 大规模退出 | Many users exit at once after detecting risk |
| Data Availability | 数据可用性 | Whether users can obtain data needed for safety |
| State Validity | 状态有效性 | Whether state transitions are correct |

---

# 44. Final Conceptual Summary  
# 44. 最终概念总结

### English
Lecture 09 explains why Layer 2 is necessary and how off-chain systems try to preserve blockchain security while avoiding the cost of global execution.

Channel-based systems work best when a fixed set of users interacts frequently. They provide speed, low cost, and privacy, but require collateral, online monitoring, and careful data storage.

Commit-chain systems relax instant finality and introduce a separate execution environment that periodically commits to Layer 1. Plasma was an early commit-chain design, but its off-chain data model created severe data availability, block withholding, and exit complexity issues. These limitations help explain why modern blockchain scaling moved toward Rollups.

### 中文
第 09 讲解释了为什么 Layer 2 是必要的，以及链下系统如何在避免全局执行成本的同时尽量保持区块链安全性。

通道型系统适合固定用户之间的高频交互。它们速度快、成本低、隐私好，但需要资金锁定、在线监控和可靠的数据保存。

Commit-chain 系统放宽即时最终性，引入一个独立执行环境，并定期向 Layer 1 提交承诺。Plasma 是早期 Commit-chain 设计，但由于交易数据链下保存，它面临严重的数据可用性、区块扣留和退出复杂性问题。这些限制也解释了为什么现代区块链扩容逐渐转向 Rollup。

