**FRAME:Framework for Runtime Aggregation of Modularized Entities** ，即运行时模块聚合框架。 FRAME 是一系列用来简化Runtime开发的模块（称为Pallet）和相关的支持库组成。 每个Pallet 是用于处理特定逻辑领域的单独模块。

FRAME 提供了一些辅助模块用于和Substrate Primitives进行交互, Substrate Primitives为核心客户端提供了接口。

## [概述](https://learnblockchain.cn/docs/substrate/docs/knowledgebase/runtime/frame/#%E6%A6%82%E8%BF%B0)

下图展示了 FRAME 总体架构和它支持库：

![FRAME Architecture](https://learnblockchain.cn/docs/substrate/docs/assets/frame-arch.png)

### [Pallet](https://learnblockchain.cn/docs/substrate/docs/knowledgebase/runtime/frame/#pallet)

当使用 FRAME 来构建时，Substrate 运行时（runtime）是由叫做 pallet 的更小的组件组成。一个 pallet 包含一组类型，存储以及为运行时定义的一组函数。

![FRAME Runtime](https://learnblockchain.cn/docs/substrate/docs/assets/frame-runtime.png)

### [系统库(System Library)](https://learnblockchain.cn/docs/substrate/docs/knowledgebase/runtime/frame/#%E7%B3%BB%E7%BB%9F%E5%BA%93system-library)

系统库为你的区块链提供了底层的类型、存储以及函数。所有其他的pallet都依赖于系统库作为Substrate运行时的基础。

系统库为Substrate运行时定义了所有核心数据类型，比如：

- Origin

- Block Number

- AccountId

- Hash

- Header

- Version

- etc...

还有一些系统关键的存储项，比如：

- Account Nonce

- Block Hash

- Block Number

- Events

- etc...

最后他还定义了一些底层函数，用于访问你的区块链存储，验证交易的签名等。

### Executive 模块

Frame Executive模块充当runtime的调度层。他将传入的外部调用分发到对应的pallet.

### 支持库(Support Library)

FRAME支持库由一组Rust宏，类型，traits以及用于简化Substrate pallet开发的函数组成。

支持宏可在编译时展开生成在运行时调用的代码，使用宏可减少pallet中最常见组件的样板代码，

### 运行时(Runtime)

运行时库把所有这些组件和pallet组合在一起。它定义了在运行时中包含哪些pallet，并把他们配置在一起来构成最终的运行时。当对运行时调用时，它使用Executive模块来分发这些调用到各自的pallet中。

### Benchmarking

用于基准测试FRAME runtime的宏。

## 一些预编译的Pallet

有一些特别通用的pallet，他们可在许多区块链中复用。任何人都可以自由的编写并分享有用的pallet。Substrate提供了一些比较流行的pallet，让我们来探索他们。

### Asset

Asset pallet是一个简单安全、用于处理可替代资产模块。

### Atomic Swap

原子交换是一个用于将资金原子从发送方origin发送到目标方target的模块。使用了一个证明去允许target批准(approve/claim)交换(swap).如果未在指定的时间内批准，发送方可以取消它。

### Aura

Aura pallet通过管理线下报告（offline reporting）实现Aura共识。

### Authority Discovery

Authority Discovery pallet在`core/authority-discovery`用来检索当前的生成者集合，获得他自己的生成者ID，以及对来自其他生成者的消息进行签名和验证。

### Authorship

Authorship pallet用于追踪区块当前的生成者以及最近的区块

### BABE

BABE pallet实现BABE共识算法，BABE通过收集从VRF的输出链上随机数并且管理epoch的交易。

> BABE（Blind Assignment for Blockchain Extension）是一种区块链共识算法，最初由Gavin Wood在Polkadot白皮书中提出。它是基于概率的区块生成算法，采用随机性轮换的方式来选择下一个参与出块的验证人（validator），并使用VRF（Verifiable Random Function）来确保选择过程的公正性。
> 
> 在BABE算法中，每个验证人都拥有一个VRF密钥对，用于生成和验证随机数，并根据这些随机数进行轮流出块。每个时间段内，将会选出一个验证人作为领袖节点（leader），负责在该时间段内出块。领袖节点可以根据先前生成的VRF随机数来确定自己在当前时间段中是否是应当出块的验证人。
> 
> 如果领袖节点确定当前该由自己出块，则它将执行打包交易和生成区块的操作，并将新生成的区块广播到网络中。其他验证人也可以通过验证领袖节点生成的VRF证明来确认其所属时间段和出块权利是否合法。由于BABE算法采用了随机性轮换和VRF证明等多重保护措施，因此能够有效地防止恶意攻击和双重花费等问题，从而保证区块链网络的安全性和可靠性。

### Balances

Balances pallet提供了管理账户和余额的功能。

### Benchmark

一个以隔离的方式包含常见的运行时模式的pallett.此pallet不用于生产环境，仅用于基准测试。

### Collective

Collective pallet允许一组账户IDs通过分发来自特定来源的调用使他们感觉像是一个集合体。

### Contracts

Contracts pallet为运行时提供了部署和执行WASM智能合约的功能。

### Democracy

Democracy pallet提供了一个可用于处理通用的利益相关者投票的民主制度。

### Elections Phragmén

Elections Phragmén pallet是一个基于[sequential Phragmén](https://wiki.polkadot.network/docs/en/learn-phragmen)的选举模块

### Elections

Elections pallet是一个选举模块，基于质押权重的成员选举。

### EVM

EVM pallet是Substrate上的以太坊虚拟机执行模块。

### Example Offchain Worker

链下工作机实例：一个简单的pallet，展示了大多数链下工作机共有的概念，API和结构。

### GRANDPA

GRANDPA pallet通过管理针对本地代码管理
