这里是为您详细整理的《SC6019 – Blockchain Privacy & Scalability (区块链隐私与可扩展性)》第10讲的详细中英对照课件文档。文档包含了PDF中所有的知识点、核心架构、对比分析以及技术细节。

---

# SC6019 – Blockchain Privacy & Scalability / 区块链隐私与可扩展性
## Lecture 10: Layer 2 Solutions: Off-Chain Scalability (Scaling Computation – Rollup)
## 第10讲：Layer 2 解决方案：链下扩展（计算扩展 – Rollup）
**Speaker / 讲师:** Neil HAN, Nanyang Technological University (NTU)

---

## 1. Introduction to Off-Chain Scalability / 链下扩展简介

### 1.1 Lightning Network / 闪电网络 (Slide 3)
*   **English**: The Lightning Network is an early Layer 2 solution primarily used for scaling Bitcoin. It utilizes state channels and routing. For instance, if Alice wants to send 1 BTC to Dave, she can route the payment through intermediate nodes using hashed timelock contracts (HTLCs) with a secret `r` and its hash `h <- H(r)`.
*   **中文**: 闪电网络是早期的 Layer 2 解决方案，主要用于比特币扩容。它利用状态通道和路由机制。例如，如果 Alice 想给 Dave 发送 1 BTC，她可以通过中间节点进行路由，利用哈希时间锁合约（HTLC），并设定一个秘密值 `r` 及其哈希 `h <- H(r)`。

### 1.2 Generic Commit-chain Workflow / 通用提交链工作流 (Slide 4)
*   **English**: A commit-chain (Layer 2) operates alongside the Ethereum Mainnet (Layer 1). The general workflow involves:
    1.  **Step 1: Deposit**: Users deposit funds into a Commit-chain Manager Contract / EthBridge Contract on L1.
    2.  **Step 2: TX in commit-chain**: Users perform fast, cheap transactions on L2.
    3.  **Step 3: Withdraw**: Users withdraw funds back to L1 using proofs.
    *   **Key Questions**:
        *   *What to commit?* State root only (Data off-chain $\rightarrow$ Plasma) vs. State root + all tx data (Data on-chain $\rightarrow$ Rollup).
        *   *Who can be a validator?* Anyone who puts up enough security deposit in the EthBridge contract (Validator sets usually range from 1 to 100+).
*   **中文**: 提交链（Layer 2）与以太坊主网（Layer 1）并行运行。通用工作流包括：
    1.  **第一步：存款 (Deposit)**：用户将资金存入 L1 上的提交链管理合约 / 桥接合约。
    2.  **第二步：提交链内交易**：用户在 L2 上进行快速、廉价的交易。
    3.  **第三步：提款 (Withdraw)**：用户利用证明将资金提回 L1。
    *   **核心问题**：
        *   *提交什么？* 仅提交状态根（数据链下 $\rightarrow$ Plasma） vs. 状态根 + 所有交易数据（数据链上 $\rightarrow$ Rollup）。
        *   *谁能成为验证者？* 通常是在 EthBridge 合约中存入足够保证金的任何人（验证者集合通常在 1 到 100+ 人之间）。

---

## 2. Optimistic Rollup (OP) / 乐观 Rollup

### 2.1 Key Ideas / 核心理念 (Slide 5)
*   **English**:
    1.  Relax validity proof to **fraud proof** to allow **optimistic execution** of general computation. (Assume transactions are valid unless challenged).
    2.  Still **use Layer 1 for data availability**.
*   **中文**:
    1.  将有效性证明放宽为**欺诈证明 (Fraud Proof)**，以允许对通用计算进行**乐观执行**。（假设交易有效，除非受到挑战）。
    2.  仍然**利用 Layer 1 来保证数据可用性 (Data Availability)**。

### 2.2 State Transition & Dispute Assertion / 状态转换与争议断言 (Slides 6-7)
*   **English**:
    *   **State Transition**: Users send Tx messages + signatures. The rollup applies the tx and calculates a new state root. The rollup block (All TX + sigs + New State Root) is committed to the L1 Rollup Contract.
    *   *Question:* Without validity proof, how does the L1 contract know the state root is correct? Some OP Rollups upload all txs first while waiting for validators to commit the new root.
    *   **Dispute Assertion**: If a malicious state is posted, a verifier can challenge it. The L1 contract verifies the challenge `verify_state_transition(pre_state, block, witness)`. If the challenge succeeds, the challenger is rewarded, and the malicious validator is slashed.
    *   *Note*: Dispute resolution can be one-round or multi-round (e.g., Arbitrum interacts back-and-forth off-chain to narrow down the dispute source, minimizing L1 referee work).
*   **中文**:
    *   **状态转换**：用户发送交易信息+签名。Rollup 节点应用交易并计算新的状态根。Rollup 区块（包含所有交易、签名和新状态根）被提交到 L1 的 Rollup 合约。
    *   *疑问：* 没有有效性证明，合约如何知道新状态根是正确的？部分 OP 会先上传所有交易，同时等待验证者提交新状态根。
    *   **争议断言**：如果提交了恶意状态，验证者可以发起挑战。L1 合约会验证挑战 `verify_state_transition(前置状态, 区块, 见证数据)`。如果挑战成功，挑战者获得奖励，恶意验证者被罚没（Slashing）。
    *   *注*：争议解决可以是单轮或多轮的（例如 Arbitrum 在链下进行多轮交互，缩小争议范围，最小化 L1 裁判的计算工作量）。

### 2.3 OP Analysis / 乐观 Rollup 优劣势分析 (Slide 8)
*   **English**:
    *   **Good (✓)**: State validity ensured by fraud proof (economically bounded security); Bypasses Data Availability (DA) issues by posting raw data; Supports EVM compatible smart contracts easily; Finality only needs 1 block confirmation (if a watchtower is validating); Low cost during dispute adjudication.
    *   **Bad (✗)**: Safety requires "always online assumption" (users or 3rd-party watchtowers must constantly monitor L1); Withdrawals take ~1 week (due to the dispute & validator unbonding period).
*   **中文**:
    *   **优势 (✓)**：通过欺诈证明确保状态有效性（经济学约束的安全）；通过将原始数据发布到链上绕过了数据可用性问题；完美支持 EVM 兼容的智能合约；只需1个区块确认即可达到最终性（假设有瞭望塔验证）；争议裁决成本相对较低。
    *   **劣势 (✗)**：安全性依赖“永远在线假设”（用户或第三方瞭望塔必须持续监控）；提款需要长达1周的时间（为了留出争议期和验证者解除绑定期）。

---

## 3. Zero-Knowledge Rollup (ZK Rollup) / 零知识 Rollup

### 3.1 ZK-Rollup Comes to Rescue / 拯救 Plasma 的困境 (Slide 9)
*   **English**:
    1.  Use **zkSNARK / zkSTARK** to provide **validity proof** of the new state root.
    2.  Use **Layer 1 for data availability** by posting all tx data on-chain (`calldata`) to ensure data needed to reconstruct the latest state is always available.
*   **中文**:
    1.  使用 **zkSNARK / zkSTARK** 为新状态根提供**有效性证明 (Validity Proof)**。
    2.  **利用 Layer 1 保证数据可用性**，通过将所有交易数据发布到链上（`calldata`），确保重建最新状态所需的数据永远可用。

### 3.2 ZK Rollup Workflows (Transfer, Deposit, Withdraw) / 工作流 (Slides 10-12)
*   **English**:
    *   **Transfer**: Tx is applied, new state root is calculated. A validity proof ($\pi$) is generated for all Txs and committed to the L1 contract. L1 verifies $\pi$ instantly.
    *   **Deposit**: Appends tx to the inbox. Calculates new state root. The proof $\pi$ also proves the topped-up balance of the address in the L2 state.
    *   **Withdraw**: User provides a Merkle Proof of their balance. The system verifies the proof, immediately refunds on L1, appends the tx, and generates $\pi$. The proof proves the deducted balance.
*   **中文**:
    *   **转账 (Transfer)**：应用交易，计算新状态根。为所有交易生成有效性证明（$\pi$）并提交给 L1 合约。L1 合约会即刻验证 $\pi$。
    *   **存款 (Deposit)**：将交易追加到收件箱中，计算新状态根。证明 $\pi$ 同时证明了收件箱中地址的充值后余额。
    *   **提款 (Withdraw)**：用户提供状态树中余额的 Merkle 证明。验证者验证证明并立即在 L1 退款，追加交易并生成 $\pi$。该证明同时证明了该地址余额已被正确扣除。

### 3.3 ZK Rollup Analysis / ZK Rollup 优劣势分析 (Slide 13)
*   **English**:
    *   **Good (✓)**: State validity strictly ensured by math (zkSNARK/zkSTARK)—no one can corrupt state or steal funds; Bypasses DA issues by posting raw data to L1; Safety does *not* require "always online assumption"; Fast finality (around 10 mins, bottleneck is proof generation time).
    *   **Bad (✗)**: General computation/arbitrary L2 contracts are very expensive and tough to implement (zkEVM is hard); Throughput was historically bounded (~300 tps) due to proof generation costs; Advanced cryptography is harder to implement and audit.
*   **中文**:
    *   **优势 (✓)**：状态有效性由数学证明（zkSNARK/zkSTARK）严格保证——任何人（包括验证者）都无法破坏状态或窃取资金；通过将原始数据发至 L1 绕过 DA 问题；安全性不依赖“永远在线假设”；最终性确认快（约10分钟，主要取决于证明生成时间）。
    *   **劣势 (✗)**：支持任意 L2 合约/通用计算非常昂贵，EVM 兼容极其困难（zkEVM开发难度大）；由于证明生成的成本，早期的吞吐量受限（约 300 tps）；高级密码学的实现和审计难度极高。

---

## 4. Commit-chain Design Space / 提交链设计空间矩阵 (Slide 14)

| Data / Proof Type             | Validity Proof (有效性证明)                        | Fraud Proof (欺诈证明)                           |
| :---------------------------- | :------------------------------------------------- | :----------------------------------------------- |
| **Data On-chain (数据链上)**  | **ZK Rollup** (e.g., ZKSync, Loopring, Starknet)   | **Optimistic Rollup** (e.g., Arbitrum, Optimism) |
| **Data Off-chain (数据链下)** | **ZK Plasma / Validium** (e.g., Validium, StarkEx) | **Plasma** (Various implementations)             |

---

## 5. Case Study 1: Base & OP Stack (Coinbase L2) / 案例分析 1：Base 与以太坊扩展

### 5.1 Overview & Superchain Vision / 概览与超级链愿景 (Slides 15-20)
*   **English**: Base is an Ethereum L2 built by Coinbase to scale the cryptoeconomy. It has **no token** and uses ETH for gas. It is built on the **OP Stack** in collaboration with Optimism. The vision is that "Onchain will be a superchain, not a monolith," with Ethereum as L1 and a thriving ecosystem of interoperable L2s. Base targets transaction costs <$0.01/tx.
*   **中文**: Base 是由 Coinbase 构建的以太坊 L2，旨在扩展加密经济。它**没有发行代币**，使用 ETH 作为 Gas 费。它基于 **OP Stack** 并与 Optimism 合作开发。其愿景是“链上将是一个超级链（Superchain），而不是单一庞然大物”，以太坊作为 L1，众多可互操作的 L2 组成繁荣生态。Base 的目标是将交易成本降至 $0.01 以下。

### 5.2 How L2s Save Gas / L2 如何节省 Gas 费？ (Slides 34-39)
*   **English**: If all txs are submitted to L1, how do L2s save gas?
    1.  **Storage**: State storage on L1, but intensive computation happens on L2.
    2.  **Compression**: Base uses `zlib` to compress data.
    3.  **Batching**: Compressing batches of transactions is much more efficient (1 big L1 tx instead of many small ones).
    4.  **Calldata**: L2 txs are submitted in L1 tx `calldata`. Calldata (16 gas/byte) is about 40x cheaper than L1 state storage (approx 625 gas/byte).
    5.  **EIP-4844 (Proto-Danksharding) - Blobs**: Introduces Shard Blob Transactions. Blobs are 10x-100x cheaper than calldata, but data is pruned after 14 days (sufficient for L2 DA).
*   **中文**: 如果所有交易都要提交到 L1，L2 是如何节省 Gas 的？
    1.  **存储与计算分离**：L1 负责最终存储，而昂贵的计算在 L2 发生。
    2.  **数据压缩**：Base 使用 `zlib` 进行交易压缩。
    3.  **批量处理 (Batching)**：将海量交易打包成一个大的 L1 交易（分摊了基础交易成本）。
    4.  **利用 Calldata**：L2 交易数据作为函数参数 (`calldata`) 提交。Calldata 成本（16 Gas/字节）比 L1 状态存储（约 625 Gas/字节）便宜近 40 倍。
    5.  **EIP-4844 (Proto-Danksharding) - Blobs 数据块**：引入 Blob 交易。Blob 的成本比 calldata 便宜 10到100倍，但数据只会保存 14 天（对 L2 的数据可用性来说已经足够）。

---

## 6. Case Study 2: StarkWare & ZK-STARKs / 案例分析 2：StarkWare 与 ZK-STARK

### 6.1 Scalable Proofs & PCP Theorem / 可扩展证明与 PCP 定理 (Slides 43-48)
*   **English**: 
    *   *Analogy*: When polling an election, the margin of error is independent of the population size. 
    *   **PCP Theorem (1990s)**: "A single reliable PC can monitor the operation of a herd of supercomputers working with possibly extremely powerful but unreliable software and untested hardware." However, early versions required proofs larger than the atoms in the solar system.
    *   **Breakthrough (2018)**: Efficient enough to write a proof on a laptop (FRI - Fast Reed-Solomon IOP of Proximity).
*   **中文**:
    *   *类比*：在进行选举民调时，误差范围与人口总数无关（抽样定律）。
    *   **PCP 定理（概率可检验证明，20世纪90年代）**：“一台可靠的个人电脑，可以监控一群可能使用极其强大但不可靠的软件和未经测试的硬件的超级计算机的运行。” 但是早期理论需要的证明大小超过了太阳系原子的总和。
    *   **突破（2018年）**：算法优化（基于 FRI），现在一台笔记本电脑就能生成证明。

### 6.2 ZK-STARK Mechanics / ZK-STARK 原理 (Slides 49-55)
*   **English**: 
    *   **Claim**: Starting from state hash `x`, I processed 10,000 txs using program `P`, and the new state hash is `y`.
    *   **How**: A Sudoku-like set of constraints is implied by the statement. The Prover submits a solution. The Verifier samples and checks just one constraint succinctly.
    *   **Magic**: A good proof satisfies *all* constraints. Any bad proof must make 99% of constraints unsatisfied, making it extremely easy for the verifier to catch fraud with minimal sampling.
    *   **Scalability Definition**: Proving time scales as $T \cdot \text{poly}(\log T, |x|)$; Verification time scales as $\text{poly}(\log T, |x|)$.
    *   **Transparency**: No trusted setup, no private keys (Arthur-Merlin protocol).
*   **中文**:
    *   **断言 (Claim)**：从状态哈希 `x` 开始，我使用程序 `P` 处理了 10,000 笔交易，新的状态哈希是 `y`。
    *   **如何证明**：这个断言会生成一组类似“数独”的约束条件。证明者提交解答。验证者只需简洁地“抽样”检查其中一个约束即可。
    *   **数学魔法**：好的证明满足所有约束；而任何“伪造”的坏证明，都会导致 99% 的约束无法满足，因此验证者只需极少的抽样就能立刻发现造假。
    *   **可扩展性定义**：生成证明的时间复杂度为 $T \cdot \text{poly}(\log T, |x|)$；验证证明的时间复杂度极低，仅为 $\text{poly}(\log T, |x|)$。
    *   **透明性**：无需可信设置 (Trusted Setup)，无需私钥（基于 Arthur-Merlin 公开随机掷币协议）。

### 6.3 STARK vs. SNARK Comparison / 对比 (Slide 66)

| Feature / 特性                   | ZK-STARK       | ZK-SNARK            |
| :------------------------------- | :------------- | :------------------ |
| **Proof Size (证明大小)**        | ~400 KB (较大) | 288 bytes (极小)    |
| **Verification Time (验证时间)** | $O(\log^2(n))$ | Constant (常数时间) |
| **Proving Time (证明生成时间)**  | 1x (极快)      | 10x (较慢)          |
| **Trusted Setup (可信设置)**     | No (不需要)    | Yes (需要)          |
| **Quantum Secure (抗量子)**      | Yes (抗量子)   | No (不抗量子)       |

### 6.4 Cairo Language vs Solidity / 编程语言对比 (Slides 63-64)
*   **English**: To optimize for ZK-STARKs, a new language was needed. `Cairo` is to StarkNet what `Solidity` is to Ethereum. Why? Because STARKs have different constraints: Algebraic steps (mod p) are very cheap in STARKs, but bitwise operations (like Keccak hashing) are extremely expensive to prove.
*   **中文**: 为了针对 ZK-STARK 进行优化，需要一种新的语言。`Cairo` 对于 StarkNet 就如同 `Solidity` 对于以太坊。原因在于 STARK 的底层约束不同：代数运算 (mod p) 在 STARK 中证明成本极低，但按位运算（如 Keccak 哈希）的证明成本极其昂贵。

### 6.5 Types of zkEVM / zkEVM 的四大类型 (Slide 68)
*   **English**:
    *   **Type 1**: No EVM changes. Full compatibility. Very Slow proving. 
    *   **Type 2**: Changes to Storage data structure. High compatibility. Slow proving.
    *   **Type 3**: Changes to Storage, hashes, precompiles. Partial compatibility. Fast proving (e.g., Scroll, Polygon zkEVM, zkSync).
    *   **Type 4**: Completely different VM. None/No EVM compatibility. Very Fast proving (e.g., Starknet, Polygon Miden).
*   **中文**:
    *   **Type 1 (完全等效)**：不改变任何 EVM 架构。完全兼容。证明生成**非常慢**。
    *   **Type 2 (完全 EVM 等效)**：更改存储数据结构。高兼容性。证明生成**较慢**。
    *   **Type 3 (部分等效)**：更改存储、哈希算法和预编译合约。部分兼容。证明生成**快**（如 Scroll, Polygon zkEVM）。
    *   **Type 4 (高级语言等效)**：采用完全不同的虚拟机。与 EVM 不兼容。证明生成**极快**（如 Starknet, Polygon Miden）。

---

## 7. Comparing L2 Solutions / L2 解决方案全面横评 (Slides 69-70)
*(Based on Matter Labs Comparison Framework)*

*   **English**:
    *   **Security (Liveness & Exit)**: State channels, Sidechains, Plasma, and Optimistic rollups rely on watch-towers or bonded validators (liveness assumption: Yes). Validium and zkRollups do NOT rely on watchtowers (liveness assumption: No).
    *   **Throughput**: 
        *   Optimistic Rollup: ~500 - 2k TPS
        *   ZK-Rollup: 2k - 20k+ TPS
    *   **Withdrawal Time**:
        *   Optimistic: 1 to 2 weeks (Dispute window)
        *   ZK-Rollup: A few minutes to hours (1..10 min)
    *   **Privacy**: Hard and expensive in OP; Easy and cheap in ZK.
*   **中文**:
    *   **安全性 (活跃度与退出机制)**：状态通道、侧链、Plasma 和 OP Rollup 都依赖瞭望塔机制或被抵押的验证者（需要在线假设）。Validium 和 zkRollup **不**依赖瞭望塔（不需要在线假设）。
    *   **吞吐量 (TPS)**：
        *   OP Rollup：约 500 - 2,000 TPS
        *   ZK-Rollup：2,000 - 20,000+ TPS
    *   **提款时间 (Withdrawal Time)**：
        *   OP Rollup：1 到 2 周（为了留出欺诈挑战期）
        *   ZK-Rollup：几分钟到几小时（通常 1~10 分钟）
    *   **隐私保护**：在 OP 中实现困难且昂贵；在 ZK 中实现相对容易且便宜。

---

## 8. Ecosystems, Superchains, and Fractal Scaling / 生态、超级链与分形扩展 (Slides 74-77)

### 8.1 Blockchain Game Spectrum / 链游图谱 (Slide 74)
*   **English**: Games range from Web 2.0+ (NFTs + off-chain logic, like Axie), Web 2.5 (Hybrid on-chain/off-chain logic), to Fully On-chain Autonomous Worlds (AW) where all game state and logic are strictly on-chain.
*   **中文**: 链游范围涵盖 Web 2.0+（NFT + 链下逻辑，如 Axie）、Web 2.5（链上链下混合逻辑），以及全链上自治世界 (Fully On-chain AW)——所有游戏状态和逻辑都严格在链上运行。

### 8.2 Fractal Scaling: L2 to L3 / 分形扩展：从 L2 到 L3 (Slide 77)
*   **English**:
    *   Recursive proofs open up novel design options. 
    *   **L3** is introduced as an application-specific layer, built recursively over L2. 
    *   L3 serves bespoke needs: hyper-scalability, better control of the tech stack, and enhanced privacy.
    *   For example, StarkEx (currently an L2 solution serving high-volume DApps like dYdX, Sorare, Immutable) will be ported to L3.
*   **中文**:
    *   递归证明技术开启了全新的架构设计可能。
    *   **L3** 被引入作为“特定应用层 (App-Specific)”，它是在 L2 基础上递归构建的。
    *   L3 满足定制化需求：超级可扩展性 (Hyper-scalability)、对技术栈的更好控制、以及更强的隐私保护。
    *   例如，StarkEx（目前作为 L2 为 dYdX、Sorare 等高频 DApp 提供服务）未来将被移植为 L3 解决方案。

---

**End of Document / 文档结束**