# Welcome to Substrate

Substrate是一个SDK，允许你构建应用于特定程序的区块链。这些区块链可以作为独立服务运行，也可以与其他链并行运行，并享受 Polkadot 生态系统提供的共享安全性。

## 简单易用，自由创新

With Substrate, you have complete creative control over the application you want to build. You can select predefined application logic from a large library of open source modules and templates to speed your development time. Not finding what you need in the current library? That’s not a problem—it’s an opportunity to build custom modules from reusable Rust macros and scaffolding code. If you’re feeling more adventurous or have a novel idea, you’re free to innovate on the blockchain design by using low-level primitives.

有了Substrate，你就可以完全掌控想要构建的应用。你可以从一个大型的开源模块和模板库中选用预定义的应用逻辑来加速开发时间。如果当前库中找不到您需要的内容，那也不是问题，这是一个构建可复用 Rust 宏和脚手架代码的自定义模块的好机会。如果有更新颖的想法，你可以使用底层结构创新区块链设计。



![Development time and complexity](https://docs.substrate.io/static/c9882d38950de8f51743890233f18ef6/3c520/development-complexity.png)

## 使用模板和模块进行构建

大多数项目从模板开始，以减少复杂性和开发时间，然后通过修改现有模块和添加新模块来进一步发展。这些模块、宏和库是 FRAME 开发环境的核心组成部分。FRAME 的主要目的是为构建定制的 Substrate 运行时提供模块化和灵活的组件集合。

FRAME 开发环境使您能够选择并配置你想在运行时中使用的特定模块——称为**pallets**。Pallets 为常用案例提供了可自定义的业务逻辑，如管理账户余额和对提案进行投票。Pallets 也为区块链操作（如质押和共识）提供了业务逻辑。

## 组合运行时

每个 Pallet 定义了特定类型、存储项和函数，以实现一组特定功能或功能集合的运行时。你选择并组合适合应用需求的 pallets 来组成一个自定义运行时。例如，如果你的应用需要管理账户余额，你可以在运行时逻辑配置中简单地包含 Balances Pallet。然后，可以修改你的自定义运行时中 Pallet 的配置，来适配应用程序。

在下图中，运行时由九个 Pallet 组成，以实现共识、包括块时间戳、管理资产和余额，并准备框架以管理汇集资金。



![Select pallets to compose the runtime](https://docs.substrate.io/static/64b2fcb61748ae77f4dd4c9ce63872b1/ee8ba/compose-runtime.png)

除了为运行时提供所需功能的 Pallets 之外，FRAME 还依赖于一些底层系统服务来构建和启用客户端外部节点服务与运行时进行交互。这些底层服务由以下必需模块提供：

- FRAME system crate frame_system 为运行时提供低级别类型、存储和函数
- FRAME support crate frame_support 是一组 Rust 宏、类型、traits和模块的集合，简化了 Substrate Pallets 的开发
- FRAME executive pallet frame_executive 协调运行时中各个pallets的传入函数调用的执行

有许多 Pallets 可供用作运行时的构建模块。可以在 Substrate 代码仓库或 Rust 文档中查看可用的 pallets 列表。有关最常见 pallets 的简要说明，请参见 FRAME pallets。

如果找不到提供所需功能的 pallet，则可以使用 FRAME 创建自己的自定义 pallet，然后将该自定义 pallet 添加到您的自定义运行时中。

## 使用自定义 Pallet 进行构建

FRAME开发环境的库使得构建自定义的pallet相对容易。通过自定义的pallet，你可以定义最适合应用需求的运行时行为。因为每个pallet都有自己离散的逻辑，您可以将现有的开源pallet与自定义pallet组合起来，以提供特定的功能或服务。例如，你可以在运行时中引入Balances pallet，以使用它的函数和存储项来管理帐户余额，同时添加一个自定义的pallet来在帐户余额发生变化时向服务发送通知。

## 为什么你应该用Substrate进行构建

Substrate是一个完全**模块化**和**灵活**的框架，它允许你通过选择和自定义最适合你项目的基础设施组件来组合链。例如，你可以更改网络栈、共识模型、交易格式或治理方法，以部署一个独特设计的区块链，也可以随着不断变化的需求而演化。

除了可组合和可适配之外，Substrate还被设计为**可升级**的。状态转换逻辑 - Substrate运行时 - 是一个自包含的WebAssembly对象，当需要引入新功能或更新现有功能时，可以修改它。由于运行时是一个自包含的对象，你可以在网络上引入运行时升级而不会影响服务或需要离线节点。在大多数情况下，节点操作新的运行时不需要任何操作，所以就可以随着时间的推移无缝地演进你的网络协议，以满足用户的需求。

Substrate也是一个**开源**项目，所有的Substrate库和工具都可在开源许可下获得。此外，Substrate框架的核心组件使用开放协议，如`libp2p`和`jsonRPC`，同时赋予自定义区块链体系结构的程度。Substrate还拥有一个庞大、活跃和乐于助人的构建者社区，为生态系统做出贡献。社区的贡献增强了可用于自己的区块链中的功能。Substrate支持**跨共识消息（XCM）**，以便不同的系统之间相互传递消息。
