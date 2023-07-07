# Rust for Substrate

在创建关键任务软件方面，Substrate能成为一个灵活可扩展的框架，很大部分原因归功于[Rust](https://www.rust-lang.org/). 作为 Substrate 的首选语言，Rust 是一种高性能的编程语言，也是首选语言的原因如下：

- Rust 速度很快：它在编译时是静态类型的，使编译器可以优化代码以提高速度，并使开发人员可以针对特定的编译目标进行优化。
- Rust 是可移植的：它被设计为在嵌入式设备上运行，支持任何类型的操作系统。
- Rust 是内存安全的：它没有垃圾收集器，它会检查你使用的每个变量和引用的每个内存地址，以避免任何内存泄漏。
- Rust 是 Wasm优先：它具有编译到 WebAssembly 的一等支持。

## Rust in Substrate

在[架构](https://docs.substrate.io/learn/rust-basics/)部分，你将了解到 Substrate 由两个不同的架构组件组成：外部节点和运行时。 虽然 Rust 中更复杂的功能（如多线程和异步 Rust）在外部节点代码中使用，但它们不会直接暴露给运行时工程师，这使得运行时工程师更容易专注于节点的业务逻辑。

通常，根据他们的重点，开发人员应当了解以下内容：

- 基本[Rust idioms](https://rust-unofficial.github.io/patterns/idioms/index.html),[使用`no_std`](https://docs.rust-embedded.org/book/intro/no-std.html)以及使用哪些宏以及为什么（用于运行时工程）。
- [异步Rust](https://rust-lang.github.io/async-book/01_getting_started/01_chapter.html)（适用于使用外节点（客户端）代码的更高级开发人员）。

在深入研究 Substrate 之前，需要对Rust 有一个大致熟悉。有许多资源可用于学习 Rust，包括[Rust Language Programming Book](https://doc.rust-lang.org/book/)和[Rust by Example](https://doc.rust-lang.org/rust-by-example/)——本节的其余部分重点介绍了 Substrate 如何使用 Rust 的一些核心功能来帮助开发人员开始进行运行时工程。

## 编译目标

在构建 Substrate 节点时，我们使用编译目标`wasm32-unknown-unknown`，这意味着 Substrate 运行时工程师只能编写必须编译为 Wasm 的运行时。 这意味着你不能依赖一些典型的标准库类型和函数，并且必须仅对大多数运行时代码使用`no_std`兼容的 crate。 Substrate具有许多自己的原始类型和相关traits可以解决`no_std`需求。

## 宏

在学习如何使用和编写 FRAME pallets时，你会发现有许多宏可用作可重用代码来抽象常见任务或强制实施特定于运行时的要求。 这些宏允许你专注于编写惯用的 Rust 和特定于应用程序的逻辑，而不是与运行时交互所需的通用代码。

Rust 宏是一个强大的工具，可以帮助确保满足某些要求（无需重写代码），例如要以特定方式格式化的逻辑、进行特定检查或某些由特定数据结构组成的逻辑。 这对于帮助开发者编写可与 Substrate 运行时的复杂性集成的代码特别有用。 例如，所有 FRAME pallets都需要`#[frame_system::pallet]`宏，以帮助你正确实现某些必需的属性（如存储项或外部可调用函数），并使其与`construct_runtime`中的build过程兼容。

开发 Substrate 运行时涉及大量使用 Rust 的属性宏，这些宏有两种风格：派生属性和自定义属性。 当你开始使用 Substrate 时，不需要确切地知道它们是如何工作的，重要的是知道它们的存在，它们使你能够编写正确的运行时代码。

派生属性对于需要满足某些traits的自定义运行时类型非常有用，例如，在运行时执行期间使类型可由节点解码。

其他属性（如宏）在 Substrate 的代码库中也很常见：

- 指定代码段是仅编译到`no_std`库还是可以使用`std`库
- 构建自定义FRAME pallets.
- 指定运行时的build方式

## 泛型和配置traits

通常与 Java 等语言中的接口相比，Rust 中的 traits 提供了为类型赋予高级功能的方法。

如果你读过pallets，你可能已经注意到每个pallet都有一个`Config` trait，允许你定义pallet所依赖的类型和接口。

trait本身从`frame_system::pallet:;Config` trait继承了许多核心运行时类型，因此在编写运行时逻辑时可以轻松访问常见类型。 此外，在任何 FRAME pallets中，`Config` trait都是基于泛型`T`（下一节将详细介绍泛型）。 这些核心运行时类型的一些常见示例： `T::AccountId`，用于标识运行时中的用户帐户的通用类型，`T::BlockNumber`运行时使用的块号类型。

有关 Rust 中的泛型类型和traits的更多信息，请参阅the Rust book的[泛型类型](https://doc.rust-lang.org/book/ch10-01-syntax.html),[Traits](https://doc.rust-lang.org/book/ch10-02-traits.html)和[高级Traits](https://doc.rust-lang.org/book/ch19-03-advanced-traits.html)部分。

使用 Rust 泛型，Substrate 运行时开发人员可以编写与特定实现细节完全无关的pallets，从而充分利用 Substrate 的灵活性、可扩展性和模块化。

`Config` trait中的所有类型通常都使用 trait 边界进行定义，并在运行时中具体实现。 这不仅意味着你可以编写支持相同类型不同规范的pallets（例如，同时适用Substrate 和 Ethereum 链的地址），而且你还可以以最小的开销（例如将块号更改为`u32`）自定义泛型实现。

这使开发人员可以灵活地编写代码，而无需对你所做的核心区块链架构决策做出任何假设。

Substrate最大限度地利用泛型类型，以提供最大的灵活性。 你可以定义如何解析泛型类型以满足你的目的。

有关 Rust 中的泛型类型和traits的更多信息，请参阅the Rust book[泛型类型](https://doc.rust-lang.org/book/ch10-01-syntax.html)。

## 接下来

现在你已经了解了 Substrate 如何依赖于一些关键的 Rust 特性（如traits、泛型类型和宏），你可以浏览以下资源以了解更多信息。

- [Rust book](https://doc.rust-lang.org/book/)
- [Why Rust？](https://www.parity.io/blog/why-rust)（Parity的博客）
- [Cargo和 crates.io](https://doc.rust-lang.org/book/ch14-00-more-about-cargo.html)
- [为什么用Rust写智能合约？](https://paritytech.github.io/ink-docs/why-rust-for-smart-contracts)（ink!文档）
