## 合约结构

每个ink!合约包含：

- 只有一个`#[ink(storage)]`结构体

- 至少一个`#[ink(constructor)]`函数

- 至少一个`#[ink(message)]`函数

## macros

- `#[ink::contract]` 定义智能合约逻辑的入口

- `#[ink(storage)]` 定义合约存储结构 

- `#[ink(constructor)]`
  
  定义storage结构体的constructor方法，用来初始化合约。可能会有多个constructors，以实现多种初始化方式

- `#[ink(message)]` 
  
  标识一个方法可以可以通过合约调用。所有的public函数必须使用该宏

- `#[ink(event)]`
  
  应用在`struct`定义上，定义合约执行期间可以触发的事件

- `#[ink(topic)]`
  
  应用在event类型的字段上，类似sodility的索引事件参数
  
  event的签名默认是其中一个topic

`new`和`default`函数初始化值

## 合约测试

`cargo test`

## 合约打包

`cargo contract build`

打包生成文件

```
xxx.contract    (code + metada)
xxx.wasm        (code)
xxx.json        (metadata)
```

**部署**合约用的是`.contract`文件

包含ABI的metadata文件

```json
spec
    constructors、messages等可调用函数
    events    查询合约的执行和链上状态
    selector  constructor或message的名称哈希后取前4个字节
storage
types
```

## Environment函数

在`#[ink(constructor)]`中使用`Self::env()`访问env函数，在`#[ink(message)]`中使用`self.env()`.

下面列举一些实用的函数：

- `caller()`：返回合约调用者的地址

- `account_id()`：返回合约的账户ID

- `balance()`：返回合约的余额

- `block_number()`： 返回当前块号

- `emit_event`：触发一个事件

- `transfer()`：从合约中转账到指定账户

- `hash_bytes()`：将给定输入的加密哈希并将结果存储在输出中

更全面的函数[详见](https://docs.rs/ink_env/4.0.0/ink_env/#functions).
