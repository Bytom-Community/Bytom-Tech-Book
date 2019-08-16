# Chapter 01  The Architecture of Public Blockchain

## 1.1 Introduction （引言）

​	This technology —— Blockchian, was born in Bitcoin in 2008  with the publication of a paper titled "Bitcoin: A Peer-to-Peer Electronic Cash System", written under the alias of Satoshi Nakamoto.

> 区块链技术起源于2008年中本聪《比特币：一种点对点电子现金系统》,区块链诞生自中本聪的比特币。

​	Blockchain is a distributed ledger, a computing scheme that  uses a decentralized and trustless way to maintain a reliable database collectively. 

> 区块链是一个分布式账本，一种通过去中心化、去信任的方式集体维护一个可靠数据库的计算方案。

​	Classifications of Blockchain:

* Public Blockchain: There is no central trusted authority、manager and central server. Nodes need to follow by the system rules to access the network, and work together based on the consensus.

* Private Blockchain: Established within an enterprise and the system rules are set according to the requirements of the enterprise. Although  the read and write permissions are owed by a few nodes, the authenticity and partial decentralization of the blockchain are still retained.

* Consortium Blockchain: Initiated by several agencies. It's between public blockchain and private blockchain, and has partial features of decentralization.

> 区块链分类：
>
> * 公链：无官方组织及管理机构，无中心服务器。参与的节点按照系统规则自由的接入网络，节点间基于共识机制开展工作。
>
> * 私链：建立在某个企业内部，系统运作规则根据企业要求进行设定，读写权限仅限于少数节点，但仍保留着区块链的真实性和部分去中心化特性。
>
> * 联盟链：若干个机构联合发起，介于公链和私联之间，兼容部分去中心化的特性。

​	This book analyses the code of Bytom,  which is an excellent project in China, to  give our readers a complete view of public blockchain technology. If Bitcoin represents the era of blockchain 1.0 and Ethereum that owes Turing completeness represents the era of blockchain 2.0, I do think Bytom represents  the era of blockchain 2.5, which supports much more functions based on UTXO( smart contracts with Turing completeness, multi-assets management, Tensority —— a new POW consensus algorithm, etc. ). Bytom is an open source project based on GO, hosted on GitHub (https://github.com/Bytom/bytom) .

> 本书基于国内优秀项目比原链bytom作为源码分析，为读者展开公链技术的完整实现。如果说比特币代表区块链1.0时代，以太坊拥有图灵完备性代表的是区块链2.0时代的话。比原链则基于UTXO模型上面支持了更丰富的功能（图灵完备的智能合约、多资产管理、Tensority新型的 PoW 共识算法等），我则认为比原链代表的是区块链2.5。比原链是一个开源项目，整个项目基于GO开发，代码托管于GitHub（https://github.com/Bytom/bytom）网站上。

​	Our analysis is based on the 1.0.5 version of Bytom. Furthermore, there is no need to wonder why this book doesn't choose Bitcoin or Ethereal as the example, since we all believe that almost all implementations of blockchain are similar, differences are just in consensus algorithms (PoW, PoS) and models (UTXO or Account) .

> 本书基于比原链的1.0.5版本源码为读者展开分析。在此读者无需纠结本书为何不使用比特币或以太坊作为本书的示例，所谓“有道无术，术尚可求也，有术无道，止于术”，作者认为大部分区块链技术实现都是相似的。目前只有在共识算法（PoW、PoS）和模型（UTXO或Account模型）以外方面有所不同。

​	This chapter aims to analyse the technical implementation architecture of Bytom based on its code. Here are mainly related to three parts:

* Description of the general architecture of Bytom
* The implementation and analysis of each module in Bytom.
* The compile and application of Bytom.

> 本章的目的是在理解比原链源码的基础上分析比原链整体技术实现架构，过程主要按照三个部分进行：
>
> * 比原链的总架构图讲解。
>
> * 比原链架构内部各模块功能实现与分析。
>
> * 比原链编译部署及应用。



## 1.2 General Architecture Design of Public Blockchain  (公链设计总架构图)

​	Bytom Blockchain Protocol (here in after referred as Bytom) is an interactive protocol of multiple byteassets and an open source public blockchain with smart contracts. As an excellent public blockchain in China, Bytom's code is not too much, and has clear structure, which makes the learning easier for programmers and amateurs. We can learn a lot from it, eg:  The Go language programming and application,  architecture design of public blockchain, the principle public blockchain operation, etc. The Bytom architecture design is as follows:
![图 1-1 比原链架构示意图](https://github.com/Bytom-Community/Bytom-Tech-Book/blob/master/Chapter01_pic/pic_01.png)

> Bytom Blockchain（简称比原链，或者Bytom）是一种多元比特资产的交互协议，也是一个开源的有智能合约功能的公共区块链平台。作为国内优秀的公链，比原链的代码量并不多，而且清晰的源码结构使得程序员和链圈爱好者的学习成本并不高。我们从中可以学到很多东西，如GO语言程序设计及应用、公链设计架构、公链运行原理等。比原链公链设计架构图如下所示：



## 1.3 Function and Implementation of Each Module of Bytom (比原链各模块功能与实现)

​	We will discompose the architecture of Bytom, then analyse and elaborate each module.

> 我们将从比原链总架构图抽离出架构中的各个模块，并对每个模块进行架构的分析及阐述。



### 1.3.1 User Interaction Layer （用户交互层）

![图 1-2 比原链架构示意图](https://github.com/Bytom-Community/Bytom-Tech-Book/blob/master/Chapter01_pic/pic_02.png)

***1. Bytomcli RPC Client （Bytomcli RPC客户端）***

​	Bytomcli is an RPC client that builds communication between the user and the bytomd by the command line. On a computer that has deployed Bytom, users can initiate many   management requests by the bytomcli executable file.

> bytomcli是用户与bytomd进程在命令行下建立通信的rpc客户端。在已经部署比原链的机器上，用户可以使用bytomcli可执行文件发起众多对比原链的管理请求。

​	The request is sent by bytomcli, then received and processed by bytomd. That is a cycle of bytomcli's request.

> bytomcli发送相应的请求，请求由bytomd进程接收并处理。bytomcli的一次请求周期结束。

***2. Bytom Dashboard***

​	Bytom dashboard is similar to bytomcli. Both build communication with bytomd by sending requests. Users can communicate with bytomd more friendly through Web pages.

>  bytom dashboard与bytomcli功能类似。都是发送请求与bytomd建立通信。用户可通过Web页面与bytomd进程进行更为友好的交互通信。

​	On a computer that has deployed Bytom,  the bytom-dashboard is turned on by default, and manual deployment is not required. In fact, users can decide to turn on or off the bytom-dashboarf by setting parameters。For example, if we set  --web.closed, the bytom dashboard will be turned off. Project Address: https://github.com/Bytom/bytom-dashboard

> 在已经部署比原链机器上，默认会开启bytom-dashboard功能。无需再手动部署bytom-dashboard。实际上通过传入的参数用户可以决定是否开启或关闭bytom-dashboard功能。如传入--web.closed则可以关闭该功能。项目源码地址：https://github.com/Bytom/bytom-dashboard



### 1.3.2  Interface Layer -ApiServer(接口层-ApiServer)

![图 1-3 比原链架构示意图](https://github.com/Bytom-Community/Bytom-Tech-Book/blob/master/Chapter01_pic/pic_03.png)

​	Api Server is a very important part in bytom, which serves bytomcli and dashboard specially to receive, process and respond to requests from users and mining pools. It starts port 9888 by default. Here are its main functions:

* Receive, process and respond to requests from users and mining pools.
* Manage transactions: package, sign, submit transactions and some other interface operations. 
* Manage the local wallet interface.
* Manage the information interface of the local P2P node.
* Manage the operation interface of local mining

> Api Server是比原链中非常重要的一个功能，在比原链的架构中专门服务于bytomcli和dashboard，它的功能是接收、处理并响应用户和矿池相关的请求。默认启动9888端口。总之主要功能如下：
>
> * 接收并处理用户或矿池发送的请求。
> * 管理交易：打包、签名、提交等接口操作。
> * 管理本地钱包接口。
> * 管理本地p2p节点信息接口。
> * 管理本地矿工挖矿操作接口等。

​	In the process of API Server,  the listener receives access requests from bytomcli or dashboard, then API Server will create a new goroutine for each request to process. First, API Server reads and parses the content of request, matches the corresponding routing, then calls the callback function of the routing Handler to process it. Finally, the Handler responds to bytomcli's request after the processing finished.

> 在Api Server服务过程中，在监听地址listener上接收bytomcli或dashboard的请求访问。对每一个请求，Api Server均会创建一个新的goroutine来处理请求。首先Api Server读取请求内容，解析请求，接着匹配相应的路由项，随后调用路由项的Handler回调函数来处理。最后Handler处理完请求之后给bytomcli响应该请求。



### 1.3.3 Kernel Layer - Blocks and Transactions Management (内核层-区块和交易管理)

![图 1-4 比原链架构示意图](https://github.com/Bytom-Community/Bytom-Tech-Book/blob/master/Chapter01_pic/pic_04.png)

​	The kernel layer is the most important part of Bytom. Its code accounts for about 54% of all.

​	Linked-list is the basic structure of blockchain, which is made up of blocks one by one. A blockchain contains thousands of blocks, while a block contains one or more transactions. In Bytom, another important tasks of the kernel layer is to validate blocks and transactions.

> 内核层是比原链中最重要的功能。代码量大约占了总量的54%。
>
> 区块链的基本结构是一个链表。链表由一个个区块串联组成。一个区块链中包含了成千上万个区块，而一个区块内又包含一个或多个交易。在比原链内核层另一个重要的功能对区块和交易进行验证。

​	In the network, when a new valid block is received , the node will validate it. If the node doesn't find its parent block in the main blockchain, it will be regarded as orphan block, waiting for its parent block. If there is its parent block, this new block will be added to the main blockchain.

> 当网络中的某个节点接收到一个新的有效区块时，节点会验证新区块。当新的区块并未在现有的主链中找到它的父区块，这个新区块会进入到孤块管理中等待父区块。如果从现有的主链中找到了父区块则加入到主链。

​	In the network, when a new transaction is received, the node will validate it. Once verified successfully, this transaction will be put into the transaction pool to wait for the miners to package.

> 当网络中的某个节点接收到一笔交易时，节点会验证交易的合法性。验证成功后，该笔交易放入交易池等待矿工打包。

​	Each transaction needs to go through the following processes to complete its life cycle after being sent :

* User A  uses his wallet to send a transaction to User B.
* The transaction is propagated in the P2P network. 
* Miners validate the transaction after receiving it.
* Miners packages the transaction and put it into a new block. A block can contains one or more transactions.
* Add the new block to the blockchain.
* The transaction is complete and becomes a part of the blockchain.

> 一笔交易发送到完成的生命周期需要经过如下过程：
>
> * A通过钱包向B发出一笔交易，交易金额为100BTM。
> * 该笔交易被广播到P2P网络中。
> * 矿工收到交易信息，验证交易合法性。
> * 打包交易，将多个交易组成一个新区块。
> * 新区块加入到一个已经存在区块链中。
> * 交易完成，成为区块链的一部分。

### 1.3.4 Kernel Layer - Smart Contract（内核层-智能合约）

![图 1-5 比原链架构示意图](https://github.com/Bytom-Community/Bytom-Tech-Book/blob/master/Chapter01_pic/pic_04.png)

​	Contract is generally regarded as the real life contract. Smart contract is a computer protocol intended to digitally facilitate, verify, or enforce the negotiation or performance of a contract. It can be executed on the blockchain. In fact, smart contract is just a "program code" running on the virtual mechain, which can complete transactions without a third party.

> 合约，从传统意义上来说就是现实生活中的合同。智能合约是一种旨在以数字化的方式让验证合约谈判或履行合约规则更加便捷的计算机协议。在可信的区块链上执行合约。智能合约本质是一段运行在虚拟机上的“程序代码”。可以在没有第三方信任机构的情况下执行可信交易。

​	Smart contract has two features:  traceability and irreversibility.

​	Smart contract is the core and the most important part of Bytom. Inthe following chapters, we will introduce the its models ( UTXO，Acccount ), principle, and working mechanism of BVM. At the same time, we will analyse the code deeply to explain how smart contract can complete transactions without a central trusted authority.

> 智能合约具有两个特性：可追踪性和不可逆转性。
>
> 智能合约是比原链中最核心的也是最重要的部分。在后面章节中，我们会详细介绍智能合约模型（主流模型：utxo模型、账户模型），运行原理，以及BVM虚拟机工作机制。深入代码了解区块链上智能合约如何解决没有第三方信任机构的情况下进行可信交易。



### 1.3.5 Kernel Layer - BVM(Bytom Virtual Machine) (内核层-BVM虚拟机)

![图 1-6 比原链架构示意图](https://github.com/Bytom-Community/Bytom-Tech-Book/blob/master/Chapter01_pic/pic_04.png)

​	Bytom Virtual Machine is designed to execute smart contract programs in a platform-independent environment. BVM is an important part of Bytom, which plays a vital role in the process of storing, processing and validating smart contracts.

> 比原链虚拟机（Bytom Virtual Machine，BVM）是建立在区块链上的代码运行环境，其主要作用是处理比原链系统内的智能合约。BVM虚拟机是比原链中是非常重要的部分。它在智能合约存储、执行和验证过程中，担当着重要角色。

​	BVM uses Equity to write smart contracts. Every node all runs a BVM and has same opcodes. It is a platform-independent environment, separated with the blockchain.

​	In a word, BVM is a code running environment build on the Bytom.

> BVM用Equity语言来编写智能合约，比原链是一个点对点的网络，每一个节点都运行着BVM虚拟机，并执行相同的指令。BVM虚拟机是在沙盒中运行，和区块链主链完全分开的。
>
> 简单来说，BVM虚拟机是建立在比原区块链上的代码运行环境。



### 1.3.6 Wallet Layer (钱包层)

![图 1-7 比原链架构示意图](https://github.com/Bytom-Community/Bytom-Tech-Book/blob/master/Chapter01_pic/pic_05.png)

​	Wallet is used to manage keys and UTXO. It's like the real life safe, and we just care about the way to open the safe(keys) and the property (UTXO) in it. In Bytom, wallet layer is in charge of storing keys, managing addresses, maintaining UTXO, processing transactions, and offering interfaces of the wallet and transactions.

> 钱包主要用于管理密钥和UTXO。钱包可以类比于我们日常生活中的保险箱，我们关心保险箱的开门方式（密钥）和其中保存的财产（UTXO）。比原链钱包层主要负责保存密钥、管理地址、维护UTXO信息，并处理交易的生成、签名, 对外提供钱包、交易相关的接口。

​	Here are three steps to send a transaction :

* Build: Build the transaction with its input and output information .
* Sign: Sign every input of the transaction with the private key.
* Submit: Submit the transaction to the network and propagate it. Wait for the transaction to be packaged.

> 比原链的交易发送分为三步：
>
> * Build：根据交易的输入和输出，构造交易数据。
> * Sign：使用私钥对每个交易输入进行签名。
> * Submit：将交易提交到网络进行广播，等待打包。



### 1.3.7 Consensus Layer （共识层）

![图 1-8 比原链架构示意图](https://github.com/Bytom-Community/Bytom-Tech-Book/blob/master/Chapter01_pic/pic_06.png)

​	Consensus layer is used to maintain the synchronization of the whole network. Blockchain is a decentralized ledger, which needs all nodes in the network to reach a consensus. Consensus layer ensures new blocks to be generated in the same way on all nodes by validating blocks and transactions. In short, consensus works in some way to compete for the right of ledger recording. The node that gets the right can add the block generated by itself to the end of the blockchain. Other nodes will validate and accept these blocks following the same rules, and discard blocks failed in validation.

> 共识层用于实现全网的一致性，区块链是去中心化账本，需要全网对账本达成共识。共识层通过验证区块和交易，保证新的区块在所有节点上以相同的方式产生。简单说，共识机制就是通过某种方式竞争“记账权”，得到记账权的节点可以将自己生成的区块追加到现有区块链最后。其它节点可以根据相同的规则，验证并接受这些区块。丢弃那些无法通过验证的区块。

​	There are two common types of consensus: PoW(Proof-of-Work), PoS(Proof-of-Stake.

​	Pow)

​	PoW is based on a super complex mathematical problem, which is  to compute a hash result less than a specific values. It is impossible to compute the input by the result according to the feature of the hash function, so the only way is to compute the input by enumeration until meeting the hash result, which requires a lot of computing. The complecity of PoW ensures that anyone has to pay a lot of computing to generate a new block and pay much computing than the whole network to tamper the block. Here are advantages and disadvantages of PoW:

> 常见的共识机制有PoW工作量证明（Proof-of-Work），PoS股权证明（Proof-of-Stake）等。
>
> PoW共识机制利用复杂的“数学难题”作为共识机制，目前一般使用“hash函数的计算结果小于特定的值”。由于hash函数的特性，不可能通过函数值来反向计算自变量，所以必须用枚举的方式进行计算，直到找出符合要求的hash值为止。这一过程需要进行大量运算。PoW的复杂性保证了任何人都需要付出大量的运算来产生新的块，如果要篡改已有的区块，则需要付出的算力要比网络上的其它节点总和更大。PoW优缺点对比如下：



| Advantages of PoW                             | Disadvantages of PoW                                         |
| --------------------------------------------- | ------------------------------------------------------------ |
| 1. The algorithm is very simple to implement. | 1. It consumes a lot of resources, which is a waste.         |
| 2. It costs a lot to destroy the consensus.   | 2. The process of computing is complex, resulting in the large interval for generating blocks. |
|                                               | 3. As the ASIC develops, the computing power is controlled by a few users. |



| PoW的优点                     | PoW的缺点                             |
| ----------------------------- | ------------------------------------- |
| 1. 算法简单，容易实现         | 1. 消耗大量资源，造成资源浪费         |
| 2. 破坏共识需要付出极大的成本 | 2. 运算过程复杂，导致区块间隔较大     |
|                               | 3. 随着ASIC的发展，算力集中于少数用户 |

​	PoS is another way. It asks the node to lock a part of the encrypted currency and then assign the ledger recording right  according to the amount and time of locked encrypted currency. PoS generally doesn't need a lot of computing, that's why it is faster and more efficicient than PoW. Here are advantages and disadvantages of PoW:

> PoS是另一种挖矿方式，这种方式要求节点将一部分加密货币锁定，并根据数量和锁定的时长等因素来分配记账权。PoS一般不需要大量的计算，所以比PoW更加迅速和高效。PoS优缺点对比如下：

| Advantages of PoS                                            | Disadvantages of PoS                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| 1. It saves energy with no much computing.                   | 1. The wealth may be gathered in a few people, resulting in a great gap between rich and poor. |
| 2. It is decentralized. All nodes can join in the PoS without extra hardware. | 2. Encrypted currency starts with ICO, which is easy to be hoarded by early users. |

| PoS 的优点                                                   | PoS 的缺点                               |
| ------------------------------------------------------------ | ---------------------------------------- |
| 1. 节能，不需要大量计算                                      | 1. 造成数字货币聚集，导致“贫富不均”      |
| 2. 去中心化，所有持币人，无需投入硬件成本，都可以参与PoS共识 | 2. 数字货币来源于ICO，早期用户容易囤积。 |

​	Nowadays, although there are a few of encrypted currency systems adopt other consensus, PoW and PoS are still main methods. According to the features of Bytom and the view of "computing is right" ( a certain pattern to distribute the benefit, which means that as long as having much computing power, you will get some control), Bytom chose the PoW as its consensus. That asks for a strict consensus on many nodes, and has higher demand for the synchronization and decentralization.

> 目前还有少量加密数字货币采用其它共识机制，但PoW和PoS是其中的主流。由于比原链的特性，结合比原链崇尚的“计算即权利”（一种已确定的利益分配的方式。只要计算力高或拥有更多的算力，那就拥有了某些控制权。）主张，需要在多节点上达成较强的共识，对全局一致性、去中心化要求较高，需要在一定程度上牺牲效率。所以比原链选择了PoW方式作为公链的共识机制。



### 1.3.8 Data Storage Layer数据存储层

![图 1-9 比原链架构示意图](https://github.com/Bytom-Community/Bytom-Tech-Book/blob/master/Chapter01_pic/pic_07.png)

​	Bytom stores all information of addresses, assets, transactions, etc. in the data storage layer. The date storage layer can be divided into two parts. One is the cache, which used to response most queries to reduce the I/O pressure on the disk. The other is the persistent storage. When query can't find the data in the cache, it will read from the persistent storage directly and add a copy to the cache at the same time.

> 比原链在数据存储层上存储所有链上地址、资产交易等信息。数据存储层分为两部分，第一部分为缓存，大部分查询首先从缓存中查询，以减少对磁盘的IO压力。第二部分为持久化存储，当缓存中查询不到数据时，直接从持久化存储中读取，并添加一份到缓存中。

​	Bytom uses LevelDB as its persistent storage by default, which is a highly efficient kv database created by Google. Because LevelDB is a single-process, it can't read and write the database by multiple processes at the same time. That means only one process or one process with multiple threads can read and write the database in the meantime.

> 比原链默认使用LevelDB数据库作为持久化存储。LevelDB是一个Google实现的非常高效的kv数据库。由于Leveldb是单进程服务，不能同时有多个进程对一个数据库进行读写。同一时间只能有一个进程，或一个进程多并发的方式进行读写。

​	By default, the data storage is in the " --data" under the "--home". Take the Darwin (MacOS) as the example, the database is in "$HOME/Library/Bytom/data".

* accesstoken.db —— token information( the wallet access control)
* core.db —— Core database, which stores related data of the main blockchain, including the information of blocks, transactions, assets , etc.
* discover.db —— The information of nodes in the distributed network. 
* trusthistory.db—— The information of malicious nodes in the distributed network.
* txfeeds.db —— Not be used in this version. 
* txfeeds.db —— Local wallet database, used to store the information of users, assets, transactions, UTXOs, etc.

> 默认情况下，数据存储目录在--home参数下的data目录。以Darwin（即MacOS）平台为例，默认数据库存储在 $HOME/Library/Bytom/data 。
>
> * accesstoken.db——token信息(钱包访问控制权限)。
> * core.db——核心数据库，存储主链相关数据。包括块信息、交易信息、资产信息等。
> * discover.db——分布式网络中端到端的节点信息。
> * trusthistory.db——分布式网络中端到端的恶意节点信息。
> * txfeeds.db——目前比原链代码版本未使用该功能，暂不介绍。
> * wallet.db——本地钱包数据库。存储用户、资产、交易、UTXO等信息。



### 1.3.9 P2P Distributed Network (P2P分布式网络)

![图 1-10 比原链架构示意图](https://github.com/Bytom-Community/Bytom-Tech-Book/blob/master/Chapter01_pic/pic_08.png)

​	As a decentralized distributed system, the pattern of communicating between nodes is very important for Bytom to maintain the stability of the whole system. The synchronization of transactions and nodes state all relys on the communication between each node in the network. The network communication of Bytom is based on the P2P communication protocol and also has some special changes according its own features. The P2P netwo communication Layer of Bytom includes four parts: nodes discovery, synchronization of transactions, synchronization of blocks and Propagation of blocks and transactions.

> 比原链作为一个去中心化的分布式系统，其底层个体间的通信机制对整个系统的稳定运行显然十分重要。个体间数据的同步、状态的更新都依赖于整个网络中每个个体之间的通信机制。比原链的网络通信基于P2P通信协议，又根据自身业务的特殊性做了特别的设计。比原链的P2P网络通信层，主要分为节点发现、区块同步、交易同步和快速广播四大部分。



 ***1. Nodes discovery（节点发现）***

​	Nodes discovery of P2P network mainly aims to solve how the new node join the network of Bytom, which requires the new node can be known by other nodes as soon as possible, and gets information from other nodes as well. The Bytom nodes discovery uses Kademli, which is a overlay network with a specific structure of the P2P network.  Each node is identified by a number or node ID, which is stored in each node. Kademlia stores nodes in the k-buckets, and every node just connects with the n closest nodes to form the topological structure of network as follows :
![图 1-11 比原链架构示意图](https://github.com/Bytom-Community/Bytom-Tech-Book/blob/master/Chapter01_pic/pic_09.png)

> P2P节点发现主要解决新加入网络的节点如何连接到比原链网络中。新加入的节点能够快速的被网络中其他的节点感知；同时节点自己也能够获得其他节点的信息，与其通信交换数据。比原链中的节点发现使用的是Kademlia算法实现的节点发现机制。Kademlia算法实现是一个结构化的P2P覆盖网络，每个节点都有一个全网唯一的标识称为Node ID。Node ID被分散的存储在各个节点上。Kademlia将发行的节点存储到k-bucket中，每个节点只连接距离自己最近的n个节点。以此形成的网络拓扑结构如下所示：



***2.Synchronization of transactions (交易同步)***

​	To ensure the security of transactions, the node needs to synchronize transactions with other nodes in the network, which is synchronization of transactions. After other nodes finish the synchronization, they will also validate transactions. If successful, these nodes will add transactions into thier transactions. Also,the synchronization of transactions enhances the efficiency of packaging transactions.

> 比原链网络为了保证交易的安全，每一笔交易达成后，节点需要将交易信息同步到网络中其他的节点。这个过程被称作交易同步。交易同步验证是为了安全性，当交易同步到网络中的其他的节点时，节点也会验证交易是否合法，当交易合法，节点才会将交易写入自己的交易池中。另一方面同步交易可以将交易同步到更多的节点，交易可以更快的被打包到区块中。

 

***3. Synchronization of blocks (区块同步)***

​	Blockchain is a decentralized distributed ledger, which needs full nodes to store all blocks information. Once a new node join the network, it needs to synchronize blocks firstly to build the blockchain. New nodes have to finish the synchronization of blocks from the height 1 to the highest with other nodes in the network, so that the complete blockchain can be built.

> 区块链是一种去中心化的分布式记账系统，全节点需要保存完整的区块信息。当一个节点加入到网络之后，第一件需要做的事情就是同步区块，构建完整的区块链。新加入的节点需要从网络中其他节点同步，从高度为1的区块开始到当前全网的最高高度，才能构建成一条完整的区块链。

​	Nodes with fast synchronization can synchronize 1–128 blocks once（ from the current height to the next height of checkpoint block). Apart from fast synchronization, there is general synchronization, which is used to synchronize the newer blocks to ensure nodes can catch up the blockchain.

> 节点启用快速同步功能时，节点每次最多能够同步1-128个区块（当前高度到下一个checkpoint高度的区块）信息。除了快速同步算法之外，还包括普通同步算法。节点主要使用普通同步算法同步网络中较新的区块信息。普通同步算法主要为了保证节点中的区块信息不落后，能够及时的同步新挖掘的区块。

​	The difference of general synchronization and fast synchronization is that the latter uses "get-headers" and "get-blocks" to get a bunch of blocks with the help of checkpoint, which can reduce the PoW computing and improve efficiency. General synchronization just can get one block once.

​	General synchronization and fast synchronization aim to meet different needs. Fast  synchronization is for building the blockchain quickly so that the node can join the network as soon as possible. General synchronization is only used when it is not satisfied with the requirements of fast synchronization and just can get one block every time. 

> 普通同步和快速同步区别是快速同步使用get-headers，get-blocks批量获取区块，使用checkpoint点的验证来避免PoW工作量验证，从而能极大提高速度。普通同步一次只能获取一个区块。
>
> 快速同步和普通同步分别是为了解决不同场景的需求。快速同步是为了节点能够快速的重建区块链信息，快速的加入的网络中，这个是通过新块广播实现的。普通同步只是不满足快速同步条件下才会使用普通同步。每次同步时请求一个区块。

 

***4. Propagation of blocks and transactions (区块和交易广播)***

​	To ensure the transaction can be confirmed and submitted to the main blockchain in time, Bytom offers the fast broadcasting. Nodes will broadcast new blocks and transactions quickly to other known nodes after receiving them. 

> 为了保证交易信息能够及时的被确认和提交到主链上。比原链提供了快速广播。网络中的节点对于接收到的新区块和新交易信息，会快速的广播到当前节点已知的其他节点。

### 1.3.10 Compilation and Deployment of Bytom

​	There are many ways to install the Bytom. Here we compile the code to show the process.

> 比原链的安装方式有多种。本书以源码分析的角度带领读者了解架构，本节使用源码编译的方式了解安装过程。

***1.Compile and deploy the Bytom***

1）Download ：

```
$ git clone https://github.com/Bytom/bytom.git$GOPATH/src/github.com/bytom
```

2）Switch to version 1.0.5：

```
$ cd $GOPATH/src/github.com/bytom
$ git fetch origin v1.0.5
$ git checkout v1.0.5
```

2）Compile：

```
$ make bytomd
$ make bytomcli
```

3）Initiate ：

```
$ cd ./cmd/bytomd
$ ./bytomd init --chain_id mainnet 
```

​	Bytom has three types of network now, distinguished by chain_id :

* mainnet
* testnet
* solonet

> 目前bytomd支持三种网络，使用chain_id进行区分。分别是：
>
> * mainnet: 主网。
> * testnet: 也称为wisdom，测试网。
> * solonet: 单机模式。

4）Start the Bytomd：

```
$ ./bytomd node

$ ps -ef|grep bytomd
  501 52318   449   0  2:00PM ttys000    0:00.85 ./bytomd node

$ ./bytomcli net-info
{
  "current_block": 36714,
  "highest_block": 36714,
  "listening": true,
  "mining": false,
  "network_id": "wisdom",
  "peer_count": 10,
  "syncing": false,
  "version": "1.0.5+2bc2396a"
}
```

​	When executing the "ps -ef", the bytomd can be seen, which means the process has been running. Using bytomcli to get the information of the node can know that the Bytom process has successfully run.

​	The mining will not be turned on by default after the first start of the bytomd. At that time, the information of the neighbored node can be got from the seed nodes of P2P network, then build communication and synchronize blocks. We will analyse the P2P network in detail in the following chapters.

> 当我们执行ps -ef命令看到bytomd进程时，说明进程已经处于运行状态。使用bytomcli获取节点状态信息。此时我们已经成功的运行比原链的进程。
>
> bytomd进程第一次启动后，默认不会开启挖矿功能。此时会从P2P网络种子节点中获取与之相邻的peer节点，建立握手连接并同步区块。在后面章节中我们深入分析P2P网络底层工作原理。



***2. The directory of the code***

Here is the  The directory of the code:

```
$ tree -L 1
.
├── accesstoken       
├── account           
├── api               
├── asset            
├── blockchain        
├── cmd               
├── common            
├── config            
├── consensus         
├── crypto            
├── dashboard        
├── database          
├── docs              
├── encoding         
├── env             
├── equity           
├── errors            
├── math             
├── metrics          
├── mining            
├── net               
├── netsync         
├── node              
├── p2p             
├── protocol          
├── test             
├── testutil          
├── util              
├── vendor           
├── version           
└── wallet            
```

***3. Start mining***

​	Use the command line to start mining :

```
$ ./bytomcli set-mining true
```

​	The mining is turned off by default in Bytom. There are two ways to start mining. One is to use the bytomcli to set the parameter of mining to true, then the bytomcli will interact with bytomd and start mining. Also, setting the parameter to false will turn off mining. The other way is to use the dashboard to set the parameter, readers can try it themselves. 

> 比原链在默认情况下挖矿模式是关闭状态，开启挖矿模式有两种方式，第一种使用bytomcli命令行交互方式，将mining参数设置为true，此时bytomcli会通过rpc协议跟bytomd进程交互并启用挖矿模式。关闭挖矿模式则指定set-mining参数为false。第二种方式使用dashboard页面启用挖矿参数，在这里请读者自行探索dashboard。

***4. SDK in other languages***

​	The technology community of Bytom offers SDK in different languages:

* PHP SDK：https://github.com/lxlxw/bytom-php-sdk
* Java SDK：https://github.com/chainworld/java-bytom
* Java SDK：https://github.com/successli/Bytom-Java-SDK
* Python SDK：https://github.com/Bytom-Community/python-bytom
* Node SDK：https://github.com/Bytom/node-sdk



## 1.4 Conclusion

​	In this chapter, we made a brief analysis based on the structure of the Bytom.  We show a complete technical architecture system of public blockchain by the diagram and introduce some basic knowledge of Bytom by giving the example of installing.

​	Through the learning of architecture, readers can have a thorough understanding of the design, function and value of public blockchain from the macro perspective.

> 本章从比原链的总体架构上逐步分层进行了简要分析。通过总架构图可以看出一条完整公链的技术架构体系。最后通过安装部署，了解了比原链的基础知识。
>
> 通过对公链架构的学习，可以从顶层宏观角度对公链设计、功能与价值等诸多方面进行全面了解。
