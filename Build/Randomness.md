# Randomness

由于区块链需要确定性的结果——相同的输入总是产生相同的输出——找到一个合适的来源，以产生看似随机的结果提出了一个独特的挑战。 但是，在许多应用中，随机性对于执行某些操作至关重要。 例如，你希望选择负责生成块的验证人包含随机性，以防止验证人选择可以被预测，从而容易被攻击。 你还可以在统计或科学分析、加密操作或涉及游戏或赌博的应用程序中使用随机性。

## 确定性随机性

在传统的计算机应用中，人们所说的随机数实际上是使用**伪随机性**生成的。 伪随机性依赖于足够随机的种子（由用户或外部来源提供）和操纵种子以生成一系列看似随机的数字的算法。 结果是一个伪随机数，由于用于生成它的算法，它很难预测，但具有确定性，因为相同的种子输入总是产生相同的输出序列。

然而，在区块链上运行的应用程序受到更严格的限制，因为网络中的所有验证节点都必须就任何链上值达成一致，包括注入的任何随机性数据。 由于此约束，你不能直接在区块链应用程序中使用真正的随机性。

对于区块链应用，提供随机性的最常见方法是[可验证随机函数](https://en.wikipedia.org/wiki/Verifiable_random_function). 可验证随机函数 （VRF） 是一种数学运算，它接受输入并生成随机数和真实性证明，证明该随机数是由提交者生成的。 任何挑战者都可以验证证明，以确保随机数生成有效。

在 Polkadot 生态系统和基于Substrate的链中，VRF是 BABE pallet提供的共识机制的一部分。 有关VRF与共识之间关系的更多信息，请参阅[共识](https://docs.substrate.io/learn/consensus/)

## 生成和使用随机性

Substrate提供了[`Randomness`](https://paritytech.github.io/substrate/master/frame_support/traits/trait.Randomness.html) trait.它定义了**生成随机性**的逻辑和**使用随机性的**逻辑之间的接口。 这个trait允许你编写用于生成随机性和使用随机性的逻辑，彼此独立。

### 生成随机性

你可以通过许多不同的方式实现`Randomness` trait，具体取决于应用程序所需的安全保证和性能权衡。 Substrate包括两个如何在pallets中实现`Randomness`trait的示例，这些示例在性能、复杂性和安全性之间提供了不同的权衡。

- [insecure randomness](https://paritytech.github.io/substrate/master/pallet_insecure_randomness_collective_flip/index.html)Pallet 提供了一个`random`函数，该函数根据前 81 个区块的区块哈希生成伪随机值。
  
  这种类型的随机性表现良好，但并不安全。 你只应在安全要求较低的应用中或测试使用随机性的应用时使用此pallet。 你不应在生产环境中使用此pallet。

- [BABE pallet](https://paritytech.github.io/substrate/master/pallet_babe/index.html)通过使用VRF提供随机性。

此pallet提供生产级的随机性，Polkadot中就是用了它。 如果你选择此pallet作为随机性源，则区块链必须使用BABE基于插槽的共识，用于生成区块。

### 使用随机性

`Randomness` trait提供了以下使用随机性的方法：

- `random_seed`方法不带任何参数并返回原始随机值。 如果在块中多次调用此方法，则每次都返回相同的值。 因此，在大多数情况下，不应直接使用此方法。
- `random`方法接收一个字节数组作为上下文标识符，并返回一个结果，该结果对于此上下文是唯一的，并且与基础随机性源允许的其他上下文无关。

需要随机值的pallets不需要提供随机性源，但它们确实需要指定实现`Randomness`trait的随机性源。

### 安全保障

记住，`Randomness`trait为在运行时中定义随机性源提供了一个方便的抽象，但trait并不提供任何安全保证。 作为运行时开发人员，你需要确保你使用的随机性源满足使用其随机性的所有*pallets*的安全要求。

## 下一步去哪里

- [How-to: Randomness](https://docs.substrate.io/reference/how-to-guides/pallet-design/incorporate-randomness/)
- [`Randomness`](https://paritytech.github.io/substrate/master/frame_support/traits/trait.Randomness.html)
