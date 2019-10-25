# Chapter 01  The Architecture of a Public Blockchain

## 1.1 Introduction （引言）

Blockchain technology or a Distributed Ledger Technology was born in 2008 with the publication of a paper titled "Bitcoin: A Peer-to-Peer Electronic Cash System", written under the alias of Satoshi Nakamoto.

> 区块链技术起源于2008年中本聪《比特币：一种点对点电子现金系统》,区块链诞生自中本聪的比特币。

Blockchain is a distributed ledger managed on a decentralized peer to peer computer network without any single trusted authority or a central server. The nodes in the network collectively adhere to well defined protocols for communication between peers, block creation, block storage and block validation to maintain ledger data reliably.

> 区块链是一个分布式账本，一种通过去中心化、去信任的方式集体维护一个可靠数据库的计算方案。

​	Types of Blockchains:

* Public Blockchains: In a public blockchain, there is no single trusted authority, manager or a central server. Any node that can follow the system rules can access the network and participate in the consensus process.

* Private Blockchains: Private blockchains are usually created and managed within an enterprise and the system rules are set according to the requirements of the enterprise. Some nodes can have special privileges compared to other nodes e.g only selected nodes can have block write permissions or only invited nodes can participate in the network. Private Blockchains can also be only partially decentralized.

* Consortium Blockchains: Consortium Blockchains are usually created and managed by a group of different agencies. Consortium Blockchain is a hybrid between a public and a private blockchain, and can also be only partially decentralized.

> 区块链分类：
>
> * 公链：无官方组织及管理机构，无中心服务器。参与的节点按照系统规则自由的接入网络，节点间基于共识机制开展工作。
>
> * 私链：建立在某个企业内部，系统运作规则根据企业要求进行设定，读写权限仅限于少数节点，但仍保留着区块链的真实性和部分去中心化特性。
>
> * 联盟链：若干个机构联合发起，介于公链和私联之间，兼容部分去中心化的特性。

Bytom Blockchain is a very popular open public blockchain originated in China. Bytom is an open source project written in GO language and hosted on Github (https://github.com/Bytom/bytom).

This book analyzes and walks though the source code of Bytom blockchain to give readers a complete understanding of public blockchain technology. 

If Bitcoin represents the era of blockchain 1.0 and Ethereum with Turing complete smart contracts represents the era of blockchain 2.0, Bytom with multi asset distributed ledger and turing complete smart contracts represents the era of blockchain 2.5. 

Bytom is based on an UTXO transaction model similar to bitcoin and offers much more functionality like multi-asset transactions, turing complete smart contracts and a new POW consensus algorithm called Tensority etc.  

> 本书基于国内优秀项目比原链bytom作为源码分析，为读者展开公链技术的完整实现。如果说比特币代表区块链1.0时代，以太坊拥有图灵完备性代表的是区块链2.0时代的话。比原链则基于UTXO模型上面支持了更丰富的功能（图灵完备的智能合约、多资产管理、Tensority新型的 PoW 共识算法等），我则认为比原链代表的是区块链2.5。比原链是一个开源项目，整个项目基于GO开发，代码托管于GitHub（https://github.com/Bytom/bytom）网站上。

​This book is based on the source code of Bytom version 1.0.5. The reader might be curious to know why the authors chose Bytom for this book instead of Bitcoin or Ethereum. The authors believe that the ideas behind the public blochchains are amlost the same and almost all implementations of public blockchains are similar. Variations in public blockchain implementations are mostly due to the choice of consensus algorithms (PoW, PoS) or transaction models (UTXO or Account). 

> 本书基于比原链的1.0.5版本源码为读者展开分析。在此读者无需纠结本书为何不使用比特币或以太坊作为本书的示例，所谓“有道无术，术尚可求也，有术无道，止于术”，作者认为大部分区块链技术实现都是相似的。目前只有在共识算法（PoW、PoS）和模型（UTXO或Account模型）以外方面有所不同。

​	This chapter analyzes the architecture and implementation details of Bytom blockchain based on it's source code. We cover the details in three parts:

* General Architecture.
* Different modules and their functionality.
* Compilation and Deployment of Bytom node.

> 本章的目的是在理解比原链源码的基础上分析比原链整体技术实现架构，过程主要按照三个部分进行：
>
> * 比原链的总架构图讲解。
>
> * 比原链架构内部各模块功能实现与分析。
>
> * 比原链编译部署及应用。


## 1.2 Architecture (公链设计总架构图)

​	Bytom Blockchain (referred as "Bytom" from here on) is a multi asset public blockchain with smart contracts. Bytom is one of the very popular public blockchains and is originated in China. 

Bytom's codebase has a clear structure and is not very large, which makes it easy for beginner and advanced programmers to get started and follow along. Readers can learn a lot of things from reading bytom source code. eg: Programming and application development in Go language, architecture and design of a public blockchain, the principles of public blockchain operations and network protocols, etc. 

The Bytom architecture design is as follows:
![图 1-1 比原链架构示意图](https://github.com/Bytom-Community/Bytom-Tech-Book/blob/master/Chapter01_pic/pic_01.png)

> Bytom Blockchain（简称比原链，或者Bytom）是一种多元比特资产的交互协议，也是一个开源的有智能合约功能的公共区块链平台。作为国内优秀的公链，比原链的代码量并不多，而且清晰的源码结构使得程序员和链圈爱好者的学习成本并不高。我们从中可以学到很多东西，如GO语言程序设计及应用、公链设计架构、公链运行原理等。比原链公链设计架构图如下所示：



## 1.3 Bytom Modules (比原链各模块功能与实现)

​	We will dissect the architecture of Bytom, then analyze and elaborate each module.

> 我们将从比原链总架构图抽离出架构中的各个模块，并对每个模块进行架构的分析及阐述。



### 1.3.1 User Interaction Layer （用户交互层）

![图 1-2 比原链架构示意图](https://github.com/Bytom-Community/Bytom-Tech-Book/blob/master/Chapter01_pic/pic_02.png)

***1. Bytomcli RPC Client （Bytomcli RPC客户端）***

Bytomcli is an RPC client executable that can interact with a bytomd process. You can run bytomcli on command line and issue RPC commands to a bytom node.

> bytomcli是用户与bytomd进程在命令行下建立通信的rpc客户端。在已经部署比原链的机器上，用户可以使用bytomcli可执行文件发起众多对比原链的管理请求。

​	Bytomd receives and processes the request sent by bytomcli. 

> bytomcli发送相应的请求，请求由bytomd进程接收并处理。bytomcli的一次请求周期结束。

***2. Bytom Dashboard***

​	Bytom dashboard is similar to bytomcli. Both communicate with bytomd by sending requests. Dashboard provides a friendly web interface for users to interact with bytomd.

>  bytom dashboard与bytomcli功能类似。都是发送请求与bytomd建立通信。用户可通过Web页面与bytomd进程进行更为友好的交互通信。

By default bytom-dashboard is deployed along with bytom node. Users can optionally turnoff bytom-dashboard by setting --web.closed parameter.
Repository for bytom-dashboard is here. https://github.com/Bytom/bytom-dashboard

> 在已经部署比原链机器上，默认会开启bytom-dashboard功能。无需再手动部署bytom-dashboard。实际上通过传入的参数用户可以决定是否开启或关闭bytom-dashboard功能。如传入--web.closed则可以关闭该功能。项目源码地址：https://github.com/Bytom/bytom-dashboard



### 1.3.2  Interface Layer -ApiServer(接口层-ApiServer)

![图 1-3 比原链架构示意图](https://github.com/Bytom-Community/Bytom-Tech-Book/blob/master/Chapter01_pic/pic_03.png)

Api Server is a very important part of Bytom. It receives, handles and responds to requests from bytomcli, bytom-dashboard, other nodes and mining pools. By default API Server starts on port 9888. 

It's main funcations are:

* Receive, handle and respond to requests from users and mining pools.
* Manage transactions: package, sign, submit transactions and some other interface operations. 
* Manage the local wallet interface.
* Manage the information interface of the local P2P node.
* Manage the operational interface of local mining.

> Api Server是比原链中非常重要的一个功能，在比原链的架构中专门服务于bytomcli和dashboard，它的功能是接收、处理并响应用户和矿池相关的请求。默认启动9888端口。总之主要功能如下：
>
> * 接收并处理用户或矿池发送的请求。
> * 管理交易：打包、签名、提交等接口操作。
> * 管理本地钱包接口。
> * 管理本地p2p节点信息接口。
> * 管理本地矿工挖矿操作接口等。

 API Server constantly listens to requests from bytomcli and bytom-dashboard. When the API Server listener receives a new request, it  creates a new goroutine to handle and process this request. 
 
 First, the API server reads and parses the request content, identifies the route for the request and then calls the callback function of the routing handler for this route to process this request. Finally, the handler responds to the request after the processing is done.

> 在Api Server服务过程中，在监听地址listener上接收bytomcli或dashboard的请求访问。对每一个请求，Api Server均会创建一个新的goroutine来处理请求。首先Api Server读取请求内容，解析请求，接着匹配相应的路由项，随后调用路由项的Handler回调函数来处理。最后Handler处理完请求之后给bytomcli响应该请求。



### 1.3.3 Kernel Layer - Blocks and Transactions Management (内核层-区块和交易管理)

![图 1-4 比原链架构示意图](https://github.com/Bytom-Community/Bytom-Tech-Book/blob/master/Chapter01_pic/pic_04.png)

​	The kernel layer is the most important part of Bytom. It's code is about 54% of overall bytom code.

A blockchain contains thousands of blocks. A block in a blockchain contains one or more transactions and each block is linked to the previous one in a linked-list like structure. 

One other important task of kernel layer is to validate transactions and blocks.

> 内核层是比原链中最重要的功能。代码量大约占了总量的54%。
>
> 区块链的基本结构是一个链表。链表由一个个区块串联组成。一个区块链中包含了成千上万个区块，而一个区块内又包含一个或多个交易。在比原链内核层另一个重要的功能对区块和交易进行验证。

When the node receives a new block from the network, it validates the block and tries to find it's parent block in the locally stored main blockchain. If the parent block exists, then the new block will be added to the main blockchain, otherwise the received block is regarded as an orphaned block and the node waits for the parent block.

> 当网络中的某个节点接收到一个新的有效区块时，节点会验证新区块。当新的区块并未在现有的主链中找到它的父区块，这个新区块会进入到孤块管理中等待父区块。如果从现有的主链中找到了父区块则加入到主链。

When a node receives a new transaction from the network, the node will first validate it. Once the tx is verified successfully, the tx will be put in the transaction pool and the node waits for the miners to pack this tx into a block.

> 当网络中的某个节点接收到一笔交易时，节点会验证交易的合法性。验证成功后，该笔交易放入交易池等待矿工打包。

​	A transaction life cycle in a blockchain goes through the following steps:

* User A creates a new transaction using his wallet to send an asset to User B.
* The transaction is propagated onto the P2P network.
* Miners validate the transaction after receiving it.
* Miners package the transaction and put it into a new block. A block can contain one or more transactions.
* The new block is propagated onto the P2P network and the new block is added to the blockchain on all nodes in the network.
* The transaction is now complete and becomes a part of the blockchain.

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

 
 A Contract is similar to a real life contract agreed upon by people. Smart contract is a set of rules involving public keys and conditional transactions specified in digital form like a computer program that can be executed on a blockchain. Different parties can agree to the set of rules coded as computer programs and can be run on the virtual machine provided by the blockchain. These programs run inside the blockchain with out depending on any third party to interpret these rules and complete transactions.

> 合约，从传统意义上来说就是现实生活中的合同。智能合约是一种旨在以数字化的方式让验证合约谈判或履行合约规则更加便捷的计算机协议。在可信的区块链上执行合约。智能合约本质是一段运行在虚拟机上的“程序代码”。可以在没有第三方信任机构的情况下执行可信交易。

​	Smart contracts have two features:  traceability and irreversibility.

​	Smart contract implementation is the core and the most important part of Bytom. In the following chapters, we will introduce different smart contract models ( UTXO，Acccount), principles, and working mechanism of BVM. We will walk through the code to explain how smart contracts can complete transactions without a central trusted authority to interpret the rules of the contract.

> 智能合约具有两个特性：可追踪性和不可逆转性。
>
> 智能合约是比原链中最核心的也是最重要的部分。在后面章节中，我们会详细介绍智能合约模型（主流模型：utxo模型、账户模型），运行原理，以及BVM虚拟机工作机制。深入代码了解区块链上智能合约如何解决没有第三方信任机构的情况下进行可信交易。



### 1.3.5 Kernel Layer - BVM(Bytom Virtual Machine) (内核层-BVM虚拟机)

![图 1-6 比原链架构示意图](https://github.com/Bytom-Community/Bytom-Tech-Book/blob/master/Chapter01_pic/pic_04.png)

​	Bytom Virtual Machine is designed to execute smart contract programs in a platform-independent environment. BVM is an important part of Bytom, which plays a vital role in the process of storing, processing and validating smart contracts.

> 比原链虚拟机（Bytom Virtual Machine，BVM）是建立在区块链上的代码运行环境，其主要作用是处理比原链系统内的智能合约。BVM虚拟机是比原链中是非常重要的部分。它在智能合约存储、执行和验证过程中，担当着重要角色。

Users can write smart contract programs in a simple high level programming language called Equity. Equity programs can be compiled into standard Bytom Virtual machine opcodes. BVM is a platform-independant virtual machine environment that is part of every bytom node in the bytom blockchain network.

​	In short, BVM is a code running environment build on the Bytom blockchain.

> BVM用Equity语言来编写智能合约，比原链是一个点对点的网络，每一个节点都运行着BVM虚拟机，并执行相同的指令。BVM虚拟机是在沙盒中运行，和区块链主链完全分开的。
>
> 简单来说，BVM虚拟机是建立在比原区块链上的代码运行环境。



### 1.3.6 Wallet Layer (钱包层)

![图 1-7 比原链架构示意图](https://github.com/Bytom-Community/Bytom-Tech-Book/blob/master/Chapter01_pic/pic_05.png)

​	Wallet is used to manage keys and UTXO. It's like the real life safe that you can open with key(s) and manage the property (UTXO) in it. In Bytom, wallet layer is mainly responsible for storing keys, managing addresses, maintaining UTXOs, creating and signing transactions and offers external interfaces to wallet and transaction functionality.

> 钱包主要用于管理密钥和UTXO。钱包可以类比于我们日常生活中的保险箱，我们关心保险箱的开门方式（密钥）和其中保存的财产（UTXO）。比原链钱包层主要负责保存密钥、管理地址、维护UTXO信息，并处理交易的生成、签名, 对外提供钱包、交易相关的接口。

​	Here are three steps to send a transaction:

* Build: Build the transaction with it's input and output information .
* Sign: Sign every input of the transaction with the private key.
* Submit: Submit the transaction to the network and propagate it. Wait for the transaction to be put in a block.

> 比原链的交易发送分为三步：
>
> * Build：根据交易的输入和输出，构造交易数据。
> * Sign：使用私钥对每个交易输入进行签名。
> * Submit：将交易提交到网络进行广播，等待打包。



### 1.3.7 Consensus Layer （共识层）

![图 1-8 比原链架构示意图](https://github.com/Bytom-Community/Bytom-Tech-Book/blob/master/Chapter01_pic/pic_06.png)

​	All the nodes in a Blockchain maintain a decentralized ledger. The Consensus layer is used to achieve blockchain data consistency across all the nodes of network. Consensus layer defines the rules for validating blocks and transactions and rules for generating new blocks which are same on all the nodes. In short, consensus provides a mechanism for nodes participating in the process to win a right to create the next valid block. The node that wins the right can add the next block generated by it to the end of the blockchain. Other nodes will validate and accept this block following the same rules, and discard any other blocks that fail the validation.

> 共识层用于实现全网的一致性，区块链是去中心化账本，需要全网对账本达成共识。共识层通过验证区块和交易，保证新的区块在所有节点上以相同的方式产生。简单说，共识机制就是通过某种方式竞争“记账权”，得到记账权的节点可以将自己生成的区块追加到现有区块链最后。其它节点可以根据相同的规则，验证并接受这些区块。丢弃那些无法通过验证的区块。

​	There are two common types of consensus: PoW(Proof-of-Work) and PoS(Proof-of-Stake).

​	PoW is based on the idea that nodes compete to solve a super complex mathematical problem, like compute a hash result less than a specific value. The hash function is designed such a way that it is impossible to compute or guess the input from the hash result. So the only way is to compute the input by trial and error in a loop until the expected hash result is found, which requires a lot of computing power. The complexity of PoW ensures that a node has to spend a lot of computing power to generate a new block and spend much more computing power than the entire network to tamper or alter the block. Here are advantages and disadvantages of PoW:

> 常见的共识机制有PoW工作量证明（Proof-of-Work），PoS股权证明（Proof-of-Stake）等。
>
> PoW共识机制利用复杂的“数学难题”作为共识机制，目前一般使用“hash函数的计算结果小于特定的值”。由于hash函数的特性，不可能通过函数值来反向计算自变量，所以必须用枚举的方式进行计算，直到找出符合要求的hash值为止。这一过程需要进行大量运算。PoW的复杂性保证了任何人都需要付出大量的运算来产生新的块，如果要篡改已有的区块，则需要付出的算力要比网络上的其它节点总和更大。PoW优缺点对比如下：



| Advantages of PoW                             | Disadvantages of PoW                                         |
| --------------------------------------------- | ------------------------------------------------------------ |
| 1. The algorithm is very simple to implement. | 1. It consumes a lot of resources, which is a waste.         |
| 2. It costs a lot to destroy the consensus.   | 2. The process of computing is complex, resulting in the large interval in between generating blocks. |
|                                               | 3. As the ASIC keeps getting better at computing hashes, the computing power is concentrated among a very few users. |



| PoW的优点                     | PoW的缺点                             |
| ----------------------------- | ------------------------------------- |
| 1. 算法简单，容易实现         | 1. 消耗大量资源，造成资源浪费         |
| 2. 破坏共识需要付出极大的成本 | 2. 运算过程复杂，导致区块间隔较大     |
|                               | 3. 随着ASIC的发展，算力集中于少数用户 |

​	PoS is another way of achieving consensus. The nodes participate in the consensus process by locking some currency and the algorithm takes into account the amount of currency locked and the age duration the amount was locked in determining who gets the right to create the next block. PoS generally doesn't need a lot of computation, that's why it is faster and more efficicient than PoW. Here are advantages and disadvantages of PoW:

> PoS是另一种挖矿方式，这种方式要求节点将一部分加密货币锁定，并根据数量和锁定的时长等因素来分配记账权。PoS一般不需要大量的计算，所以比PoW更加迅速和高效。PoS优缺点对比如下：

| Advantages of PoS                                            | Disadvantages of PoS                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| 1. It saves energy with no need for a lot of computing.    | 1. The wealth may be gathered among a few people, resulting in a great gap between rich and poor. |
| 2. It is decentralized. All nodes can join in the PoS without extra hardware. | 2. CryptoCurrency usually starts with an ICO, which makes it easy for early users to hoard a lot of it. |

| PoS 的优点                                                   | PoS 的缺点                               |
| ------------------------------------------------------------ | ---------------------------------------- |
| 1. 节能，不需要大量计算                                      | 1. 造成数字货币聚集，导致“贫富不均”      |
| 2. 去中心化，所有持币人，无需投入硬件成本，都可以参与PoS共识 | 2. 数字货币来源于ICO，早期用户容易囤积。 |

​	Though there are a small number of cryptocurrency implementations which chose other consensus mechanisms, PoW and PoS are still the most popular methods of achieving consensus. Bytom believes in the idea of "Computing is Power" so anyone who can bring computing power can participate in the consensus process and have some control. Due to higher need for decentralization and consistency, efficiency had to be sacrifised to get multiple nodes across the globe reach consensus reliably and for this reason bytom chose PoW as the consensus mechanism for mainnet public chain.

> 目前还有少量加密数字货币采用其它共识机制，但PoW和PoS是其中的主流。由于比原链的特性，结合比原链崇尚的“计算即权利”（一种已确定的利益分配的方式。只要计算力高或拥有更多的算力，那就拥有了某些控制权。）主张，需要在多节点上达成较强的共识，对全局一致性、去中心化要求较高，需要在一定程度上牺牲效率。所以比原链选择了PoW方式作为公链的共识机制。



### 1.3.8 Data Storage Layer数据存储层

![图 1-9 比原链架构示意图](https://github.com/Bytom-Community/Bytom-Tech-Book/blob/master/Chapter01_pic/pic_07.png)

​	Bytom stores all blockchain data like addresses, assets, transactions, etc. in the data storage layer. The data storage layer can be divided into two parts. Cache and Persistent storage. Cache is used to respond to most queries quickly and reduce the disk I/O. When a query can't find the data in the cache, it will read from the persistent storage directly and add a copy to the cache for future lookup.

> 比原链在数据存储层上存储所有链上地址、资产交易等信息。数据存储层分为两部分，第一部分为缓存，大部分查询首先从缓存中查询，以减少对磁盘的IO压力。第二部分为持久化存储，当缓存中查询不到数据时，直接从持久化存储中读取，并添加一份到缓存中。

​	Bytom uses LevelDB as its persistent storage by default, which is a highly efficient key-value database created by Google. Because LevelDB runs in a single-process, multiple processes cannot read and write to the database at the same time. That means only one process or one process with multiple threads can read and write the database at one time.

> 比原链默认使用LevelDB数据库作为持久化存储。LevelDB是一个Google实现的非常高效的kv数据库。由于Leveldb是单进程服务，不能同时有多个进程对一个数据库进行读写。同一时间只能有一个进程，或一个进程多并发的方式进行读写。

​	By default, the data storage is in the " --data" under the "--home". On Darwin (MacOS), the database is in location "$HOME/Library/Bytom/data".

* accesstoken.db —— token information( the wallet access control)
* core.db —— Core database, which stores data related to the main blockchain, including the information of blocks, transactions, assets , etc.
* discover.db —— The information of nodes in the distributed network. 
* trusthistory.db—— The information of malicious nodes in the distributed network.
* txfeeds.db —— Not yet used in this version. 
* wallet.db —— Local wallet database, used to store the information of users, assets, transactions, UTXOs, etc.

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

​	As a decentralized distributed system, the pattern of communicating between nodes is very important for Bytom to maintain the stability of the whole system. The synchronization of transactions and nodes state all relys on the communication between each node in the network. The network communication of Bytom is based on the P2P communication protocol and also has some special changes according its own features. The P2P netwo communication Layer of Bytom includes four parts: nodes discovery, synchronization of transactions, synchronization of blocks and propagation of blocks and transactions.

> 比原链作为一个去中心化的分布式系统，其底层个体间的通信机制对整个系统的稳定运行显然十分重要。个体间数据的同步、状态的更新都依赖于整个网络中每个个体之间的通信机制。比原链的网络通信基于P2P通信协议，又根据自身业务的特殊性做了特别的设计。比原链的P2P网络通信层，主要分为节点发现、区块同步、交易同步和快速广播四大部分。



 ***1. Nodes discovery（节点发现）***

​	Nodes discovery of P2P network mainly aims to solve how the new node join the network of Bytom, which requires the new node can be known by other nodes as soon as possible, and gets information from other nodes as well. The Bytom nodes discovery uses Kademli, which is a overlay network with a specific structure of the P2P network. Each node is identified by a number or node ID, which is stored in each node. Kademlia stores nodes in the k-buckets, and every node just connects with the n closest nodes to form the topological structure of network as follows : 
![图 1-11 比原链架构示意图](https://github.com/Bytom-Community/Bytom-Tech-Book/blob/master/Chapter01_pic/pic_09.png)

> P2P节点发现主要解决新加入网络的节点如何连接到比原链网络中。新加入的节点能够快速的被网络中其他的节点感知；同时节点自己也能够获得其他节点的信息，与其通信交换数据。比原链中的节点发现使用的是Kademlia算法实现的节点发现机制。Kademlia算法实现是一个结构化的P2P覆盖网络，每个节点都有一个全网唯一的标识称为Node ID。Node ID被分散的存储在各个节点上。Kademlia将发行的节点存储到k-bucket中，每个节点只连接距离自己最近的n个节点。以此形成的网络拓扑结构如下所示：



***2.Synchronization of transactions (交易同步)***

​	To ensure the security of transactions, the node needs to synchronize transactions with other nodes in the network, which is synchronization of transactions. After other nodes finish the synchronization, they will also validate transactions. If successful, these nodes will add transactions into thier transactions. Also,the synchronization of transactions enhances the efficiency of packaging transactions.

> 比原链网络为了保证交易的安全，每一笔交易达成后，节点需要将交易信息同步到网络中其他的节点。这个过程被称作交易同步。交易同步验证是为了安全性，当交易同步到网络中的其他的节点时，节点也会验证交易是否合法，当交易合法，节点才会将交易写入自己的交易池中。另一方面同步交易可以将交易同步到更多的节点，交易可以更快的被打包到区块中。

 

***3. Synchronization of blocks (区块同步)***

​	Blockchain is a decentralized distributed ledger, which needs full nodes to store information of all the blocks. Once a new node joins the network, it first needs to synchronize blocks since height 1 to the most recent block from the network and build the complete chain of blocks to date locally.

> 区块链是一种去中心化的分布式记账系统，全节点需要保存完整的区块信息。当一个节点加入到网络之后，第一件需要做的事情就是同步区块，构建完整的区块链。新加入的节点需要从网络中其他节点同步，从高度为1的区块开始到当前全网的最高高度，才能构建成一条完整的区块链。

​	Nodes with fast synchronization can synchronize 1–128 blocks once（ from the current height to the next height of checkpoint block). Apart from fast synchronization, there is general synchronization, which is used to synchronize the newer blocks to ensure nodes can catch up the blockchain.

> 节点启用快速同步功能时，节点每次最多能够同步1-128个区块（当前高度到下一个checkpoint高度的区块）信息。除了快速同步算法之外，还包括普通同步算法。节点主要使用普通同步算法同步网络中较新的区块信息。普通同步算法主要为了保证节点中的区块信息不落后，能够及时的同步新挖掘的区块。

​	The difference between general synchronization and fast synchronization is that the latter uses "get-headers" and "get-blocks" to get a bunch of blocks upto a checkpoint, which can reduce the PoW computing and improve efficiency. General synchronization just can get one block at a time.

​	General synchronization and fast synchronization aim to meet different needs. Fast  synchronization is for building the local copy of blockchain quickly so that the node can join the network as soon as possible. General synchronization is only used when requirements for fast synchronization are not satisfied and the node can just get one block at a time. 

> 普通同步和快速同步区别是快速同步使用get-headers，get-blocks批量获取区块，使用checkpoint点的验证来避免PoW工作量验证，从而能极大提高速度。普通同步一次只能获取一个区块。
>
> 快速同步和普通同步分别是为了解决不同场景的需求。快速同步是为了节点能够快速的重建区块链信息，快速的加入的网络中，这个是通过新块广播实现的。普通同步只是不满足快速同步条件下才会使用普通同步。每次同步时请求一个区块。

 

***4. Propagation of blocks and transactions (区块和交易广播)***

​	To ensure the transaction can be confirmed and submitted to the main blockchain in time, Bytom offers fast broadcasting. Nodes will broadcast new blocks and transactions quickly to other known nodes immediately after receiving them. 

> 为了保证交易信息能够及时的被确认和提交到主链上。比原链提供了快速广播。网络中的节点对于接收到的新区块和新交易信息，会快速的广播到当前节点已知的其他节点。

### 1.3.10 Compilation and Deployment of Bytom

​	There are many ways to install Bytom. Here we show how to compile the code.

> 比原链的安装方式有多种。本书以源码分析的角度带领读者了解架构，本节使用源码编译的方式了解安装过程。

***1.Compile and deploy Bytom node***

1）Download ：

```
$ git clone https://github.com/Bytom/bytom.git $GOPATH/src/github.com/bytom
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

3）Initialize ：

```
$ cd ./cmd/bytomd
$ ./bytomd init --chain_id mainnet 
```

​	Bytom currently has three types of networks, identified by chain_id :

* mainnet: Main Network
* testnet: Also known as Wisdom test network
* solonet: Stand-alone mode useful for development

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
 When you run "ps -ef", you can see the bytomd process is running. You can also use bytomcli to get the information of node from current running bytomd process. We will analyse the P2P network in detail in the following chapters.

 By default, mining is turned off when you run bytomd for the first time. The bytomd process establishes a communication with neighbour nodes obtained from the seed nodes of the P2P network and will start synchronizing blocks.

> 当我们执行ps -ef命令看到bytomd进程时，说明进程已经处于运行状态。使用bytomcli获取节点状态信息。此时我们已经成功的运行比原链的进程。
>
> bytomd进程第一次启动后，默认不会开启挖矿功能。此时会从P2P网络种子节点中获取与之相邻的peer节点，建立握手连接并同步区块。在后面章节中我们深入分析P2P网络底层工作原理。



***2. The directory listing of the source code***

Here is the directory listing of the source code:

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

​	Use this command line to start mining :

```
$ ./bytomcli set-mining true
```

​	Mining is turned off by default. There are two ways to start mining. 
- Use bytomcli to set the parameter for mining to true. bytomcli will interact with bytomd and the node starts mining. Similarly, set the parameter to false to turn off mining. 
- Use the bytom-dashboard to set the parameter

Readers can try it themselves. 

​	Readers can try it themselves.
> 比原链在默认情况下挖矿模式是关闭状态，开启挖矿模式有两种方式，第一种使用bytomcli命令行交互方式，将mining参数设置为true，此时bytomcli会通过rpc协议跟bytomd进程交互并启用挖矿模式。关闭挖矿模式则指定set-mining参数为false。第二种方式使用dashboard页面启用挖矿参数，在这里请读者自行探索dashboard。

***4. SDKs in other languages***

​	Bytom community offers SDKs in different languages:

* PHP SDK：https://github.com/lxlxw/bytom-php-sdk
* Java SDK：https://github.com/chainworld/java-bytom
* Java SDK：https://github.com/successli/Bytom-Java-SDK
* Python SDK：https://github.com/Bytom-Community/python-bytom
* Node SDK：https://github.com/Bytom/node-sdk



## 1.4 Conclusion

 In this chapter we started with a brief introduction of Bytom blockchain and it's overall structure, then we walked through the technical architecture diagram and different components to get a thorough understanding of a public blockchain and finally we learned how to compile and deploy a bytom node on mainnet public chain. 

 By reviewing the overall architecture, one can achieve a thorough understanding of design, functionality and value of public blockchain from a macro perspective.

> 本章从比原链的总体架构上逐步分层进行了简要分析。通过总架构图可以看出一条完整公链的技术架构体系。最后通过安装部署，了解了比原链的基础知识。
>
> 通过对公链架构的学习，可以从顶层宏观角度对公链设计、功能与价值等诸多方面进行全面了解。
