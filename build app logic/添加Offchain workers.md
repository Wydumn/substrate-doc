# 添加Offchain workers

本教程说明如何修改apllet以包含offchain worker，并配置pallet和运行时以使ocw能够提交更新链上状态的交易。

## 使用ocw

如果你使用ocw来执行长时间运行的计算或从离线源获取数据，则你可能希望将这些操作的结果存储在链上。

> ???? 为什么我会想着把这玩意儿存链上
> 
> 将离线计算或从离线来源获取数据的结果存储在链上可以确保这些结果是不可变的，无法被篡改或更改。这种方式可以提高数据的可靠性和安全性，并允许整个网络共享和验证这些结果。此外，存储结果在链上还可以使其易于查询和检索，因为它们可以通过智能合约或其他工具访问和处理。

 但是，链下存储与链上资源是分开的，你无法将ocw的数据直接保存到链上存储中。 要将ocw处理的任何数据存储为链上状态的一部分，你必须创建交易以将数据从ocw存储发送到链上存储系统。

本教程说明了如何创建能够发送签名或未签名交易以在链上存储链下数据的ocw。 一般来说，签名交易更安全，但需要调用账户处理交易费用。 例如：

- 如果要记录关联的交易调用方帐户并从调用方帐户中扣除交易费用，请使用**签名交易**
- 如果要记录关联的交易调用方，但不希望调用方负责支付交易费，请使用**具有已签名payload的未签名交易**。

## 使用未签名的交易

也可以在没有签名有效数据（without a signed payload）的情况下提交**未签名的**交易，例如，因为你根本不想记录关联的交易调用方。 但是，允许未签名的交易修改链状态存在重大风险。 未签名的交易表示恶意用户可以利用的潜在攻击媒介。 如果你打算允许ocw发送未签名的交易，你应该包括确保交易获得授权的逻辑。 有关如何使用链上状态验证未签名交易的示例，请参阅[`enact_authorized_upgrade`](https://github.com/paritytech/cumulus/blob/445f9277ab55b4d930ced4fbbb38d27c617c6658/pallets/parachain-system/src/lib.rs#L694)中的`ValidateUnsigned`实现。 在该示例中，调用通过验证给定的代码哈希之前是否已获得授权来验证未签名的交易。

同样重要的是要考虑到，即使是具有签名有效数据的未签名交易也可能被利用，因为除非你实施严格的逻辑来检查交易的有效性，否则不能假设offchain worker是可靠的来源。 在大多数情况下，在写入存储之前检查交易是否已由ocw提交并不足以保护网络。 与其假设ocw可以在没有保护措施的情况下被信任，不如有意设置限制性权限来限制对进程的访问及其可以执行的操作。

请记住，未签名的交易本质上是进入运行时的**open door**。 只有在仔细考虑允许执行它们的条件后，才应使用它们。 如果没有保护措施，恶意参与者可能会冒充ocw并访问运行时存储。

> "Open door"是一个英语短语，意思为“敞开的门”。这个短语可以用来描述一个门或房间里的门是打开的，也可以用来比喻某个机会或资源是向所有人敞开的，任何人都可以利用它。在商业或政治上，“open door policy”（敞开的门政策）指的是一个国家或组织对外开放、欢迎各种合作和交流的政策。

## 开始之前

在开始之前，请验证以下内容：

- 你已通过安装为Substrate开发配置了环境[Rust 和 Rust 工具链](https://docs.substrate.io/install/).
- 你已完成[构建本地区块链](https://docs.substrate.io/tutorials/build-a-blockchain/build-local-blockchain/)教程，并在本地安装Developer Hub的 Substrate node template。
- 你熟悉如何使用 FRAME 宏和编辑pallet的逻辑。
- 你熟悉如何在运行时修改pallet的Config trait。

## 教程目标

完成本教程后，你将能够：

- 识别使用未签名交易所涉及的风险
- 将一个ocw函数添加到pallet
- 配置pallet和运行时，使ocw能够提交签名的交易。
- 配置pallet和运行时，使ocw能够提交未签名的交易。
- 配置pallet和运行时，使ocw能够提交具有签名有效数据的未签名交易。

## 签名的交易记录

要提交已签名的交易，你必须配置pallet和运行时，以启用至少一个账户供链下工作人员使用。 在高级pallet，将托盘配置为使用办公室连锁工作人员并提交已签名的交易记录涉及以下步骤：

- [在pallet中配置OCW](https://docs.substrate.io/tutorials/build-application-logic/add-offchain-workers/#configure-the-offchain-worker-in-the-pallet).
- [在运行时实现pallet和所需trait](https://docs.substrate.io/tutorials/build-application-logic/add-offchain-workers/#implement-the-pallet-in-the-runtime).
- [添加用于签署交易的帐户](https://docs.substrate.io/tutorials/build-application-logic/add-offchain-workers/#add-an-account-for-signing-transactions).

### 在pallet中配置OCW

要使链下工作者能够发送签名的交易，请执行以下操作：

1. 在文本编辑器中打开pallet的文件。`src/lib.rs`

2. 将链下工作者的宏和入口点添加到代码中。`#[pallet::hooks]`
   
   例如：
   
   ```rust
   #[pallet::hooks]
   impl<T: Config> Hooks<BlockNumberFor<T>> for Pallet<T> {
      /// Offchain worker entry point.
      ///
      /// By implementing `fn offchain_worker` you declare a new offchain worker.
      /// This function will be called when the node is fully synced and a new best block is
      /// successfully imported.
      /// Note that it's not guaranteed for offchain workers to run on EVERY block, there might
      /// be cases where some blocks are skipped, or for some the worker runs twice (re-orgs),
      /// so the code should be able to handle that.
      fn offchain_worker(block_number: T::BlockNumber) {
          log::info!("Hello from pallet-ocw.");
          // The entry point of your code called by offchain worker
      }
      // ...
   }
   ```

3. 添加函数的逻辑。`offchain_worker`

4. 加[`CreateSignedTransaction`](https://paritytech.github.io/substrate/master/frame_system/offchain/trait.CreateSignedTransaction.html)到你的pallet的特性。 例pallet你的托盘特征应如下所示：`Config``Config`
   
   ```rust
   /// This pallet's configuration trait
   #[pallet::config]
   pub trait Config: CreateSignedTransaction<Call<Self>> + frame_system::Config {
      // ...
   }
   ```

5. 向pallet特征添加类型：`AuthorityId``Config`
   
   ```rust
   #[pallet::config]
   pub trait Config: CreateSignedTransaction<Call<Self>> + frame_system::Config {
      // ...
   type AuthorityId: AppCrypto<Self::Public, Self::Signature>;
   }
   ```

6. 添加具有签名密钥的模块，以确保你的pallet拥有可用于签署交易的帐户。`crypto``sr25519`
   
   ```rust
   use sp_core::{crypto::KeyTypeId};
   
   // ...
   
   pub const KEY_TYPE: KeyTypeId = KeyTypeId(*b"demo");
   
   // ...
   
   pub mod crypto {
      use super::KEY_TYPE;
      use sp_core::sr25519::Signature as Sr25519Signature;
      use sp_runtime::{
          app_crypto::{app_crypto, sr25519},
          traits::Verify, MultiSignature, MultiSigner
      };
      app_crypto!(sr25519, KEY_TYPE);
   
      pub struct TestAuthId;
   
      // implemented for runtime
      impl frame_system::offchain::AppCrypto<MultiSigner, MultiSignature> for TestAuthId {
      type RuntimeAppPublic = Public;
      type GenericSignature = sp_core::sr25519::Signature;
      type GenericPublic = sp_core::sr25519::Public;
      }
   }
   ```
   
   这[`app_crypto`宏观](https://paritytech.github.io/substrate/master/sp_application_crypto/macro.app_crypto.html)声明具有由 标识的签名的帐户。 在此示例中，是 . 请注意，此宏不会创建新帐户。 宏只是声明一个帐户可供此pallet使用。`sr25519``KEY_TYPE``KEY_TYPE``demo``crypto`

7. 初始化一个账户，供链下工作者用于将签名交易发送到链上存储。
   
   ```rust
   fn offchain_worker(block_number: T::BlockNumber) {
      let signer = Signer::<T, T::AuthorityId>::all_accounts();
   
      // ...
   }
   ```
   
   此代码使你能够检索此pallet拥有的所有签名者。

8. 用于创建已签名的事务调用：`send_signed_transaction()`
   
   ```rust
   fn offchain_worker(block_number: T::BlockNumber) {
      let signer = Signer::<T, T::AuthorityId>::all_accounts();
   
      // Using `send_signed_transaction` associated type we create and submit a transaction
      // representing the call we've just created.
      // `send_signed_transaction()` return type is `Option<(Account<T>, Result<(), ()>)>`. It is:
      //     - `None`: no account is available for sending transaction
      //     - `Some((account, Ok(())))`: transaction is successfully sent
      //     - `Some((account, Err(())))`: error occurred when sending the transaction
      let results = signer.send_signed_transaction(|_account| {
          Call::on_chain_call { key: val }
      });
   
      // ...
   }
   ```

9. 检查交易是否在链上成功提交，并通过检查返回的 .`results`
   
   ```rust
   fn offchain_worker(block_number: T::BlockNumber) {
      // ...
   
      for (acc, res) in &results {
          match res {
              Ok(()) => log::info!("[{:?}]: submit transaction success.", acc.id),
              Err(e) => log::error!("[{:?}]: submit transaction failure. Reason: {:?}", acc.id, e),
          }
      }
   
      Ok(())
   }
   ```

### 在运行时实现pallet

1. 在文本编辑器中打开节点模板的文件。`runtime/src/lib.rs`

2. 将 添加到pallet的配置中，并确保它使用 来自模块：`AuthorityId``TestAuthId``crypto`
   
   ```rust
   impl pallet_your_ocw_pallet::Config for Runtime {
    // ...
    type AuthorityId = pallet_your_ocw_pallet::crypto::TestAuthId;
   }
   ```

3. 在运行时实现特征。`CreateSignedTransaction`
   
   由于你已将pallet配置为实现该特性，因此你还需要为运行时实现该特性。`CreateSignedTransaction`
   
   通过查看[`CreateSignedTransaction`](https://paritytech.github.io/substrate/master/frame_system/offchain/trait.CreateSignedTransaction.html)，你可以看到你只需要为运行时实现该函数。 例如：`create_transaction()`
   
   ```rust
   use codec::Encode;
   use sp_runtime::{generic::Era, SaturatedConversion};
   
   // ...
   
   impl<LocalCall> frame_system::offchain::CreateSignedTransaction<LocalCall> for Runtime
   where
      RuntimeCall: From<LocalCall>,
   {
      fn create_transaction<C: frame_system::offchain::AppCrypto<Self::Public, Self::Signature>>(
             call: RuntimeCall,
         public: <Signature as Verify>::Signer,
           account: AccountId,
           nonce: Index,
       ) -> Option<(RuntimeCall, <UncheckedExtrinsic as traits::Extrinsic>::SignaturePayload)> {
           let tip = 0;
           // take the biggest period possible.
           let period =
                BlockHashCount::get().checked_next_power_of_two().map(|c| c / 2).unwrap_or(2) as u64;
           let current_block = System::block_number()
                .saturated_into::<u64>()
                // The `System::block_number` is initialized with `n+1`,
                // so the actual block number is `n`.
                .saturating_sub(1);
           let era = Era::mortal(period, current_block);
           let extra = (
                frame_system::CheckNonZeroSender::<Runtime>::new(),
                frame_system::CheckSpecVersion::<Runtime>::new(),
                frame_system::CheckTxVersion::<Runtime>::new(),
                frame_system::CheckGenesis::<Runtime>::new(),
                frame_system::CheckEra::<Runtime>::from(era),
                frame_system::CheckNonce::<Runtime>::from(nonce),
                frame_system::CheckWeight::<Runtime>::new(),
                pallet_transaction_payment::ChargeTransactionPayment::<Runtime>::from(tip),
           );
           let raw_payload = SignedPayload::new(call, extra)
                .map_err(|e| {
                     log::warn!("Unable to create signed payload: {:?}", e);
                })
                .ok()?;
           let signature = raw_payload.using_encoded(|payload| C::sign(payload, public))?;
           let address = account;
           let (call, extra, _) = raw_payload.deconstruct();
           Some((call, (sp_runtime::MultiAddress::Id(address), signature, extra)))
     }
   }
   ```
   
   此代码片段很长，但实质上它说明了以下主要步骤：
   
   - 创建和准备类型，并将各种检查器放置到位。`extra``SignedExtra`
   - 根据传入的 和 创建原始有效负载。`call``extra`
   - 使用帐户公钥对原始有效负载进行签名。
   - 将所有数据捆绑在一起，并返回调用、调用方、其签名和任何已签名分机数据的元组。
   
   你可以在[基板代码库](https://github.com/paritytech/substrate/blob/master/bin/node/runtime/src/lib.rs#L1239-L1279).

4. 在运行时实现 和 以支持提交事务，无论它们是已签名的还是未签名的。`SigningTypes``SendTransactionTypes`
   
   ```rust
   impl frame_system::offchain::SigningTypes for Runtime {
      type Public = <Signature as traits::Verify>::Signer;
      type Signature = Signature;
   }
   
   impl<C> frame_system::offchain::SendTransactionTypes<C> for Runtime
   where
      RuntimeCall: From<C>,
   {
      type Extrinsic = UncheckedExtrinsic;
      type OverarchingCall = RuntimeCall;
   }
   ```
   
   你可以在[基板代码库](https://github.com/paritytech/substrate/blob/master/bin/node/runtime/src/lib.rs#L1280-L1292).

### 添加用于签署交易的帐户

此时，你已准备好使用链下工人的palletpallet准备托盘涉及以下步骤：

- 添加用于发送已签名事务的函数和相关逻辑。`offchain_worker`
- 为你的pallet添加和特性。`CreateSignedTransaction``AuthorityId``Config`
- 添加模块以描述pallet将用于签署交易的帐户。`crypto`

你还使用代码更新了运行时，以支持ocw和发送签名交易。 更新运行时涉及以下步骤：

- 将 添加到pallet的运行时配置中。`AuthorityId`
- 实现特征和功能。`CreateSignedTransaction``create_transaction()`
- 从pallet上实施和为链下工人。`SigningTypes``SendTransactionTypes``frame_system`

但是，在pallet链下工作人员可以提交已签名的交易之前，你必须指定至少一个帐户供链下工作人员使用。 要使链下工作人员能够签署交易，你pallet生成托盘拥有的帐户密钥，并将该密钥添加到节点密钥库。

有几种方法可以完成最后一步，你选择的方法可能会有所不同，具体取决于你是在开发模式下运行节点进行测试、使用自定义链规范还是部署到生产环境中。

### 使用开发帐户

如果你在开发模式下运行节点（使用命令行选项），则可以通过修改文件来手动生成和插入开发帐户的帐户密钥，如下所示：`--dev``node/src/service.rs`

```rust
pub fn new_partial(config: &Configuration) -> Result <SomeStruct, SomeError> {

//...

  if config.offchain_worker.enabled {
  // Initialize seed for signing transaction using offchain workers. This is a convenience
  // so learners can see the transactions submitted simply running the node.
  // Typically these keys should be inserted with RPC calls to `author_insertKey`.
       sp_keystore::SyncCryptoStore::sr25519_generate_new(
           &*keystore,
           node_template_runtime::pallet_your_ocw_pallet::KEY_TYPE,
           Some("//Alice"),
       ).expect("Creating key with account Alice should succeed.");
       }
}
```

此示例手动将帐户的密钥添加到由pallet中定义的标识的密钥库。 有关工作示例，请参阅此示例`Alice``KEY_TYPE`[service.rs](https://github.com/jimmychu0807/substrate-offchain-worker-demo/blob/v2.0.0/node/src/service.rs#L87-L105)文件。

### 使用其他帐户

在生产环境中，你可以使用其他工具（例如）生成专门供ocw使用的密钥。 生成一个或多个密钥供链下工ocw，可以通过以下方式将它们添加到节点密钥库：`subkey`

- 修改链规范文件的配置。
- 使用 RPC 方法传递参数。`author_insertKey`

例如，你可以使用[波卡点/基板门户](https://polkadot.js.org/apps/#/rpc)、Polkadot-JS API 或用于选择方法并指定帐户要使用的密钥类型、密码短语和公钥参数的命令：`curl``author_insertKey`

![使用“author_insertKey”方法插入帐户](https://docs.substrate.io/static/8551cc611f38a7a6c6ac76edf377e436/9432b/author_insertKey.png)

请注意，此示例中的 keyType 参数与ocwpallet中声明的参数匹配。`demo``KEY_TYPE`

现在，你的pallet已准备好从链下工人发送链上签名的交易。

## 未签名的交易

默认情况下，所有未签名的交易在 Substrate 中都会被拒绝。 要使 Substrate 能够接受某些未签名的交易，你必须实现pallet的特征。`ValidateUnsigned`

尽管你必须实现 trait 才能发送未签名的交易，但此检查并不能保证只有ocw**才能**发送交易。 你应该始终考虑恶意行为者发送这些交易作为篡改链状态的尝试的后果。 未签名的交易始终代表恶意用户可以利用的潜在攻击媒介，如果没有额外的保护措施，就不能假定链下工作者是可靠的来源。`ValidateUnsigned`

你永远不应该假设未签名的交易只能由链下工作者提交。 根据定义，**任何人都可以**提交它们。

### 配置pallet

要使链下工作者能够发送未签名的交易，请执行以下操作：

1. 在文本编辑器中打开pallet的文件。`src/lib.rs`

2. 添加[`validate_unsigned`](https://paritytech.github.io/substrate/master/frame_support/attr.pallet.html#validate-unsigned-palletvalidate_unsigned-optional)宏观。
   
   例如：
   
   ```rust
   #[pallet::validate_unsigned]
   impl<T: Config> ValidateUnsigned for Pallet<T> {
      type Call = Call<T>;
   
          /// Validate unsigned call to this module.
          ///
          /// By default unsigned transactions are disallowed, but implementing the validator
          /// here we make sure that some particular calls (the ones produced by offchain worker)
          /// are being whitelisted and marked as valid.
          fn validate_unsigned(source: TransactionSource, call: &Self::Call) -> TransactionValidity {
          //...
          }
   }
   ```

3. 按如下方式实现特征：
   
   ```rust
   fn validate_unsigned(source: TransactionSource, call: &Self::Call) -> TransactionValidity {
      let valid_tx = |provide| ValidTransaction::with_tag_prefix("my-pallet")
          .priority(UNSIGNED_TXS_PRIORITY) // please define `UNSIGNED_TXS_PRIORITY` before this line
          .and_provides([&provide])
          .longevity(3)
          .propagate(true)
          .build();
      // ...
   }
   ```

4. 检查调用外部函数以确定是否允许调用，并返回是否允许调用或不允许调用。`ValidTransaction``TransactionValidityError`
   
   例如：
   
   ```rust
   fn validate_unsigned(source: TransactionSource, call: &Self::Call) -> TransactionValidity {
      // ...
      match call {
          RuntimeCall::my_unsigned_tx { key: value } => valid_tx(b"my_unsigned_tx".to_vec()),
          _ => InvalidTransaction::Call.into(),
      }
   }
   ```
   
   在此示例中，用户只能在没有签名的情况下调用特定函数。 如果还有其他函数，调用它们将需要签名事务。`my_unsigned_tx`
   
   有关如何在pallet中实现的示例，请参阅`ValidateUnsigned`[链下工作者](https://github.com/paritytech/substrate/blob/master/frame/examples/offchain-worker/src/lib.rs#L301-L329).

5. 添加宏和函数以发送未签名的事务，如下所示：`#[pallet::hooks]``offchain_worker`
   
   ```rust
   #[pallet::hooks]
   impl<T: Config> Hooks<BlockNumberFor<T>> for Pallet<T> {
      /// Offchain worker entry point.
      fn offchain_worker(block_number: T::BlockNumber) {
          let value: u64 = 10;
          // This is your call to on-chain extrinsic together with any necessary parameters.
          let call = RuntimeCall::unsigned_extrinsic1 { key: value };
   
          // `submit_unsigned_transaction` returns a type of `Result<(), ()>`
          //     ref: https://paritytech.github.io/substrate/master/frame_system/offchain/struct.SubmitTransaction.html
          SubmitTransaction::<T, Call<T>>::submit_unsigned_transaction(call.into())
              .map_err(|_| {
              log::error!("Failed in offchain_unsigned_tx");
          });
      }
   }
   ```
   
   此代码准备线路中的调用，使用`let call = ...`[`SubmitTransaction::submit_unsigned_transaction`](https://paritytech.github.io/substrate/master/frame_system/offchain/struct.SubmitTransaction.html)，并在传入的回调函数中执行任何必要的错误处理。

### 配置运行时

1. 通过将类型添加到宏，在运行时启用pallet的特征。`ValidateUnsigned``ValidateUnsigned``construct_runtime`
   
   例如：
   
   ```rust
   construct_runtime!(
      pub enum Runtime where
          Block = Block,
          NodeBlock = opaque::Block,
          UncheckedExtrinsic = UncheckedExtrinsic
      {
          // ...
          OcwPallet: pallet_ocw::{Pallet, Call, Storage, Event<T>, ValidateUnsigned},
      }
   );
   ```

2. 实现运行时的特征，如中所述`SendTransactionTypes`[发送已签名的交易](https://docs.substrate.io/tutorials/build-application-logic/add-offchain-workers/#sending-signed-transactions).
   
   有关完整示例，请参阅 [ocw]（https://github.com/paritytech/substrate/blob/master/frame/examples/offchain-worker示例pallet。

## 已签名的有效负载

发送具有签名有效负载的未签名事务类似于发送未签名事务。 你需要：

- 实现pallet的特性。`ValidateUnsigned`
- 使用此pallet时，将类型添加到运行时。`ValidateUnsigned`
- 准备要签名的数据结构（已签名的有效负载），方法是实现[`SignedPayload`](https://paritytech.github.io/substrate/master/frame_system/offchain/trait.SignedPayload.html)特性。
- 发送带有签名有效负载的事务。

你可以参考以下部分[发送未签名的交易](https://docs.substrate.io/tutorials/build-application-logic/add-offchain-workers/#sending-unsigned-transactions)有关实现特征和将类型添加到运行时的详细信息。`ValidateUnsigned``ValidateUnsigned`

请记住，未签名的交易始终代表潜在的攻击媒介，如果没有额外的保护措施，就不能假定链下工作者是可靠的来源。 在大多数情况下，你应该实现限制性权限或其他逻辑来验证链下工作者提交的交易是否有效。

以下代码示例说明了发送未签名事务与发送具有已签名有效负载的未签名事务之间的差异。

要使数据结构可签名，请执行以下操作：

1. 实现[`SignedPayload`](https://paritytech.github.io/substrate/master/frame_system/offchain/trait.SignedPayload.html).
   
   例如：
   
   ```rust
   #[derive(Encode, Decode, Clone, PartialEq, Eq, RuntimeDebug, scale_info::TypeInfo)]
   pub struct Payload<Public> {
      number: u64,
      public: Public,
   }
   
   impl<T: SigningTypes> SignedPayload<T> for Payload<T::Public> {
      fn public(&self) -> T::Public {
      self.public.clone()
   }
   }
   ```

有关已签名有效负载的示例，请参阅[链下工作者](https://github.com/paritytech/substrate/blob/master/frame/examples/offchain-worker/src/lib.rs#L348-L361).

1. 在函数中，调用签名者，然后调用函数发送事务：`offchain_worker`
   
   ```rust
   #[pallet::hooks]
   impl<T: Config> Hooks<BlockNumberFor<T>> for Pallet<T> {
      /// Offchain worker entry point.
      fn offchain_worker(block_number: T::BlockNumber) {
          let value: u64 = 10;
   
          // Retrieve the signer to sign the payload
          let signer = Signer::<T, T::AuthorityId>::any_account();
   
          // `send_unsigned_transaction` is returning a type of `Option<(Account<T>, Result<(), ()>)>`.
          //     The returned result means:
          //     - `None`: no account is available for sending transaction
          //     - `Some((account, Ok(())))`: transaction is successfully sent
          //     - `Some((account, Err(())))`: error occurred when sending the transaction
          if let Some((_, res)) = signer.send_unsigned_transaction(
              // this line is to prepare and return payload
              |acct| Payload { number, public: acct.public.clone() },
              |payload, signature| RuntimeCall::some_extrinsics { payload, signature },
          ) {
              match res {
                  Ok(()) => log::info!("unsigned tx with signed payload successfully sent.");
                  Err(()) => log::error!("sending unsigned tx with signed payload failed.");
              };
          } else {
              // The case of `None`: no account is available for sending
              log::error!("No local account available");
          }
      }
   }
   ```
   
   此代码检索具有两个函数闭包的 then 调用。 第一个函数闭包返回要使用的有效负载。 第二个函数闭包返回链上调用，并传递有效负载和签名。 此调用返回结果类型以允许以下结果：`signer``send_unsigned_transaction()``Option<(Account<T>, Result<(), ()>)>`
   
   - `None`如果没有可用于发送交易的帐户。
   - `Some((account, Ok(())))`如果事务成功发送。
   - `Some((account, Err(())))`如果在发送事务时发生错误。

2. 检查提供的签名是否与用于对有效负载进行签名的公钥匹配：
   
   ```rust
   #[pallet::validate_unsigned]
   impl<T: Config> ValidateUnsigned for Pallet<T> {
      type Call = Call<T>;
   
      fn validate_unsigned(_source: TransactionSource, call: &Self::Call) -> TransactionValidity {
          let valid_tx = |provide| ValidTransaction::with_tag_prefix("ocw-demo")
              .priority(UNSIGNED_TXS_PRIORITY)
              .and_provides([&provide])
              .longevity(3)
              .propagate(true)
              .build();
   
          match call {
              RuntimeCall::unsigned_extrinsic_with_signed_payload {
              ref payload,
              ref signature
              } => {
              if !SignedPayload::<T>::verify::<T::AuthorityId>(payload, signature.clone()) {
                  return InvalidTransaction::BadProof.into();
              }
              valid_tx(b"unsigned_extrinsic_with_signed_payload".to_vec())
              },
              _ => InvalidTransaction::Call.into(),
          }
      }
   }
   ```
   
   此示例使用[`SignedPayload`](https://paritytech.github.io/substrate/master/frame_system/offchain/trait.SignedPayload.html)验证有效负载中的公钥是否与提供的签名相同。 但是，你应该注意，示例中的代码仅检查所提供的代码对于其中包含的键是否有效。 此检查不会验证签名者是ocw还是有权调用指定函数。 此简单检查不会阻止未经授权的参与者使用已签名的有效负载来修改状态。`signature``public``payload`
   
   有关此代码的工作示例，请参阅[链下函数调用](https://github.com/paritytech/substrate/blob/master/frame/examples/offchain-worker/src/lib.rs#L508-L536)和实施[`ValidateUnsigned`](https://github.com/paritytech/substrate/blob/master/frame/examples/offchain-worker/src/lib.rs#L305-L329).

## 下一步去哪里

本教程提供了如何使用ocw发送链上存储交易的简单示例。 要了解更多信息，请浏览以下资源：

- [链下操作](https://docs.substrate.io/learn/offchain-operations/)
- [ocw示例](https://github.com/paritytech/substrate/tree/master/frame/examples/offchain-worker)
- [链下工作者演示](https://github.com/jimmychu0807/substrate-offchain-worker-demo/tree/v2.0.0/pallets/ocw)
