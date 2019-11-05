# Chapter 04 Interface Layer

## 4.1 Introduction 

​	ApiServer is an important component of Bytom blockchain. ApiServer listens to user requests, routes them to different code blocks based on the routing rules and returns responses to users.

> 比原链架构中，ApiServer接口层是一个重要组件，是一个监听请求、处理请求、响应请求的服务端。主要功能接收用户发送的请求，并按照相应的路由规则实现请求的路由分发，最终将请求处理后的结果响应给用户。

​	This chapter walks through APIServer implementation and shows:

* How to create a simple HTTP Server in Go language.
* How ApiServer implements a http server which accepts requests and returns responses using HTTP protocol.
* HTTP request life cycle.

> 本章将从源码角度分析比原链的ApiServer的工作流程。内容如下：
>
> * 使用GO语言实现一个简易HTTP Server服务，让读者了解HTTP服务的创建过程
> * ApiServer如何创建HTTP服务、监听端口、接收请求的过程，以及HTTP协议中请求处理和响应流程。
> * 一个HTTP请求完整的生命周期详细介绍。

 

## 4.2 A Simple HTTP Server Implementation(实现一个简易HTTP Server)

​	The `net/http` package from Golang standard library provides an implementation of HTTP protocol with necessary interfaces and functionality such as TCP connection and message parsing. You can write a simple http request handler with two arguments `http.request` and `http.ResponseWriter`. The http handler can access request parameters and write out the result as a `Response`. A very basic implementation of HTTP Server looks as follows:

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

​	Start this HTTP Service and make a http request on localport `8080` to http://localhost:8080/ you will get a `Hello, World!` response.

> 当我们运行上面的HTTP服务以后，请求本地的8080端口，HTTP服务会返回给我们“Hello, World”信息。

​	An HTTP Service can be created in three steps:

* Instantiate a new HTTP request multiplexer using `http.NewServeMux()` into `mux` variable, then add a route and a handler for this route using `mux.HandleFunc`. Each route has HTTP request method (GET, POST, PUT, DELETE), a URL path and callback Handler function.
* Listen to local `8080` port and create a listener object.
* Using the `listener` as an argument, start http service by calling `Serve(listener) `.

> 一个HTTP服务的创建主要有三部分组成：
>
> * 实例化http.NewServeMux()得到mux路由。为mux.Handle添加多个有效的router路由项。每一个路由项由HTTP请求方法（GET、POST、PUT、DELETE）、URL和Handler回调函数组成。
> * 监听本地的8080端口。
> * 将监听地址作为参数，最终执行Serve(listener)开始服务于外部请求。

 

## 4.3 ApiServer HTTP Service (ApiServer创建HTTP服务)

### 4.3.1 Create an API Object (创建API对象)

​	`API` object manages the `ApiServer`. Before running `ApiServer`, We need to initialize the `API` object. The code looks as follows:

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

​	`api.NewAPI` creates a new `API` object. `ApiServer` manages a lot of objects so it needs many function arguments. Some of these are:

* listenAddr：Local port, If the environment variable `LISTEN` is not set, the default port of 9888 from `config.ApiAddress` will be used as `listenAddr`.
* n.api.StartServer: Start a HTTP Service on local port.

> 首先，api.NewAPI初始化API对象。ApiServer管理的事情很多，所以参数也相对较多，其中：
>
> * listenAddr：本地端口，如果系统没有设置LISTEN环境变量则使用config.ApiAddress配置地址，默认为9888。
> * n.api.StartServer：监听本地的地址端口，启动HTTP服务。

 	`NewAPI()` method has these three instructions:

* Instantiate a new `API` object.
* Add routes by calling `api.buildHandle()` method.
* Initialize `http.Server`, set `auth`, etc. by calling `api.initServer()` method.

> NewAPI函数我们看到有三个操作：
>
> * 实例化API对象。
> * api.buildHandler添加router路由项。
> * api.initServer实例化http.Server，配置auth验证等。

 

### 4.3.2 Create routes for router (创建router路由项)

​	HTTP requests are routed to corresponding handler functions through a router that matches the url paths and methods to handlers. Bytom implements many such handlers and all of them have same interface. Here we only introduce account related handler implementation. The code is as follows:

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

​	Here we use an http multiplexer `http.NewServeMux()` from Go standard library to create a router. As we can see, each route is made of a url and a handler callback function. Once the url requested matches `/net-info`, `ApiServer` will run `a.getNetInfo` function and pass arguments.

> 我们使用Golang标准库http.NewServeMux()创建一个router路由器，提供请求的路由分发功能。我们可以看到一条router项由url和对应的handle回调函数组成。当我们请求的url匹配到/net-info时，ApiServer会执行a.getNetInfo回调函数，并将用户的传参也带过去。

​	Other handler work:

* latencyHandler：When there is no matching route for the request URL path, codeblock for error path `/error` is called.
* maxBytesHandler：It limits the size of requests to no more than 10MB for each request.
* webAssetsHandler：It adds routes needed of dashboard and equity pages. These paths are hard-coded in bytomd in `dashboard/dashboard.go` and `equity/equity.go`.
* gzip.Handler：gzip handler provides a way of compressing the response body. To enable `gzip` data compression, the level of `gzip.BestSpeed` need to be set (default level is 1 and the highest level is 9). Higher level uses more cpu.

> 额外的handler处理：
>
> * latencyHandler：当请求找不到网址路径时，将请求重定向到"/error"错误路径。
> * maxBytesHandler：限制请求的字节大小，一个请求不允许超过10MB。
> * webAssetsHandler：添加dashboard和equity的路由项，dashboard和equity页面代码被硬编码到bytomd中，路径分别在dashboard/dashboard.go、equity/equity.go。
> * gzip.Handler：启用gzip数据压缩，需指定gzip.BestSpeed压缩级别（默认level 1）。最高可指定level 9，随着压缩级别增加，cpu耗时也会增加，可根据HTTP服务接收的数据量调整该压缩级别。



### 4.3.3 Instantiate http.Server (实例化http.Server)

​	`http.Server` struct makes it easy to build and run an HTTP Service. Here is the code:

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

1) AuthHandler: It handles `authentication` for incoming http requests, which means users need to enter valid `access token` on `dashboard` to login.  `authentication` is enabled by default .

2) RedirectHandler：When users enter  "/", it will redirect the request to "/dashboard/" and set the status to "302" by default.

3) http.Server: Set timeouts for reading and writing and instantiate an `http.Server` object.

> 实例化http.Server分为以下几步：
>
> 1) AuthHandler：启用auth验证功能，用户打开dashboard时需要输入access token进行验证登录。默认开启auth功能。
>
> 2) RedirectHandler：重定向，当用户输入"/"时，默认跳转到"/dashboard/"，并设置状态为302。
>
> 3) http.Server：设置读写超时时间并实例化http.Server。

​	So far, all handers have been registered in HTTP Server and `http.Server` object has been instantiated. Next, we will listen on local ports and start HTTP Service.

> 到目前为止，所有的handle已经被注册到HTTP 服务器中并实例化http.Server服务对象，下面操作监听端口，启动HTTP服务。



### 4.3.4 Start Api Server (启动ApiServer服务)

​	We use `net.listen` from Golang standard library to listen on local port. goroutines are functions that run concurrently with other functions. This is a concurrency feature available in golang. goroutines are used to start long running services like the Http server. If there are no error messages so far when running `a.server.Serve`, the http server is started on `9888` port. At this point, the `ApiServer` is running and is in a waiting state for user requests. For every new request, `ApiServer` starts a goroutine to process that request. The code looks as follows:

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

​	`server.Serve` works as follows:

1) `a.server.Serve` starts a forever `for` loop that keeps receiving incoming requests.

2) Instantiate a new Connection `Conn` for each request and also start a `goroutine` to handle for this request.

3) `c.readRequest(ctx)` reads contents of each request.

4) Chose a `handler` for this request and call ServeHTTP of this handler.

5) Call `w.finishRequest` to delete conn buffer data, close the tcp connection and end the request.

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

 

### 4.3.5 Receive and Respond to Requests (接收并响应请求)

​	We use `curl` to initate an HTTP request from commandline and get the network status of current node. Here is a sample `ApiServer` response: 

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

​	Let's look at how `ApiServer` processes the requests:

> 解析ApiServer的处理过程：

 api/api.go

```
m.Handle("/net-info", jsonHandler(a.getNetInfo))
```

​	`ApiServer` parses request http headers, matches the path with `/net-info` and jumps to `a.getNetInfo` callback function.

> ApiServer解析HTTP头，匹配到path路径/net-info，跳转至a.getNetInfo回调函数。

 api/nodeinfo.go

```
func (a *API) getNetInfo() Response {
	return NewSuccessResponse(a.GetNodeInfo())
}
```

​	`getNetInfo()` method gets more information from objects like `P2P`, `cpuMinner`, etc and serializes the `NetInfo` structure to JSON format and  returns it to users by instantiating a new `Response` object.

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

func NewSuccessResponse(data interface{}) Response {
	return Response{Status: SUCCESS, Data: data}
}
```

​	A new `Response` object is instantiated by `NewSuccessResponse` method with HTTP status code 200. Here is the description:

* Status: The status string
* Code: The status code
* Msg: The status description
* ErrorDetail: Error message. It is null when request is successfully
* Data: Data response to user's request.

> NewSuccessResponse实例化Response，并设置HTTP状态码为200。Response结构描述如下：
>
> * Status：状态码字符描述。
> * Code：状态码。
> * Msg：状态码对应的字符描述。
> * ErrorDetail：错误信息，请求成功时，错误信息为空。
> * Data：响应用户的数据。

## 4.4 Life Cycle of an HTTP Request (HTTP请求的完整生命周期)

​	`ApiServer` creates a short lived HTTP connection for every request. The connection will be closed after the HTTP request is handled and the response is sent back. A new connection needs to be rebuild for the next HTTP request/response. Here is a picture that shows the life cycle of an HTTP request : 

> ApiServer服务提供HTTP短连接（非持久连接）服务请求。客户端和服务端进行一次HTTP请求/响应之后，就关闭连接。下一次的HTTP请求/响应操作就需要重新建立连接。一次HTTP请求的完整生命周期如下图所示：

![4-1 The life cycle of an HTTP request](https://github.com/Bytom-Community/Bytom-Tech-Book/blob/master/Chapter04_pic/pic_01.png)

The complete life cycle of HTTP request:

1) User sends an HTTP request to ApiServer.

2) ApiServer receives the HTTP request.

3) A goroutine is started to process the request.

4) Verify the authentication information in the request.

5) Parse the request content.

6) Call the handler callback function based on the routing rules.

7) Get data returned by handler.

8) Set response status code.

9) Send response to user.

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

 

## 4.5 Bytom APIs(比原链API接口描述)

#### 1. Wallet related APIs (钱包相关接口)

| API                      | Description                                                  |
| ------------------------ | ------------------------------------------------------------ |
| /create-account          | Create an account                                            |
| /list-accounts           | Returns the list of all available accounts.                  |
| /delete-account          | Delete an existing account                                   |
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
| /delete-key              | Delete existing key                                           |
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
| /restore-wallet          | Restore wallet from image file                               |
| /rescan-wallet           | Trigger a rescan of block information into related wallet    |
| /wallet-info             | Return the wallet information                                |

#### 2.Token related  APIs (Token验证相关接口)

| API                  | Description                                     |
| -------------------- | ----------------------------------------------- |
| /create-access-token | Create access token                             |
| /list-access-tokens  | Returns the list of all available access tokens |
| /delete-access-token | Delete existed access token                     |
| /check-access-token  | Check access token is valid                     |

#### 3.Transaction related APIs (交易相关接口)

| API                            | Description                                                  |
| ------------------------------ | ------------------------------------------------------------ |
| /submit-transaction            | Submit a transaction                                         |
| /estimate-transaction-gas      | Submit transactions used for batch submit transactions       |
| /get-unconfirmed-transaction   | Estimate consumed neu(1BTM = 10^8NEU) for the transaction    |
| /list-unconfirmed-transactions | Query mempool transaction by transaction ID                  |
| /decode-raw-transaction        | Decode a serialized transaction hex string into a JSON object describing the transaction |

#### 4. Block related APIs (块信息相关接口)

| API               | Description                                                    |
| ----------------- | ---------------------------------------------------------------|
| /get-block        | Returns the block details by block height or block hash        |
| /get-block-hash   | Returns the current block hash for blockchain                  |
| /get-block-header | Returns the block header details by block height or block hash |
| /get-block-count  | Returns the current block height for blockchain                |
| /get-difficulty   | Returns the block difficulty by block height or block hash     |
| /get-hash-rate    | Returns the block hash rate by block height or block hash      |
| /is-mining        | Returns the mining status of current node                      |
| /set-mining       | Start up node mining                                           |

#### 5. Mining Pool related APIs (矿池相关接口)

| API               | Description                               |
| ----------------- | ----------------------------------------- |
| /get-work         | Get the proof of work in Varint format    |
| /get-work-json    | Get the proof of work in JSON             |
| /submit-work      | Submit the proof of work in Varint format |
| /submit-work-json | Submit the proof of work in JSON          |
| /is-mining        | Returns the mining status                 |
| /set-mining       | Start mining on this node                 |

#### 6. Contract related APIs (合约相关接口)

| API             | Description                                                |
| --------------- | ---------------------------------------------------------- |
| /verify-message | Verify a signed message with derived pubkey of the address |
| /decode-program | Decode program                                             |
| /compile        | Compile equity contract                                    |

#### 7. P2P related APIs (P2P网络相关接口)

| API              | Description                                     |
| ---------------- | ----------------------------------------------- |
| /net-info        | Returns the information of current network node |
| /list-peers      | Returns the list of connected peers             |
| /disconnect-peer | Disconnect a specified peer                     |
| /connect-peer    | Connect to a specified peer                     |

 

## 4.6  API Tools (API接口调用工具)

​	We need api tools to test and try out the api interfaces available on public blockchain. There are many tools available for this but we choose a command line tool `curl` and a graphical user interface tool `postman` to show how to make api calls. We use the `create-key` api as an example.

> 在公链开发中，我们需要对API接口进行测试。测试工具有很多种，本节推荐使用两种方式curl命令或postman图形工具，在这里以create-key接口示例演示。

### 4.6.1 Call APIs using `curl` commandline (使用curl命令行调用API接口)

`curl` is an open source command line tool and library for interacting with http/s urls. Using `curl` command line, you can make http requests to any urls, get response and show it in the `standard console output`.

> curl命令是利用URL语法在命令行方式下工作的开源文件传输工具。curl命令发出网络请求，然后得到和提取数据，并显示在"标准输出"中。

​	We use `curl` commandline to make a `create-key` api call to create private key and show response. Here is the code:

> 我们使用curl命令，请求访问create-key接口，创建私钥，并返回密钥的信息。代码示例如下：

```
$ curl -X POST http://localhost:9888/create-key -d '{ "alias" :"user1" , "password":"123456"}'

{"alias":"user1",
"xpub":"e441f31e16276902d305ab5eb6fb686ec3954a59c00712c5ee7565c93f16589b75ea5e11aa09122962ff7bd04c9dfff5027ca10ac9bea64722934850f6954f56",
"file":"C:\\Users\\Mac\\AppData\\Roaming\\Bytom\\keystore\\UTC--2018-09-21T11-51-52.401546400Z--e35c76e1-0f6c-4431-bb0f-eca37206d885"}}
```

​	The arguments used with `curl` commandline are:

* -X: Set the method of request, such as GET, POST, DELETE, etc.
* -d: Send data to the api url using `POST` method.
* Other parameters for curl can be found online or try `curl -h`

> curl命令参数如下：
>
> * -X：指定请求方式，请求方式包括GET、POST、DELETE等。
> * -d：使用POST方式向ApiServer发送数据。
> * 其他参数请读者自行学习。

​	Values in API response: 

* alias: The alias of the account is `user1`.
* xpub: Public key.
* file: The path of keystore.

> API接口返回字段结果为：
>
> * alias：别名user1。
> * xpub：公钥。
> * file：密钥文件的存储路径。

### 4.6.2 Call APIs using Postman (使用Postman调用API接口)

   Postman is a graphical REST api client which helps you develop and test http REST apis. With postman, you can make api calls using http methods like GET, POST etc.

​	Download Postman from here https://www.getpostman.com/apps.

> Postman是一个图形化接口请求工具，可以帮助我们更方便地模拟GET、POST及其他方式的请求来调用接口。
>
> 下载postman（官网下载：https://www.getpostman.com/apps）。

​	Use Postman to call create-key API with `POST` method. Enter request URL ( `http://127.0.0.1:9888/create-key` ) , enter `{"alias": "user0", "password": "123456"}` in `raw` under `Body` in `Text` format, and click `send` button. The return response is same as when we used `curl` above: 

> 使用postman调用API接口，选择POST请求方式，输入http://127.0.0.1:9888/create-key；在Body标签下raw，选择Text格式，输入:{"alias": "user0", "password": "123456"};点击send，返回如上述curl下create-key相同格式内容：

![4-2 The Postman](https://github.com/Bytom-Community/Bytom-Tech-Book/blob/master/Chapter04_pic/pic_02.png)

​	Similarly, you can try other apis using postman. `create-access-token` will create a new access token. `list-keys` will list all the private keys created.

> postman下进行create-access-token同理，此处不再加以赘述。此外可通过list-keys来查看所创建的所有私钥。

***Note: Everytime you call create an account api, change the alias value, or it will return fail message that the alias has already been created.***




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

​	This chapter describes how to implement a simple http service using golang and it's standard library. We offer many practical details for how requests are received, processed and responses sent back with a request response life cycle diagram. We showed how public blockchains implement HTTP apis to handle user requests and finally we covered the APIs implemented in all public blockchains along with bytom specific apis like mining pool related API, token related API etc.

> 本章介绍了如何使用GO语言标准库实现一个简易的HTTP Server服务。其中包括HTTP服务的工作原理，接收、处理、响应等过程。并以图文的方式介绍了一个HTTP请求的完整生命周期。最后详解了作为公链如何提供HTTP相关的接口。比原链API接口中已经包含了大部分公链所具备的接口，相比其他公链中比原链扩展了其他接口，比如矿池相关接口、token验证相关接口等。
