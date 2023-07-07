reserve    一次性的

lock    

transfer

slash

#### ReservableCurrency

- reserve()

- unreserve()

在 Substrate 中，`ReservableCurrency` 是一个 trait，它定义了一种货币类型，该货币类型具有一些特殊的行为。具体来说，这个货币类型可以被“保留”，以便稍后使用。

这个 trait 包含了一些方法，其中最重要的是 `reserve()` 和 `unreserve()` 方法。这些方法分别用于将一定数量的货币保留起来或释放出来。当调用 `reserve()` 方法时，系统会检查当前余额是否足够，如果足够，则会将指定数量的货币标记为“已保留”。这些已保留的货币不能被其他账户或模块使用，直到它们被取消保留或者被使用。

`ReservableCurrency` 被广泛地应用在 Substrate 的经济模型中，例如在处理抵押、锁定交易费用等方面。

#### PalletId

`frame_support::PalletId`

PalletId 可以转换为一个账户，token mint的时候



### 链上升级

假设kitty数据结构需要调整

原来的数据结构

```rust
pub struct Kitty([u8; 16]);


// 新的结构
pub struct Kitty {
    pub dna: [u8; 16],
    pub name: [u8; 4],
}
```

之前已经创建的Kitty在链上都存储为旧格式，新的Kitty结构已经变了，怎么办？

pallet 的storage版本常量

```rust
const STORAGE_VERSION: StorageVersion = StorageVersion::new(1);
```

通过属性宏给pallet增加属性

```rust
#[pallet::pallet]
#[pallet::storage_version(STORAGE_VERSION)]
pub struct Pallet<T>(_);
```
