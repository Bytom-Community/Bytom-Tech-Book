# Chapter 02 Interactive Tools

## 2.1 Introduction 

​	Bytomcli and dashboard are two Bytom tools to interact with bytomd. Bytomcli is a command line tool and dashboard is a more user friendly Web Application. Both of them use RPC protocol to interact with bytomd process.

​	Any operation related to tokens, accounts, transactions, wallets, mining, etc. can be managed by these interactive tools. 

​	In this chapter, we will analyze the ideas and code behind bytomcli and bytomd and briefly introduce dashboard.

> bytomcli和dashboard是比原链提供的与bytomd交互的工具（基于RPC协议）。bytomcli是命令行客户端，dashboard是Web图形界面。dashboard相比bytomcli使用体验更友好，用户可随意选择。
>
> 通过交互工具可以完成token、账户、交易、钱包、挖矿等比原链相关的管理操作。
>
> 本章中，我们对bytomcli与bytomd的交互过程原理和代码进行分析。对dashboard做一些简介。



## 2.2 Bytomcli (bytomcli交互工具)

### 2.2.1 Commandline arguments for Bytomcli (bytomcli命令flag参数详解)

​	There are two ways to use bytomcli. 
​	
​	1. Using the `flag` (bytomcli [flags]). Use `-h` to open help. 
​	
​	2. Using a sub-command (bytomcli [command]). All sub-commands can be seen by `-h` or `-help`. The help section is pretty detailed, useful and easy to understand.

> bytomcli的使用方式分为两种，一种是后面接flags 参数，bytomcli [flags]。bytomcli使用 -h 参数查看命令的帮助文档。另一种是后面接子命令的形式，bytomcli [command]。所有的命令选项都可以通过执行 -h 或 --help 获得指定命令的帮助信息。bytomcli的帮助信息、示例相当详细，而且简单易懂。建议大家习惯使用帮助信息。

```
$ ./bytomcli -h
Bytomcli is a commond line client for bytom core (a.k.a. bytomd)

Usage:
  bytomcli [flags]
  bytomcli [command]

Available Commands:
  build-transaction             
  check-access-token            
  create-access-token           
  create-account                
  create-account-receiver       
  create-asset                  
  create-key                  
  create-transaction-feed     
  decode-program                
  decode-raw-transaction        
  delete-access-token       
  delete-account               
  delete-key                 
  delete-transaction-feed      
  estimate-transaction-gas      
  gas-rate                      
  get-asset                  
  get-block                     
  get-block-count               
  get-block-hash              
  get-block-header              
  get-difficulty                
  get-hash-rate                
  get-transaction             
  get-transaction-feed        
  get-unconfirmed-transaction   
  help                          
  is-mining                     
  list-access-tokens            
  list-accounts                 
  list-addresses               
  list-assets                
  list-balances                
  list-keys                   
  list-pubkeys                  
  list-transaction-feeds    
  list-transactions             
  list-unconfirmed-transactions 
  list-unspent-outputs         
  net-info                      
  rescan-wallet                 
  reset-key-password            
  set-mining                    
  sign-message                
  sign-transaction             
  submit-transaction           
  update-asset-alias            
  update-transaction-feed      
  validate-address             
  verify-message             
  version                     
  wallet-info                 
```

​	The following sections will cover how to use sub-commands 

> 以上内容已经详细介绍了bytomcli命令行工具的详细参数。下面，将会以一个实际的用例介绍如何使用bytomcli命令行工具。

### 2.2.2 Use Bytomcli to View Nodes (使用bytomcli查看节点状态信息)

​	Use `net-info` to get the information about running Bytom node. 

Run `bytomcli net-info -h` to get help information.

> 使用net-info参数可以查看bytomd节点运行的网络状态。首先，使用 bytomcli net-info -h 命令查看net-info命令的帮助信息。命令执行如下：

```
$ ./bytomcli net-info -h
Prints the summary of node
Usage:
  bytomcli net-info [flags]
Flags:
  -h, --help   help for net-info
```

​	You can see that `net info` is very simple and has only one `flag` `-h`.

> 由net-info参数输出的帮助信息可知，net-info命令仅支持一个可选的flags参数，flags 参数列表又仅包含 -h 参数用来查看net-info命令的帮助信息。所以net-info命令的用法是十分简单的，使用bytomcli net-info即可查看节点的网络状态：

```
$ ./bytomcli net-info
{
  "current_block": 36714,
  "highest_block": 36714,
  "listening": true,
  "mining": false,
  "network_id": "mainnet",
  "peer_count": 10,
  "syncing": false,
  "version": "1.0.5+2bc2396a"
}
```

​	Description of the response :

* current_block：The height of the current block is 36714 on this node.
* highest_block：The highest block of the network is 36714, which means the node has all generated blocks.
* listening：Whether the node is listening to the network.
* mining：Whether the node is mining.
* network_id：Identify the type of network (mainnet, wisdom, solonet) this node is listening to.
* peer_count：Show the number of other nodes it connects to.
* syncing：Whether the node is synchronizing or not.
* version：The bytom version this node is running.

> net-info参数返回数据详解：
>
> * current_block：当前节点的当前区块高度为36714。
> * highest_block：代表网络中最高区块高度为36714，表明节点已经拥有完整的区块数据。
> * listening：当前节点处于监听状态。
> * mining：当前节点是否启用挖矿功能。
> * network_id：标识节点连接的网络类型（mainnet为正式主网，wisdom为测试网络，solonet为单节点网络）。
> * peer_count：显示节点当前连接的其他节点个数。
> * syncing：当前节点是否正在同步区块数据。
> * version：节点的版本号。

### 2.2.3 Bytomcli Sub-command example(bytomcli运行案例)

​	In this section, we will analyze `net-info` part of bytomcli code. Other sub-commands also follow the same pattern.

​	Here is the structure of bytomcli code.

> 本小节我们以net-info命令为例对bytomcli的源码进行剖析，其他命令参数与net-info参数执行的过程大同小异。
>
> bytomcli相关代码文件结构：

```
$ tree cmd/bytomcli/
cmd/bytomcli/
├── bytomcli
├── commands			
│   ├── accesstoken.go		
│   ├── account.go		
│   ├── asset.go			
│   ├── block.go			
│   ├── bytomcli.go	
│   ├── key.go			
│   ├── mining.go		
│   ├── net.go			
│   ├── program.go	
│   ├── template.go		
│   ├── transaction.go		
│   ├── txfeed.go	
│   ├── util.go			
│   ├── version.go	
│   └── wallet.go		
└── main.go			
```

***1. Introduction to Cobra (Cobra 库介绍)***

​	Bytomcli is based on Cobra. Cobra is both a library for creating powerful modern CLI (command-line interface) applications as well as a program to generate scaffolding for commandline applications and related command files. Many of the most widely used Go projects such as Kubernetes, docker, etcd, etc also use Cobra.

> bytomcli 命令行生成工具是基于Cobra实现。Cobra既是用来创建强大的现代CLI应用程序的库，也可以用来生成应用和程序的文件。很多知名的开源软件都使用Cobra实现其CLI部分，例如 kubernetes，docker，etcd等。

​	Cobra is built on a structure of commands, arguments & flags. 

Command is the central element of the application. Each interaction that the application supports will be contained in a Command. A command can have sub commands and optionally run an action.

Args are things and Flags are modifiers for those actions. A flag is a way to modify the behavior of a command. Cobra supports fully POSIX-compliant flags as well as the Go flag package. 

A Cobra command can define flags that persist through to sub commands and flags that are only available to that command.

​	Here is an example to show how to create a CLI using Cobra:

> Cobra基于三个基本的概念commands，arguments和flags实现对命令参数解析和行为控制。commands是应用程序的中心。应用程序支持的每个交互都包含在命令中。命令可以具有子命令并可选的运行子命令。flags是修改命令行为的方法。Cobra支持POSIX标准和GO的flags包，并可以同时用于父命令和子命令的参数，也支持只在父命令或者子命令有效的参数。
>
> 如何使用Cobra库来构建我们自己的CLI程序呢？下面我们简单的介绍Cobra库的使用。

***（1）Installing Cobra (安装Cobra库)***

​	Using Cobra is easy. First, use `go get` to install the latest version of the library. This command will install the `cobra `generator executable along with the library and it's dependencies. After installing, you can find cobra in `GOPATH/bin`.

> Cobra是非常容易使用的，使用go get来安装最新版本的库。当然这个库还是相对比较大的，安装它可能需要相当长的时间，这取决于你的网速。安装完成后，在GOPATH/bin目录下应该有已经编译好的cobra程序。

```
$ go get -v github.com/spf13/cobra/cobra
```

***（2）Using Cobra to build an application (使用cobra生成应用程序)***

​	We are now going to create a command line program named `demo`. Open the terminal, navigate to `GOPATH/src`, and run the following command:

> 假设现在我们要开发一个基于CLIs的命令程序，名字为demo。首先打开CMD，切换到GOPATH的src目录下，执行如下命令：

```
$ cobra init demo
```

​	There will be a `demo` under `GOPATH/src` as follows:

> 在src目录下会生成一个demo的文件夹，如下：

```
$ tree demo/
demo/
├── LICENSE
├── cmd
│   └── root.go
└── main.go
```

​	this program does not have any sub commands. We can add sub commands later if our applicaiton needs it.

​To add a bit more functionality, Open `main.go`:

> 如果此时我们的应用不需要子命令，那么cobra生成应用程序的操作就结束了。这里我们先实现一个没有子命令的CLI程序，之后再为程序添加子命令。
>
> 接下来继续demo的功能设计。我们打开cobra自动生成的main.go文件查看：

```
package main

import "demo/cmd"

func main() {
    cmd.Execute()
}
```

​The `main()` function runs `Execute()`, which is defined in `demo/cmd/root.go`. 
`root.go` file has some initial imports and the function definition for `Execute`. Cobra creates many imports by default but all of them are not necessary. For eg., `viper`, it is a library for reading configuration files. We can comment it out since we don't intend to use it here. If we don't comment it out, this will be a 10MB application.

​	Create a new file `imp.go` in `demo` and copy these contents:

> main函数执行cmd包的Execute()方法，打开cmd包下的root.go文件查看，发现里面进行了一些初始化操作并提供了Execute接口，其实Cobra自动生成的root.go文件中有很多初始化操作是不需要的，其中viper是cobra集成的配置文件读取的库，这里不需要使用，可以注释掉，不注释生成的应用程序会大10M左右。
>
> 在demo下面新建一个imp包，imp.go内容如下：

```
package imp

import(
    "fmt"
)

func Show(name string, age int) {
    fmt.Printf("My Name is %s, My age is %d\n", name, age)
}
```

​	In `imp.go`, `show()` has two arguments: `name` and `age`, which are printed by `fmt.Printf`. The `demo` directory now looks as follows:

> 在imp.go文件中，Show函数接收两个参数name和age，使用fmt打印出来。此时，整个demo项目目录结构如下：

```
$ tree demo/
demo/
├── LICENSE
├── cmd
│   └── root.go
├── imp
│   ├── imp.go
└── main.go
```

​	All Cobra commands need to be defined by GO struct `cobra.Command`. To implement `demo`, we shall make some changes to RootCmd in `demo/cmd/root.go`.

> cobra的所有命令都是通过cobra.Command这个结构体实现的。为了实现demo功能，显然我们需要修改RootCmd。修改demo/cmd/root.go文件。

```
var RootCmd = &cobra.Command{
    Use:   "demo",
    Short: "A test demo",
    Long:  `Demo is a test appcation for print things`,
    // Uncomment the following line if your bare application
    // has an action associated with it:
    Run: func(cmd *cobra.Command, args []string) {
        if len(name) == 0 {
            cmd.Help()
            return
        }
        imp.Show(name, age)
    },
}
```

​	Here, we defined RootCmd as a `Command` struct with an impementation for `Run` function, and this function will be executed by `Execute` method. 

We still need to parse command line arguments before actually running the command's `Run` method and that is done in `init()` function in cmd package. `init()` has higher priority than `main()` and will be executed by default after each package initializes. `init()` is usually used to initialize variables. Here we use `init()` to parse command line arguments before `main()`.

Make the following changes in `demo/cmd/root.go`:

> 虽然我们已经定义好了command结构，也能通过Execute接口调用到RootCmd定义的回调方法。但是想实现demo的功能还需要从命令行解析传入的参数，这部分应该如何实现？这部分可以在cmd包的init方法中实现。init()函数会在每个包完成初始化后自动执行，并且执行优先级比main函数高。init 函数通常被用来对变量进行初始化等操作。这里我们使用init函数在main函数执行之前解析命令行参数。修改demo/cmd/root.go文件。

```
var (
    name string
    age  int
)

func init() {
    RootCmd.Flags().StringVarP(&name, "name", "n", "", "person's name")
    RootCmd.Flags().IntVarP(&age, "age", "a", 0, "person's age")
}
```

​	We can run `demo` from command line now:

> 至此，demo的功能已经实现了，我们编译运行一下看看实际效果：

```
$ go run main.go
Usage:
  demo [flags]

Flags:
  -a, --age int         person's age
      --config string   config file (default is $HOME/.demo.yaml)
  -h, --help            help for demo
  -n, --name string     person's name
  -t, --toggle          Help message for toggle
```

​	If we want to create a CLI application with sub commands, just need to run `cobra add` to add sub commands to it. Here we add a sub command `server` as follows:

> 如果我们想实现一个带有子命令的CLI程序，只需要再执行cobra add为程序新增子命令。这里我们演示一下为demo程序新增一个server的子命令。

```
$ cd demo
$ cobra add server
```

​	This will add a file `server.go` in `demo/cmd/` :

> 在demo目录下生成了一个cmd/server.go文件，如下：

```
$ tree demo/
demo/
├── LICENSE
├── cmd
│   ├── root.go
│   └── server.go
├── imp
│   └── imp.go
└── main.go
```

We now configure the `server` just like we did in `root.go`.
​Here it is :  

> 接下来的操作就和上面修改root.go文件一样去配置server子命令。效果如下：

```
$ go run main.go
Usage:
  demo [flags]
  demo [command]

Available Commands:
  help        Help about any command
  server      A brief description of your command

Flags:
  -a, --age int         person's age
      --config string   config file (default is $HOME/.demo.yaml)
  -h, --help            help for demo
  -n, --name string     person's name
  -t, --toggle          Help message for toggle

Use "demo [command] --help" for more information about a command.
```



***2. bytomcli design (bytomcli运行流程与原理)***

![图 2-1 bytomcli运行原理示意图](https://github.com/Bytom-Community/Bytom-Tech-Book/blob/master/Chapter02_pic/pic_01.png)

​	In the last section, we looked at how to use Cobra to create a CLI application. With a good understanding of how Cobra works, it is easy to read the rest of bytomcli code. Just like in the `demo` command application, bytomcli also runs `cmd.Execute()` in the main function at the start of the bytomcli application. `cmd.Execute()` calls the `Execute()` method from the package named `commands`. `cmd` is just another name given to package `commands` when it is imported into `main.go`.

> 在上一节中我们学习了，如何使用Cobra库构建CLI应用程序。当我们对Cobra库有了了解之后，再来学习bytomcli的代码就会感觉非常的容易。同上一节中的demo程序一样，bytomcli同样是在main函数中使用cmd.Execute()来启动应用程序。其实cmd.Execute()是调用的commands包的Execute()方法。在mian.go中引入commands包的时候，给它起了一个别名cmd。

```
cmd/bytomcli/main.go
func main() {
    runtime.GOMAXPROCS(runtime.NumCPU())
    cmd.Execute()
}
```

​	When the program runs `cmd.Execute()`, it first runs `init()` function in `commands` to parse command line arguments. After that, `Execute()` will be run. In `Execute()`, `AddCommands()` is run first and this adds sub commands to Bytomcli Cmd, while `AddTemplateFunc()` is for adding templates to sub commands. After these (`init()`,`AddCommands()` ,`AddTemplateFunc()`) are run, it will start to execute the commands. When running Bytomcli `cmd.Execute()`, Cobra will jump into the sub command given by the user at command input and run its `Run`.

> 当程序调用commands包的Execute()方法时，会先执行commands包中所有的init()方法，这些init()主要用来解析命令行参数的。当执行完commands包中所有的init方法之后，才会执行Execute()。在Execute()方法中首先执行AddCommands()方法，该方法主要用来给BytomcliCmd命令添加子命令。而AddTemplateFunc()方法则是为子命令添加使用模板的。在完成解析flags参数、添加子命令和添加使用模板这些初始化操作之后，才真正进入命令的执行过程。在BytomcliCmd.ExecuteC()方法中，Cobra库 会根据用户输入的命令跳转到相应的子命令，同时执行命令定义的Run方法。

```
cmd/bytomcli/commands/bytomcli.go
func Execute() {

    AddCommands()
    AddTemplateFunc()

    if _, err := BytomcliCmd.ExecuteC(); err != nil {
        os.Exit(util.ErrLocalExe)
    }
}
```

​	Here we take `create-access-token` as an example to analyze what it's `Run` does. It takes `arg[0]` as token.ID, then calls `ClientCall` in `util` to get the path of `/create-access-token` and sends tokenID to create a new token.

> 下面我们以create-access-token命令为例，看一下这个命令的Run方法都执行了哪些操作。首先会取args参数的第一位作为tokenID，然后调用util包下的ClientCall方法请求/create-access-token路径，并将tokenID传入,创建token。

```
bytom/cmd/bytomcli/commands/accesstoken.go
var createAccessTokenCmd = &cobra.Command{
    Use:   "create-access-token <tokenID>",
    Short: "Create a new access token",
    Args:  cobra.ExactArgs(1),
    Run: func(cmd *cobra.Command, args []string) {
        var token accessToken
        token.ID = args[0]

        data, exitCode := util.ClientCall("/create-access-token", &token)
        if exitCode != util.Success {
            os.Exit(exitCode)
        }
        printJSON(data)
    },
}
```

​	`ClientCall` encapsulates an RPC client, which can compose requests based on path and arguments given by the caller and parse the response from RPC server.

> ClientCall方法封装了一个RPC的client，根据用户传入的路径和参数发送请求，并解析RPC server返回的内容。

```
bytom/util/util.go
func ClientCall(path string, req ...interface{}) (interface{}, int) {

    var response = &api.Response{}
    var request interface{}

    if req != nil {
        request = req[0]
    }

    client := MustRPCClient()
    client.Call(context.Background(), path, request, response)

    switch response.Status {
    case api.FAIL:
        jww.ERROR.Println(response.Msg)
        return nil, ErrRemote
    case "":
        jww.ERROR.Println("Unable to connect to the bytomd")
        return nil, ErrConnect
    }

    return response.Data, Success
}
```

​	The `create-access-token` completes its execution path after the response is displayed.

> 至此，create-access-token命令执行完成，其生命周期终止。


## **2.3 Interactive Dashboard (Dashboard交互工具)**

​Bytom Dashboard is a seperate project available here. (https://github.com/Bytom/bytom-dashboard). Compiled and compressed version of Dashboard is hard-coded in bytomd and you can find it's code in `dashboard/dashboard.go`. The `hanlder` information is in `api/api.go`. 

> 在比原链中，dashboard是单独一个项目，项目源码地址：https://github.com/Bytom/bytom-dashboard. 可能会有读者疑惑，既然dashboard是一个单独的项目，为什么在启动bytomd进程的时候会启用dashboard服务呢？这是因为dashboard编译后被硬编码到bytomd中，源码在dashboard/dashboard.go中。而在ApiServer中api/api.go我们可以看到引用handle信息：

```
mux := http.NewServeMux()
mux.Handle("/dashboard/", http.StripPrefix("/dashboard/", static.Handler{
	Assets:  dashboard.Files,
	Default: "index.html",
}))
```

### 2.3.1 Using Dashboard to send a Transaction (使用dashboard发送一笔交易)

​	Open dashboard in http://127.0.0.1:9888/, the status of block synchronization is shown on the lower left of the page. Click the green button on the upper right to create a transaction.
![图 2-2 dashboard示意图](https://github.com/Bytom-Community/Bytom-Tech-Book/blob/master/Chapter02_pic/pic_02.png)

> 在浏览器中输入http://127.0.0.1:9888/, 打开dashboard页面。页面左下方显示当前节点同步区块的进度。点击右上角->新建交易。

![图 2-3 dashboard示意图](https://github.com/Bytom-Community/Bytom-Tech-Book/blob/master/Chapter02_pic/pic_03.png)
​	In Bytom, there are simple transactions and advanced transactions. Here we create a simple transaction. More details about advanced transactions will be introduced in the following chapters.

> 在比原链中，新建交易分为两种，一种是简单交易，一种是高级交易。本节中使用简单交易发送一笔交易。高级交易会在后面章节进行详细讲解。

​	From this page, we can get following information:

​	There are `8254800 NEU` sent to `bm1q5p9d4gelfm4cc3zq3slj7vh2njx23ma2cf866j` from local wallet named Derek, and `500000 NEU` among them is gas. Input the password to submit the transaction. Here is the result :
![图 2-4 dashboard示意图](https://github.com/Bytom-Community/Bytom-Tech-Book/blob/master/Chapter02_pic/pic_04.png)

> 从图中看出，我们从本地钱包的derek账户下的BTM资产往bm1q5p9d4gelfm4cc3zq3slj7vh2njx23ma2cf866j地址中打入了8254800 NEU，其中gas的手续费估算是500000个NEU。然后输入钱包密码并提交交易。得到结果如下：

​	Transaction ID is 3ebe89ddb64f3a7ef1742e..., and it is broadcasted to the main network. This transaction will be complete once it is confirmed. A transaction is considered irreversible after being validated by 6 blocks.

> 交易ID为3ebe89ddb64f3a7ef1742e...目前正在往主网中广播，待到交易确认后则说明这笔交易最终交易成功。一般我们认为这笔交易经过6个区块的验证以后则说明该笔交易最终无法逆转。

### 2.3.2 Mining Settings in Dashboard (使用dashboard开启挖矿模式)

​	In 1.3.10, we used bytomcli to enable mining in the node. Here, we will show how to use dashboard to enable or disable mining.
![图 2-5 dashboard示意图](https://github.com/Bytom-Community/Bytom-Tech-Book/blob/master/Chapter02_pic/pic_05.png)

> 在1.3.10小节中我们使用bytomcli命令行交互的方式开启节点的挖矿模式，同样我们也可以使用dashboard来启用或关停挖矿模式。

![图 2-6 dashboard示意图](https://github.com/Bytom-Community/Bytom-Tech-Book/blob/master/Chapter02_pic/pic_06.png)

​	Click the gear button on the upper left and it will show the status and configuration of the node. We can enable/disable mining here, which is disabled by default.

> 点击dashboard左上角齿轮按钮，点击核心状态，显示节点的配置信息。默认情况下挖矿选项是关闭状态，在这里我们可以启用节点的挖矿模式。



## **2.4** 本章总结

In this chapter, we looked into more implementation details behind the user interaction layer i.e, bytomcli and dashboard. We then introduced Cobra library and how to use it to make a sample application to understand how bytomcli also works similarly. We saw the code related to how bytomcli and bytomd communicate via RPC. We finally looked at how to use Dashboard to make a sample transaction.

> 本章对用户交互层做了详细介绍。对bytomcli命令行和dashboard做了应用和分析。在bytomcli章节中对cobra库的使用做了详细的用例。让读者对cobra如何生成cli下flag的流程更加透彻。从源码的角度深入分析了bytomcli与bytomd的RPC交互过程。以dashboard发送一笔交易为例，展示了dashboard使用方式。
