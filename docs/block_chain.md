# 📚 Block Chain

此节笔记主要参考了 PKU 肖臻老师的《区块链技术与应用》公开课。

肖臻老师的homepage：[http://zhenxiao.com/](http://zhenxiao.com/)

公开课网址：[https://www.bilibili.com/video/av37065233/](https://www.bilibili.com/video/av37065233/)

---

## 什么是区块链？

区块链是一种去中心化、分布式且通常是公共的数字分类账，由称为区块的记录组成，这些区块用于记录交易信息到多台电脑上，以便任何涉及的区块都不能被追溯更改，否则会影响后续的区块。

从数据结构的角度看，区块链是一个单向链表，其中每个节点是一个区块而节点之间的指针采用 hash pointer。

每个区块都包含两个部分：block header 以及 block body。

block header 中主要维护了指向上一个区块的 block header 的 hash pointer，以及组织本区块数据的各种树的 root pointer。

block body 中主要保存了本区块的各种数据。BTC 链的 block body 保存了本区区块的交易记录，而 ETH 链的 block body 保存了本区块的用户状态、交易记录以及收据。

![ETH chain's block](img/ETH_chain.png)
**ETH chain's block**

区块链这个数据结构本身被维护在一个 P2P 的网络中，该网络拓扑有两类节点：全节点和轻节点。
每个全节点保存区块链的一个完整副本，而每个轻节点只保存区块链所有 block header 的副本。当轻节点需要 block body 的信息是，它会请求全节点发送相应信息。

---

## 区块链如何保证不能被篡改？
这得益于 hash pointer。所谓的 hash pointer 只是一种形象的说法，实际上 block header里只有哈希值，没有指针。

那么怎么才能找到前一个区块的内容呢？全节点一般是把这些区块存储在一个 (key，value) 数据库里面。key 是区块的哈希，value就是区块的内容。一个常用的 key value 数据库是 level DB。所谓的区块链这种链表结构实际上是在 level DB 里面用哈希值算出来的。只要你掌握了最后一个区块的哈希值，那么你通过 level DB 的查找，哈希值 key 对应的 value 就可以把最后一个区块的内容取出来。然后这个区块块头里面，又有指向前一个区块的哈希值。那么再去查找 key 和 value，可以找到前一个区块的内容，以此类推，一步一步往前找，最终能够把整个区块链都找出来。

所以说在实际系统当中，所谓的哈希指针，只有哈希，没有指针，或者也可以认为哈希值的本身就是指针。

有一些节点没有保存完整的区块链的信息，只保存了最近的几千个区块，如果需要用到前面的区块的信息可以问其他的全节点要。哈希指针的性质保证了整个区块链的内容是不可篡改的。

在验证的时候，**节点会重新计算前一个区块的哈希值**，然后把重新算出来的真实哈希 hash_actual 和当前区块里声明的 previous_hash 对比。

你改了前一个区块，但后一个区块早就记着原来的哈希了，
而验证的时候，**用新的前区块内容重新算哈希 ➔ 新算出来的哈希值和后区块声明的值不一样 ➔ 对不上，验证失败。**

---

## 维护区块链的网络和共识机制

---

## 挖矿
并非所有的用户都能向区块链上添加新的块，只有提供了 PoW 或 PoS 以此获得了**记账权**的用户有权向区块链添加新的区块。
而获得 PoW （往往伴随着 block reward）的方法称之为挖矿。

挖矿的逻辑是，miner 需要不断调整一个叫做 nonce 的参数，使得一个复杂的 hash function 的输出小于规定的 target 值。 在申请提交区块时，miner 将这个 nonce 参数写入 block header，维护区块链的节点会验证这个 nonce 是否使得 hash function 的输出小于 target。如果确实小于 target，发布这个区块。

由于尝试的 nonce 是随机的，所以**挖矿本身是无记忆的**。也就是说已经尝试的 nonce 数量，和找到一个小于 target 的哈希值的概率无关。

---

## 验证

---

## 区块链的分叉与合并

___

## Merkle Proof
Merkle Proof 是指在区块链中，轻节点用来验证交易合法性的证明机制。它是 SPV（Simplified Payment Verification） 的实现基础。

交易者向维护区块链的网络节点提交区块时需要证明将要写入区块的交易合法。在 BTC 中，交易者需要提供币的来源信息，在 ETH 中，交易者需要提供账号状态信息（账户余额信息）；这些信息或保存在 Merkle tree 中，或保存在 Modified Merkle Patricia Trie 中。当交易者要提交相应信息的节点时，实际上是提供了从 root 节点到相应叶子节点的链路上的所有 hash pointer。维护区块链的网络节点将给定的 hash pointer 与同节点中的 hash pointer 组合，自底向上计算出最终的实际根节点的 atul_hash 并和 block header 中维护的根节点 hash 对比。若两者相等，验证通过；反之不通过。

下面的展示的是一个轻节点进行 Merkle Proof 的过程。

![mpt](img/merkle_proof-1.png)
![mpt](img/merkle_proof-2.png)
![mpt](img/merkle_proof-3.png)
![mpt](img/merkle_proof-4.png)
**A Merkle Proof in Light Node**

---

## ETH

### ETH 的数据结构
ETH 的数据结构主要有两种：Block Chain 以及 Modified Merkle Patricia Trie。

这里主要讨论 Modified Merkle Patricia Trie。简单地讲，Modified Merkle Patricia Trie 是改良后的 Merkle Patricia Trie， 而 Merkle Patricia Trie 就是使用 hash pointer 的 Radix Tree。

不同于 BTC 只在 block body 中维护交易的内容，ETH 显式地在 block body 中维护账户状态，交易以及收据三种信息。这三种信息被组织成 Merkle Patricia Trie，其 root pointer 被维护在 block header 中。

![mpt](img/16-ETH-5.png)
**ETH Modified Merkle-Patricia-Trie System**

ETH 使用 Modified Merkle Patricia Trie 主要是基于需要修改用户状态的考虑。如果像 BTC 一样直接使用 Merkle Tree，每次修改一个账户状态，所有节点上的 Merkle Tree 必须重构，但不同的节点可能会构造出不同的 Merkle Tree，导致数据不一致。这会为 Merkle Proof 带来困难。

实际上，ETH 对账户的修改并非原地的，而是在新区块的 Modified Merkle Patricia Trie 写入更新后的分支，其他所有节点指向原来的地址。这是因为存在 roll back 的需求，并且这样做不需要把整棵树发布，降低了网络负担。

![ETH chain's block](img/16-ETH-6.png)
**Updating For Modified Merkle Patricia Trie**

ETH 区块内的交易记录与收据同样也被组织为 Modified Merkle Patricia Trie，它们的节点是一一对应的。

### Bloom Filter


### GHOST

### ETH 的挖矿算法
为了防止 ASIC 芯片的滥用，ETH 设计了 Memory Hard Miner Puzzle（ASIC miner 的计算性能远高于普通个人电脑，但访存速度却没有这种优势）。其主要思想是让 miner 在运行 hash function 时需要维护一个大的数组，通过大量访存时间来稀释计算时间上的优势。

计算 ETH 的 hash 需要一个 16M 的 cache 数组以及一个 1G 的 DAG 数组（需要的 cache 和 DAG 的大小还会随着时间增长）。

cache 通过 cache[0] 位置上的 seed 元素不断往后进行 hash function 的递推生成。
$$
cache_0 = seed,
$$
$$
cache_n = hash(cache_{n-1})
$$

DAG 的每个位置由伪随机挑选的 256 个 cache 中的元素生成。最后，从 DAG 中挑选 128 个元素（和 nonce 相关）生成 hash function 的输出。

miner 挖矿时需维护 cache 和 DAG，而进行验证的轻节点只需要维护 cache，动态生成所需要的 DAG 元素。这样做的好处是使得 miner 有 memory hard 的同时，轻节点验证不出现此问题。

### ETH 的挖矿难度调整策略
ETH 进行挖矿难度的主要作用是稳定出块速度以及稳定货币供给量， 挖矿难度 $diff$ 与 miner puzzle 的 $target$ 负相关。每个区块的挖矿难度基于以下公式：

$$
D(H) \equiv 
\begin{cases}
D_0, & \text{if } H_i = 0 \\
\max\left( D_0, P(H)_{H_d} + x \times \zeta_2 \right) + \epsilon, & \text{otherwise}
\end{cases}
$$

$where：$

$$
D_0 \equiv 131072
$$

- $H_i$ 是本区块序号。
- $D(H)$ 是本区块的难度，由基础部分 $P(H)_{H_d} + x \times \zeta_2$ 和难度炸弹部分 $\epsilon$ 相加得到。
- $P(H)_{H_d}$ 是父区块的难度，每个区块的难度都是在父区块难度的基础上进行调整。
- $x \times \zeta_2$ 用于自适应调节出块难度，维持稳定的出块速度。
- $\epsilon$ 表示设定的难度炸弹，其作用是在一段时间后使得挖矿难度爆炸，迫使 PoW 向 PoS 转型。
- 基础部分有下界，为最小值 $D_0 = 131072$。

自适应难度调整 $x \times \zeta_2$:

$$
x \equiv \left\lfloor \frac{P(H)_{H_d}}{2048} \right\rfloor
\quad
$$

$$
\zeta_2 \equiv \max\left( y - \left\lfloor \frac{H_s - P(H)_{H_s}}{9} \right\rfloor, -99 \right)
\quad
$$

- $x$ 是调整的粒度，$\zeta_2$ 为调整的系数。
- $y$ 和父区块的 uncle 数有关。如果父区块中包括了 uncle，则 $y$ 为 2，否则为 1。
- 父块包含 uncle 时，难度会大一个单位，因为包含 uncle 时新发行的货币量大，需要适当提高难度以保持货币发行量稳定。
- 难度降低的上界设置为 $-99$，主要是应对被黑客攻击或其他目前想不到的黑天鹅事件。

$$
y - \left\lfloor \frac{H_S - P(H)_{H_S}}{9} \right\rfloor
$$

- $H_S$ 是本区块的时间戳，$P(H)_{H_S}$ 是父区块的时间戳，均以秒为单位，并规定 $H_S > P(H)_{H_S}$ 。$H_S - P(H)_{H_S}$ 就是出块时间。
  - 该部分是稳定出块速度的最重要部分：出块时间过短则调大难度，出块时间过长则调小难度。

- 以父块不带 uncle 的情况（$y=1$）为例：
  - 出块时间在 $[1,8]$ 之间，出块时间过短，难度调大一个单位。
  - 出块时间在 $[9,17]$ 之间，出块时间可以接受，难度保持不变。
  - 相差时间在 $[18,26]$ 之间，出块时间过长，难度调小一个单位。

难度炸弹 $\epsilon$:

$$\epsilon \equiv \left\lfloor 2^{\left\lfloor \frac{H_i'}{100000} \right\rfloor - 2} \right\rfloor $$

$$H_i' \equiv \max(H_i - 3,000,000)$$

- 每十万个区块扩大一倍，是 2 的指数函数，后期增长极快，因此被称为难度"炸弹"。
- 降低迁移到 PoS 协议时的分叉风险：当挖矿难度激增时，矿工有动力迁移至 PoS 协议。
- 由真实区块号 $H_i$ 减去 $300$ 万得到（即 $H_i' = \max(H_i - 3,000,000)$）。  
- 此设计源于低估 PoS 协议的开发难度，需延长约一年半的过渡期（通过 [EIP-100](https://eips.ethereum.org/EIPS/eip-100) 实现）。

### PoS
PoS 是 Proof-of-Stake 的缩写，是一种基于用户持有的代币数量来进行记账权分配的共识机制。基于 PoW 获得记账权广受诟病的一点是，需要大量的算力和能源，而这两者根本上是由资金大小决定的。因此，PoS 的一个核心想法是**直接通过资本金大小分配记账权**。

而 PoS 则是基于用户持有的代币数量来进行记账权分配，用户持有的代币数量越多，获得记账权的概率就越大。

这样做的第一个好处是降低了能源消耗，因为不需要大量的能源来维持挖矿的过程。同时，由于必须持有代币才能获得记账权，这使得在币种发行早期记账权被开发团队控制，也避免了 Altcoin Infanticide。

ETH 准备采用的 PoS 协议是 Casper，它和 PoW 混合使用，为经过 PoW 验证的区块提供记 finality。获得 finality 状态的区块会成为合法链的部分，即使有恶意攻击使得与之平行的更长分支出现。

在 Casper 协议中引入了 Validator 的概念，Validator 是指一些特殊的账户，它们可以行使投票权使得一个 epoch 获得 finality 状态。在 Casper 中，同一分支上的 50 个连续区块组成一个 epoch，每一轮投票作用于同一分支上的两个连续的 epoch。每次投票需要 2/3 的 Validator 同意才能通过；通过后，前一个 epoch 获得 commit message，后一个 epoch 获得 prepare message；只有获得 commit message 的 epoch 才能获得 finality。

约束 Validator 的行为利用了 Validator Reward 和保证金制度。当 Validator 行为良好时会得到 Reward，反之则销毁一部分的保证金。设计上没有硬性限制 Validator 不可以为多个平行分支投票，但是一旦被发现其保证金会被没收。

目前 PoS 协议还有许多需要解决的问题，PoW 仍然是主流。

### 智能合约

### Code is Law!!!






