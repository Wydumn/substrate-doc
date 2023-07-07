# 运行时原语

Substrate 运行时由一组原语类型组成，Substrate框架的其余部分需要这些原语类型。

## [核心原语](https://learnblockchain.cn/docs/substrate/docs/knowledgebase/runtime/primitives/#core-primitives)

Substrate 框架对运行时必须向Substrate的其他层提供的内容做了最小化的假设。它们必须定义并且实现一个特定的接口，以便在Substrate框架中工作。

它们是：

- `Hash`：对某些数据的加密摘要进行编码的类型。通常只有256位。

- `DigestItem`：该类型必须能够编码许多与共识和变更跟踪相关的“硬连接”备选方案中的一种，以及与运行时中的特定模块相关的任意数量的“软编码”变体

- `Digest`：一系列的DigestItems。它编码了轻客户端在区块内需要掌握的所有相关信息。

- `Extrinsic`：表示区块链所识别的单个外部数据的类型。这通常涉及一个或多个签名，以及某种编码指令（例如，用于转移资金所有权或调用智能合约）。

- `Header`：该类型代表着区块相关的所有信息（无论是加密还是其他方式）。它包括父哈希、存储根和extrinsics trie根、摘要以及块号。

- `Block`：本质上是`Header`和一系列`Extrinsic`s的组合，以及要使用的哈希算法的规范。

- `BlockNumber`：对任何有效块具有的祖先总数编码的类型。通常是32 位

## [FRAME原语](https://learnblockchain.cn/docs/substrate/docs/knowledgebase/runtime/primitives/#frame-primitives)

使用Substrate构建的运行时中 ，还有一组额外的原语：

- `Call`：可以通过外部交易(extrinsic)调用的调度类型。

- `Origin`：表示call来自哪里。例如，签名消息（交易）、 未签名的消息（an inherent extrinsic）和运行时本身的调用（root call）。

- `Index`：帐户索引（又名nonce随机数）类型。这存储了与发件人帐户关联的以前的交易数量。

- `Hashing`：运行时中使用的哈希系统（算法）（例如 Blake2）。

- `AccountId`：用于在运行时中标识用户帐户的类型。

- `Event`：用于运行时发出的事件的类型。

- `Version`：表示运行时版本的类型。

## [后续步骤](https://learnblockchain.cn/docs/substrate/docs/knowledgebase/runtime/primitives/#next-steps)

### [了解更多信息](https://learnblockchain.cn/docs/substrate/docs/knowledgebase/runtime/primitives/#learn-more)

- 了解[Substrate FRAME](https://learnblockchain.cn/docs/substrate/docs/knowledgebase/runtime/primitives/frame)。

### [例子](https://learnblockchain.cn/docs/substrate/docs/knowledgebase/runtime/primitives/#examples)

- 了解如何[在Substrate node中](https://github.com/paritytech/substrate/blob/master/bin/node/runtime/src/lib.rs)实现这些泛型类型。

### [引用](https://learnblockchain.cn/docs/substrate/docs/knowledgebase/runtime/primitives/#references)

- 查看[`node-primitives`中定义的原语类型](https://substrate.dev/rustdocs/v2.0.0/node_primitives/index.html)。

- 查看 [`sp-runtime`中定义的`traits`](https://substrate.dev/rustdocs/v2.0.0/sp_runtime/traits/index.html)
