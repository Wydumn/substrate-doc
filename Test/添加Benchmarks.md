# 添加Benchmarks

本指南说明了如何为pallet编写简单的基准（benchmarks）测试、测试benchmarks并运行[benchmarking](https://github.com/paritytech/substrate/tree/master/frame/benchmarking)命令，以生成pallet中函数所需执行时间的实际估计值。 本指南不介绍如何使用基准测试结果来更新交易权重。

## 为pallet添加benchmarking

1. 在编辑器中打开pallet中的[`Cargo.toml`](https://github.com/paritytech/substrate/blob/master/frame/examples/basic/Cargo.toml)。

2. 使用与pallet中其他依赖项相同的版本和分支，将`frame-benchmarking`crate添加到pallet的 [dependencies] 中。
   
   例如：
   
   ```toml
   frame-benchmarking = { version = "4.0.0-dev", default-features = false, git = "https://github.com/paritytech/substrate.git", branch = "polkadot-v0.9.25", optional = true }
   ```

3. 添加`runtime-benchmarks`pallet的 [features] 列表中。
   
   例如：
   
   ```toml
   [features]
   runtime-benchmarks = ["frame-benchmarking/runtime-benchmarks"]
   ```

4. 添加`frame-benchmarking/std`到pallet的`std`功能列表中。
   
   例如：
   
   ```toml
   std = [
    ...
    "frame-benchmarking/std",
    ...
   ]
   ```

## 添加benchmarking模块

1. 在pallet的`src`文件夹中创建新的文本文件（例如 ）`benchmarking.rs`。

2. 在编辑器中打开`benchmarking.rs`文件并创建一个 Rust 模块来定义pallet的benchmarks。
   
   你可以使用`benchmarking.rs`作为任何预构建pallet Rust 模块中包含的内容的示例。 通常，该模块应包含类似于以下内容的代码：
   
   ```rust
   #![cfg(feature = "runtime-benchmarks")]
   mod benchmarking;
   
   use crate::*;
   use frame_benchmarking::{benchmarks, whitelisted_caller};
   use frame_system::RawOrigin;
   
   benchmarks! {
    // Add individual benchmarks here
    benchmark_name {
       /* code to set the initial state */
    }: {
       /* code to test the function benchmarked */
    }
    verify {
       /* optional verification */
    }
   }
   ```

3. 编写单独的基准测试，以测试pallet中函数计算成本最高的路径。
   
   基准测试宏会为你包含在基准测试模块中的每个基准测试自动生成一个测试函数。 例如，宏创建类似于以下内容的测试函数：
   
   ```rust
   fn test_benchmarking_[benchmark_name]<T>::() -> Result<(), &'static str>
   ```
   
   基准测试模块[pallet-example-basic](https://github.com/paritytech/substrate/blob/master/frame/examples/basic/src/benchmarking.rs)提供了一些简单的示例基准测试。 例如：
   
   ```rust
   benchmarks! {
   set_dummy_benchmark {
     // Benchmark setup phase
     let b in 1 .. 1000;
   }: set_dummy(RawOrigin::Root, b.into()) // Execution phase
   verify {
         // Optional verification phase
     assert_eq!(Pallet::<T>::dummy(), Some(b.into()))
   }
   }  
   ```
   
   在此示例代码中：
   
   - 基准测试的名称是 `set_dummy_benchmark`.
   - 变量`b`存储用于测试`set_dummy`函数执行时间的输入。
   - `b`的值在 1 到 1,000 之间变化，因此你可以重复运行基准测试，以使用不同的输入值测量执行时间。

## 测试benchmarks

在pallet的基准测试模块中向`benchmarks!`宏添加基准测试后，你可以使用mock运行时进行单元测试，并确保基准测试的测试函数返回`Ok(()`。

1. 在编辑器中打开基`benchmarking.rs`。

2. 将`impl_benchmark_test_suite!`宏添加到基准测试模块的底部：
   
   ```rust
   impl_benchmark_test_suite!(
       MyPallet,
       crate::mock::new_test_ext(),
       crate::mock::Test,
   );
   ```
   
   `impl_benchmark_test_suite!`宏接收以下输入：
   
   - pallet生成的Pallet结构，本例中的`MyPallet`
   - 生成测试创世存储的函数 .`new_text_ext()`
   - 完整的模拟运行时结构 .`Test`
   
   此信息与用于为单元测试设置模拟运行时的信息相同。 如果所有基准测试都在模拟运行时测试环境中通过，则当你在实际运行时运行基准测试时，它们很可能会预期工作。

3. 在模拟运行时中通过运行类似如下命令，执行为pallet生成的基准单元测试
   
   ```bash
   cargo test --package pallet-mycustom --features runtime-benchmarks
   ```

4. 验证测试结果。

例如：

```text
running 4 tests
test mock::__construct_runtime_integrity_test::runtime_integrity_tests ... ok
test tests::it_works_for_default_value ... ok
test tests::correct_error_for_none_value ... ok
test benchmarking::bench_do_something ... ok

test result: ok. 4 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

## 运行时添加基准测试

向pallet添加基准测试后，还必须更新运行时以包括pallet和pallet的基准测试。

1. 在编辑器中打开runtime的`Cargo.toml`。

2. 将你的pallet添加到运行时的`[dependencies]`列表中：
   
   ```toml
   pallet-mycustom = { default-features = false, path = "../pallets/pallet-mycustom"}
   ```

3. 更新运行时的`[features]`以包含`runtime-benchmarks`：
   
   ```toml
   [features]
   runtime-benchmarks = [
       ...
       'pallet-mycustom/runtime-benchmarks'
       ...
   ]
   ```

4. 更新运行时的`std` features以包含pallet：
   
   ```toml
   std = [
       # -- snip --
       'pallet-mycustom/std'
   ]
   ```

5. 将pallet的configuration trait添加到运行时。

6. 将pallet添加到`construct_runtime!`宏中。
   
   如果你需要有关将pallet添加到runtime的更多详细信息，请参阅[将pallet添加到runtime](https://docs.substrate.io/tutorials/build-application-logic/add-a-pallet/)或[import pallet](https://docs.substrate.io/reference/how-to-guides/basics/import-a-pallet/).

7. 将pallet添加到`runtime-benchmarks`feature的`define_benchmark!`宏中。
   
   ```rust
   #[cfg(feature = "runtime-benchmarks")]
   mod benches {
     define_benchmarks!(
       [frame_benchmarking, BaselineBench::<Runtime>]
       [pallet_assets, Assets]
       [pallet_babe, Babe]
       ...
       [pallet_mycustom, MyPallet]
       ...
     );
   }
   ```

## 运行基准测试

更新运行时后，你就可以在启用`runtime-benchmarks`功能的情况下对其进行编译，并开始对pallet进行benchmarking分析。

1. 通过运行以下命令，使用启用的`runtime-benchmarks`功能打包项目：
   
   ```bash
   cargo build --package node-template --release --features runtime-benchmarks
   ```

2. 查看 node 子命令`benchmark pallet`的命令行选项：
   
   ```bash
   ./target/release/node-template benchmark pallet --help
   ```
   
   `benchmark pallet`子命令支持多个命令行选项，可帮助你自动执行基准测试。 例如，可以将`--steps`和`--repeat`命令行选项设置为使用不同的值多次执行函数调用。

3. 通过运行类似如下的命令开始对pallet进行基准测试：
   
   ```bash
   ./target/release/node-template benchmark pallet \
   --chain dev \
   --pallet pallet_mycustom \
   --extrinsic '*' \
   --steps 20 \
   --repeat 10 \
   --output pallets/pallet-mycustom/src/weights.rs
   ```
   
   此命令在指定目录中创建一个`weight.rs`文件。 有关如何配置pallet以使用这些权重的信息，请参阅[使用自定义权重](https://docs.substrate.io/reference/how-to-guides/weights/use-custom-weights/).

## 例子

你可以使用任何预构建pallet的`benchmarking.rs`和`weights.rs` 文件来了解有关对不同类型的函数进行基准测试的更多信息。

- [pallet示例：Benchamarks](https://github.com/paritytech/substrate/blob/master/frame/examples/basic/src/benchmarking.rs)
- [pallet示例：Weights](https://github.com/paritytech/substrate/blob/master/frame/examples/basic/src/weights.rs)
- [Balances pallet: Benchmarks](https://github.com/paritytech/substrate/blob/master/frame/balances/src/benchmarking.rs)
- [Balances pallet: Weights](https://github.com/paritytech/substrate/blob/master/frame/balances/src/weights.rs)
