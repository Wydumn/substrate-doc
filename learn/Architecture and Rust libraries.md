# 架构和Rust库

正如在“区块链基础”中所指出的那样，区块链依赖于一个分布式计算机网络进行通信。

由于节点是任何区块链的核心组件，因此了解什么使 Substrate 节点独特以及默认提供的核心服务和库以及如何自定义和扩展节点以适配不同的项目目标非常重要。

## 客户端和运行时

Substrate 节点由以下两个主要部分组成：

- 包含**外部节点服务**的**核心客户端**，负责网络活动，如peer discovery、处理交易请求、与peers达成共识和响应 RPC 调用
- 包含用于执行区块链状态转换功能的所有业务逻辑的**运行时**

下图以简化形式说明了这种职责分离，帮助理解架构和 Substrate 如何提供一种模块化的框架来构建区块链。

![Substrate architecture](https://docs.substrate.io/static/ba5a48a1993a5eddabf1e91c3eb9974f/aec80/simplified-architecture.png)

## 客户端外部节点服务

核心客户端包含几个外部节点服务，负责处理运行时之外的活动。例如，核心客户端中的外部节点服务处理peer discovery、管理交易池、与其他节点通信以达成共识，并响应来自外部世界的RPC请求。

一些最重要的由核心客户端服务处理的活动涉及以下组件：

- [Storage](https://docs.substrate.io/learn/state-transitions-and-storage/) 存储：外部节点使用简单且高效的键值存储层持久化Substrate区块链状态的演变状态
- [Peer-to-peer networking](https://docs.substrate.io/learn/node-and-network-types/) 点对点网络：外部节点使用libp2p网络堆栈的Rust实现与其他网络参与者进行通信
- [Consensus](https://docs.substrate.io/learn/consensus/) 共识：外部节点与其他网络参与者进行通信，确保他们对区块链的状态达成一致
- [Remote procedure call (RPC) API](https://docs.substrate.io/build/remote-procedure-calls/) 远程过程调用（RPC）API：外部节点接受入站HTTP和WebSocket请求，允许区块链用户与网络交互
- [Telemetry](https://docs.substrate.io/maintain/monitor/) 遥测：外部节点通过嵌入的[Prometheus](https://prometheus.io/)服务器收集并提供节点指标的访问 
- [Execution environment](https://docs.substrate.io/build/build-process/) 执行环境：外部节点负责选择运行时要使用的执行环境——WebAssembly或本机Rust，然后将调用分派到所选的运行时

Substrate通过核心区块链组件提供了处理这些活动的默认实现。原则上，你可以用自己的代码修改或替换任何组件的默认实现。实际上，一个应用程序很少需要对任何底层区块链功能进行更改，但Substrate允许你进行更改，以便在你认为合适的地方自由创新。

执行这些任务通常需要客户端节点服务与运行时进行通信。这种通信是通过调用专门的[runtime APIs](https://docs.substrate.io/reference/runtime-apis/)来处理的。

## 运行时

运行时确定交易是否有效，并负责处理区块链状态的更改。来自外部的请求通过客户端进入运行时，运行时负责状态转换函数并存储结果状态。

因为运行时执行接收到的函数，所以它控制着交易如何打包入块，以及如何将区块返回给外部节点进行传播或导入到其他节点。实质上，运行时负责处理发生在链上的所有事情。它也是构建 Substrate 区块链的核心组件。

Substrate 运行时被设计为编译成[WebAssembly (Wasm)](https://docs.substrate.io/reference/glossary/#webassembly-wasm) 字节码。这个设计决策使得：

- 支持无分叉升级
- 多平台兼容
- 运行时有效性检查
- 中继链共识机制的验证证明

类似于外部节点有一种方式向运行时提供信息，运行时使用专门的[主机函数](https://paritytech.github.io/substrate/master/sp_io/index.html)与外部节点或外部世界进行通信。

## 核心库

为了让节点模板保持简单，区块链的许多方面都配置了默认实现。例如，网络层、数据库和共识机制都有默认实现，你可以直接使用它们来运行区块链，无需进行大量的定制。但同时，支撑基本架构的库为自定义区块链组件提供了很大的灵活性。

与节点由提供不同服务的两个主要部分(核心客户机和运行时)组成非常相似，底层库被划分为三个主要职责领域：

- 提供外部节点服务的核心客户端库
- 运行时的 FRAME库
- 用于基础函数的原始库和用于库之间通信的接口

下图说明了这些库如何反映核心客户机外部节点和运行时职责，以及原语库如何提供两者之间的通信层。

![Core node libraries for the outer node and runtime](https://docs.substrate.io/static/dae77f7ece855ad265b5c93651f4881b/c337e/libraries.png)

### 核心客户端库

使substrate节点能够处理其网络职责（包括共识和块执行）的库其实就是Rust crates，使用`sc_` 作为crate名字的前缀。例如，`sc_service` 库负责构建Substrate区块链的网络层，管理网络参与者和交易池之间的通信。

### 用于运行时的FRAME库

使你能够构建运行时逻辑并对传入和传出运行时的信息进行编码和解码的库是使用`frame_`前缀的Rust crate。

`frame_*`库为运行时提供基础设施。例如，`frame_system` 库提供了一组基本函数，用于与其他Substrate组件交互，而`frame_support` 使你能够声明运行时存储项、错误和事件。

除了由`frame_*` 库提供的基础设施外，运行时可以包括一个或多个`pallet_*`库。每个使用`pallet_` 前缀的Rust crate表示一个单独的FRAME模块。在大多数情况下，使用`pallet_*` 库来组装要合并到区块链中以满足项目的功能。

你可以使用**primitives**库构建Substrate运行时，而无需使用`frame_*`或`pallet_*`库。但是`frame_*`或`pallet_*`库提供了最有效的组合Substrate运行时的方法。

### Primitive库

在 Substrate 架构的最底层，有primitive库可以控制底层操作并允许核心客户端服务和运行时进行通信。primitive库是以 `sp_` 前缀的Rust crate.

primitive 库提供最低层次的抽象，以公开接口，核心客户端或运行时可以使用这些接口来执行操作或相互交互。

例如:

-  [`sp_arithmetic`](https://paritytech.github.io/substrate/master/sp_arithmetic/index.html) 库定义了运行时使用的定点运算priimtive和类型
-  [`sp_core`](https://paritytech.github.io/substrate/master/sp_core/index.html) 库提供了一组共享的 Substrate 类型
- [`sp_std`](https://paritytech.github.io/substrate/master/sp_std/index.html) 库导出了 Rust 标准库的primitive，使得任何依赖于运行时的代码都可以使用它们

## 模块化架构

将核心 Substrate 库分离提供了一种灵活而模块化的架构，用于编写区块链逻辑。primitive库提供了一个基础，核心客户端和运行时可以在此之上构建，而无需直接相互通信。primitive类型和traits在它们自己的单独 crate 中公开，因此它们可供外部节点服务和运行时组件使用，而不会引入循环依赖问题。

## Where to go next

既然您已经熟悉了用于构建和交互底层节点的体系结构和库，那么你可能希望更深入地研究这些库。要了解更多关于任何库的技术细节，你应该查看该库的  [Rust API](https://paritytech.github.io/substrate/master/) 文档。
