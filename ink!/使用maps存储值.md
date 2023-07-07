# 使用maps存储值

本文将介绍如何扩展上一个合约，使得每个用户都有一个number.你将使用`Mapping`类型添加该功能。

ink!语言提供了Mapping类型是你能够以键值对的形式存储数据。如下：

```rust
// Import `Mapping` type
use ink::storage::Mapping;

#[ink(storage)]
pub struct MyContract {
    // Store a mapping from AccountId to a u32
    my_map: Mapping<AccountId, u32>,
}
```

有了`Mapping`类型，你可以为storage value的每一个key存储唯一实例。

在本文中，每个`AccountId`代表了my_map的唯一键。

每个用户只能存储、增加、检索与其`AccountId`相关的值。

## 初始化`Mapping`

第一步就是初始化mapping

- 指定mapping的key及其对应的值

下面的例子演示了如何初始化一个`Mapping`并检索值

```rust
#[ink::contract]
mod my_contract {

  use ink::storage::Mapping;

  #[ink(storage)]
  pub struct MyContract {
    my_map: Mapping<AccountId, u32>,
  }

  impl MyContract {
    #[ink(constructor)]
    pub fn new(count: u32) -> Self {
      let mut my_map = Mapping::default();
      let caller = Self::env().caller();
      my_map.insert(&caller, &count);

      Self { my_map }
    }

    #[ink(message)]
    pub fn get(&self) -> u32 {
      let caller = Self::env().caller();
      self.my_map.get(&caller).unwrap_or_default()
    }
  }
}
```

### 识别合约调用者

在上面的例子中，你可能已经注意到了`Self::env().caller()`函数调用

该函数在整个合约逻辑中都可以访问，并返回**合约调用者**。

注意：合约调用者并不是**origin caller**.

如果一个用户访问了一个合约，但是该合约调用了后续的合约，第二个合约中的`Self::env().caller()`是第一个合约的地址，不是original user.

### 使用合约调用者

许多场景下访问合约调用者是很有用的。

比如，你可以使用`Self::env().caller()`来创建一个访问控制层，只允许用户访问自己的值；你也可以用来在合约部署时，保存合约的所有者。

比如：

```rust
#![cfg_attr(not(feature = "std"), no_std)]

#[ink::contract]
mod my_contract {

  use ink::storage::Mapping;

  #[ink(storage)]
  pub struct MyContract {
    owner: AccountId,
  }

  impl MyContract {
    #[ink(constructor)]
    pub fn new(count: u32) -> Self {
      Self {
        owner: Self::env().caller(),
      }      
    }
  }
}
```

因为你已经通过`owner`保存了合约调用者，可以在之后写一个函数，检查当前的合约调用者是不是合约的所有者。

### 添加mapping到合约中

要添加一个storage map到合约中：

1. 引入`Mapping`类型
   
   ```rust
   #[ink::contract]
   mod incrementer {
       use ink::storage::Mapping;
   }
   ```

2. 添加mapping key
   
   ```rust
   pub struct Incrementer {
       value: i32,
       my_map: Mapping<AccountId, i32>,
     }
   ```

3. 在`new`constructor中创建一个新的`Mapping`，并用来初始化合约
   
   ```rust
   #[ink(constructor)]
     pub fn new(init_value: i32) -> Self {
       let mut my_map = Mapping::new();
       let caller = Self::env().caller();
       my_map.insert(&caller, &0);
   
       Self {
         value: init_value,
         my_map,
       }
     }
   ```

4. 在`default`constructor中添加一个新的默认Mapping和已经定义的默认`value`
   
   ```rust
   #[ink(constructor)]
     pub fn default() -> Self {
       Self {
         value: 0,
         my_map: Mapping::default(),
       }
     }
   ```

5. 添加一个`get_mine()`函数，通过Mapping API的`get()`方法读取`my_map`并返回给合约调用者
   
   ```rust
   #[ink(message)]
     pub fn get_mine(&self) -> i32 {
       let caller = self.env().caller();
       self.my_map.get(&caller).unwrap_or_default()
     }
   ```

6. 添加新的test来初始化账户
   
   ```rust
   #[ink::test]
   fn my_map_works() {
      let contract = Incrementer::new(11);
      assert_eq!(contract.get(), 11);
      assert_eq!(contract.get_mine(), 0);
   }
   ```

7. 
   

### 插入、更新或删除值

`Incrementer`合约的最后一步是允许用户更改自己的值。你可以通过调用Mapping API来提供这个功能。`Mapping`提供了直接访问存储项的能力。

比如，你可以用已经存在的key调`Mapping::insert()`替换之前的值；你也可以通过`Mapping::get()`先读取，然后再用`Mapping::insert()`修改值；如果给定的key没有对应的值，`Mapping::get()`返回`None`.

你可以用`Mapping::remove()`方法通过给定key从storage中删除值。

步骤如下：

1. 添加`inc_mine()`方法，允许合约调用者获取`my_map`存储项，插入一个增加的`value`到mapping中。
   
   ```rust
   #[ink(message)]
     pub fn inc_mine(&mut self, by: i32) {
       let caller = self.env().caller();
       let my_value = self.get_mine();
       self.my_map.insert(caller, &(my_value + by));
     }
   ```

2. 添加`remove_mine()`函数，允许合约调用者清除`my_map`存储项。
   
   ```rust
   #[inc(message)]
     pub fn remove_mine(&self) {
       let caller = self.env().caller();
       self.my_map.remove(&caller)
     }
   ```

3. 添加新的测试以验证`inc_mine()`函数如期工作
   
   ```rust
   #[ink::test]
       fn inc_mine_works() {
         let mut contract = Incrementer::new(11);
         assert_eq!(contract.get_mine(), 0);
         contract.inc_mine(5);
         assert_eq!(contract.get_mine(), 5);
         contract.inc_mine(5);
         assert_eq!(contract.get_mine(), 10);
       }
   ```

4. 添加新的测试以验证`remove_mine()`函数如期工作
   
   ```rust
   #[ink::test]
       fn remove_mine_works() {
         let mut contract = Incrementer::new(11);
         assert_eq!(contract.get_mine(), 0);
         contract.inc_mine(5);
         assert_eq!(contract.get_mine(), 5);
         contract.remove_mine();
         assert_eq!(contract.get_mine(), 0);
       }
   ```

## 接下来




