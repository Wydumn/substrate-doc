# 自定义pallet

构建自定义运行时最常见方法是从[现有pallets](https://docs.substrate.io/reference/frame-pallets/)开始. 例如，你可以开始构建特定于应用的staking pallet，该pallet使用现有Collective和Balances pallets中暴露的类型，但包括你的应用及其质押规则所需的自定义运行时逻辑。

虽然[FRAME pallets](https://docs.substrate.io/reference/frame-pallets/)提供最常见pallets的概述，查找有关现有pallets最新信息的来源是[Rust API](https://docs.substrate.io/reference/rust-api/)文档，他们都以`pallet_*`命名。

如果找不到满足需求的pallet，可以使用 FRAME 宏为自定义pallet构建脚手架。

## Pallet宏和属性

FRAME 广泛使用 Rust 宏来封装复杂的代码块。 用于构建自定义pllet的最重要宏是[`pallet`](https://paritytech.github.io/substrate/master/frame_support/attr.pallet.html)宏。 `pallet`宏定义了一个pallet必须提供的核心属性。 例如：

- `#[pallet::pallet]`是必需的pallet属性，可用于定义pallet的结构（struct），以便它可以存储方便检索的数据
- `#[pallet::config]`是必需的pallet属性，可用于定义pallet的配置trait

`pallet`宏还定义了pallets通常提供的核心属性。 例如：

- `#[pallet::call]`是使你能够为pallet实现可调度函数调用的属性
- `#[pallet::error]`是生成可调度错误的属性
- `#[pallet::event]`是生成可调度事件的属性
- `#[pallet::storage]`是使你能够在运行时及其元数据中生成存储实例的属性

这些核心属性与你在编写自定义pallet时需要做出的决策一致。 例如，你需要考虑：

- 存储。你的pallet存储哪些数据？数据存储在链上还是链下？
- 功能。你的pallet公开了哪些可调用的功能？
- 事务性。你的函数调用是否旨在以原子方式修改存储？
- Hooks。你的pallet会调用任何运行时hooks吗？

宏简化了实现自定义运行时逻辑所需的代码。 但是，某些宏对函数声明强制实施特定要求。 例如，`Config` trait必须受 `frame_system::Config`约束，并且`#[pallet::pallet]`结构必须声明为`pub struct Pallet<T>(_)` 。 有关 FRAME pallets中使用的宏的概述，请参阅[FRAME macros](https://docs.substrate.io/reference/frame-macros/).
