# Chapter 05 Kernel Layer - Block and Chain

## 5.1 Introduction

​	Blockchain is a growing list of blocks that are linked in chronological order containing many transactions. This data is a typical link-list structure, all blocks are linked in one blockchain. Each block includes the cryptography hash of the prior block in the blockchain, linking the two. That cryptography hash comes from hashing(SHA256) the previous blockheader and can uniquely identifies a block. 

> 区块链是由包含交易信息的区块按照时间先后顺序依次链接起来的数据结构。这种数据结构是一个形象的链表结构，所有区块在有序的链接在同一条区块链上，每个区块通过一个哈希指针指向前一个区块。哈希指针其实是前一个区块头进行SHA256哈希得到。通过这个哈希值，可以唯一的识别一个区块。每个区块头都包含他的前一个区块头的哈希值。通过把每个区块链接到其区块头中前一个区块哈希值代表的区块后面就可以构建一条完整的区块链。

​	As blockchain only allows to link a block to the end of the chain, it is also said that it is like a stack, the first block is in the bottom and then add blocks above it in order, of course, the latest block is the top of stack. The data of blocks can be stored in relational database as well as non-relational database like LevelDB. There are some special words in blockchain, such as genesis block ( the first block of chain, also known as the bottom stack block), which is usually hard-coded and is mined by miners. The height of block: the number of blocks from genesis block to current block.

> 由于区块链是一种只允许从链尾添加区块到链表的结构，因此有人也将其比作一个数据栈，第一个区块作为栈底的首区块，之后的区块依次被放置到第一个区块之上，最新的区块作为栈顶。区块数据既可以保存在关系型数据库中，也可以保存在LevelDB这种非关系型数据库中。区块链中也有一些比较形象的描述词。例如：创世区块，也称“首区块”、“栈底区块”，指区块链上的第一个区块，创世区块一般都是硬编码生成的，需要矿工挖掘。区块高度：区块与创世区块之间的距离。 

​	The block structure is similar to HTTP request, consisting of `Header` and `Body`, storing some metadata and valid data respectively. `Header` is just the head of block, while `Body` is the main part of block which normally represents a block and stores all transaction information.

> 区块的结构和HTTP请求的结构类似，都是由一个Header和Body组成，Header中存储一些元数据，Body中存储有效数据。区块中的Header被称为区块头，区块头中保存了区块的元数据。区块Body也别称为区块，通常说的区块一般都是指的这部分，区块中保存的是所有的交易信息。

​	Although each block only has one parent block, one block may have many children blocks. It is known as forks, happens when many blocks take a same block as their parent block and will be found by miners. Forks are very common, but will not exist for a long time under the blockchain consensus that ensures one chain. Because of forks, the latest few blocks in blockchain might be changed by recomputing. In Bytom, the latest 6-8 blocks might be changed, but it is almost impossible over 6 blocks. Therefore, a transaction that is confirmed by six consecutive blocks can be seen a success. 

> 每个区块只能有一个父区块，但是一个区块有多个子区块是可能发生的。当多个区块将同一个区块作为其父区块时就会出现这种情况。这种情况通常被称为分叉。区块链分叉是一个比较常见的状态。当有多个区块同时被矿工发现时就会出现这种情况。分叉链并不会长久的存在，区块链的共识机制保证了最终只有一条链存在。由于分叉的存在，区块链中最近的几个区块很可能会因为分叉引发的重新计算而被修改。比原链中前6-8个区块可能会因为分叉而被修改。但是超过6个区块之后，被修改的可能性已经很小了。因此，一笔交易只有被连续6个块确认之后，我们才认为这笔交易是成功的。

​	Generally, because of the chain structure, currencies based on blockchain are believed to be very safe compared with traditional currencies. The `previous block hash` field is inside the block header and thereby affects the current block’s hash. The child’s own identity changes if the parent’s identity changes. This cascade effect ensures that once a block has many generations following it, it cannot be changed without forcing a recalculation of all subsequent blocks, while such a recalculation would require enormous computation which can hardly be met.

> 与传统货币相比，通常认为基于区块链的货币是十分安全的。主要就是由于区块链的这种链式结构决定的。由于每个区块头里面包含“父区块哈希值”字段，如果父区块的内容发生变化，将会迫使子区块内父区块哈希值字段发生改变，从而又将导致子区块的哈希值发生改变。一旦一个区块有很多代以后，想要修改区块内某一笔交易，将会引发瀑布式的效应，这种瀑布式的效应会导致之后的区块全部需要重新计算。这样的重新计算需要耗费巨大的计算量，通常没有任何人可以提供如此大的算力。

This chapter covers:

* Structure of the block and the chain
* The genesis Block
* Block validation, link to the blockchain and the difficulty target
* Orphan block management

> 本章主要内容如下：
>
> * 区块和链数据结构介绍
> * 创世区块数据结构及初始化
> * 区块验证、区块上链以及区块难度的计算方式
> * 孤块管理



## 5.2 Block (区块)

​	Data structure is defined into two layers: types layer (protocol/bc/types) and bc layer (protocol/bc). Such as `Tx` is defined in both of them, and transformed by `MapTx` method between two layers. Layers make the data structure more specific. 

* types layer: It is for raw data and transferred between nodes
* bc layer: It is for visual machine computing, especially for improving transactions validation. 

> 在比原链中，数据结构的定义是分层的：types层（protocol/bc/types）和bc层（protocol/bc）。比如Tx数据结构在types层和bc层同时定义。types层和bc层之间相互转换是通过MapTx方法。数据结构分层使得整体结构明确。
>
> * types层：是原始数据结构层，用于在节点与节点之间相互传输。
> * bc层：是虚拟机计算层，专门用于提高交易验证的性能。

 

### 5.2.1 Structure of a block (区块数据结构)

​	The block is made of a header, containing metadata, and a body, storing all transactions. A block can be seen as a ledger that records transactions and has a title page for brief introduction, namely, block header. Here is the `Block` struct:

> 区块(block)由区块头（Header）和区块体（Body）组成，区块头记录了区块的元数据信息，区块体中包含了打包的所有交易信息。我们可以把区块形象的类比成一本账薄，交易就是记录在账薄中的一笔笔转账记录，区块头则是账本的扉页，记录了账本的交易的概括信息。区块数据结构如下所示：

 

protocol/bc/block.go

```
type Block struct {
	*BlockHeader
	ID           Hash
	Transactions []*Tx
}
```

The structure of a block:

###### **chart 5-1**   **The structure of a block:**

| Field        | Bytes    | Description                                   |
| ------------ | -------- | --------------------------------------------- |
| BlockHeader  | Variable | Several fields form the block header          |
| ID           | 32       | Block hash which can identify a certain block |
| Transactions | Variable | The transactions recorded in this block       |

### 5.2.2 Structure of Block Header (区块头数据结构)

​	The block header consists of three parts. The first is a reference to a previous block hash, which connects this block to the previous block in the blockchain. The second part relates to mining, including the `Timestamp`, `Noce` and `Bits`. Miner need to compute the `Nonce` until it meets the `Bits`. The last part is the merkle tree root, a data structure used to efficiently summarize all the transactions in the block. Here is the `Block Header` structure: 

> 区块头主要由三大部分组成：区块链的链接信息、挖矿信息、区块交易信息。区块链的链接信息是指区块中引用的父区块哈希值，用于将区块与区块链中前一区块相连接。与挖矿相关的信息主要包括：Timestamp、Nonce和Bits。挖矿时，矿机可以不断的修改Nonce值，计算满足Bits要求的区块头。区块的交易信息主要是区块中交易的Merkle树。比原区块头保存了两个Merkle树：区块中所有交易的哈希值组成的交易信息的Merkle树，区块中所有交易的验证结果组成的交易结果的Merkle树。区块头的数据结构如下所示：

 protocol/bc/bc.pb.go

```
type BlockHeader struct {
	Version               uint64
	Height                uint64
	PreviousBlockId       *Hash
	Timestamp             uint64
	TransactionsRoot      *Hash
	TransactionStatusHash *Hash
	Nonce                 uint64
	Bits                  uint64
	TransactionStatus     *TransactionStatus
}
```

 

The structure of a block header:

###### **chart5-2** **The structure of a block header:**

| Field                 | Bytes    | Description                                                  |
| --------------------- | -------- | ------------------------------------------------------------ |
| Version               | 8        | A version number to track software/protocol upgrades         |
| Height                | 8        | The height of the block, counted from the genesis block      |
| PreviousBlockId       | 32       | A reference to the hash of the previous (parent) block in the chain |
| Timestamp             | 8        | The approximate creation time of this block (seconds from Unix Epoch) |
| TransactionsRoot      | 32       | A hash of the root of the merkle tree of this block’s transactions |
| TransactionStatusHash | 32       | A hash of the root of the merkle tree of validation results of this block’s transactions |
| Nonce                 | 8        | A counter used for the Proof-of-Work algorithm               |
| Bits                  | 8        | The Proof-of-Work algorithm difficulty target for this block |
| TransactionStatus     | Variable | The status of transaction after being validated              |

 

### 5.2.3 Block Identifiers (区块标识符)

​	The block identifier can identify a block uniquely. Block identifiers include block header hash and block Height.

> 区块标识符是区块的唯一标识。根据区块标识符，我们可以唯一确定一个区块。有两种区块标识符：区块头哈希值和区块高度。

The block header hash is made by hashing the block header twice through the SHA256 algorithm. The resulting 32- byte hash is called the *block hash* but is more accurately the *block header hash*. The SHA256 algorithm6 is a digtial signature algorithm defined by digtial signature standard, mainly used to compute the digest of message. No matter many bytes the input is, there will always be a 32- byte hash computed by the SHA256 algorithm. After receiving the message, it can be checked by this 32-byte hash. A tampered message can be catch since its 32-byte hash has been changed. 

> 区块头哈希值通过使用SHA256算法对区块头进行二次哈希得到32位散列值。区块头哈希值也被成为“区块哈希值”。SHA256算法是数字签名标准中的定义的数字签名算法，主要用于计算消息的摘要信息。对于任意长度的消息经过SHA256都会生成一个32字节长度的摘要数据。接收端收到消息时，可以根据摘要信息验证数据是否发生篡改。为什么我们根据摘要信息就可以判断消息是否发生改变？这主要是由SHA256算法的性质决定。

​	Features of the SHA256 algorithm:

* No one can get the original message by the 32- byte hash 
* Messages that are different can not produce a same 32- byte hash 

> SHA256算法有如下特性：
>
> * 任何人都不能从摘要信息中复原原始消息。
> * 两个不同的消息不会产生相同的消息摘要。

​	Because of the second feature, this 32-byte hash is also called digital fingerprint and identify a block uniquely.

> 由SHA256算法的性质2，我们也将生成的消息摘要称为消息指纹。由此每个区块头的哈希值也是不同的。我们就可以通过区块头的哈希值唯一确定一个区块。

​	Note that the block hash is not actually included inside the block’s data structure, and there will not go through all blocks from the highest and check their `PreviousBlockHash` . Instead, the block hash is computed by each node as the block is received from the network. The block hash is stored in a LevelDB as the `key`, to facilitate indexing and faster retrieval of blocks from disk, and the block is its `value`.

> 还有一点需要注意，区块头哈希值并不保存在当前区块的数据结构中。那我们又是如何通过区块哈希值检索区块的呢。总不能从最高区块开始遍历所有区块，查找哪个区块头的PreviousBlockHash等于要检索的区块。如果这样想那就错了。实际上，当节点从网络中同步一个区块的时候，节点会计算区块头哈希值，区块的哈希值会作为key，区块数据作为value存储到LevelDB中。这样我们就可以通过区块哈希值快速检索到对应的区块。

​	A second way to identify a block is by its position in the blockchain, called the *block height*. But unlike the block hash, the block height is not a unique identifier. That because of the blockchain forks, two or more blocks might have the same block height, competing for the same position in the blockchain. In this scenario, their heights are totally same, while the block hash differs from each other. Obviously, the height can not identify a block uniquely. Blockchain soft fork is just a temporary status, ending up being selected by the consensus algorithm. It is generally believed that in the height smaller than `the highest - 6`, there is not the blockchain soft forks 

> 除区块头哈希值可以唯一确定一个区块外，在某些情景下，通过区块高度也可以唯一确定一个区块。为什么是有些情景下才能唯一确定呢？这主要是由于区块软分叉的原因。我们知道当有多个区块同时被矿工发现时，这时候产生的多个区块都会引用同一个父区块，虽然他们的区块头哈希值是不同的，但是他们此时高度是相同的。这时候我们通过区块高度是无法唯一确定一个区块的。区块软分叉是一个临时状态，当分叉数达到一定阈值时，共识算法会比较每条分叉的工作量，保留工作量最多的那条分叉。我们通常认为区块高度比当前最高区块高度小于6的区块高度不存在软分叉。

### 5.2.4 The Genesis Block (创世区块)

​	The first block in the blockchain is called the genesis block.It is the common ancestor of all the blocks in the blockchain, meaning that if you start at any block and follow the chain backward in time, you will eventually arrive at the genesis block. Every node always starts with a blockchain of at least one block because the genesis block is statically encoded within the core of the code, such that it cannot be altered. 

> 区块链中的第一个区块创被称为创世区块。它是区块链里面所有区块的共同祖先。一般情况下创世区块被硬编码到代码核心中，每一个节点都始于同一个创世区块，这能确保创世区块不会被改变。每个节点都把创世区块作为区块链的首区块，从而构建了一个安全的、可信的区块链。

​	The genesis block can be got by `bytomcli` commandline tool, here it is :

> 通过比原命令行工具bytomcli获取创世区块的信息。命令执行如下：

```
$ ./bytomcli get-block 0

{
  "bits": 2161727821137910500,
  "difficulty": "15154807",
  "hash": "a75483474799ea1aa6bb910a1a5025b4372bf20bef20f246a2c2dc5e12e8a053",
  "height": 0,
  "nonce": 9253507043297,
  "previous_block_hash": "0000000000000000000000000000000000000000000000000000000000000000",
  "size": 546,
  "timestamp": 1524549600,
  "transaction_merkle_root": "58e45ceb675a0b3d7ad3ab9d4288048789de8194e9766b26d8f42fdb624d4390",
  "transaction_status_hash": "c9c377e5192668bc0a367e4a4764f11e7c725ecced1d7b6a492974fab1b6d5bc",
  "transactions": [
  // ...
  ],
  "version": 1
}
```

​	The description of the genesis block:

* Bits: The target. Only if the miner produces a hash that is less than the target, the block can be packaged successfully by him. Although the target of genesis block is *216172782113791050* , it is not produced by miners but statically encoded within the block, so in fact it can be written whatever.
* Difficulty：The difficulty, computed from bits. One can estimate the amount of work it takes to succeed from the difficulty.
* Height: The height of the block. The genesis block is height-0, which means blocks are counted from height-0.
* Nonce: A counter used for the Proof-of-Work algorithm. It is used to vary the output of a cryptographic function to meet the bits.
* previous_block_hash：A reference to the hash of the previous (parent) block in the chain. Since the genesis block is the first block and doesn't have a parent block, here is statically encoded with *000000...000000* by default.
* timestamp：The approximate creation time of this block. The genesis block's is `152454960` (2018-04-24 14:00:00), namely the time of Bytom Mainnet implementation. 
* transaction_merkle_root: A hash of the root of the merkle tree of this block’s transactions. It is used to validate the integrity of transactions in this block.
* transaction_status_hash: A hash of the root of the merkle tree of validation results of this block’s transactions.
* transactions: The list of transactions in this block.

> 创世区块说明：
>
> * bits：目标难度值，挖矿时计算的hash之后要小于或等于该值，则矿工节点才能打包交易构建区块。创世区块的难度值为2161727821137910500，但是创世区块不是矿工节点挖出来的，而是一个硬编码的区块。所以该难度值没有任何意义，理论上是可以随便写的。
> * difficulty：难度值，由bits值计算出来，我们只看bits值是很难判断某次挖矿的难度的，而通过难度值可以比较直观的比较出两个区块在挖矿时的难度。
> * height：区块高度，创世区块高度为0，区块高度从0开始计数。
> * nonce：随机数，挖矿时矿工节点反复尝试不同的nonce值来生成小于等于bits的区块头哈希值。
> * previous_block_hash：当前区块的父区块hash值，由于创世区块是第一个块，它是不存在父区块的，因此在硬编码时，给了他一个默认的区块哈希值为000000...000000
> * timestamp：区块生成时间。创世区块的时间戳为1524549600，标准时间为2018-04-24 14:00:00，这个时间也是比原主网上线的时间。
> * transaction_merkle_root：创世区块的merkle树根节点，根据交易生成不同的merkle树根节点，用于验证当前区块的交易是否完整。
> * transaction_status_hash：区块交易状态哈希值。
> * transactions：当前块中的交易信息。

 

### 5.2.5 Generate the Genesis Block (生成创世区块)

​	When starting a node, it will firstly try to get the blocks information from local Level DB. If the node didn't get blocks, it would regard itself as a new node and initialize the local blockchain according to the statically encoded genesis block. Here is the code to generate the genesis block: 

> 当我们启动一个节点的时候，节点会先尝试从本地LevelDB数据库中加载区块信息。如果节点没有从LevelDB中获取到区块，则节点会认为自己是一个全新的节点。此时，节点会根据硬编码的创世区块信息初始化本地链。硬编码的创世区块信息如下所示： 

config/genesis.go

```
func mainNetGenesisBlock() *types.Block {
   tx := genesisTx()
   txStatus := bc.NewTransactionStatus()
   if err := txStatus.SetStatus(0, false); err != nil {
      log.Panicf(err.Error())
   }
   txStatusHash, err := types.TxStatusMerkleRoot(txStatus.VerifyStatus)
   // ...

   merkleRoot, err := types.TxMerkleRoot([]*bc.Tx{tx.Tx})
   // ...
   block := &types.Block{
      BlockHeader: types.BlockHeader{
         Version:   1,
         Height:    0,
         Nonce:     9253507043297,
         Timestamp: 1524549600,
         Bits:      2161727821137910632,
         BlockCommitment: types.BlockCommitment{
            TransactionsMerkleRoot: merkleRoot,
            TransactionStatusHash:  txStatusHash,
         },
      },
      Transactions: []*types.Tx{tx},
   }
   return block
}
```

​	`mainNetGenesisBlock()` function is used to generate the genesis block. Firstly, a coinbase transaction is produced by `genesisTx()` function and here puts additional information, *Information is power. -- Jan/11/2013. Computing is power. -- Apr/24/2018.*, in this coinbase transaction to memorize the computer genius *Aaron Swart* for his innovative spirit of openning network. There are about 1.4 billion BTM in the output of this coinbase transaction bound with a lock script *00148c9d063ff74ee6d9ffa88d83aeb038068366c4c4*.  These 1.4 billion BTM is kept by the Bytom official, partly used to exchange tokens with the exchanges.

> mainNetGenesisBlock方法用来生成创世区块。在mainNetGenesisBlock方法中首先通过genesisTx方法构造一笔coinbase交易。coinbase交易输入中附加信息为“Information is power. -- Jan/11/2013. Computing is power. -- Apr/24/2018.”。主要是为了纪念计算机天才Aaron Swartz，纪念他致力于网络信息开放的创新精神。coinbase交易的输出为140700041250000000/1e8 = 1407000412.5，大约为14亿BTM币。这些币的锁定脚本为00148c9d063ff74ee6d9ffa88d83aeb038068366c4c4。这些币是为官方预留的，部分用于与交易所兑换代币。

 	After building the coinbase transaction, the node will set the coinbase transaction validation status successful. Here usually has a misunderstanding, why it is successful with a `false` argument? That because Bytom uses `StatusFail` to record transaction status (succeed or fail), and setting `false` means the transaction is successful. Later, the node will generate a hash of the root of the merkle tree of the genesis block’s transactions to build the block.

> 构造完coinbase交易后，节点将coinbase交易的验证状态设置为成功。这里可能有个误解，明明传入的值是false，怎么会设置为成功呢。这是因为比原链中使用StatusFail记录交易的状态，记录交易是否是失败的。设置为false则说明交易是成功的。之后，节点根据交易信息生成Merkle树根节点，构造区块结构。

 

### 5.2.6 Blocks Validation (区块验证)

​	Blocks validation is vital to ensure the blockchian doesn't have forks. If there is no validation, many forks will be produced in the process of blocks sychrolization between nodes. Forks are great threats to the safety of assets in the blockchain.

> 区块验证是保证区块链不产生分叉的重要手段。如果没有区块验证的过程，则节点间在同步区块的过程中，会产生较多的分叉。我们知道分叉对于链上资金和财产安全具有极大的威胁。

​	Usually, blocks are validated in the following situations:

* Once a mining node successfully mines a block and submits it to the blockchain, this block will be validated firstly.
* When users submit blocks to the blockchain by API, the node validate blocks.
* In the process of syncing blocks, every time the node receives blocks from others, it will validate blocks and only link valid blocks to the local blockchain.
* When mining nodes submit their work to the mining pool, the mining pool will validate the blocks they submit.

> 一般会在以下四种情况下对区块进行验证：
>
> * 挖矿节点在成功挖掘到一个区块后，向链上提交区块时，节点会先验证区块是否合法。
> * 用户通过API接口向节点提交区块到区块链时，节点也会验证区块是否合法。
> * 同步区块时，节点收到其他节点同步过来的区块，节点会先验证同步的区块是否合法，如果合法将其加入到本地链。
> * 矿池中的节点向矿池提交工作时，矿池会验证矿机提交的区块。

​	In order to improve the efficiency of  blocks synchronization, Bytom offers a safe, simple also fast methond to validate blocks compared with *Etherum*. The blocks validation strategy of *Etherum* is a little complicated, which always makes machine beep  when the blocks validation starts. Here is the code: 

> 相比以太坊等区块链，比原链为了提高区块的同步效率，提供了一种即安全又简单快速的验证方法。以太坊的区块验证策略比较复杂，每次节点在同步区块时，机器都会嗡嗡作响。代码示例如下：

protocol/block.go

```
func (c *Chain) processBlock(block *types.Block) (bool, error) {
   blockHash := block.Hash()
   if c.BlockExist(&blockHash) {
      log.WithFields(log.Fields{"hash": blockHash.String(), "height": block.Height}).Info("block has been processed")
      return c.orphanManage.BlockExist(&blockHash), nil
   }
   if parent := c.index.GetNode(&block.PreviousBlockHash); parent == nil {
      c.orphanManage.Add(block)
      return true, nil
   }
   if err := c.saveBlock(block); err != nil {
      return false, err
   }
   bestBlock := c.saveSubBlock(block)
   bestBlockHash := bestBlock.Hash()
   bestNode := c.index.GetNode(&bestBlockHash)
   if bestNode.Parent == c.bestNode {
      log.Debug("append block to the end of mainchain")
      return false, c.connectBlock(bestBlock)
   }
   if bestNode.Height > c.bestNode.Height && bestNode.WorkSum.Cmp(c.bestNode.WorkSum) >= 0 {
      log.Debug("start to reorganize chain")
      return false, c.reorganizeChain(bestNode)
   }
   return false, nil
}
```

​	`processBlock()` function works in the following steps:

1) Compute the block hash and check whether the block exists in the local blockchain or orphan blocks pool by its hash. If the block has been in the local blockchain, then it would be thrown away directly.

2) Find the block's parent block in the local blockchain according to its `PreviousBlockHash`. If the parent block doesn't exist, this block will be put into orphan block pool.

3) If its parent block exists in the local blockchain, link this block to the local blockchain after verifing its transactions and metadata.

4) Check if there are blocks in orphan block pool that can be linked to the blockchain. If there are, link the blocks to the blockchain in some order and then get the highest block `bestNode`.

5) If the `bestNode`'s  parent block is the highest block `c.betNode` before this operation, link the `bestNode` after `c.betNode`. If not, that means the `bestNode` is in the sidechain instead of the main blockchain.

6）If the `bestNode` is in the sidechain and its height is over the main blockchain, which means the work of this sidechain is more than the main blockchain, the node will reorganize the blockchain.

> processBlock函数处理过程：
>
> 1) 计算区块的哈希值，根据区块哈希值，查看本地链或孤块池中是否已经存在该区块。如果本地已经存在了相同的区块，则将区块直接丢弃。
>
> 2) 根据区块中PreviousBlockHash从本地链中查找父区块。如果本地链中没有其父区块，则将区块加入孤块池中。
>
> 3) 本地链中存在父区块，则将区块加入本地链。但是区块在加入本地链时，需要对区块中的交易信息和元数据进行验证。
>
> 4) 查找孤块池中是否存在可以上链的孤块。如果存在，则将孤块池中的孤块依次上链，并得到此时链上最高块bestNode。
>
> 5) 如果bestNode的父区块等于本次操作之前链上的最高区块c.betNode，则将bestNode连接到c.betNode之后。否则说明bestNode并不在主链上，而是被添加到了侧链上。
>
> 6) 如果bestNode被添加到侧链上，其高度大于主链的高度，工作量之和超过了主链的工作量，节点会重组区块链。

​	In step (3), it needs to validate the block header before linking the block to the blockchain. This work covers 5 parts. Here is the code:

> 在步骤3中，区块存储到链上时，需要对区块的头内容进行验证，区块头内容的验证主要分为以下5部分。代码示例如下：

 protocol/validation/block.go

```
func ValidateBlockHeader(b *bc.Block, parent *state.BlockNode) error {
   if b.Version < parent.Version {
      // ...
   }
   if b.Height != parent.Height+1 {
      // ...
   }
   if b.Bits != parent.CalcNextBits() {
      // ...
   }
   if parent.Hash != *b.PreviousBlockId {
      // ...
   }
   if err := checkBlockTime(b, parent); err != nil {
      // ...
   }
   if !difficulty.CheckProofOfWork(&b.ID, parent.CalcNextSeed(), b.BlockHeader.Bits) {
      // ...
   }
   return nil
}
```

​	The block header validation includes the following work:

- Check if the version of the block equals to the version of its parent block
- Check if the height of the block is the height of the parent block plus 1
- Check if the block target equals the next block target that is calculated by the parent block height. 
- Check if the block's `Previous Block Hash` is the hash of its parent block
- Check that the timestamp isn't later more than one hours compared with the current time and not earlier than the intermediate time of the last 11 blocks.
- Check if the hash of the block equals to the target.

> 区块头验证主要包括以下几部分：
>
> * 当前区块的版本号要与父区块的版本号相同。
> * 当前区块的高度只能比父区块的高度大1。
> * 当前区块的难度目标需要等于根据父区块高度计算出的下个区块的难度目标。
> * 当前区块的父区块的哈希值要等于父区块的哈希值。
> * 当前区块的时间戳不能比当前时间延后一小时，并且不能早于之前11个区块时间戳的中位数。
> * 验证区块的哈希值是否等于给定的难度目标。

### 5.2.7 Calculate the Next Target (计算下一个区块的难度目标)

​	Here is the code that calculates the next target according the parent block height: 

> 根据父区块高度计算下一个区块的难度目标算法：

 protocol/state/blockindex.go

```
func (node *BlockNode) CalcNextBits() uint64 {
   if node.Height%consensus.BlocksPerRetarget != 0 || node.Height == 0 {
      return node.Bits
   }

   compareNode := node.Parent
   for compareNode.Height%consensus.BlocksPerRetarget != 0 {
      compareNode = compareNode.Parent
   }
   return difficulty.CalcNextRequiredDifficulty(node.BlockHeader(), compareNode.BlockHeader())
}
```

**Algorithm**:  If `the parent block height` *mod* `consensus.BlocksPerRetarget（2016)` doesn't equal 0 as well as the parent block height, the next target will be the same as the parent block's. Or the target will be adjusted. See more detials about `retarget` in the consensus part.

> **算法描述**：如果父区块对consensus.BlocksPerRetarget（2016）取模不等于0并且父区块的高度也不等于0，则下个区块的难度与父区块的难度相同。否则进行难度动态调整（retarget），更多关于难度动态调整内容，请读者参考共识章节。

### **5.2.8 Orphan Block Management (孤块管理)**

​	If a valid block is received and no parent is found in the existing chains, that block is considered an “orphan.” Orphan blocks are saved in the orphan block pool where they will stay until their parent is received. Once the parent is received and linked into the existing chains, the orphan can be pulled out of the orphan pool and linked to the parent, making it part of a chain. 

> 当节点收到了一个有效的区块，而在现有的主链中却未找到它的父区块，那么这个区块是“孤块”。接收到的孤块会存储在孤块池中，直到它们的父区块被节点收到。一旦收到了父区块，节点就会将孤块从孤块池中取出，并且连接到它的父区块，让它作为区块链的一部分。

​	Orphan blocks usually occur when two blocks that were mined within a short time of each other are received in reverse order (child before parent). Here assumes that there are three blocks and their heights are 100, 101, 102 respectively, then they are received by the node in the order of 102, 101, 100. In this situation, the node will put 102, 101 blocks into orphan block pool where they will stay until their parent is received. Once the 100 block is received and linked to the existing chain after validation, those two orphan blocks will be found by a recursive function, and linked to the parent.

> 当两个或多个区块在很短的时间间隔内被挖出来，节点有可能会以不同的顺序接收到它们，这个时候孤块现象就会出现。我们假设有三个高度分别为100、101、102的块，分别以102、101、100的颠倒顺序被节点接收。此时节点将102、101放入到孤块管理缓存池中，等待彼此的父块。当高度为100的区块被同步进来时，会被验证区块和交易，然后存储到区块链上。这时会对孤块缓存池进行递归查询，根据高度为100的区块找到101的区块并存储到区块链上，再根据高度为101的区块找到102的区块并存储到区块链上。

​	Here is the orphan block structure:

> 孤块结构如下所示：

protocol/orphan_manage.go

```
type orphanBlock struct {
   *types.Block
   expiration time.Time
}
```

​	The description of orphan block:

* types.Block: The orphan block structure.
* Expiration: The expiration of the orphan block, which means if this orphan block doesn't be linked to the blockchain before its expiration, it will be thrown away.

> 孤块结构说明：
>
> * types.Block：孤块的区块结构。
> * Expiration：孤块超时时间，如果一个孤块到Expiration时间仍然没有上链，则这个孤块将会被丢弃。

​	Here is the orphan block pool structure :

> 孤块管理缓存池结构体如下所示：

```
type OrphanManage struct {
   orphan      map[bc.Hash]*orphanBlock
   prevOrphans map[bc.Hash][]*bc.Hash
   mtx         sync.RWMutex
}
```

​	The description of orphan block pool:

* Orphan: Used to store orphan blocks, key is block hash and value is the block.
* prevOrphans: Orphan block's parent block.
* Mtx: Exclusive lock, used to ensure the consistency of `map` structure data in multiple concurrency.

> 孤块管理缓存池结构说明：
>
> * Orphan：用于存储孤块，key为block hash，value为孤块结构体。
> * prevOrphans：用于孤块的父块。
> * Mtx： 互斥锁，保护map结构在多并发读写状态下保持数据一致。

​	The orphan block manager has main functions as follow:

* BlockExist(): Check if a block is orphan
* Add(): Add new orphan blocks into the orphan block pool
* Delete(): Delete an orphan block from the orphan block pool
* Get(): Get a block from the orphan block pool according to its hash
* GetPrevOrphans(): Get all blocks that have a same specific parent block hash from the orphan block pool.
* orphanExpireWorker(): A timer that sets the time to clean the timeout orphan blocks
* orphanExpire(): Used to clean the timeout orphan block

> 孤块管理器的主要方法如下：
>
> * BlockExist()：判断一个区块是否是孤块。
> * Add()：向孤块池中新增孤块。
> * Delete()：从孤块池中删除孤块。
> * Get()：根据区块哈希从孤块池中获取孤块。
> * GetPrevOrphans()：从孤块池中获取所有父区块哈希值为hash的孤块。
> * orphanExpireWorker()：定时器，定时清理超时孤块。
> * orphanExpire()：清理超时孤块的工作方法。



1) Here is the code that adds an orphan block into the orphan block pool:

> 1）添加孤块到孤块池，代码示例如下：

```
func (o *OrphanManage) Add(block *types.Block) {
   blockHash := block.Hash()
   o.mtx.Lock()
   defer o.mtx.Unlock()
   if _, ok := o.orphan[blockHash]; ok {
      return
   }
   o.orphan[blockHash] = &orphanBlock{block, time.Now().Add(orphanBlockTTL)}
   o.prevOrphans[block.PreviousBlockHash] = append(o.prevOrphans[block.PreviousBlockHash], &blockHash) 
}
```

​	Adding a orphan block means not only putting the orphan block into the orphan block pool, also recording the relationship between the orphan block and its parent block, so that the orphan blocks that have the same parent can be found easily.

> 添加孤块到孤块管理池的方法中不仅仅是将孤块加入孤块管理池，而且还需要在管理器中记录父区块与孤块的对应关系。这步主要是为了在孤块上链时可以快速找到以当前孤块作为父区块的其他孤块。

2) Here is the code that deletes an orphan block from the orphan block pool: 

> 2）从孤块池中删除孤块，代码示例如下：

```
func (o *OrphanManage) delete(hash *bc.Hash) {
   block, ok := o.orphan[*hash]
   if !ok {
      return
   }
   delete(o.orphan, *hash)
   prevOrphans, ok := o.prevOrphans[block.Block.PreviousBlockHash]
   if !ok || len(prevOrphans) == 1 {
      delete(o.prevOrphans, block.Block.PreviousBlockHash)
      return
   }
   for i, preOrphan := range prevOrphans {
      if preOrphan == hash {
         o.prevOrphans[block.Block.PreviousBlockHash] = append(prevOrphans[:i], prevOrphans[i+1:]...)
         return
      }
   }
}
```

​	Accordingly,deleting an orphan block from the orphan block pool needs to delete the orphan block as well as its recorded relationship.

> 同样从孤块池中删除孤块的时候，不仅需要清除孤块池中的记录，也需要请求孤块管理器中父区块与孤块对应关系的记录。

​	Here is the code that cleans the orphan blocks regularly:

> 3）定时清理孤块，代码示例如下：

```
orphanExpireScanInterval = 3 * time.Minute

func (o *OrphanManage) orphanExpireWorker() {
   ticker := time.NewTicker(orphanExpireWorkInterval) //3分钟
   for now := range ticker.C {
      o.orphanExpire(now)
   }
   ticker.Stop()
}

func (o *OrphanManage) orphanExpire(now time.Time) {
   o.mtx.Lock()
   defer o.mtx.Unlock()
   for hash, orphan := range o.orphan {
      if orphan.expiration.Before(now) { //孤块超时时间60分钟
         o.delete(&hash)
      }
   }
}
```

​	Regular orphan block cleaning is for preventing a lot of useless orphan blocks keeping using memory and wasting resources. There might be some blocks that are produced by the malicious nodes and have no existed parent block, so if these blocks are not cleaned up, it will keep occupying a lot local memory. Therefore, orphan block manager starts a goroutine to clean them regularly, running a timer every 3 minutes to clean the orphan blocks that have existed over 60 minutes.

> 定时清理孤块是为了防止大量无用的孤块长期占用内存，浪费资源。孤块中很可能会存在一些恶意节点构造的恶意块，这些块很可能已用了一个不存在的父区块，如果不清理这些区块，将会导致节点内存暴涨。孤块管理器会使用一个gorouting定时清理超时的孤块。定时器每3分钟执行一次，清理60分钟之前加入孤块池中的孤块。

 

## 5.3 Blockchain (区块链)

​	The blockchain data structure is an ordered, back-linked list of blocks of transactions. Here are more details about blockchain data structure in the following part.

> 区块是区块链上的一个元素，区块链按照时间顺序将区块顺序相连，最终形成一条以时间为序列的链表结构。下面详细讲解区块链相关的核心数据结构。

### 5.3.1 The Blockchain Data Structure  (区块链数据结构)

​	The chain-related functions are mainly done by `Chain` structure. `Chain` offers many functions, such as initializing chain, getting the block according to a specific height or hash, getting the current blockchain height, validating the blocks, link and reorganize blocks, etc. Here is the structure of `Chain`: 

> 链上操作主要通过定义Chain结构来实现，Chain提供了多种链上操作的方法：链的初始化、获取链上指定高度或者哈希值的区块、获取当前链的高度、验证提交到链上的区块、链上区块的链接以及链重组等操作。Chain结构如下所示：

protocol/protocol.go

```
type Chain struct {
	index          *state.BlockIndex
	orphanManage   *OrphanManage
	txPool         *TxPool
	store          Store
	processBlockCh chan *processBlockMsg

	cond     sync.Cond
	bestNode *state.BlockNode
}
```

​	The description of `Chain` structure:

* Index: The index of block that is stored in the memory. It is designed to be a tree structure to seek blocks quickly.
* orphanManage: Orphan block manager.
* txPool: Transaction pool.
* Store: The interface of blocks persistent storage.
* processBlockCh: The cache of blocks that wait for validation.
* cond: It is a structure defined in the `sync` package of GoLang, used as the concurrency control between goroutines.
* bestNode: Record the highest block of the current blockchain.

> Chain结构说明：
>
> * index: 存储在内存中的区块索引，该索引被构造成一个树形结构，可以快速查找区块。
> * orphanManage: 孤块管理器。
> * txPool: 交易池。
> * store: 区块持久化存储接口。
> * processBlockCh: 待验证区块的缓存队列。
> * cond: GO语言sync包中Cond结构，用于goroutine之间并发控制。
> * bestNode: 记录当前主链的最高区块。

​	The functions of chain is the most important part in Bytom, which realize many operations, such as the transaction validation, the blocks linkage and validation, etc. Here are more details:

* func (c *Chain) BestBlockHeight()：Get the height of the main blockchain
* func (c *Chain) BestBlockHash()：Get the highest block of the current main blockchain
* func (c *Chain) BestBlockHeader()：Get  the highest block's header of the current main blockchain
* func (c *Chain) InMainChain()：Check if the block is in the main blockchain
* func (c *Chain) CalcNextSeed()：Calculate the seed in Tensority algorithm  
* func (c *Chain) CalcNextBits()：Calculate the next bits according to the parent block information
* func (c *Chain) GetTransactionStatus()：Get the status of all transactions in the block
* func (c *Chain) GetTransactionsUtxo()：Get UTXOs referred by all transactions in the block 
* func (c *Chain) ValidateTx()：Validate transactions
* func (c *Chain) ProcessBlock() ：Process blocks. Blocks that are validated successfully will be link to the main blockchain 

> 链上的操作方法是整个比原系统中最重要的一部分，很多重要的方法都在这里实现了，例如交易的验证、区块链接、区块校验等等。具体方法如下所示：
>
> * func (c *Chain) BestBlockHeight()：获取主链的高度
> * func (c *Chain) BestBlockHash()：获取主链目前高度最高的区块
> * func (c *Chain) BestBlockHeader()：获取主链目前高度最高的区块头
> * func (c *Chain) InMainChain()：判断区块是否在主链上
> * func (c *Chain) CalcNextSeed()：计算Tensority算法的seed
> * func (c *Chain) CalcNextBits()：根据前父区块计算下一个区块的难度值
> * func (c *Chain) GetTransactionStatus()：获取区块中所有交易的状态
> * func (c *Chain) GetTransactionsUtxo()：获取区块中所有交易引用的urxo
> * func (c *Chain) ValidateTx()：验证交易
> * func (c *Chain) ProcessBlock() ：处理区块，校验通过的区块将被连接到主链上

 

### 5.3.2 Add Blocks into the Blockchain (区块上链)

​	It is clear that the block will be linked to the blockchain after a series operations of validation mentioned in the block validation part. But there is no more details about how the blocks are store into the blockchain. 

​	In this section, we will go to the details. This work is mainly done by `func (c *Chain) saveBlock(block *types.Block)` function in `protocol/block.go`, and the `saveBlock()` function works in the following steps.

> 在区块验证部分，我们讲到区块在完成一系列的验证步骤后，就会被存储到链上，但是区块具体的存储方法，我们没有详细讲解。本小节，我们将会介绍区块是怎么存储到链上的。该部分主要由protocol/block.go中的func (c *Chain) saveBlock(block *types.Block)  方法实现。saveBlock方法主要依靠以下几个步骤将区块上链。

**Step 1**: Store the block into the local LevelDB.

​	In the `saveBlock()` function of the `Chain`, the block will be stored into the local LevelDB by the `SaveBlock()` interface after being validated. Here is the code of this interface : 

> **步骤1，**将区块存储到本地LevelDB数据库中。
>
> 在Chain的saveBlock方法中，在完成区块的校验后，会通过LevelDB提供的SaveBlock接口将区块存储到本地LevelDB数据库中。下面我们看一下这个接口的具体实现。代码示例如下：

database/leveldb/store.go

```
func (s *Store) SaveBlock(block *types.Block, ts *bc.TransactionStatus) error {
   binaryBlock, err := block.MarshalText()
   // ...
   binaryBlockHeader, err := block.BlockHeader.MarshalText()
   // ...
   binaryTxStatus, err := proto.Marshal(ts)
   // ...
   blockHash := block.Hash()
   batch := s.db.NewBatch()
   batch.Set(calcBlockKey(&blockHash), binaryBlock)
   batch.Set(calcBlockHeaderKey(block.Height, &blockHash), binaryBlockHeader)
   batch.Set(calcTxStatusKey(&blockHash), binaryTxStatus)
   batch.Write()
   
   return nil
}
```

​	In the `SaveBlock()` interface, there are three types of block information needed to be stored into the LevelDB. The first is the serialized information of the whole block, using  `B`  as its prefix of key, follwed by the block hash. The second is the serialized information of the block header, using  `BH`  as its prefix of key, follwed by the block height and hash. The third is the serialized information of all transactions is the block, using  `BTS`  as its prefix of key, follwed by the block hash. 

> 在SaveBlock接口中，需要将区块的三种信息存储到LevelDB中。第一个是完整区块的序列化信息。这些信息的key将会以“B:”作为前缀，后接区块哈希值；第二个则是区块头的序列化信息。这些信息的key将会以”BH”作为前缀，后接区块高度和区块哈希值；第三种信息是区块中所有交易状态的序列化信息。这些信息的key将会以“BTS:”作为前缀，后接区块哈希值。 

**Step 2**: Delete a block from the orphan block pool that has been stored into the LevelDB.

> **步骤2**，从孤块池中删除一存储到LevelDB数据库中的区块。

```
c.orphanManage.Delete(&bcBlock.ID)
```

**Step 3**: Update the new block into the memory of block index.

> **步骤3，**将区块更新到内存中的区块索引上，代码如下：

```
node, err := state.NewBlockNode(&block.BlockHeader, parent)
if err != nil {
   return err
}

c.index.AddNode(node)
```

​	In the memory, the `BlockIndex` structure is used to conduct and store the block information. In the `BlockIndex` structure, the `index` maps the block, using the hash of the block header as its key. A block can be found quickly by using its hash to match the index. Meanwhile, there is also a `blockNode` array to store the blocks. ???

> 在内存中，我们使用一个结构BlockIndex结构组织和保存区块信息。在BlockIndex结构中，我们使用index做区块索引，在index中使用区块头哈希值做key。如果我们需要通过区块哈希值搜索区块时，可以直接从index中快速搜索。同时我们还使用一个blockNode的数组结构存储区块信息。在mainChain中保存的无分叉的主链数据，所有区块数据按照区块高度依次排列。当我们需要根据区块高度搜索区块时，我们可以直接从mainChain中检索。数据结构如下所示：

```
type BlockIndex struct {
	sync.RWMutex

	index     map[bc.Hash]*BlockNode
	mainChain []*BlockNode
}
```

​	For a block, being store into the blockchain doesn't equal to being in the main blockchain, there are chances to be in the side chain. Therefore, when a block is add by the `AddNode()` function, only the block information of the `index` will be updated with no change in the `mainChain` structure.

> 当我们将一个区块保存到链上时，这个区块会上链，但是不一定会上主链。索引我们通过AddNode方法添加区块的时候，只需要更新index中的区块索引信息，当时不会更新mainChain中保存的主链信息。

```
func (bi *BlockIndex) AddNode(node *BlockNode) {
  bi.Lock()
  bi.index[node.Hash] = node
  bi.Unlock()
}
```

### **5.3.3 Linking Blocks in the Blockchain (区块链接)**

​	Blocks need to be connected after being add into the blockchain. This work mainly involves operations on `UTXO`,  including marking all UTXOs in the block  "spent" and removing all transactions out of the transaction pool. Here is the code: 

> 完成区块上链的操作后，节点还需要完成区块的连接。区块链接主要是将区块中所有交易花费的utxo的状态标记为已被消费，并将交易从交易池中移除。代码示例如下：

protocol/block.go

```
func (c *Chain) connectBlock(block *types.Block) (err error) {
   bcBlock := types.MapBlock(block)
   if bcBlock.TransactionStatus, err = c.store.GetTransactionStatus(&bcBlock.ID); err != nil {
      return err
   }
   utxoView := state.NewUtxoViewpoint()
   if err := c.store.GetTransactionsUtxo(utxoView, bcBlock.Transactions); err != nil {
      return err
   }
   if err := utxoView.ApplyBlock(bcBlock, bcBlock.TransactionStatus); err != nil {
      return err
   }
   node := c.index.GetNode(&bcBlock.ID)
   if err := c.setState(node, utxoView); err != nil {
      return err
   }
   for _, tx := range block.Transactions {
      c.txPool.RemoveTransaction(&tx.Tx.ID)
   }
   return nil
}
```

​	First of all, get all UTXOs spent by transactions of this block by the `GetTransactionsUtxo()` function, then put them into the `UtxoViewpoint`, which represents a set of UTXOs. The `ApplyBlock()` function is used to update the status of UTXOs, that is the UTXOs in the input will be marked "spent" if the following two conditions are met:

* The transaction is successful and the asset in the UTXOs is BTM
* If it is an UTXO produced by a coinbase transaction, it can only be used in a transaction after 100 blocks, which means the coinbase UTXO will be unlocked after 100 blocks.

> 首先通过GetTransactionsUtxo方法找到区块中所有交易引用的UTXO，并将其保存到UtxoViewpoint中。UtxoViewpoint可以用来表示一组未花费交易输出的集合。ApplyBlock方法则用来更新UTXO的状态，对于交易输入引用的UTXO如果满足以下两个条件，则将UTXO的状态更新为已消费：
>
> * 交易状态为成功，并且UTXO的资产类型为BTM资产。
> * 如果交易引用的UTXO是一个coninbae交易的输出，则该coinbase交易所在的区块高度至少要比现在小100。即coinbase交易的输出会被阻塞100个区块之后，才能正式解锁使用。

​	After the  `ApplyBlock()` function updates the status of the UTXOs , it will store the UTXO that is produced by the coinbase transaction into the UTXOs set of the LevelDB. Apart from that, the status of the UTXOs that has been spent also needs to be updated into the UTXOs set of the LevelDB by the   `setState()` function.

> ApplyBlock方法完成UTXO状态的更新之后，将区块中coinbase交易产生的UTXO存储到LevelDB中的UTXO集合中。还需通过setState方法将这些已消费的UTXO更新到LevelDB中的UTXO集合中。

### 5.3.4 Reorganize the Blockchain (链重组)

​	The node adds the received block into the blockchain. The soft fork or the side chain is a forward-compatible change to the consensus rules that allows unupgraded clients to continue to operate in consensus with the new rules. The side chain also have chances to be the main chain, which all depends on the work of chain. In the `processBlock` function, if there is a block whose parent block is not in the main chain, the work of the main and side chains will be measured. The blockchain will be reorganized if the side chain wins. The side chain becomes the main chain. Her is the flow chart of the blockchain reorganization.

> 节点将接收到的区块，链接到Chain链上。当新共识规则发布后，没有升级的节点会因为不知道新共识规则下，而生产不合法的区块，就会产生临时性分叉。这种临时性分叉可以称为软分叉或侧链（这里的侧链是临时性产生的软分叉，叫做侧链）。当主链与侧链冲突时，侧链也有会有可能成为主链。不过这取决于哪个链累积了最多的工作量证明。在processBlock方法中，如果添加区块的父区块不是主链上的尾节点，则这个区块被添加到侧链上。此时，就需要判断主链与侧链的工作量哪个更多。如果侧链的工作量比主链的工作量还要多，则需要对区块链进行重组，将侧链变为主链。链重组流程如图5-1所示。

![5-1 The process of block reorganization](https://github.com/Bytom-Community/Bytom-Tech-Book/blob/master/Chapter05_pic/pic-01.png)

​	The calculation algorithm of the amout of work is here:

> 区块链节点工作量总和的算法如下所示：

```
func CalcWork(bits uint64) *big.Int {
   difficultyNum := CompactToBig(bits)
   if difficultyNum.Sign() <= 0 {
      return big.NewInt(0)
   }
   denominator := new(big.Int).Add(difficultyNum, bigOne)
   return new(big.Int).Div(oneLsh256, denominator)
}
```

​	The formula of the amount of work:

> 节点工作量总和计算公式为：

```
SumPoW = 2^256 / (difficultyNum+1)
```

​	If the amount of work on the new side chain is over than the older best chain, this side chain will become the new main chain. To achieve that, it needs to find the detach node then cut the link with the old chain and link the following new blocks to the new main chain. All operations above are mainly made by the `reorganizeChain()` function.

> 如果这个新的侧链的累积工作量总和超过了旧的最佳链，则这个侧链需要成为新的主链，为了实现这一点，需要找到主链和侧链的分叉点，断开形成（现在）旧分叉的块与主链的链接，并将形成新链的块连接到从主链开始的主链ancenstor（链分叉的点）。该部分操作主要由reorganizeChain方法实现，步骤如下。

**Step 1**, figure out the blocks that need to be reorganized and deleted.

​	The  `reorganizeChain()` functions finds the detach node and records `detachNodes` and `attachNodes`.  `detachNodes` are nodes that need to be cut when reorganization happens, while  `attachNodes` needs to be appended to the blockchain. Here is the code:

> **步骤1，**区分出需要重组和删除的块。
>
> 通过calcReorganizeNodes方法查找到主链和侧链的分叉点，同时记录detachNodes节点和attachNodes节点。detachNodes节点是需要重组时断开与主链连接的节点，attachNodes节点是需要重新添加到主链的节点。代码示例如下：

protocol/block.go

```
func (c *Chain) calcReorganizeNodes(node *state.BlockNode) ([]*state.BlockNode, []*state.BlockNode) {
	var attachNodes []*state.BlockNode
	var detachNodes []*state.BlockNode

	attachNode := node
	for c.index.NodeByHeight(attachNode.Height) != attachNode {
		attachNodes = append([]*state.BlockNode{attachNode}, attachNodes...)
		attachNode = attachNode.Parent
	}

	detachNode := c.bestNode
	for detachNode != attachNode {
		detachNodes = append(detachNodes, detachNode)
		detachNode = detachNode.Parent
	}
	return attachNodes, detachNodes
}
```

​	The `calcReorganizeNode` function will start with the last block to go through the whole side chain. 

> calcReorganizeNodes方法首先从侧链最后节点依次向前遍历，通过NodeByHeight方法判断是否遍历到主链的一个节点attachnode（主链的这个节点attachnode就是此时的分叉点）。如果不是，则将遍历的节点加入attachNodes数组中。NodeByHeight方法会根据传入的高度从主链上查找节点返回。但是，如果传入的高度大于主链目前的高度，则NodeByHeight方法会返回一个空值。之后是从最高节点开始依次遍历主链，将主链上attachnode父节点之后的所有区块加入到detachNodes 数组中。

**Step 2** , process the ` detachNodes`.

​	Find all blocks that need to be deleted in the `detachNode` array and then process the status of transactions in these blocks by the `DetachTransaction` function. The  `DetachTransaction` function set all UTXOs used in transactions "unspent" so that they can still be spent later. Meanwhile, the `OUTPUT` of these transactions will be set referred to avoid being used in the new transactions. The UTXO set is returned to the original status by operations above. Here is the code:

> **步骤2，**处理detachNodes。
>
> 遍历detachNodes数组中保存的需要删除的节点，通过DetachTransaction方法将区块中所有交易的状态还原。DetachTransaction方法中，首先将交易中引用的UTXO的状态变为未花费，这样之后的交易就又可以引用这些UTXO。然后将交易产生的OUTPUT状态变为已引用。这样可以防止用户在构建交易的输入中继续引用被被重组区块中的交易。通过这两步，就可以将UTXO集的状态恢复到之前的状态。代码示例如下：

```
func (view *UtxoViewpoint) DetachTransaction(tx *bc.Tx, statusFail bool) error {
	for _, prevout := range tx.SpentOutputIDs {
		spentOutput, err := tx.Output(prevout)
		if err != nil {
			return err
		}
		if statusFail && *spentOutput.Source.Value.AssetId != *consensus.BTMAssetID {
			continue
		}

		entry, ok := view.Entries[prevout]
		if ok && !entry.Spent {
			return errors.New("try to revert an unspent utxo")
		}
		if !ok {
			view.Entries[prevout] = storage.NewUtxoEntry(false, 0, false)
			continue
		}
		entry.UnspendOutput()
	}

	for _, id := range tx.TxHeader.ResultIds {
		output, err := tx.Output(*id)
		if err != nil {
			// error due to it's a retirement, utxo doesn't care this output type so skip it
			continue
		}
		if statusFail && *output.Source.Value.AssetId != *consensus.BTMAssetID {
			continue
		}

		view.Entries[*id] = storage.NewUtxoEntry(false, 0, true)
	}
	return nil
}
```

**Step 3**, process the `attachNode`

​	This step works just in the opposite of step 2. Here, the node goes through the `attachNodes` array to find the blocks that need to be linked to the main chain. They are linked to the chain by the `ApplyBlock()` function and the `ApplyTransaction()` function  locks all transactions of the block by setting all UTXOs "spent" and instantiates the new UTXO object。

> **步骤3，**处理attachNodes。
>
> 这步的操作和上一步正好有些相反。在这一步中，节点遍历attachNodes 数组中保存的需要变更到主链上的节点。通过ApplyBlock方法重新链接区块上链，ApplyTransaction方法将区块中所有交易的状态锁定。ApplyTransaction方法中，将交易中引用的UTXO状态更新为已消费。同时实例化新的UTXO对象。

 **Step 4**, Update the UTXO set.

​	Add all UTXOs instantiated in the last step into the UTXO set. Update the chain status and change the best block to the highest block of the side chain.

> **步骤4，**更新UTXO集合。
>
> 将上一步中新实例化的UTXO写入UTXO集合中，同时更新链的状态，将链的最高区块更新为原来侧链的最高区块。

 

### 5.3.5 the Status of the Main Blockchain (主链的状态)

​	At the first time to start a node, initializing the main blockchain or not depends on the `blockStore` key in the database. When initializing the main blockchain, the height of the blockchain is 0 and the hash is the genesis block hash. Here is the code:

> 当节点第一次启动时，节点会根据数据库中key为blockStore的内容判断是否初始化主链。初始化主链时，主链状态的高度是0，hash是创世区块。代码示例如下：

protocol/store.go

```
type BlockStoreState struct {
	Height uint64
	Hash   *bc.Hash
}
```

​	The description of the main blockchain status:

* Height: The biggest height of the current main blockchain.
* Hash: The hash of the highest block in the current main blockchain.

> 主链状态说明：
>
> * Height：当前主链最高的高度。
> * Hash：当前主链最高的hash。

​	The status of the main blockchain needs to be store in persistent storage since it will be used every time the node is restarted. 

> 主链状态是需要持久化到存储中，每次节点重启时会加载该值，存储与主链状态相关的key如下：

database/leveldb/store.go

```
blockStoreKey     = []byte("blockStore")
```

​	The steps of updating the main blockchain status:

1) Update the main blockchain status at the first time the node is started.

2) Update the main blockchain status when new blocks are linked to the main blockchain.

3) Update the main blockchain status when reorganization happens on the main blockchain.

> 主链状态更新操作流程：
>
> 1) 当节点第一次启动时，会更新主链状态。
>
> 2) 当新的区块添加到主链的末尾，会更新主链状态。
>
> 3) 当重组主链时，会更新主链状态。

## 5.4 Conclusion 

​	This chapter analyzes in the detail on the structure of block and blockchain as well as  their basic operations. Analysis mainly includes the genesis block, block validation, target, orphan block management, link and reorganize blocks, etc., offering a macro view of the blockchain.

> 本章详细介绍了区块和链的核心数据结构，并对区块和链的基本操作做了深入细致的分析。其中主要包括：创世区块的生成、区块校验、区块难度计算、孤块管理、区块上链、区块重组等。通过本章的学习，我们已经对区块链的全貌有了详细了解。
