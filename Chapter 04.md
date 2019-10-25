# Chapter 04 Interface Layer

## 4.1 Introduction 

​	ApiServer is an important component of Bytom, which is used to listen to, process and response to requests. Its main functions are receiving users' requests, distributing them according to the routing rules and returningneir results to users.

> 比原链架构中，ApiServer接口层是一个重要组件，是一个监听请求、处理请求、响应请求的服务端。主要功能接收用户发送的请求，并按照相应的路由规则实现请求的路由分发，最终将请求处理后的结果响应给用户。

​	This chapter covers:

* Create a simple HTTP Server in GO language to make readers understand the process of creating HTTP server.
* Introduce how ApiServer creates HTTP Server, listen ports and receive requests and the process of requests processing and response in HTTP protocol.
* A complete HTTP request life cycle.

> 本章将从源码角度分析比原链的ApiServer的工作流程。内容如下：
>
> * 使用GO语言实现一个简易HTTP Server服务，让读者了解HTTP服务的创建过程
> * ApiServer如何创建HTTP服务、监听端口、接收请求的过程，以及HTTP协议中请求处理和响应流程。
> * 一个HTTP请求完整的生命周期详细介绍。

 

## 4.2 Create a Simple HTTP Server(实现一个简易HTTP Server)

​	In GoLang, the `net/http` library offers HTTP programming related interfaces and encapsulates many functions such as TCP link and message parsing. `http.request` object and `http.ResponseWriter` object are already enough for users interaction. The arguments of requests will be sent and processed by the handler, which also writes the result to the `Response`. Here is the code of a simple HTTP Server:

> GO语言的标准库net/http提供了HTTP编程相关的接口，封装了内部TCP连接和报文解析等功能，使用者只需要http.request和http.ResponseWriter两个对象交互就足够了。我们实现handler，请求会通过参数传递进来，会根据请求的数据做处理，把结果写到Response中。下面我们实现一个简易的HTTP服务，代码如下所示：

```
package main

import (
	"net"
	"net/http"
)

func sayHello(w http.ResponseWriter, r *http.Request) {
	w.Write([]byte("Hello, World!"))
}

func main() {
	mux := http.NewServeMux()
	mux.HandleFunc("/", sayHello)

	server := &http.Server{
		Handler: mux,
	}

	listener, err := net.Listen("tcp", ":8080")
	if err != nil {
		panic(err)
	}

	err = server.Serve(listener)
	if err != nil {
		panic(err)
	}
}
```

​	When this HTTP Service runs, it sends a request to local `8080` port and returns `Hello,Word!` .

> 当我们运行上面的HTTP服务以后，请求本地的8080端口，HTTP服务会返回给我们“Hello, World”信息。

​	An HTTP Service is created by these three steps:

* Instantiate `http.NewServeMux()` to get the route of `mux`, then add valid routers to `mux.Handle`. Each router has HTTP request method (GET, POST, PUT, DELET), URL and Handler callback function.
* Listen the `8080` port.
* Use the listening address as an argument. The HTTP Service serves external requests by running`Serve(listener) `.

> 一个HTTP服务的创建主要有三部分组成：
>
> * 实例化http.NewServeMux()得到mux路由。为mux.Handle添加多个有效的router路由项。每一个路由项由HTTP请求方法（GET、POST、PUT、DELET）、URL和Handler回调函数组成。
> * 监听本地的8080端口。
> * 将监听地址作为参数，最终执行Serve(listener)开始服务于外部请求。

 

## 4.3 Create HTTP Service (ApiServer创建HTTP服务)

### 4.3.1 Create API Object (创建API对象)

​	`ApiServer` is managed by `API` object. Before running `ApiServer`, `API` object need to be initialized. Here is the code:

> API对象负责管理整个ApiServer的管理工作，在运行ApiServer之前，首先要初始化它。代码如下所示：

node/node.go

```
func (n *Node) initAndstartApiServer() {
	n.api = api.NewAPI(n.syncManager, n.wallet, n.txfeed, n.cpuMiner, n.miningPool, n.chain, n.config, n.accessTokens)

	listenAddr := env.String("LISTEN", n.config.ApiAddress)
	env.Parse()
	n.api.StartServer(*listenAddr)
}
```

api/api.go

```
func NewAPI(sync *netsync.SyncManager, wallet *wallet.Wallet, txfeeds *txfeed.Tracker, cpuMiner *cpuminer.CPUMiner, miningPool *miningpool.MiningPool, chain *protocol.Chain, config *cfg.Config, token *accesstoken.CredentialStore) *API {
	api := &API{
		sync:          sync,
		wallet:        wallet,
		chain:         chain,
		accessTokens:  token,
		txFeedTracker: txfeeds,
		cpuMiner:      cpuMiner,
		miningPool:    miningPool,
	}
	api.buildHandler()
	api.initServer(config)

	return api
}
```

​	First of all, `API` object is initialized by `api.NewAPI`. There are many arguments in `ApiServer`:

* listenAddr：Local port. If the system environment variable `LISTEN` isn't set, `config.ApiAddress` (default 9888) will be used as `listenAddr`.
* n.api.StartServer: Listen local port addresses and start HTTP Service.

> 首先，api.NewAPI初始化API对象。ApiServer管理的事情很多，所以参数也相对较多，其中：
>
> * listenAddr：本地端口，如果系统没有设置LISTEN环境变量则使用config.ApiAddress配置地址，默认为9888。
> * n.api.StartServer：监听本地的地址端口，启动HTTP服务。

 	`NewAPI()` method has three operations:

* Instantiate `API` object.
* Add routers by `api.buildHandle()` method.
* Instantiate `http.Server`, set `auth`, etc. by `api.initServer()` method.

> NewAPI函数我们看到有三个操作：
>
> * 实例化API对象。
> * api.buildHandler添加router路由项。
> * api.initServer实例化http.Server，配置auth验证等。

 

### 4.3.2 Create router (创建router路由项)

​	HTTP requests are sent to corresponding callback functions through router that has been matched successfully. Only handlers related to account will be introduced here, since Bytom has too many handler and they are all similar. Here is the code:

> router路由被匹配后，负责将HTTP请求交给对应的回调函数。比原链代码中路由项过多。这里只介绍关于账号相关的handler，其它的handler大同小异。代码如下所示：

```
func (a *API) buildHandler() {
	walletEnable := false
	m := http.NewServeMux()
	
	// ...
	m.Handle("/net-info", jsonHandler(a.getNetInfo))
	// ...

	handler := latencyHandler(m, walletEnable)
	handler = maxBytesHandler(handler) // TODO(tessr): consider moving this to non-core specific mux
	handler = webAssetsHandler(handler)
	handler = gzip.Handler{Handler: handler}
	// ...
}
```

​	Here it uses `http.NewServeMux()` in Go library to create a router to distribute routes. As we can see, each router is made of a url and a handle callback function. Once the url user requested matches `/net-info`, `ApiServer` will run `a.getNetInfo` function and send arguments.

> 我们使用Golang标准库http.NewServeMux()创建一个router路由器，提供请求的路由分发功能。我们可以看到一条router项由url和对应的handle回调函数组成。当我们请求的url匹配到/net-info时，ApiServer会执行a.getNetInfo回调函数，并将用户的传参也带过去。

​	Other handler work:

* latencyHandler：When requests can't find the web path, it will match `/error` with requests.
* maxBytesHandler：It limit the size of requests with no more than 10MB for each.
* webAssetsHandler：It adds routers of dashboard and equity pages, which are hard-coded in Bytom. Their paths are `dashboard/dashboard.go` and `equity/equity.go` respectively.
* gzip.Handler：To enable `gzip`, the level of `gzip.BestSpeed` need to be set (default level-1 and the highest is level-9). Higher level, more time cpu uses. The level can be adjusted according to the the amount of data HTTP Service receives .

> 额外的handler处理：
>
> * latencyHandler：当请求找不到网址路径时，将请求重定向到"/error"错误路径。
> * maxBytesHandler：限制请求的字节大小，一个请求不允许超过10MB。
> * webAssetsHandler：添加dashboard和equity的路由项，dashboard和equity页面代码被硬编码到bytomd中，路径分别在dashboard/dashboard.go、equity/equity.go。
> * gzip.Handler：启用gzip数据压缩，需指定gzip.BestSpeed压缩级别（默认level 1）。最高可指定level 9，随着压缩级别增加，cpu耗时也会增加，可根据HTTP服务接收的数据量调整该压缩级别。



### 4.3.3 Instantiate http.Server (实例化http.Server)

​	`http.Server` is an HTTP Service object. It is the basis of HTTP Service work. Here is the code:

> http.Server是一个HTTP服务对象， 是运行HTTP服务的基础。代码如下所示：

```
func (a *API) initServer(config *cfg.Config) {
	// ...

	coreHandler.wg.Add(1)
	mux := http.NewServeMux()
	mux.Handle("/", &coreHandler)

	handler = mux
	if config.Auth.Disable == false {
		handler = AuthHandler(handler, a.accessTokens)
	}
	handler = RedirectHandler(handler)

	secureheader.DefaultConfig.PermitClearLoopback = true
	secureheader.DefaultConfig.HTTPSRedirect = false
	secureheader.DefaultConfig.Next = handler

	a.server = &http.Server{
		Handler:      secureheader.DefaultConfig,
		ReadTimeout:  httpReadTimeout,
		WriteTimeout: httpWriteTimeout,
		TLSNextProto: map[string]func(*http.Server, *tls.Conn, http.Handler){},
	}

	coreHandler.Set(a)
}
```

​	There are three steps to instantiate `http.Server` :

1) AuthHandler: It enables `auth`, which means users need to enter `access token` on `dashboard` to login.  `auth` is enabled by default .

2) RedirectHandler：When users enter  "/", it will jump to "/dashboard/" and set the status "302" by default.

3) http.Server: Set the timeout of reading and writing and instantiate `http.Server`。

> 实例化http.Server分为以下几步：
>
> 1) AuthHandler：启用auth验证功能，用户打开dashboard时需要输入access token进行验证登录。默认开启auth功能。
>
> 2) RedirectHandler：重定向，当用户输入"/"时，默认跳转到"/dashboard/"，并设置状态为302。
>
> 3) http.Server：设置读写超时时间并实例化http.Server。

​	By now, all handers have been registered in HTTP Server and `http.Server` object also has been instantiated. We will listen ports and enable HTTP Service next.

> 到目前为止，所有的handle已经被注册到HTTP 服务器中并实例化http.Server服务对象，下面操作监听端口，启动HTTP服务。



### 4.3.4 Enable Api Server (启动ApiServer服务)

​	Use `net.listen` in GoLang library to listen local ports addresses. Here starts a goroutine to run HTTP service due to HTTP service is a persistent working. If there was no error message when running `a.server.Serve`, the `9888` port is started successfully. By now, `ApiServer ` has been waiting for users' requests and starts a goroutine to process each request once received. Here is the code:

> 通过Golang标准库net.listen方法，监听本地的地址端口。由于HTTP服务是一个持久运行的服务，我们启动一个goroutine专门运行HTTP服务。当运行a.server.Serve没有任何报错时，我们可以看到服务器上启动的9888端口。此时ApiServer已经处于等待接收用户的请求的状态，每当有请求被接收后，ApiServer会单独启用一个goroutine来处理该请求。代码如下所示： 

api/api.go

```
func (a *API) StartServer(address string) {
	log.WithField("api address:", address).Info("Rpc listen")
	listener, err := net.Listen("tcp", address)
	if err != nil {
		cmn.Exit(cmn.Fmt("Failed to register tcp port: %v", err))
	}

	go func() {
		if err := a.server.Serve(listener); err != nil {
			log.WithField("error", errors.Wrap(err, "Serve")).Error("Rpc server")
		}
	}()
}
```

​	The process of `server.Serve` working is as follows:

1) `a.server.Serve` starts a `for` loop to keep receiving accept requests.

2) Instantiate a  `Conn` for each request and also start a `goroutine` for this request to operate `handle`.

3) `c.readRequest(ctx)` reads each request.

4) Chose `handler` according to request and go into the HTTP Server of this handler.

5) Call `w.finishRequest` to delete conn buffer data and stop tcp link. Request is over.

> 我们来跟踪server.Serve的代码流程:
>
> 1) a.server.Serve内部启动一个for循环，在循环体中接收Accept请求.
>
> 2) 对每个请求实例化一个Conn，并开启一个goroutine为这个请求执行handle.
>
> 3) c.readRequest(ctx)读取每个request请求的内容 。
>
> 4) 根据request选择handler，并且进入到这个handler的ServeHTTP。
>
> 5) 最后调用w.finishRequest()刷掉conn缓冲区的数据，关闭tcp连接，结束请求。

 

### 4.3.5 Receive and Respond to Request (接收并响应请求)

​	When starting an HTTP request, we can use `curl` commdline to make an HTTP request to get the network status of the current node. Here is the information of `ApiServer` response: 

> 当我们发起一个HTTP请求，使用curl命令发起HTTP请求获取当前节点运行的网络状态，ApiServer响应给我们如下数据：

```
$ curl -s http://localhost:9888/net-info|jq .
{
  "status": "success",
  "data": {
    "listening": true,
    "syncing": false,
    "mining": false,
    "peer_count": 0,
    "current_block": 63304,
    "highest_block": 63304,
    "network_id": "mainnet",
    "version": "1.0.5+2bc2396a"
  }
}
```

​	Anyalse the process of  `ApiServer` :

> 解析ApiServer的处理过程：

 api/api.go

```
m.Handle("/net-info", jsonHandler(a.getNetInfo))
```

​	`ApiServer` parses the head of HTTP, matches the path with `/net-info` and jumps to `a.getNetInfo` function.

> ApiServer解析HTTP头，匹配到path路径/net-info，跳转至a.getNetInfo回调函数。

 api/nodeinfo.go

```
func (a *API) getNetInfo() Response {
	return NewSuccessResponse(a.GetNodeInfo())
}
```

​	`getNetInfo()` function gets the object information by `P2P`, `cpuMinner`, etc. `getNetInfo()` serializes `NetInfo` structure to JSON format and then returns it to users by instantiating `Response`.

> GetNetInfo函数通过P2P、cpuMinner等对象获取信息。getNetInfo函数将NetInfo结构体序列化至JSON格式，并通过实例化Response返回给用户。

 api/api.go

```
type Response struct {
	Status      string      `json:"status,omitempty"`
	Code        string      `json:"code,omitempty"`
	Msg         string      `json:"msg,omitempty"`
	ErrorDetail string      `json:"error_detail,omitempty"`
	Data        interface{} `json:"data,omitempty"`
}

func NewS\uccessResponse(data interface{}) Response {
	return Response{Status: SUCCESS, Data: data}
}
```

​	`Response` is instantiated by `NewSuccessResponse`, and the HTTP status  code is 200. Here is the description:

* Status: The status string
* Code: The status code
* Msg: The status description
* ErrorDetail: Error message. It is null when requesting successfully
* Data: Data to response users.

> NewSuccessResponse实例化Response，并设置HTTP状态码为200。Response结构描述如下：
>
> * Status：状态码字符描述。
> * Code：状态码。
> * Msg：状态码对应的字符描述。
> * ErrorDetail：错误信息，请求成功时，错误信息为空。
> * Data：响应用户的数据。

## 4.4 The Life Cycle of an HTTP Request (HTTP请求的完整生命周期)

​	`ApiServer` just offers HTTP short link. The link will be shut down after an HTTP request/response and needs to be rebuild when the next HTTP request/response happens. Here is a picture to show the life cycle of an HTTP request : 

> ApiServer服务提供HTTP短连接（非持久连接）服务请求。客户端和服务端进行一次HTTP请求/响应之后，就关闭连接。下一次的HTTP请求/响应操作就需要重新建立连接。一次HTTP请求的完整生命周期如下图所示：

![4-1 The life cycle of an HTTP request](https://github.com/Bytom-Community/Bytom-Tech-Book/blob/master/Chapter04_pic/pic_01.png)

The complete life cycle of HTTP request:

1) User sends an HTTP request to ApiServer.

2) ApiServer receives the HTTP request.

3) Start a goroutine to process the request.

4) Verify the auth of the request.

5) Parse the request.

6) Call the handle callback function by router.

7) Get handle information.

8) Set request status code.

9) Respond to user's request.

> HTTP请求的完整生命周期过程：
>
> 1) 用户向ApiServer发出HTTP请求。
>
> 2) ApiServer接收到用户发出的请求。
>
> 3) 启用goroutine处理接收到的请求。
>
> 4) 验证请求内容中的auth信息。
>
> 5) 解析请求内容。
>
> 6) 调用路由项对应的handle回调函数。
>
> 7) 获取handle的数据信息。
>
> 8) 设置请求状态码。
>
> 9) 响应用户的请求。

 

## 4.5 Bytom API Description (比原链API接口描述)

#### 1. Wallet related API (钱包相关接口)

| API                      | Description                                                  |
| ------------------------ | ------------------------------------------------------------ |
| /create-account          | Create account                                               |
| /list-accounts           | Returns the list of all available accounts.                  |
| /delete-account          | Delete existed account                                       |
| /create-account-receiver | Create address and control program                           |
| /list-addresses          | Returns the list of all available addresses by account.      |
| /validate-address        | Verify the address is valid, and judge the address is own.   |
| /list-pubkeys            | Returns the list of all available pubkeys by account.        |
| /get-mining-address      | Query the current mining address                             |
| /set-mining-address      | Set the current mining address                               |
| /get-coinbase-arbitrary  | Get coinbase arbitrary                                       |
| /set-coinbase-arbitrary  | Set coinbase arbitrary                                       |
| /create-asset            | Create asset definition                                      |
| /update-asset-alias      | Update asset alias by assetID                                |
| /get-asset               | Query detail asset by asset ID                               |
| /list-assets             | Returns the list of all available assets                     |
| /create-key              | Create private key                                           |
| /list-keys               | Returns the list of all available keys.                      |
| /delete-key              | Delete existed key                                           |
| /reset-key-password      | Reset key password                                           |
| /check-key-password      | Check key password                                           |
| /sign-message            | Sign a message with the key password(decode encrypted private key) of an address |
| /build-transaction       | Build transaction                                            |
| /sign-transaction        | Sign transaction                                             |
| /get-transaction         | Query the account related transaction by transaction ID      |
| /list-transactions       | Returns the sub list of all the account related transactions |
| /list-balances           | Returns the list of all available account balances.          |
| /list-unspent-outputs    | Returns the sub list of all available unspent outputs for all accounts in your wallet. |
| /backup-wallet           | Backup wallet to image file                                  |
| /restore-wallet          | Restore wallet by image file                                 |
| /rescan-wallet           | Trigger to rescan block information into related wallet      |
| /wallet-info             | Return the information of wallet                             |

#### 2.Token related  API (Token验证相关接口)

| API                  | Description                                     |
| -------------------- | ----------------------------------------------- |
| /create-access-token | Create access token                             |
| /list-access-tokens  | Returns the list of all available access tokens |
| /delete-access-token | Delete existed access token                     |
| /check-access-token  | Check access token is valid                     |

#### 3.Transaction related API (交易相关接口)

| API                            | Description                                                  |
| ------------------------------ | ------------------------------------------------------------ |
| /submit-transaction            | Submit transaction                                           |
| /estimate-transaction-gas      | Submit transactions used for batch submit transactions       |
| /get-unconfirmed-transaction   | Estimate consumed neu(1BTM = 10^8NEU) for the transaction    |
| /list-unconfirmed-transactions | Query mempool transaction by transaction ID                  |
| /decode-raw-transaction        | Decode a serialized transaction hex string into a JSON object describing the transaction |

#### 4. Block related API (块信息相关接口)

| API               | Description                                                  |
| ----------------- | ------------------------------------------------------------ |
| /get-block        | Returns the detail block by block height or block hash       |
| /get-block-hash   | Returns the current block hash for blockchain                |
| /get-block-header | Returns the detail block header by block height or block hash |
| /get-block-count  | Returns the current block height for blockchain              |
| /get-difficulty   | Returns the block difficulty by block height or block has    |
| /get-hash-rate    | Returns the block hash rate by block height or block hash    |
| /is-mining        | Returns the mining status                                    |
| /set-mining       | Start up node mining                                         |

#### 5. Mining Pool related API (矿池相关接口)

| API               | Description                               |
| ----------------- | ----------------------------------------- |
| /get-work         | Get the proof of work in Varint format    |
| /get-work-json    | Get the proof of work by JSON             |
| /submit-work      | Submit the proof of work in Varint format |
| /submit-work-json | Submit the proof of work by JSON          |
| /is-mining        | Returns the mining status                 |
| /set-mining       | Start up node mining                      |

#### 6. Contract related API (合约相关接口)

| API             | Description                                                |
| --------------- | ---------------------------------------------------------- |
| /verify-message | Verify a signed message with derived pubkey of the address |
| /decode-program | Decode program                                             |
| /compile        | Compile equity contract                                    |

#### 7. P2P related API (P2P网络相关接口)

| API              | Description                                     |
| ---------------- | ----------------------------------------------- |
| /net-info        | Returns the information of current network node |
| /list-peers      | Returns the list of connected peers             |
| /disconnect-peer | Disconnect to specified peer                    |
| /connect-peer    | Connect to specified peer                       |

 

## 4.6  API Tools (API接口调用工具)

​	API needs to be tested when developing a public blockchain. There are many test tools, `curl` commandline and `postman` platform are recommended here. We take the `create-key` as an example in the following part.

> 在公链开发中，我们需要对API接口进行测试。测试工具有很多种，本节推荐使用两种方式curl命令或postman图形工具，在这里以create-key接口示例演示。

### 4.6.1 Call API via `curl` commandline (使用curl命令行调用API接口)

`curl` is a command line tool and library for transferring data with URLs. Use `curl` commandline to send request in the network, then get and collect data and finally show it in the `standard output`.

> curl命令是利用URL语法在命令行方式下工作的开源文件传输工具。curl命令发出网络请求，然后得到和提取数据，并显示在"标准输出"中。

​	Here we use `curl` commandline to call `create-key` to create private key and return it. Here is the code:

> 我们使用curl命令，请求访问create-key接口，创建私钥，并返回密钥的信息。代码示例如下：

```
$ curl -X POST http://localhost:9888/create-key -d '{ "alias" :"user1" , "password":"123456"}'

{"alias":"user1",
"xpub":"e441f31e16276902d305ab5eb6fb686ec3954a59c00712c5ee7565c93f16589b75ea5e11aa09122962ff7bd04c9dfff5027ca10ac9bea64722934850f6954f56",
"file":"C:\\Users\\Mac\\AppData\\Roaming\\Bytom\\keystore\\UTC--2018-09-21T11-51-52.401546400Z--e35c76e1-0f6c-4431-bb0f-eca37206d885"}}
```

​	Here are the arguments of `curl` commandline:

* -X: Set the method of request, such as GET, POST, DELETE, etc.
* -d: Use `POST` to send ApiServer data.
* Others can be learn on the internet.

> curl命令参数如下：
>
> * -X：指定请求方式，请求方式包括GET、POST、DELETE等。
> * -d：使用POST方式向ApiServer发送数据。
> * 其他参数请读者自行学习。

​	Fields of the API result: 

* alias: The alias of the account is `user1`.
* xpub: Public key.
* file: The path of keystore.

> API接口返回字段结果为：
>
> * alias：别名user1。
> * xpub：公钥。
> * file：密钥文件的存储路径。

### 4.6.2 Call API via Postman (使用Postman调用API接口)

​	Postman is a collaboration platform for API development. Its features simplify each step of building an API and streamline collaboration so you can create better APIs—faster.

​	Dowland Postman on https://www.getpostman.com/apps.

> Postman是一个图形化接口请求工具，可以帮助我们更方便地模拟GET、POST及其他方式的请求来调用接口。
>
> 下载postman（官网下载：https://www.getpostman.com/apps）。

​	Use Postman to call API by `POST` method. Enter request URL ( `http://127.0.0.1:9888/create-key` ) , enter `{"alias": "user0", "password": "123456"` in `raw` under `Body` in `Text` format, and click `send` button. It returns information that is as same `curl` above: 

> 使用postman调用API接口，选择POST请求方式，输入http://127.0.0.1:9888/create-key；在Body标签下raw，选择Text格式，输入:{"alias": "user0", "password": "123456"};点击send，返回如上述curl下create-key相同格式内容：

![4-2 The Postman](https://github.com/Bytom-Community/Bytom-Tech-Book/blob/master/Chapter04_pic/pic_02.png)

​	Call `create-access-token` via postman is as same. Moreover, the list of all available keys will be returned by `list-keys`.

> postman下进行create-access-token同理，此处不再加以赘述。此外可通过list-keys来查看所创建的所有私钥。

***Note: Everytime create an account needs to change the alias, or it will return fail message that the alias has been created.***





## 4.7 Bytom Error Code (比原链HTTP错误码一览)

bytom/api/errors.go

* 0xx—— API errors
* 1xx—— network errors
* 2xx—— signature related errors
* 7xx—— transaction related errors
* 72x - 73x—— transaction building errors
* 73x - 75x—— transaction validation errors
* 76x - 78x—— BVM errors
* 8xx—— HSM related errors

> * 0xx——API错误。
> * 1xx——网络错误。
> * 2xx——签名相关的错误。
> * 7xx——交易相关的错误。
> * 72x - 73x——构建交易错误。
> * 73x - 75x——验证交易错误。
> * 76x - 78x——虚拟机错误。
> * 8xx——HSM相关错误。

 

## 4.8 Conclusion (本章总结)

​	This chapter makes a practical example to show the process of creating a simple HTTP Server in GoLang. It includes the principle and the working process (receiving, processing, responding ) of HTTP Server and shows a whole life cycle by a diagram. In the last place, more attention is paid to how public blockchain offers HTTP API service to users. Bytom API has covered most of public blockchain related API, and also developed some extra API compared with other public blockchains, such as mining pool related API, token related API, etc.

> 本章介绍了如何使用GO语言标准库实现一个简易的HTTP Server服务。其中包括HTTP服务的工作原理，接收、处理、响应等过程。并以图文的方式介绍了一个HTTP请求的完整生命周期。最后详解了作为公链如何提供HTTP相关的接口。比原链API接口中已经包含了大部分公链所具备的接口，相比其他公链中比原链扩展了其他接口，比如矿池相关接口、token验证相关接口等。
