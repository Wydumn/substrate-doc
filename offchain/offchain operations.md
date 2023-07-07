# 链下操作

在许多用例中，你可能希望在更新链上状态之前从链下源查询数据或在不使用链上资源的情况下处理数据。 整合链下数据的传统方式涉及连接到[预言机](https://docs.substrate.io/reference/glossary/#oracle)从某些传统来源提供数据。 尽管使用预言机是处理链下数据源的一种方法，但预言机可以提供的安全性、可扩展性和基础设施效率存在局限性。

为了使链下数据集成更加安全和高效，Substrate通过以下功能支持链下操作：

- **链下工作机**（Offchain workers）是一个组件子系统，支持执行长时间运行且可能不确定的任务，例如：
  
  - 网站服务请求
  - 数据的加密、解密和签名
  - 随机数生成
  - CPU 密集型计算
  - 链上数据的枚举或聚合
  
  Offchain workers使你能够将执行时间可能超出块处理管道允许时间的任务移出。 任何可能需要超过允许的最大区块执行时间的任务都是链下处理的理想对象。

- **链下存储**（Offchain storage）是 Substrate 节点的本地存储，offchain workers和链上逻辑都可以访问：
  
  - offchain workers对链下存储具有读写权限。
  - 链上逻辑通过链下索引具有写入权限，但没有读取权限。 链下存储允许不同的工作线程相互通信，并存储用户特定或节点特定的数据，这些数据不需要在整个网络上达成共识。

- **链下索引**（Offchain indexing）是一项可选服务，它允许运行时独立于offchain workers直接写入链下存储。 链下索引为链上逻辑提供临时存储，补充链上状态。

> 在计算机术语中，"worker"通常指的是一种用于执行任务或处理请求的计算机程序或进程。这个术语常常出现在分布式计算系统、Web应用程序和后台处理任务等场景中。在这些环境中，worker会从一个队列中获取任务，并根据其能力和可用性来执行它们。worker的任务可能涉及到数据处理、计算、网络通信等各种操作。通过将任务分配给多个worker，可以提高计算机系统的并发性和可扩展性。

## Off-chain workers

Offchain workers在 Substrate 运行时之外的自己的 Wasm 执行环境中运行。 这种关注点分离可确保区块生成不受长期运行的链下任务的影响。 但是，由于offchain workers是在与运行时相同的代码中声明的，因此他们可以轻松访问链上状态以进行计算。

![链下工作者](https://d33wubrfki0l68.cloudfront.net/04485e45edfb524bfbebdf0270455902ff5912af/78670/static/505c4ec510929c01c0e608225c5c598a/432c6/off-chain-workers-v2.png)

Offchain workers可以访问扩展的 API，以便与外部世界（external world）进行通信：

- [提交交易](https://paritytech.github.io/substrate/master/sp_runtime/offchain/trait.TransactionPool.html)（签名或未签名）到链以发布计算结果。
- 功能齐全的 HTTP 客户端，允许worker从外部服务访问和获取数据。
- 访问本地密钥库以签署和验证语句（statements）或交易
- 额外的本地[键值数据库](https://paritytech.github.io/substrate/master/sp_runtime/offchain/trait.OffchainStorage.html)在所有offchain workers之间共享
- 用于随机数生成的安全本地熵源（entropy source）。
- 访问节点的精确[本地时间](https://paritytech.github.io/substrate/master/sp_runtime/offchain/struct.Timestamp.html).
- 待机和恢复工作的能力。

请注意，offchain workers的结果不受定期交易验证的约束。 因此，你应该确保链下操作包括一种验证方法，以确定哪些信息进入链中。 例如，你可以通过实现投票、平均或检查发件人签名的机制来验证链下交易。

> 这段话讲的是区块链技术中所谓的离线工作（offchain workers），它们是在主区块链网络之外进行的计算过程。与在链上的交易不同，离线工作的结果没有经过同样的验证，因此可能不可信。为了确保只有准确的信息被添加到主区块链上，应该在离线操作中包含验证方法。例如，可以通过实行投票系统、对多个来源进行平均或检查发送方签名等方式来验证离线交易的准确性。总而言之，这段话强调了在将离线操作添加到主区块链之前必须进行验证的重要性，以防止虚假或不准确的信息被添加到区块链中。

你还应该注意，默认情况下，offchain workers没有任何特定的特权或权限，因此offchain workers也是恶意用户可以利用的潜在攻击媒介。 在大多数情况下，在写入存储之前检查交易是否已由offchain worker提交并不足以保护网络。 与其假设offchain worker可以在没有保护措施的情况下被信任，不如有意设置限制性权限来限制对进程的访问及其可以执行的操作。

在每次区块导入期间都会生成offchain worker。 但是，它们不会在初始区块链同步期间执行。

> “Entropy source”的含义是指一种能够生成高度随机性和不可预测性的数据源。在计算机系统中，熵源通常用于生成加密密钥或其他需要高度随机性的数据。
> 
> 一个安全的本地熵源可以是一些物理过程，比如放射性衰变、硬盘读取、网络流量等，或者是使用专门的硬件设备，如随机数生成器（RNG）等。
> 
> 熵源的重要性在于，如果密钥是基于容易预测的伪随机数生成算法生成的，则攻击者有可能通过破解该算法来获取密钥。因此，使用熵源产生的随机数序列可以提供更高的安全性，因为攻击者无法确定这些随机数的模式或规律，从而更难以破解出密钥。
> 
> "The ability to sleep and resume work" 意思是指计算机系统或设备具有在不关闭电源的情况下进入低功耗状态，并能够自动唤醒以恢复工作的能力。这种功能通常被称为“睡眠模式”、“待机模式”或“休眠模式”。
> 
> 在这种模式下，计算机系统或设备将关闭大部分硬件设备，并停止运行大多数进程和服务，从而达到节省能源和延长电池寿命的目的。但是，一些关键的硬件设备和进程仍然会保持开启，以便在唤醒时能够迅速恢复工作。
> 
> 这种功能对于移动设备和网络服务器等需要连续运行的设备非常重要，因为它可以在无人操作时自动降低功耗，同时又能够快速响应用户请求，提高了用户体验和设备的可靠性。

## 链下存储

链下存储始终是Substrate 节点的本地存储，不会与任何其他区块链节点在链上共享或达成共识。 你可以使用具有读写权限的offchain worker或使用链下索引通过链上逻辑，访问存储在链下存储中的数据。

由于在每次块导入期间都会生成一个offchain worker，因此任意时间内可能有多个offchain worker运行。 与任何多线程编程环境一样，当offchain worker线程访问时，可以通过utilities使用[互斥锁]([Lock (computer science) - Wikipedia](https://en.wikipedia.org/wiki/Lock_(computer_science)))锁定链下存储，以确保数据一致性。

链下存储充当offchain worker线程之间通信以及链下和链上逻辑之间通信的桥梁。 也可以使用远程过程调用 （RPC） 进行读取，因此它适合存储无限增长数据的用例，而不会过度消耗链上存储。

## 链下索引

在区块链的背景下，存储通常与链上状态有关。 然而，链上状态是昂贵的，因为它必须达成一致并填充到网络中的多个节点。 因此，你不应使用链上存储存储历史或用户生成的数据（这些数据会随着时间的推移无限增长）。

为了满足访问历史或用户生成数据的需求，Substrate 使用链下索引提供对链下存储的访问。 链下索引允许运行时直接写入链下存储，而无需使用offchain worker 线程。 你可以通过使用命令行选项`--enable-offchain-indexing`启动 Substrate 节点来启用此功能以持久化数据。

与offchain workers不同，每次处理区块时，链下索引都会填充链下存储。 通过在每个区块填充数据，链下索引可确保数据始终保持一致，并且对于启用索引的每个节点完全相同。

## 接下来

现在你已经熟悉了offchain worker、链下存储和链下索引如何使你能够处理未存储在链上的数据，你可能希望探索下面的offchain workers示例以及如何在运行时开发中使用它们

- [示例：offchain worker](https://github.com/paritytech/substrate/tree/master/frame/examples/offchain-worker)
- [示例：提交交易](https://github.com/JoshOrndorff/recipes/blob/master/text/off-chain-workers/transactions.md)
- [示例：使用 HTTP 请求数据](https://github.com/JoshOrndorff/recipes/blob/master/text/off-chain-workers/http-json.md)
- [示例：链下存储](https://github.com/JoshOrndorff/recipes/blob/master/text/off-chain-workers/storage.md)
- [示例：链下索引](https://github.com/JoshOrndorff/recipes/blob/master/text/off-chain-workers/indexing.md)
