# 构建一个token合约

本文介绍如何使用ink!构建一个 [ERC-20 token](https://eips.ethereum.org/EIPS/eip-20)合约。

ERC-20规范为同质化代币定义了一个通用标准。

对于定义token的属性有了标准后，开发者就能遵循该规范来构建可以与其他产品和服务互操作的应用。

ERC-20 token标准不是唯一的token标准，而是最常用的之一。



## ERC-20标准基础

 [ERC-20 token standard](https://eips.ethereum.org/EIPS/eip-20)为以太坊链上的大多数合约定义了接口。这些标准的接口允许个体在已有的合约平台上部署自己的加密货币。

该标准定义有如下核心函数

```solidity
contract ERC20Interface {
    // Storage Getters
    function totalSupply() public view returns (uint);
    function balanceOf(address tokenOwner) public view returns (uint balance);
    function allowance(address tokenOwner, address spender) public view returns (bool success);

    // Public Functions
    function transfer(address to, uint tokens) public returns (bool success);
    function approve(address spender, uint tokens) public returns (bool success);
    function transferFrom(address from, address to, uint tokens) public returns (bool success);

    // Contract Events
    event Transfer(address indexed from, address indexed to, uint tokens);
    event Approval(address indexed tokenOwner, address indexed spender, uint tokens);
}
```

用户余额映射到账户地址，接口允许用户转账token，或允许嗲是南方代表他们转账。

更重要地是，智能合约逻辑的实现，必须要确保资金不会无意中创建或销毁，并确保用户的资金不受恶意行为者的侵害。

注意，所有的public函数都返回一个`bool`值，标识调用是否成功。

在Rust中，这些函数都将返回`Result`.



## 创建token supply

处理ERC-20 token的合约和使用Maps存储值的Increment合约类似。

对于本教程，ERC-20合约由一个固定的token supply组成，当合约部署时，这些token全部存入与合约所有者关联的帐户中。然后合约所有者可以将这些token分配给其他用户。

在本教程中创建的简单 ERC-20合约并不代表你可以创建和分发token的唯一方法。但是，这个 ERC-20合约提供了一个良好的基础，扩展你在其他教程中所学到的，以及如何使用ink!语言建立更健全的智能合约。

对于ERC-20 token合约，初始的存储构成：

- `total_supply`表示合约token的总供应量

- `balances`表示每一个账户的余额



### 转账tokens

ERC-20合约有一个拥有`total_supply`的用户账户。合约所有者必须能够将token转账给其他用户。

这个简单的ERC-20 合约中，你要添加一个public `transfer`函数，让你作为合约调用者，将你拥有的token转账给其他用户。

public `transfer`函数调用一个private `transfer_from_to`函数。由于这是一个内部的函数，可以在没有任何授权检查的情况下调用。

但是，转账的逻辑必须确定`from`账户有足够的token转账给`to`账户。`transfer_from_to()`函数使用合约调用者(`self.env().caller()`)作为`from`账户。

在这个上下文中，`transfer_from_to()`函数接着做了如下操作：

- 得到`from`和`to`账户的当前余额。

- 检查`from`账户余额是否小于要发送的token数量
  
  ```rust
  let from_balance = self.balance_of(*from);
  if from_balance < value {
      return Err(Error::InsufficientBalance)
  }
  ```

- 从转账账户减去`value`，接收账户加上`value`
  
  ```rust
  self.balances.insert(&from, &(from_balance - value));
  let to_balance = self.balance_of(*to);
  self.balances.insert(&to, &(to_balance + value));
  ```

将transfer函数添加到智能合约中：

1. 添加一个`Error`声明，当账户中的转账余额不足时返回错误。
   
   ```rust
   #[derive(Debug, PartialEq, Eq, scale::Encode, scale::Decode)]
   #[cfg_attr(feature = "std", derive(scale_info::TypeInfo))]
   pub enum Error {
       InsufficientBalance,
   }
   ```

2. 添加一个`Result`返回类型 ，返回`InsufficientBalance`错误
   
   ```rust
   pub type Result<T> = core::result::Result<T, Error>;
   ```

3. 添加`transfer` pub 函数，使合约调用者可以转账给其他账户
   
   ```rust
   #[ink(message)]
   pun fn transfer(&mut self, to: AccountId, value: Balance) -> Result<()> {
       let from = self.env().caller();
       self.transfer_from_to(&from, &to, value)
   }
   ```

4. 添加`transfer_from_to()` private 函数将token从与合约调用者相关的账户转账到接收账户
   
   ```rust
   fn transfer_from_to(
       &mut self,
       from: &AccountId,
       to: &AccountId,
       value: Balance,
   ) -> Result<()> {
       let from_balance = self.balance_of(*from);
       if from_balance < value {
           return Err(Error::InsufficientBalance)
       }
       
       self.balances.insert(&from, &(from_balance - value));
       let to_balance = self.balance_of(*to);
       self.balances.insert(&to, &(from_balance + value));
   
       Ok(())
   }
   ```

5. 添加transfer的测试
   
   ```rust
   #[ink::test]
   fn transfer_works() {
       let mut contract = Erc20::new(100);
       assert_eq!(contract.balance_of(alice()), 100);
       assert!(contract.transfer(bob(), 10).is_ok());
       assert_eq!(contract.balance_of(bob()), 10);
       assert!(contract.transfer(bob(), 100).is_err());
   }
   ```



### 创建events

ERC-20 token标准规定，合约调用在提交交易时不能直接返回值。但是，你可能希望智能合约以某种方式发出事件已经发生的信号。例如，你可能希望智能合约指示交易何时完成或转账是否得到批准。你可以使用事件来发送这些类型的信号。

你可以使用事件来通信任何类型的数据。定义事件的数据类似于定义结构体。事件应该使用 `# [ ink (event)]`属性声明。



### 添加一个transfer事件

在本文中，你将声明一个`Transfer`事件来提供转账操作完成的信息。`Transfer`event包含如下信息：

- 一个`Balance`类型的值

- 一个Option包装的`AccountId`变量：`from`账户

- 一个Option包装的`AccountId`变量：`to`账户

为了更快访问event数据，他们可以添加*indexed field*.你可以在字段上使用`#[ink(topic)]`属性标记。

添加`Transfer`事件：

```rust
#[ink(event)]
pub struct Transfer {
    #[ink(topic)]
    from: Option<AccountId>,
    #[ink(topic)]
    to: Option<AccountId>,
    value: Balance,
}
```

> 在 Solidity 中，通过 indexed field 来检索事件需要使用 Web3.js 等工具。
> 
> 以 Web3.js 为例，我们可以通过 `web3.eth.getPastLogs` 方法来过滤 Transfer 事件，并指定相应的 indexed 参数值。例如：
> 
> ```js
> // 引入 web3
> const Web3 = require('web3');
> const web3 = new Web3('http://localhost:8545');
> 
> // 获取 Transfer 事件
> const transferEvent = web3.eth.Contract(ABI, contractAddress).events.Transfer({
>   filter: {
>     from: '0x123...',
>     to: '0x456...'
>   },
>   fromBlock: 0, // 起始区块号
>   toBlock: 'latest' // 终止区块号
> });
> 
> // 处理 Transfer 事件
> transferEvent.on('data', (event) => {
>   console.log(event);
> });
> 
> ```
> 
> 在上述代码中，我们使用了 `filter` 参数来指定需要过滤的 from 和 to 地址，从而仅获取与这些地址相关的 Transfer 事件。由于我们将 from 和 to 参数标记为 indexed，因此 Web3.js 可以使用这些参数值来过滤事件。
> 
> 当 Transfer 事件被触发时，我们可以通过 `on('data')` 方法来处理事件数据。你可以在这里将事件数据存储到数据库中，或者以其他方式对其进行处理。
> 
> 
> 
> 如果事件参数没有被标记为 indexed，那么就不能通过该参数来过滤事件。
> 
> 在这种情况下，你可以通过 `web3.eth.getPastLogs` 方法来获取所有的 Transfer 事件，并在本地进行过滤。例如：
> 
> ```js
> // 引入 web3
> const Web3 = require('web3');
> const web3 = new Web3('http://localhost:8545');
> 
> // 获取指定区块范围内的 Transfer 事件
> const transferEvents = await web3.eth.getPastLogs({
>   address: contractAddress, // 智能合约地址
>   topics: [
>     web3.utils.sha3('Transfer(address,address,uint256)'),
>     null, // from 地址
>     null, // to 地址
>     web3.utils.padLeft(web3.utils.toHex(100), 64) // value 值
>   ],
>   fromBlock: 0, // 起始区块号
>   toBlock: 'latest' // 终止区块号
> });
> 
> // 处理 Transfer 事件
> transferEvents.forEach((event) => {
>   console.log(event);
> });
> 
> ```
> 
> 在上述代码中，我们使用了 `topics` 参数来指定事件主题（也称为事件签名）。由于 Transfer 事件参数没有被标记为 indexed，因此我们将其传递为 null。我们还使用了 `web3.utils.padLeft` 方法来将 value 值转换为 64 位十六进制字符串，以便与其他主题参数匹配。
> 
> 当 Transfer 事件被触发时，我们可以通过 `forEach` 方法遍历所有事件，并对其进行处理。由于我们在本地进行了过滤，因此处理的事件数据不一定与合约中实际触发的 Transfer 事件一致。因此，你需要仔细检查事件数据，并对其进行验证。

#### 触发事件

定义了事件之后，就可以添加触发事件的代码。

你可以使用`self.env().emit_event()`函数，以事件名为参数，触发相应事件。

在ERC-20合约中，每转账一次就要触发一个`Transfer`事件，代码中有两个地方会有：

- 初始化合约的`new`调用中

- 每次调用`traansfer_from_to`时

`from`和`to`字段的值时`Option<AccountId>`数据类型。但是，token的初始转账中，设置初始supply的值并不是来自其他账户。这种情况下，Transfer事件的`from`值就是`None`.

触发Transfer事件：

1. 在`new()`constructor中 添加`Transfer`事件
   
   ```rust
   #[ink(constructor)]
   pub fn new(total_supply: Balance) -> Self {
       Self::env().emit_event(Transfer {
           from: None,
           to: Some(caller),
           value: total_supply,
       })
   }
   ```

2. 在`transfer_from_to()`函数中添加`Transfer`事件
   
   ```rust
   fn transfer_from_to(
       &mut self,
       from: &AccountId,
       to: &AccountId,
       value: Balance,
   ) -> Result<()> {
       // -- snip --
       self.env().emit_event(Transfer {
           from: Some(*from),
           to: Some(*to),
           value,
       });
       // -- snip --
   }
   ```



### 允许第三方转账

ERC-20 token合约现在可以在账户间转账token，并在转账时触发事件。最后一步，你可以添加`approve`和`transfer_from`函数来允许第三方转账。

允许一个账户代表另一个账户消费token允许你的合约支持去中心化交易所。

不是在合约中直接将你的token转账给另一个用户，而是你可以批准另一个用户代表你，交易一部分token. 在你等待这类交易执行时，你仍然可以控制和消费你的token.

你也可以批准多个多个合约或用户获取你的token使用权，所以当一个合约出价最高时，你不需要把token从一个合约move到另一个合约，这种可能代价高又耗时的过程。

为了确保批准和转账安全操作，ERC-20合约使用了一个两步的流程，分别操作**appprove**和**transfer From**.

#### 添加approval逻辑

批准另一个账户消费你的token是第三方转账流程的第一步。

作为token的所有者，你可以指定代表你转账的任意账户、任意金额。你不需要批准账户中所有的token都可以转账，只需指定批准的账户可以转账的最大额度。

当你多次调用`approve`后，就会用新的值覆盖之前批准的值。默认情况下，任意两个账户之间批准的金额都是`0`.如果你想撤回批准的权限，你可以调用`approve`函数，设置值为`0`.

为了存储这种approvals关系，你需要使用稍微复杂一点的`Mapping`key.

因为每个账户可以为任意账户设置不同的批准额度，所以要使用一个tuple作为key，映射到一个额度value.

```rust
pub struct Erc20 {
    // -- snip --
    /// 可以被非所有者转账的金额：(owner, spender) -> allowed
    allowances: Mapping<(AccountId, AccountId), Balance>,
}
```

tuple使用`(owner, spender)`来标识允许代表`owner`访问一定数量的`allowance`的`spender`账户.

添加approval逻辑：

1. 使用`#[ink(event)]`属性宏声明`Approval` 事件
   
   ```rust
   #[ink(event)]
   pub struct Approval {
       #[ink(topic)]
       owner: AccountId,
       #[ink(topic)]
       spender: AccountId,
       value: Balance,
   }
   ```

2. 添加`Error`变体表示当转账请求超出了账户的allowance抛出的错误
   
   ```rust
   #[derive(Debug, PartialEq, Eq, scale::Encode, scale::Decode)]
   #[cfg_attr(feature = "std", derive(scale_info::TypeInfo))]
   pub enum Error {
       InsufficientBalance,
       InsufficientAllowance,
   }
   ```

3. 添加一个`allowances` `Mapping`到存储声明中,所有者和非所有者的组合映射到账户金额
   
   ```rust
   allowances: Mapping<(AccountId, AccountId), Balance>,
   ```

4. 在new() constructor中初始化并添加`allowance` `Mapping`
   
   ```rust
   #[ink(constructor)]
   pub fn new(total_supply: Balance) -> Self {
       // -- snip --
       let allowances = Mapping::default();
   
       Self {
           total_supply,
           balances,
           allowances
       }
   }
   ```

5. 添加`approve()`函数，授权`spender`账户从调用者账户中提取最大`value`的token
   
   ```rust
   #[ink(message)]
   pub fn approve(
       &mut self,
       spender: AccountId,
       value: Balance,
   ) -> Result<()> {
       let owner = self.env().caller();
       self.allowances.insert((owner,spender), &value);
   
       self.env().emit_event(
           Approval {
               owner,
               spender,
               value,
           }
       );
       Ok(())
   }
   ```

6. 添加一个`allowance()`函数，返回`owner`账户允许`spender`提取的金额
   
   ```rust
   #[ink(message)]
   pub fn allowance(&self, owner: AccountId, spender: AccountId) -> Balance {
       self.allowance.get((owner, spender)).unwrap_or_default()
   }
   ```



### 添加转账逻辑

设置了approval之后，需要创建一个`transfer_from`函数来转账token.

`transfer_from`函数调用了private `transfer_from_to`函数来处理大部分转帐逻辑。

授权非所有者转账有一些要求：

- `self.env().caller()`合约调用者必须分配`from`账户中的token

- 作为allowance存储的审批额度必须大于要转账值

如果这些要求都满足了，合约插入更新后的allowance到`allowance`变量中，使用指定的`from` 和`to`账户调用`transfer_from_to()`函数。

记住当调用`transfer_from`时，`self.env().caller()`和`from`账户用来查询当前的allowance，但是`transfer_from`函数是在指定的`from`和`to`账户之间调用的。

调用`transfer_from`会用到三个账户，你需要确保正确地使用他们。

添加`transfer_from`逻辑：

1. 添加`transfer_from()`函数来代表`from`账户向`to`账户转账`value`金额的token.
   
   ```rust
   #[ink(message)]
   pub fn transfer_from(
       &mut self,
       from: AccountId,
       to: AccountId,
       value: Balance,
   ) -> Result<()> {
       let caller = self.env().caller();
       let allowance = self.allowance(from, caller);
       if allowance < value {
           return Err(Error::InsufficientAllowance);
       }
   
       self.transfer_from_to(&from, &to, value)?;
       self.allowances.insert((from, caller), &(allowance -value));
   
       Ok(())
   }
   ```

2. 添加`transfer_from()`函数的测试
   
   ```rust
   #[ink::test]
   fn transfer_from_works() {
       let mut contract = Erc20::new(100);
       assert_eq!(contract.balance_of(alice()), 100);
       let _ = contract.approve(alice(), 20);
       let _ = contract.transfer_from(alice(), bob(), 10);
       assert_eq!(contract.balance_of(bob()), 10);
   }
   ```

3. 添加`allowance()`函数的测试
   
   ```rust
   #[ink::test]
   fn allowances_works() {
      let mut contract = Erc20::new(100);
      assert_eq!(contract.balance_of(alice()), 100);
      let _ = contract.approve(alice(), 200);
      assert_eq!(contract.allowance(alice(), alice()), 200);
   
      assert!(contract.transfer_from(alice(), bob(), 50).is_ok());
      assert_eq!(contract.balance_of(bob()), 50);
      assert_eq!(contract.allowance(alice(), alice()), 150);
   
      assert!(contract.transfer_from(alice(), bob(), 100).is_err());
      assert_eq!(contract.balance_of(bob()), 50);
      assert_eq!(contract.allowance(alice(), alice()), 150);
   }
   ```
