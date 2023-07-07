# 你可以构建什么

在设计应用程序时，首先需要做出的决策之一是要使用的方法。 例如，你需要确定你的项目是否最适合以智能合约、单个pallet、自定义运行时或平行链的形式交付。 关于构建什么的决定将影响你要做出的几乎所有其他决策。 为了帮助你初步确定要构建的内容，本节重点介绍可用的选项、它们之间的差异以及你可能选择这一种方法而不是另一种方法的原因。

## 智能合约

许多开发者熟悉智能合约，就自然倾向于认为他们的项目非常适合智能合约模型。 但是，在确定智能合约方法是否适合你的项目时，需要考虑优点和缺点。

### 智能合约必须遵守区块链规则

智能合约是部署在特定链上并运行在指定定链地址上的指令集合。 由于智能合约在它们无法控制的底层区块链上运行，因此它们必须遵守底层链施加的任何规则或限制。 例如，底层区块链可能会限制对存储的访问或阻止某些类型的交易。

此外，接受智能合约的区块链通常将代码视为来自不受信任的来源 - 可能是恶意行为者或缺乏经验的开发人员。 为了防止不受信任的代码破坏区块链操作，底层区块链实施原生保护措施来限制恶意或有缺陷的智能合约可以做的事情。 例如，底层链可能会收取费用或强制计量，以确保合约开发人员或用户为合约使用的计算和存储付费。 合约执行的费用和规则由底层链自行决定。

### 智能合约和状态

你可以将智能合约视为在沙盒环境中执行。 他们不会直接修改底层区块链存储或其他合约的存储。 一般来说，智能合约只修改自己的状态，不会调用其他合约或运行时函数。 运行智能合约通常会有一些额外的开销，以确保底层区块链可以还原交易，以防止在合约中的错误导致执行失败时更新状态。

### 使用智能合约的场景

尽管智能合约存在限制，但在某些情况下，你的项目可能会从使用智能合约中受益。 例如，智能合约的进入门槛较低，通常可以在短时间内构建和部署。 缩短的开发时间可能会让你在确定产品到市场的契合度和快速迭代方面具有优势。

同样，如果你熟悉使用Solidity等语言构建智能合约，则可以缩短项目的学习曲线和上线时间。 由于智能合约遵循其部署的链的功能，因此你可以更专注于实现合约的应用程序逻辑，而不必担心区块链基础设施或经济性。

如果你计划构建平行链，还可以使用智能合约以不影响底层网络的隔离方式对功能或服务进行原型设计，然后再投入更全面的解决方案。 如果你是运行时开发人员，则可以合并合约，以允许社区扩展和开发运行时的功能，而无需授予他们访问底层运行时逻辑的权限。 你还可以使用智能合约来测试未来的运行时更改。

通常，在决定是否使用智能合约构建项目时，应考虑以下特征：

- 它们本质上对网络更安全，因为保护措施内置于底层链中，但你无法控制这些保护措施施加的任何限制、限制或计算开销。
- 底层链提供了防止滥用的内在经济激励，但收费和计量系统也由底层链定义。
- 它们在代码复杂性和部署时间方面有更低的准入门槛。
- 它们可以为原型设计、测试和社区参与提供一个隔离的环境。
- 它们的部署和维护开销较低，因为你利用了现有网络

以下示例说明了智能合约的用例：

- 将衍生品添加到现有的去中心化交易所（DEX）。
- 实施自定义交易算法。
- 定义特定方之间的合同逻辑。
- 在将应用程序转换为平行链之前对其进行原型设计和测试。
- 在现有链上引入 layer-2 代币和自定义资产。

### 智能合约支持

Polkadot中继链不支持智能合约。 但是，连接到 Polkadot 的平行链可以支持任意状态转换，因此任何平行链都可以成为智能合约部署的潜在平台。 例如，当前的 Polkadot 生态系统中有几个平行链支持不同类型的智能合约部署。 如果你计划为 Polkadot 生态系统开发智能合约，必须首先决定要构建的智能合约类型，并确定支持该类型智能合约的平行链。 Substrate提供了支持两种类型智能合约的工具：

- FRAME 库中的`contracts` pallet使基于Substrate的链能够执行编译为WebAssembly的智能合约，而不管用于编写智能合约的语言如何。
- Frontier项目中的`evm` pallet使基于Substrate的链能够运行用Solidity编写的以太坊虚拟机（EVM）合约。

### 探索智能合约

如果你的项目看起来非常适合智能合约，你可以在以下教程中看到一些简单的示例：

- [开发智能合约](https://docs.substrate.io/tutorials/smart-contracts/)
- [访问 EVM 帐户](https://docs.substrate.io/tutorials/integrate-with-tools/access-evm-accounts/)

## 单个pallets

在某些情况下，你可能希望将应用程序逻辑实现为独立的pallet，并将功能作为库提供给社区，而不是构建自己的自定义运行时。 例如，如果您不想部署和管理特定于应用程序的区块链，则可以构建一个或多个单独的pallets，以提供在所有基于Substrate的链中广泛有用的功能，改进现有功能或定义Polkadot生态系统的标准。 使用FRAME很容易开发单个pallets，并且易于Substrate链集成。

### 编写正确的代码

值得注意的是，pallets本身并不能提供智能合约提供的任何类型的保护或保障。 使用pallets，你可以控制可供运行时开发者实现的逻辑。 你可以提供模块所需的方法、存储项、事件和错误。 pallets本身并不引入费用或计量系统。 这取决于你确保你的pallet逻辑不允许不良行为或远离pallet易受攻击的网络。缺乏内置保护措施意味着你要负责编写避免错误的代码。

### 运行时开发之外的pallets

通常，编写pallet是运行时开发的入门，让你有机会试验现有的pallets和编码模式，而无需构建完整的区块链应用程序。 单个pallets还提供了一种替代方法，你无需编写自己的应用程序即可为项目做出贡献。

尽管编写和测试pallets通常是构建更大规模应用程序的垫脚石，但有许多示例说明了单个pallets对整个生态的价值。

即使你正在构建单个托盘，也需要在运行时上下文中对其进行测试。 开发单个托盘的主要缺点是你无法控制你使用的运行时的任何其他部分。 如果你将pallet视为孤立的代码，则可能会错过增强或改进它的机会。 此外，如果你不更新代码以与这些更改保持同步，则对 FRAME 或 Substrate 的更改可能会给你的单个pallets带来维护问题。

### 探索构建pallets

如果你的项目看起来非常适合作为单个pallet，你可以在以下部分中查看一些简单的示例来帮助开始：

- [定制pallets](https://docs.substrate.io/build/custom-pallets/)
- [构建应用逻辑](https://docs.substrate.io/tutorials/build-application-logic/)
- [收藏品workshop](https://docs.substrate.io/tutorials/collectibles-workshop/)

## 自定义运行时

在大多数情况下，决定构建自定义运行时是构建和部署特定于应用程序的并行区块链（平行链）作为 Polkadot 生态系统的一部分的关键步骤。 通过使用 Substrate 和 FRAME 进行构建，你可以开发完全自定义的运行时。 使用自定义运行时，你可以完全控制应用的所有方面，包括经济激励、治理、共识和资源管理。

有些pallets为其中许多功能提供可插拔模块。 但是，需要你自己决定使用哪些模块、如何根据需要修改它们以及需要自定义模块的位置。 由于你控制网络中每个节点运行的所有底层逻辑，因此在编码技能和经验方面的有更高的准入门槛，高于编写智能合约或单个pallet。

与单个pallets一样，自定义运行时不提供任何内置保护措施来防止不良行为者或不正确的代码造成伤害。 你可以正确评估资源消耗以及如何在运行时逻辑中应用交易费，以充分保护网络和用户社区。

与智能合约或单个pallet不同，自定义运行时是一个功能齐全的区块链。 使自定义运行时可供其他人访问且安全使用涉及获取物理或云计算资源、构建可在服务中发现价值的社区以及管理网络基础结构。

使用智能合约，你的应用在现有执行模型之上运行，从而限制了你的应用可以执行的操作。 使用自定义运行时，你可以控制该基础执行模型，并可以选择对其进行扩展以支持其他开发人员的智能合约执行。 使用自定义运行时，你还可以提供比智能合约或单个pallets提供的更复杂的功能和用户交互。

### 探索构建自定义运行时

如果要构建更完整的自定义运行时而不是单个pallet，可以从一个简单的示例开始，例如[收藏品工作坊](https://docs.substrate.io/tutorials/collectibles-workshop/). 但是，如果你想构建一个自定义的运行时作为一个单链或平行链链的概念验证，你需要对运行时组件和 FRAME pallets有更广泛和更深入的理解。 最相关的主题在[Build](https://docs.substrate.io/build/)和[Test](https://docs.substrate.io/test/)以及以下各节：

- [运行时存储结构](https://docs.substrate.io/build/runtime-storage/)
- [交易、权重和费用](https://docs.substrate.io/build/tx-weights-fees/)
- [应用开发](https://docs.substrate.io/build/application-development/)
- [FRAME pallets](https://docs.substrate.io/reference/frame-pallets/)
- [FRAME macros](https://docs.substrate.io/reference/frame-macros/)

## 平行链

自定义运行时可以作为独立链或私有网络的业务逻辑独立存在，但如果希望项目成为可行的生产链，则为应用程序部署业务逻辑和状态转换功能作为平行链或平行线程有几个优点。

平行链和平行线程充当独立的 Layer-1 区块链。 每个平行链都有自己的逻辑，并与其生态中的其他链并行运行。 生态中的所有链都受益于网络的共享安全性、治理、可扩展性和互操作性。

### 平行链提供最大程度的灵活性

将项目开发为平行链，在链的设计和功能方面你拥有很大的自由度和灵活性。 决定构建什么完全取决于自己。 例如，你可以定义要存储在链上或链下的数据。 你可以定义自己的经济结构、交易要求、费用策略、治理模型、资金账户和访问控制规则。 平行链可以根据你的决定，每笔交易的开销可以或多或少，并且你的平行链可以随着时间的推移而升级和优化。 唯一的要求是平行链或平行线程必须与 Polkadot API 兼容。

### 规划平行链资源需求

作为平行链，你的项目可以以比私有链或独立链更安全的方式为更广泛的社区提供功能。 但是，如果要构建准备上线的平行链，则应牢记以下附加要求：

- 你需要一个具有足够技能和经验的开发团队，无论是 Rust 编程还是 UX 设计背景。
  
  平行链开发可能需要比其他选项更多的资源。

- 需要通过营销、外展或生态开发计划来建立你的社区。

- 需要资源来维护基础设施和网络。
  
  平行链是一个完整的区块链。 尽管中继链为你的项目提供了安全性和共识，但你必须维护自己的链和网络基础设施。 除了DevOps之外，你还需要保护平行链插槽，设计众贷或拍卖策略，并积累足够的资源来扩展插槽。

- 你需要足够的时间在沙盒或模拟网络以及功能齐全的测试网络上，测试和验证链操作

### 平行链用例

一般来说，如果需要复杂的操作，你应该将项目构建为平行链，因为平行链提供了更快、更高效的交易执行。 例如，构建平行链可能是以下用例的最佳选择：

- 去中心化金融 （DeFi） 应用
- 数字钱包
- 物联网 （IOT） 应用
- 游戏应用
- Web 3.0 基础设施

### 探索构建平行链

如果你有一个自定义运行时，想要将其部署为平行链，以利用中继链和 Polkadot 或 Kusama 生态的安全性、治理和互操作性，则可以首先在本地构建并设置自己的测试网络进行初始测试。

有关入门的一些示例，请参阅以下部分：

- [将平行链连接到网络](https://docs.substrate.io/tutorials/build-a-parachain/)
- [在测试网络中模拟平行链](https://docs.substrate.io/test/simulate-parachains/)
- [平行链](https://docs.substrate.io/reference/how-to-guides/parachains/)

若要详细了解可以构建的内容，请浏览以下资源：

- [使用 Polkadot 构建](https://wiki.polkadot.network/docs/build-build-with-polkadot)
- [平行链开发](https://wiki.polkadot.network/docs/build-pdk)
- [智能合约](https://wiki.polkadot.network/docs/build-smart-contracts)