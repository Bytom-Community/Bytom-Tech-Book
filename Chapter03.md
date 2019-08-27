# Chapter 03 Preprocessing, initialization, Start and Stop of bytomd（bytomd预处理、初始化、启动与停止）

## 3.1 Introduction（引言）

​	Initialization is for the first time to start the node. According to the arguments users send to set configuration (network, database, the initialzation of blockchain and P2P network). Those are all done completely by bytomd at the first running.

> 节点初始化是节点首次使用时，根据用户传入的参数进行参数设置，并根据参数进行网络、数据库、本地区块链以及P2P分布式网络等模块的初始化。使得节点能够正常运行。节点初始化由bytomd执行，在初次运行时一次性完成。

​	The main contents of this chapter:

* The preprocessing of bytomd, flag description
* The initialzation of node, network, database, local blockchain, etc.
* The process of starting daemon.

> 本章主要内容：
>
> * bytomd代码预处理，解析flag等。
> * 节点初始化，网络、数据库、本地区块链初始等。
> * 进程守护进程启动流程。



## 3.2 Description of Bytomd's Flag (bytomd命令flag参数详解)

​	When writing command lines, it is necessary to parse the arguments. In general, programming languages offer functions or library for programers to parse the arguments. In Golang, there are `flag` packages in its library. Here are options of bytomd: 

> 在编写命令行程序时，对命令参数进行解析是常见的需求。不同语言一般都会提供解析命令行参数的方法或库，以方便程序员使用。在GO语言标准库中提供了flag包，方便进行命令行解析。bytomd进程支持的传参如下：

```
$ ./bytomd -h
Multiple asset management.

Usage:
  bytomd [command]

Available Commands:
  help        
  init        
  node       
  version     

coral[coralmac] bytomd $ ./bytomd node -h
Run the bytomd

Usage:
  bytomd node [flags]

Flags:
      --auth.disable                Disable rpc access authenticate
      --chain_id string             Select network type
  -h, --help                        Help for node
      --log_file string             Select file to log
      --log_level string            Select level of log
      --mining                      Enable mining
      --p2p.dial_timeout int        Set dial timeout (default 3)
      --p2p.handshake_timeout int   Set handshake timeout (default 30)
      --p2p.laddr string            Node listen address.
      --p2p.max_num_peers int       Set max num peers (default 50)
      --p2p.pex                     Enable Peer-Exchange  (default true)
      --p2p.seeds string            Comma delimited host:port seed nodes
      --p2p.skip_upnp               Skip UPNP configuration
      --prof_laddr string           Use http to profile bytomd programs
      --simd.enable                 Enable simd, which is used to optimize Tensority
      --vault_mode                  Run in the offline enviroment
      --wallet.disable              Disable wallet
      --wallet.rescan               Rescan wallet
      --web.closed                  Lanch web browser or not

Global Flags:
      --home string   Root directory for config and data
      --trace         Enable trace and show information bout stack when something going wrong
```

## 3.3 The Proprocessing of Bytom Daemon (bytom 守护进程预处理)

​	Daemon is a sepecial process, which will keep running as a background program after being started. Only when the `kill term` can stop it.

​	The Cobra preprocessing of Bytom daemon is pretty similar to the Bytomcli, so we will skip it here and analyze more about Bytom daemon itself. Here are the directory of some relevant files:

> 守护进程(daemon)是一种在特殊进程，启动后会在后台一直运行，只有当触发kill term信号时，才会执行退出操作。
>
> bytom 的守护进程称为bytomd（bytom daemon），它的Cobra预处理流程跟bytomcli过程非常相似。所以在此略去相同部分的讲解。主要对后续bytom 守护进程重要内容进行深入分析。在这里我们看下bytomd预处理过程中使用到的代码文件结构，命令执行如下：

```
$ tree cmd/bytomd/
cmd/bytomd/
├── bytomd
├── commands
│   ├── init.go	     Initialize the network of the node
│   ├── root.go		   The directory of root
│   ├── run_node.go  node deamon
│   └── version.go	 node version
├── main.go		

```

## 3.4 The Initialization of Bytom Daemon (bytom 守护进程初始化具体实现)

​	Bytom daemon initializes different modules according to the flag arguments when it is started, and then keeps running. All work of bytom daemon is in `node.NewNode(config)`.

> bytom 守护进程启动时根据不同的命令行flag参数，初始化不同的模块，最终以守护进程的方式运行。有关bytom 守护进程所有运行的工作都在node.NewNode(config)的具体实现中。



### 3.4.1 Node (Node对象)

```
node/node.go
type Node struct {
	cmn.BaseService
 
	// config
	config *cfg.Config
 
	syncManager *netsync.SyncManager
 
	wallet       *w.Wallet
	accessTokens *accesstoken.CredentialStore
	api          *api.API
	chain        *protocol.Chain
	txfeed       *txfeed.Tracker
	cpuMiner     *cpuminer.CPUMiner
	miningPool   *miningpool.MiningPool
	miningEnable bool
}
```

​	Description of `Node`:

- cmn.BaseService：Service management
- config：Global configuration of the node
- syncManager：Synchronization of transactions and blocks
- wallet：Local wallet management
- accessTokens：Token management, user's access credentials
- api：Api Server
- chain：Local chain
- txfeed：Not be used
- cpuMiner：cpu mining
- miningPool：mining pool
- miningEnable：Enable mining

> Node对象说明：
>
> * cmn.BaseService：服务管理
> * config：当前节点的全局配置
> * syncManager：区块和交易同步管理
> * wallet：本地钱包管理
> * accessTokens：token管理，用户访问凭证
> * api：Api Server接口服务
> * chain：本地区块链管理对象
> * txfeed：目前版本该功能未使用
> * cpuMiner：cpu挖矿管理对象
> * miningPool：矿池管理对象
> * miningEnable：是否启用挖矿模式

​	`node.NewNode(config)` is to create `Node`, which is the basis of bytomd.

​	`cmn.BaseService` is a service management framwork based on tendermint. We ususally regard `Node` as a service and you can set `OnStart/OnStop/IsRunning` to manage it. Tendermint can ensure these operations not to be run repeatedly.

> 在node.NewNode(config)整个过程是为了创建Node对象。Node对象是整个bytomd的所有模块运行的基础。
>
> cmn.BaseService是一个tendermint框架的服务管理模块，在这里我们可以把Node作为一个服务，对该服务进行OnStart/OnStop/IsRunning等操作管理。tendermint框架可以保证这些操作不会被重复执行多次。

### 3.4.2 Initialize the Configuration (配置初始化)

​	Before running `node.NewNode(config)`, there has been the default configuration. Here we will describe the default configuration in details firstly:

> 在执行node.NewNode(config)之前，config默认配置就已经定义好了。在深入node.NewNode(config)分析之前，我们需要了解默认配置都有哪些。

 cmd/bytomd/commands/root.go

```
config = cfg.DefaultConfig()
```

​	Bytom daemon defines a global variable named `config`, which represents the configuration of bytom daemon and will be set by defaulted when daemon being started.

> 首先，bytom 守护进程声明一个config的全局变量，表示整个bytom 守护进程的配置信息。进程启动时config对象被赋予一个默认的配置参数。

config/config.go

```
func DefaultConfig() *Config {
	return &Config{
		BaseConfig: DefaultBaseConfig(),
		P2P:        DefaultP2PConfig(),
		Wallet:     DefaultWalletConfig(),
		Auth:       DefaultRPCAuthConfig(),
		Web:        DefaultWebConfig(),
		Simd:       DefaultSimdConfig(),
	}
}
```

​	There are six default arguments for different modules. We can divide them into three groups: BaseConfig, P2PConfig and other configuration.

> 默认的配置参数有六个。每个针对于不同的模块。下面对每个模块的配置进行说明，我们将默认参数归纳为三块，分别为Base基础配置、P2P网络配置及其他配置：

 

***（1）BaseConfig (Base基础配置说明): ***

​	BaseConfig includes data, log, listening address and other relevant configuration.

> BaseConfig用于bytomd节点所需的基础参数，包括数据目录、日志、监听地址等相关参数的配置。

config/config.go

```
type BaseConfig struct {
	RootDir string `mapstructure:"home"` 		// The root directory for all data.
	ChainID string `mapstructure:"chain_id"` 	// The ID of the network to json. There are three options: mainnet,testnet and solonet
	LogLevel string `mapstructure:"log_level"` 	// log level to set
	PrivateKey string `mapstructure:"private_key"` 	// A JSON file containing the private key to use as a validator in the consensus protoco
	Moniker string `mapstructure:"moniker"` 	// A custom human readable name for this node(default anonymous) 
	ProfListenAddress string `mapstructure:"prof_laddr"` 	// TCP or UNIX socket address for the profiling server to listen on.(default disabled)
	FastSync bool `mapstructure:"fast_sync"` 	// Fast synchronization(default enabled)
	Mining bool `mapstructure:"mining"`		 // Mining.(default disabled)
	FilterPeers bool `mapstructure:"filter_peers"` // not be used(default false)
	TxIndex string `mapstructure:"tx_index"` 	// not be used
	DBBackend string `mapstructure:"db_backend"` // Database backend: leveldb | memdb
	DBPath string `mapstructure:"db_dir"` 		// Database directory
	KeysPath string `mapstructure:"keys_dir"` 	// Keystore directory
	HsmUrl string `mapstructure:"hsm_url"` 		// not be used
	ApiAddress string `mapstructure:"api_addr"` 	// Address of Api Server(default 0.0.0.0:9888)
	VaultMode bool `mapstructure:"vault_mode"`    // No network
	Time time.Time 			// not be used
	LogFile string `mapstructure:"log_file"` // log file name
```

​	A part of default arguments can be got from config file, such as ApiAddress, whose tag is api_addr, can be got its default from `config/toml.go`.

> 部分参数从配置文件中获取默认值，比如ApiAddress参数，它的tag是api_addr。我们可以从config/toml.go中获取默认值。

config/toml.go

```
var defaultConfigTmpl = `# This is a TOML config file.
fast_sync = true
db_backend = "leveldb"
api_addr = "0.0.0.0:9888"
`
var mainNetConfigTmpl = `chain_id = "mainnet"
[p2p]
laddr = "tcp://0.0.0.0:46657"
seeds = "45.79.213.28:46657,198.74.61.131:46657,212.111.41.245:46657,47.100.214.154:46657,47.100.109.199:46657,47.100.105.165:46657"
`
```

***（2）P2PConfig (P2P网络配置说明)：***

​	P2PConfig is used in P2P protocol, including local listening ports, dial timeout and address book, etc.

> P2PConfig用于bytomd P2P通信协议中使用的参数，包括本机监听端口、通信节点超时、地址簿等相关参数的配置。

config/config.go

```
type P2PConfig struct {
	RootDir          string `mapstructure:"home"` 	// 跟BaseConfig中的RootDir相同
	ListenAddress    string `mapstructure:"laddr"` 	// P2P分布式网络监听的端口，用于节点之间的相互通信。默认tcp://0.0.0.0:46656
	Seeds            string `mapstructure:"seeds"` 	// 种子节点
	SkipUPNP         bool   `mapstructure:"skip_upnp"` 	// 是否不使用upnp功能，默认为false
	AddrBook         string `mapstructure:"addr_book_file"` 	// p2p地址簿路径，用于存储已知的peer节点
	AddrBookStrict   bool   `mapstructure:"addr_book_strict"` 	// 目前版本该参数未使用
	PexReactor       bool   `mapstructure:"pex"` 		// 目前版本该参数未使用
	MaxNumPeers      int    `mapstructure:"max_num_peers"`	 // 最大节点连接数，默认为50
	HandshakeTimeout int    `mapstructure:"handshake_timeout"` 	// 节点连接握手超时时间，默认30s
	DialTimeout      int    `mapstructure:"dial_timeout"` 	// 节点连接超时时间，默认3s
}
```

***Note***: In Bitcoin, nodes use DNS to find seeds to get the IP address of other nodes. In Bytom, seeds are IP address, which is usually hard coded into the code. See more details in ***Chapter10 P2P Network***.

> 提示：在比特币中，节点会使用DNS的方式来询问种子节点从而查询到其他节点的IP地址。而在比原链中，种子节点则是IP地址。种子节点一般是硬编码到代码里。详细技术细节我们会在“第10章 P2P分布式网络”中详细讲解该实现过程。



***（3）Other Configuration(其他配置说明)：***

​	WalletConfig is used to set the local wallet, including enabling the wallet and set updating configuration .

> WalletConfig用于bytomd本地钱包使用的参数，包括是否启用本地钱包和更新相关参数的配置。

config/config.go

```
type WalletConfig struct {
	Disable bool `mapstructure:"disable"`	// Enable the local wallet(default false) 
	Rescan  bool `mapstructure:"rescan"`	// Rescan the wallet
}

type RPCAuthConfig struct {
	Disable bool `mapstructure:"disable"` // Enable the authorization of Api Server(default false)
}

type WebConfig struct {
	Closed bool `mapstructure:"closed"` // Start bytom-dashboard(default false)
}

type SimdConfig struct {
	Enable bool `mapstructure:"enable"` // Enable the optimization of Tenaority CPU
}
```

​	After declaring `config = DefaultConfig()`, `init()` will assign the config attributes. Here is the code:

> bytom 守护进程声明config = DefaultConfig()之后，init()函数实现了config对象中各属性的赋值。具体实现代码：

cmd/bytomd/commands/run_node.go

```
func init() {
	runNodeCmd.Flags().String("prof_laddr", config.ProfListenAddress, "Use http to profile bytomd programs")
	runNodeCmd.Flags().Bool("mining", config.Mining, "Enable mining")

	runNodeCmd.Flags().Bool("simd.enable", config.Simd.Enable, "Enable SIMD mechan for tensority")

	runNodeCmd.Flags().Bool("auth.disable", config.Auth.Disable, "Disable rpc access authenticate")

	runNodeCmd.Flags().Bool("wallet.disable", config.Wallet.Disable, "Disable wallet")
	runNodeCmd.Flags().Bool("wallet.rescan", config.Wallet.Rescan, "Rescan wallet")
	runNodeCmd.Flags().Bool("vault_mode", config.VaultMode, "Run in the offline enviroment")
	runNodeCmd.Flags().Bool("web.closed", config.Web.Closed, "Lanch web browser or not")
	runNodeCmd.Flags().String("chain_id", config.ChainID, "Select network type")

	// log level
	runNodeCmd.Flags().String("log_level", config.LogLevel, "Select log level(debug, info, warn, error or fatal")

	// p2p flags
	runNodeCmd.Flags().String("p2p.laddr", config.P2P.ListenAddress, "Node listen address. (0.0.0.0:0 means any interface, any port)")
	runNodeCmd.Flags().String("p2p.seeds", config.P2P.Seeds, "Comma delimited host:port seed nodes")
	runNodeCmd.Flags().Bool("p2p.skip_upnp", config.P2P.SkipUPNP, "Skip UPNP configuration")
	runNodeCmd.Flags().Bool("p2p.pex", config.P2P.PexReactor, "Enable Peer-Exchange ")
	runNodeCmd.Flags().Int("p2p.max_num_peers", config.P2P.MaxNumPeers, "Set max num peers")
	runNodeCmd.Flags().Int("p2p.handshake_timeout", config.P2P.HandshakeTimeout, "Set handshake timeout")
	runNodeCmd.Flags().Int("p2p.dial_timeout", config.P2P.DialTimeout, "Set dial timeout")

	// log flags
	runNodeCmd.Flags().String("log_file", config.LogFile, "Log output file")

	RootCmd.AddCommand(runNodeCmd)
}
```

​	In `init()`, there are many different types of flag, which can be bound to `config`. Here is an example: 

> 在init()函数中定义了很多不同类型的flag参数，并将flag的参数值绑定到config对象上，比如：

```
runNodeCmd.Flags().Bool("mining", config.Mining, "Enable mining")
```

​	Description of the above code:

* Define a flag argument, which is Bool
* The name of this flag is mining
* The assignment object of this flag is `config.Mining`
* The description of this flag is `Enable mining`

> 上面语句的含义为：
>
> * 定义一个Bool类型的flag参数。
> * 该flag的名称为mining。
> * 该flag的赋值对象为config.Mining。
> * 该flag的描述信息为”Enable mining”。

​	The initialization of bytom daemon configuration ends here.

> 至此，bytom 守护进程所需要的配置信息初始化完毕。



### 3.4.3 Create File Lock (创建文件锁)

​	In this section, the program begins running.

​	In Bytom, a data directory(specified by `--root`) can only be read and written by one bytom daemon. That is because the Key-Value storage way of LevelDB is a single process. If there is a data file being written or read by many processes at the same time,  the consistency of data can't be ensured. Therefore, using file lock can ensure that there is a data file being written or read just by one process a given time.

> 从本小节开始，程序运行真正进入初始阶段。下面对此进行深入分析。
>
> 在比原链中，一份数据目录(--root参数指定)只能同时由一个bytom 守护进程读写。主要原因是由于LevelDB高性能K/V存储是单进程模式。如果存在多个进程同时读写一份数据会造成数据不一致的情况。所以使用文件锁可以保证同一时间一个进程读写一份数据目录。

node/node.go

```
if err := lockDataDirectory(config); err != nil {
	cmn.Exit("Error: " + err.Error())
}

func lockDataDirectory(config *cfg.Config) error {
	_, _, err := flock.New(filepath.Join(config.RootDir, "LOCK"))
	if err != nil {
		return errors.New("datadir already used by another process")
	}
	return nil
}
```

​	Once bytom starts, `lockDataDirectory()` will use `flock` to create a `LOCK` file in the `RootDir`. If bytomd locks the `inode` of a file, the error message of `errors.New` will be reported when bytomd restarts, and then the peocess will exit. The `flock` is for detecting the existing of process.

> bytomd启动时，lockDataDirectory函数使用flock在RootDir目录下创建一个LOCK文件。如果bytomd进程在一个文件的inode上加了锁，那么再次启动bytomd进程则会将errors.New中的内容报错并退出进程。flock的作用是检测进程是否已经存在。

​	There are three types of flock:

* LOCK_SH：Shared lock. More than one process may hold a shared lock for a given file at a given time.
* LOCK_EX：Exclusive lock. Only one process may hold an exclusive lock for a given file at a given time.
* LOCK_UN：Remove an existing lock held by this process.

> flock有如下几种主要的操作类型：
>
> * LOCK_SH：共享锁，多个进程使用同一把锁用于读锁。
> * LOCK_EX：排他锁，同时只允许一个进程使用，一般用于写锁。
> * LOCK_UN：释放锁。

​	Go deep into the pacakage `flock`, we can find there uses LOCK_EX, which means only one process may hold an exclusive lock for a given file at a given time. Here is the code:

> 在深入flock包的函数中我们可以看到，这里使用了LOCK_EX锁，同时只允许一个进程使用，代码示例如下：

vendor/github.com/prometheus/prometheus/util/flock/flock_unix.go

```
func (l *unixLock) set(lock bool) error {
	how := syscall.LOCK_UN
	if lock {
		how = syscall.LOCK_EX
	}
	return syscall.Flock(int(l.f.Fd()), how|syscall.LOCK_NB)
}
```



### 3.4.4 Types of Initial Network (初始化网络类型)

​	We have mentioned before that Bytom has three types of network, mainnet, testnet, solonet.

> 前面我们有讲到，比原链的三种网络模式。分别是mainnet主网、testnet测试网、solonet单机模式。

node/node.go

```
initActiveNetParams(config)

func initActiveNetParams(config *cfg.Config) {
	var exist bool
	consensus.ActiveNetParams, exist = consensus.NetParams[config.ChainID]
	if !exist {
		cmn.Exit(cmn.Fmt("chain_id[%v] don't exist", config.ChainID))
	}
}
```

​	Here `initActiveNetParams()` initializes the network according to the `chain_id` users send. `consensus.ActiveNetParams()` saves the type of present network, which is usually used in Bytom.

> 在这里initActiveNetParams函数则根据用户传入的chain_id来初始化网络类型。consensus.ActiveNetParams对象保存了当前使用的网络模式。consensus.ActiveNetParams对象在比原链代码中会经常引用到，用来识别当前节点连接的网络类型。

consensus/general.go

```
var ActiveNetParams = MainNetParams

var NetParams = map[string]Params{
	"mainnet": MainNetParams, 
	"wisdom":  TestNetParams, 
	"solonet": SoloNetParams,
}

var MainNetParams = Params{
	Name:            "main",
	Bech32HRPSegwit: "bm",
	Checkpoints: []Checkpoint{
		{10000, bc.NewHash([32]byte{0x93, 0xe1, 0xeb, 0x78, 0x21, 0xd2, 0xb4, 0xad, 0x0f, 0x5b, 0x1c, 0xea, 0x82, 0xe8, 0x43, 0xad, 0x8c, 0x09, 0x9a, 0xb6, 0x5d, 0x8f, 0x70, 0xc5, 0x84, 0xca, 0xa2, 0xdd, 0xf1, 0x74, 0x65, 0x2c})},
		{20000, bc.NewHash([32]byte{0x7d, 0x38, 0x61, 0xf3, 0x2c, 0xc0, 0x03, 0x81, 0xbb, 0xcd, 0x9a, 0x37, 0x6f, 0x10, 0x5d, 0xfe, 0x6f, 0xfe, 0x2d, 0xa5, 0xea, 0x88, 0xa5, 0xe3, 0x42, 0xed, 0xa1, 0x17, 0x9b, 0xa8, 0x0b, 0x7c})},
		{30000, bc.NewHash([32]byte{0x32, 0x36, 0x06, 0xd4, 0x27, 0x2e, 0x35, 0x24, 0x46, 0x26, 0x7b, 0xe0, 0xfa, 0x48, 0x10, 0xa4, 0x3b, 0xb2, 0x40, 0xf1, 0x09, 0x51, 0x5b, 0x22, 0x9f, 0xf3, 0xc3, 0x83, 0x28, 0xaa, 0x4a, 0x00})},
		{40000, bc.NewHash([32]byte{0x7f, 0xe2, 0xde, 0x11, 0x21, 0xf3, 0xa9, 0xa0, 0xee, 0x60, 0x8d, 0x7d, 0x4b, 0xea, 0xcc, 0x33, 0xfe, 0x41, 0x25, 0xdc, 0x2f, 0x26, 0xc2, 0xf2, 0x9c, 0x07, 0x17, 0xf9, 0xe4, 0x4f, 0x9d, 0x46})},
		{50000, bc.NewHash([32]byte{0x5e, 0xfb, 0xdf, 0xf5, 0x35, 0x38, 0xa6, 0x0b, 0x75, 0x32, 0x02, 0x61, 0x83, 0x54, 0x34, 0xff, 0x3e, 0x82, 0x2e, 0xf8, 0x64, 0xae, 0x2d, 0xc7, 0x6c, 0x9d, 0x5e, 0xbd, 0xa3, 0xd4, 0x50, 0xcf})},
		{62000, bc.NewHash([32]byte{0xd7, 0x39, 0x8f, 0x23, 0x57, 0xf9, 0x4c, 0xa0, 0x28, 0xa7, 0x00, 0x2b, 0x53, 0x9e, 0x51, 0x2d, 0x3e, 0xca, 0xc9, 0x22, 0x59, 0xfc, 0xd0, 0x3f, 0x67, 0x1a, 0x0a, 0xb1, 0x02, 0xbf, 0x2b, 0x03})},
	},
}
```

​	`ActiveNetParams` means using mainnet by default. Here is description of parameters  of `MainNetParam`：

* Bech32HRPSegw：Segregated Witness, an upgrade of protocol. See more details in Chapter05.
* Checkpoints：Checkpoint represents the height and hash of a block, is used to validate blocks when using fast synchronization. In general, the history of blocks will be hard coded in Checkpoints when updating the mainnet.

> ActiveNetParams默认使用主网。MainNetParams中的参数说明如下:
>
> * Bech32HRPSegwit：隔离见证，是一种协议升级，我们会在“第5章 内核层-区块与链”中详解隔离见证。
> * Checkpoints：检查点，指定一个高度，以及与这个高度相匹配的hash值，用于快速同步时验证区块的正确性。一般会在主网升级时，会将历史的块信息硬编码在Checkpoints中。

​	Checkpoints have two uses, one is to prevent forking, which means that the fork that is created before checkpoints will not be accepted by nodes. This can also peotect the network from 51% attack since attackers can't change transactions that are before checkpoints. The other use is for fast synchronization between nodes, see more details in Chapter10.

> Checkpoints检查点有两种作用，第一种作用防止分叉，如果有人试图从检查点之前的区块进行分叉，当前节点不会接受这个分叉。也用于保护网络不受全网51%的算力攻击，因为攻击者不可能逆转检查点之前的交易。第二种作用用于节点间的快速同步，我们在后面P2P章节中详细讲解。

### 3.4.5 Initialize Database (初始化数据库（持久化存储）)

​	All data (blocks information, transactions information, etc.) on the blockchain need to be stored in local K/V database once a pubilc blockchain is created. Bytom uses LevelDB as its database .

> 创建一条公链，需要将链上的所有数据（包含块信息、交易信息等）存储在本地K/V数据库中。在比原链中使用LevelDB作为链上数据存储。

node/node.go

```
coreDB := dbm.NewDB("core", config.DBBackend, config.DBDir())
store := leveldb.NewStore(coreDB)
```

database/leveldb/store.go

```
type Store struct {
	db    dbm.DB
	cache blockCache
}
```

​	`dbm` uses the tendermint db library. `dbm.NewDB` returns  `DB` , which offers the interface of database and many functions, including memory mapping, the directory structure of file system, LevelDB implementation in GO.

> dbm使用tendermint框架的db管理库。dbm.NewDB返回一个DB对象，DB对象提供了数据库接口和许多方法实现，包括使用内存映射，文件系统目录结构，GO中LevelDB的实现。

​	 `dbm.NewDB` needs three parameters: db name, db database (default  LevelDB), db path to return `DB`. `leveldb.NewStore()` reruns `Store` , which is the encapsulation of LevelDB. Bytom adds blocks cache, blocks validation, blocks status, blocks query, etc. based on LevelDB.

> dbm.NewDB返回一个DB对象，DB对象需要传入三个参数：db的名称、db使用的k/v存储（默认LevelDB）、db数据存储的路径。而leveldb.NewStore函数返回一个Store对象，Store是比原链对LevelDB进行了封装，在LevelDB的基础上增加了区块缓存cache、区块验证、区块状态、区块查询等功能。

### 3.4.6 Initialize Transaction Pool (初始化交易池)

​	When transactions are broadcasting in network, miners receive them and add them to local TxPool (transaction pool). Bytom TxPool is a limited buffer that can store 10000 transactions at most by default. If the number of transactions is over 10000, the error message "transaction pool reach the max number" will be returned.

> 当交易被广播到网络中并且被矿工接收到时，矿工会将收到的交易加入到本地的TxPool交易池中，TxPool对象是管理本地交易池。交易池相当于一个缓冲区，它并不是无限大。默认情况下在比原链中交易池最大可以存储10000笔交易。如果超出这个限制，则会返回"transaction pool reach the max number"的错误信息。

node/node.go

```
txPool := protocol.NewTxPool()

protocol/txpool.go
func NewTxPool() *TxPool {
	return &TxPool{
		lastUpdated: time.Now().Unix(),
		pool:        make(map[bc.Hash]*TxDesc),
		utxo:        make(map[bc.Hash]bc.Hash),
		errCache:    lru.New(maxCachedErrTxs),
		msgCh:       make(chan *TxPoolMsg, maxMsgChSize),
	}
}
```

​	`protocol.NewTxPool()` returns  `TxPool`. Here we only introduce the initialization od TxPool, see more details about the code in following chapters.	

> protocol.NewTxPool()返回一个一个TxPool实例对象。本小节只介绍交易池初始化部分，交易池实现原理的代码放到交易池部分深入剖析。

### 3.4.7 Create a Local Blockchain (创建一条本地区块链)

​	When the node firstly starts, it will check the status of local persistent storage. If the status is initialized,  the initialization of local blockchain will start, which means to add the genesis block to the blockchain at height-0.

> 当节点第一次启动时，判断本地持久化存储的状态，当状态为初始化时会初始化本地的区块链。区块链的第一个区块（创世区块）会被加入到区块高度为0的地方。

node/node.go

```
chain, err := protocol.NewChain(store, txPool)
if err != nil {
	cmn.Exit(cmn.Fmt("Failed to create chain structure: %v", err))
}
```

​	Here, `protocol.NewChain()`needs two parameters (store and txPool) to return `Chain`  . The `Chain` manages the whole Bytom blockchain .

> protocol.NewChain返回一个Chain对象，NewChain需要接收两个参数：store区块链的存储对象，txPool交易池。Chain对象管理着比原链的整个区块链条。

protocol/protocol.go

```
func NewChain(store Store, txPool *TxPool) (*Chain, error) {
	c := &Chain{
		orphanManage:   NewOrphanManage(),
		txPool:         txPool,
		store:          store,
		processBlockCh: make(chan *processBlockMsg, maxProcessBlockChSize),
	}
	c.cond.L = new(sync.Mutex)

	storeStatus := store.GetStoreStatus()
	if storeStatus == nil {
		if err := c.initChainStatus(); err != nil {
			return nil, err
		}
		storeStatus = store.GetStoreStatus()
	}

	var err error
	if c.index, err = store.LoadBlockIndex(); err != nil {
		return nil, err
	}

	c.bestNode = c.index.GetNode(storeStatus.Hash)
	c.index.SetMainChain(c.bestNode)
	go c.blockProcesser()
	return c, nil
}
```

​	The work of `NewChain()` is as follows:

（1） Instantiate  `Chain`

（2）`store.GetStoreStatus()` can gets the storage stautus of local blockchain. If it is nil, the blockchain hasn't been initialized. Run `initChainStatus` to add the genesis block to initialize local blockchain.

（3）`store.LoadBlockIndex()`  loads the block index and stores all `Block Header` read from database in memory. That is for improving the efficiency of getting block header.

（4）`c.index.SetMainChain` is used to set the newest block on the local node.

（5）`go c.blockProcesser()` will start a go routine to update blocks information of local blockchain. 

> NewChain函数可分为下面几个步骤:
>
> 1) 实例化Chain对象。
>
> 2) store.GetStoreStatus获取本地区块链的存储状态，如果状态为nil则说明区块链未被初始化。执行initChainStatus初始化本地区块链，该函数中初始化创世区块（第一个区块）并添加到本地链上。
>
> 3) store.LoadBlockIndex加载块索引，从数据库中读取所有Block Header信息并缓存在内存中，目的是能够加速访问区块头信息。
>
> 4) c.index.SetMainChain设置当前节点已同步的最新区块。
>
> 5) go c.blockProcesser()，启动一个go程，用于本地区块链上的区块信息更新。

### 3.4.8 初始化本地钱包

​	Bytom nodes will enable local wallet by default. Here is the code:

> 比原链节点默认情况下会启用本地钱包功能。代码实例如下：

node/node.go

```
hsm, err := pseudohsm.New(config.KeysDir())
if err != nil {
	cmn.Exit(cmn.Fmt("initialize HSM failed: %v", err))
}

if !config.Wallet.Disable {
	walletDB := dbm.NewDB("wallet", config.DBBackend, config.DBDir())
	accounts = account.NewManager(walletDB, chain)
	assets = asset.NewRegistry(walletDB, chain)
	wallet, err = w.NewWallet(walletDB, accounts, assets, hsm, chain)
	if err != nil {
		log.WithField("error", err).Error("init NewWallet")
	}

	// trigger rescan wallet
	if config.Wallet.Rescan {
		wallet.RescanBlocks()
	}
}
```

The process of the code above is as follows :（When bytom node starts)

（1）Create `hsm`, which manages keystore file that stores private key in JSON format. Keystore file is just code produced by encrypting private key and need to be used with the wallet password.

（2）Create wallet database

（3）Create account management object `walletDB`

（4）Create asset management object `accounts`

（5）Instantiate `Wallet`

（6）`RescanBlocks()` rescans all local blocks and starts wallet updating.

> 上述代码流程主要逻辑为，在比原节点启动时：
>
> 1) 创建加密机hsm对象，hsm对象管理keystore文件，该文件是存储私钥的一种格式（JSON）。keystore是一串代码，本质上是加密后的私钥，需配合钱包的密码来使用。
>
> 2) 创建钱包数据库。
>
> 3) 创建账户管理对象。
>
> 4) 创建资产管理对象。
>
> 5) 实例化Wallet对象。
>
> 6) RescanBlocks扫描本地所有区块，触发钱包更新操作。

### 3.4.9 Initialize Network Synchronization (初始化网络同步管理)

​	P2P communication module is managed by SyncManager, which manages the synchronization of information(transactions and blocks) between nodes in the business layer. Here is the code:

> P2P通信模块主要由SyncManager管理，SyncManager负责节点业务层信息的同步工作，即区块和交易信息的同步。代码如下：

node/node.go

```
const (
	maxNewBlockChSize = 1024
)

newBlockCh := make(chan *bc.Hash, maxNewBlockChSize)
syncManager, _ := netsync.NewSyncManager(config, chain, txPool, newBlockCh)
go newPoolTxListener(txPool, syncManager, wallet)
```

​	The description of main parameters :

* newBlockCh：Chanel is used to mine blocks and broadcast quickly to other nodes. The size of chanel is 1024.
* netsync.NewSyncManager：Instantiate `syncManager`, which manages the synchronization of blocks and transactions between nodes.
* newPoolTxListenner：Run a go routine to listen transactions in TxPool and send transactions to `syncManager` or local wallet.

> 主要参数说明如下：
>
> * newBlockCh：通道用于新挖掘出的区块进行快速广播给其他节点。通道大小为1024。
> * netsync.NewSyncManager：实例化syncManager同步管理对象，它管理节点与节点之间的区块、交易信息同步。
> * newPoolTxListenner：启动一个go程，监听交易池中的交易，将交易发送给syncManager同步管理对象或本地钱包。

​	See more details in `Chapter 10 P2P Network`.

> 详细实现机制内容在P2P章节详细讲解。

### 3.4.10 初始化Pprof性能分析工具

​	`pprof` is a tool for visualization and analysis of profiling data in GO library. `ppoof` is used to analyze memory and CPU, track code, etc. It can also generate both text and graphical reports (through the use of the dot visualization package). (See more details on https://golang.org/pkg/net/http/pprof/). In bytom, `ppof` is disabled by default, users can enable it by `--prof_laddr`. Here is the code:

> pprof是GO语言标准库中自带的性能分析工具。用于内存分析、CPU分析、代码追踪等，还可以生成性能分析图表。（详细参考： https://golang.org/pkg/net/http/pprof/）。在比原链中默认不启用该功能，可以使用--prof_laddr参数启动代码性能分析功能，代码示例如下：

node/node.go

```
profileHost := config.ProfListenAddress
if profileHost != "" {
	go func() {
		http.ListenAndServe(profileHost, nil)
	}()
}
```

### 3.4.11 Initialize CPU Mining (初始化CPU挖矿功能)

​	Bytom code just has CPU mining, which seems impossible to mine BTM according to the present computing power of network. Nowadays, mining equipments are mainly produced by Bitmain with its special mining chips, or use GPU like most mining pools. See more details about mining and mining pools in following chapters. Here is the code:

> 在比原链节点源码中只提供了CPU设备的挖矿功能，以目前全网的算力来看，CPU设备挖矿几乎挖不到BTM币了。目前主流的挖矿设备，由比特大陆定制的挖矿芯片或各大矿池使用GPU设备挖矿。挖矿和矿池细节我们在后面章节中详细解读。代码实例如下：

node/node.go

```
node.cpuMiner = cpuminer.NewCPUMiner(chain, accounts, txPool, newBlockCh)
node.miningPool = miningpool.NewMiningPool(chain, accounts, txPool, newBlockCh)

if config.Simd.Enable {
	tensority.UseSIMD = true
}
```

​	`sim` is used to optimize Tensority.

> 其中simd参数是用于Tenaority CPU指令优化。

 

## **3.5** Enable Bytom Daemon (bytom守护进程方式启动)

​	Daemon is a long-term running process for running some specific system tasks. It will keep running as a background program until receiving a specific signal and then exit. Most processes in Linux run as daemons.

> 守护进程是在一种长久运行的进程，用于执行特定的系统任务。守护进程一般长久运行在后台直到收到特定信号则退出。Linux系统中大部分进程以守护进程的方式运行。

​	Daemon listens for `SIGTERM`. In this way, it can keep blocked and will not exit unless receiving `SIGTERM`. And only if daemon receives `SIGTERM`, it will not be blocked and exit. See more details on http://man7.org/linux/man-pages/man7/signal.7.html. Here is the code:

> 我们在GO语言下实现守护进程的方式一般以监听标准的SIGTERM信号，进程监听SIGTERM信号后，进程处于阻塞状态并不会退出，这样便实现了守护进程的方式。只有当进程收到来自外部的SIGTERM信号时时，进程则处于非阻塞状态。实现进程退出。LINUX信号参考（http://man7.org/linux/man-pages/man7/signal.7.html）。代码实例如下：

cmd/bytomd/commands/run_node.go

```
func runNode(cmd *cobra.Command, args []string) error {
	// ...
	n.RunForever()
	// ...
}
```

node/node.go

```
func (n *Node) RunForever() {
	// Sleep forever and then...
	cmn.TrapSignal(func() {
		n.Stop()
	})
}
```

vendor/github.com/tendermint/tmlibs/common/os.go

```
func TrapSignal(cb func()) {
	c := make(chan os.Signal, 1)
	signal.Notify(c, os.Interrupt, syscall.SIGTERM)
	go func() {
		for sig := range c {
			fmt.Printf("captured %v, exiting...\n", sig)
			if cb != nil {
				cb()
			}
			os.Exit(1)
		}
	}()
	select {}
}
```

​	`signal.Notify()` listens for the signal of `Interrupt` and `Term`. Go routine will be enabled to get `c` and `select{}` keeps blocking  `TrapSignal()`. After receiving `Term`, `os.Exit(1)` will be run and then daemon exits.

> signal.Notify监听中断和Term信号。启用GO程读取c对象，select进入阻塞状态。当进程接收到Term信号则通知c对象则执行os.Exit退出守护进程。

​	There are two ways to send `Term`: one is running `kill -15 pid`, the other is using `ctrl+c`.

> 发送Term信号有两种方式：第一种方式执行命令kill -15 pid。第二种方式，如果进程运行在前台则（ctrl+c）即可。

## **3.6** Stop Bytom Daemon (bytom 守护进程停止流程)

​	After receiving `Term`, the daemon needs to finish the final work to exit, such as disabling mining, P2P synchronization, etc. Here is the code: 

> 当守护进程接收到Term信号后，守护进程退出之前需要做扫尾工作，如退出挖矿模式，退出P2P同步功能等。代码示例如下：

node/node.go

```
func (n *Node) OnStop() {
	n.BaseService.OnStop()
	if n.miningEnable {
		n.cpuMiner.Stop()
	}
	if !n.config.VaultMode {
		n.syncManager.Stop()
	}
}
```

## **3.7** Conclusion (本章总结)

​	In this chapter, we use the coed to analyze the sart process of bytomd, especially the initialization of `Node`. There are many relevant modules which all need to be initialized. The emphasis is to summarize the logic of bytomd implementation.

> 本章从源码的角度深入分析了bytomd启动过程中的Node对象创建和初始化。在这一章内容中涉及的模块和内容极多，对所有模块进行初始化操作。本章着重归纳总结bytomd实现的逻辑。

 
