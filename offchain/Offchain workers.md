# Offchain workers

*OCW*类别中的操作指南说明了链下操作的常见用例。

- [发出链下 HTTP 请求](https://docs.substrate.io/reference/how-to-guides/offchain-workers/offchain-http-requests/)
- [链下本地存储](https://docs.substrate.io/reference/how-to-guides/offchain-workers/offchain-local-storage/)
- [链下索引](https://docs.substrate.io/reference/how-to-guides/offchain-workers/offchain-indexing/)

需要注意的是，链下存储与链上存储是分开的。 你不能将OCW收集或处理的数据直接发送到链上存储。 要存储OCW收集或处理的任何数据（即修改链的状态），你必须允许OCW发送修改链上存储系统的交易。 有关如何准备链下OCW以发送交易的示例，请参阅[添加OCW](https://docs.substrate.io/tutorials/build-application-logic/add-offchain-workers/).
