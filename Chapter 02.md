# Chapter 02 Interactive Tools

## 2.1 Introduction 

​	Bytomcli and dashboard are Bytom tools to interact with bytomd based on RPC protocol. Bytomcli is command line, while dashboard is a Website, which is more friendly than bytomcli to users.

​	Operations related to token, accounts, transactions, wallets, mining, etc. can be managed by these interactive tools. 

​	In this chapter, we will analyse the principle and code of bytomcli and bytomd. Give a brief introduction about dashboard.

> bytomcli和dashboard是比原链提供的与bytomd交互的工具（基于RPC协议）。bytomcli是命令行客户端，dashboard是Web图形界面。dashboard相比bytomcli使用体验更友好，用户可随意选择。
>
> 通过交互工具可以完成token、账户、交易、钱包、挖矿等比原链相关的管理操作。
>
> 本章中，我们对bytomcli与bytomd的交互过程原理和代码进行分析。对dashboard做一些简介。



## 2.2 Bytomcli (bytomcli交互工具)

### 2.2.1 Arguments of Bytomcli (bytomcli命令flag参数详解)

​	There are two ways to use bytomcli. One is using the `flag` (bytomcli [flags]). Bytomcli uses `-h` to open the help. The other is using a sub command (bytomcli [command]). All options of commands can be seen by `-h` or `-help`. The help of bytomcli is pretty detailed, useful and easy to understand.

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

​	After introducing the arguments of bytomcli, here gives an example to show how to use the bytomcli. 

> 以上内容已经详细介绍了bytomcli命令行工具的详细参数。下面，将会以一个实际的用例介绍如何使用bytomcli命令行工具。

### 2.2.2 Use Bytomcli to View Nodes (使用bytomcli查看节点状态信息)

​	Use `net-info` to get the information of Bytom nodes. Firstly, we can get the help by `bytomcli net-info -h`. Here are the results:

> 使用net-info参数可以查看bytomd节点运行的网络状态。首先，使用 bytomcli net-info -h 命令查看net-info命令的帮助信息。命令执行如下：

```
$ ./bytomcli net-info -h
Print the summary of network
Usage:
  bytomcli net-info [flags]
Flags:
  -h, --help   help for net-info
```

​	According to the resuresults, we can see the `net info` just have one choice for the `flag`, which is exactly the `-h`. Therefore, the use of `net-info` is quite simple to view nodes.

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

​	Description of the results :

* current_block：The height of the current block is 36714 on the node.
* highest_block：The highest block of the network is 36714, which means the node has had all blocks now.
* listening：Whether the node is listening to the network.
* mining：Whether the node is mining.
* network_id：Identify the type of network (mainnet, wisdom, solonet) of this node.
* peer_count：Show the number of other nodes it connect.
* syncing：Whether the node is syncsynchronizing or not.
* version：The version of the node.

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

### 2.2.3 The Example of Bytomcli (bytomcli运行案例)

​	In this section, we use `net-info` to analyse the bytomcli code. Other arguments almost follow the same pattern.

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

***1. Introduction of Cobra (Cobra 库介绍)***

​	Bytomcli is based on Cobra, which is both a library for creating powerful modern CLI (command-line interface) applications as well as a program to generate applications and command files. Many of the most widely used Go projects are built using Cobra, such as: Kubernetes, docker, etcd, etc.

> bytomcli 命令行生成工具是基于Cobra实现。Cobra既是用来创建强大的现代CLI应用程序的库，也可以用来生成应用和程序的文件。很多知名的开源软件都使用Cobra实现其CLI部分，例如 kubernetes，docker，etcd等。

​	Cobra is built on a structure of commands, arguments & flags. Commands is the central point of the application. Each interaction that the application supports will be contained in a Command. A command can have children commands and optionally run an action.Args are things and Flags are modifiers for those actions. A flag is a way to modify the behavior of a command. Cobra supports fully POSIX-compliant flags as well as the Go flag package. A Cobra command can define flags that persist through to children commands and flags that are only available to that command.

​	How to create a CLI by Cobra?  Here is an example as follows:

> Cobra基于三个基本的概念commands，arguments和flags实现对命令参数解析和行为控制。commands是应用程序的中心。应用程序支持的每个交互都包含在命令中。命令可以具有子命令并可选的运行子命令。flags是修改命令行为的方法。Cobra支持POSIX标准和GO的flags包，并可以同时用于父命令和子命令的参数，也支持只在父命令或者子命令有效的参数。
>
> 如何使用Cobra库来构建我们自己的CLI程序呢？下面我们简单的介绍Cobra库的使用。

***（1）Installing Cobra (安装Cobra库)***

​	Using Cobra is easy. First, use `go get` to install the latest version of the library. This command will install the `cobra `generator executable along with the library and its dependencies. After instalinstalling, there will be cobra in `GOPATH/bin`.

> Cobra是非常容易使用的，使用go get来安装最新版本的库。当然这个库还是相对比较大的，安装它可能需要相当长的时间，这取决于你的网速。安装完成后，在GOPATH/bin目录下应该有已经编译好的cobra程序。

```
$ go get -v github.com/spf13/cobra/cobra
```

***（2）使用cobra生成应用程序***

​	Here we are going to create a program based on CLIs named `demo`. Firstly, open the CMD, go under `GOPATH/src`, do the following command:

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

​	If the program doesn't need children commands, it has been done here. Certinly, we can add children commands later.

​	Then go on designing the demo functions. Open `main.go`:

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

​	The `main()` runs `Execute()`, which can be seen in `demo/cmd/root.go` In `root.go`, it has some initial operations and offers `Execute` interface. However, many initial operations are not necessary but created by Cobra by default. Like `viper` , it is a library for reading configuration files of Cobra. We can comment it out since we don't use here. If we don't comment it out, there will be a 10M application.

​	Create  `imp`  in `demo`. Here it is:

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

​	In `imp.go`, `show()` has two arguments: `name` and `age`, which are printed by `fmt`. Here is the directory of `demo` :

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

​	All commands of Cobra will be run by `cobra.Command`. In order to reach the function requirements of `demo`, we need to make some changes on RootCmd in `demo/cmd/root.go`.

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

​	Here we have defined RootCmd as a `Command` struct with callback function, and this function can be executed by `Execute` interface. But still, we need to parse command line arguments before running `Execute` and that is processed by `init()` function in package cmd.  `init()` has higher priority than `main()` and will be executed by default after each package initialization. `init()` is usually used to initialize variables. Here we use `init()` to parse command line arguments before `main()`. Make changes in `demo/cmd/root.go`:

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

​	`demo` has been done until now. We can run it :

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

​	If we want to create a CLI application with children commands, just need to run `cobra add` to add children commands to it. Here we add a children command `server` as follows:

> 如果我们想实现一个带有子命令的CLI程序，只需要再执行cobra add为程序新增子命令。这里我们演示一下为demo程序新增一个server的子命令。

```
$ cd demo
$ cobra add server
```

​	There will be a `server.go` under `demo/cmd/` :

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

​	The next work is to configure the `server` just following the way we did on `root.go`. Here it is :  

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



***2.bytomcli运行流程与原理***

![图 2-1 bytomcli运行原理示意图](https://github.com/Bytom-Community/Bytom-Tech-Book/blob/master/Chapter02_pic/pic_01.png)

​	In the last section, we learned how to use Cobra to create a CLI application. After having a view of Cobra, it is easy to learn the bytomcli code. Just like the `demo` before , bytomcli also runs `cmd.Execute()` in the main function to start the application. In fact, `cmd.Execute()` means calling the `Execute()` which is in a package named `commands`. `cmd` is just another name, made when import `commands` into `main.go`.

> 在上一节中我们学习了，如何使用Cobra库构建CLI应用程序。当我们对Cobra库有了了解之后，再来学习bytomcli的代码就会感觉非常的容易。同上一节中的demo程序一样，bytomcli同样是在main函数中使用cmd.Execute()来启动应用程序。其实cmd.Execute()是调用的commands包的Execute()方法。在mian.go中引入commands包的时候，给它起了一个别名cmd。

```
cmd/bytomcli/main.go
func main() {
    runtime.GOMAXPROCS(runtime.NumCPU())
    cmd.Execute()
}
```

​	When the program running `cmd.Execute()`, it runs all `init()` in `commands` firstly, which are mainly used to parse arguments of command-lines. After that, `Execute()` can be run. In `Execute()`, `AddCommands()` is run in the first place and used to add children commands to Bytomcli Cmd, while `AddTemplateFunc()` is for adding templates to children commands. After these (`init()`,`AddCommands()` ,`AddTemplateFunc()`) are done, it will start to execute commands really. When running Bytomcli `cmd.Execute()`, Cobra will jump into the children commands according to user's commands and run its `Run`.

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

​	Here we take `create-access-token` as the example to analyse what its `Run` does. Firstly, it takes `arg[0]` as tokenID, then calls `ClientCall` in `util` to get the path of `/create-access-token` and sends tokenID to create token.

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

​	`ClientCall` encapsulates a RPC client, which can send requests according to the path and arguments given by users as well as parse the content got from RPC server.

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

​	The `create-access-token` is done here.

> 至此，create-access-token命令执行完成，其生命周期终止。



## **2.3 dashboard交互工具**

​	On Bytom, dashboard is an individual program. (https://github.com/Bytom/bytom-dashboard). Confusion may occur here: why dashboard will be used when bytomd is working since it is an individual program？That is because dashboard is hard-coded in bytomd, and you can find its code in `dashboard/dashboard.go`. The `hander` information is in `api/api.go`.

> 在比原链中，dashboard是单独一个项目，项目源码地址：https://github.com/Bytom/bytom-dashboard. 可能会有读者疑惑，既然dashboard是一个单独的项目，为什么在启动bytomd进程的时候会启用dashboard服务呢？这是因为dashboard编译后被硬编码到bytomd中，源码在dashboard/dashboard.go中。而在ApiServer中api/api.go我们可以看到引用handle信息：

```
mux := http.NewServeMux()
mux.Handle("/dashboard/", http.StripPrefix("/dashboard/", static.Handler{
	Assets:  dashboard.Files,
	Default: "index.html",
}))
```

### 2.3.1 使用dashboard发送一笔交易

​	Open dashboard in http://127.0.0.1:9888/, the status of block synchronization is shown on the lower left of the page. We can click the green button on the upper right to create a transaction.
![图 2-2 dashboard示意图](https://github.com/Bytom-Community/Bytom-Tech-Book/blob/master/Chapter02_pic/pic_02.png)

> 在浏览器中输入http://127.0.0.1:9888/, 打开dashboard页面。页面左下方显示当前节点同步区块的进度。点击右上角->新建交易。

![图 2-3 dashboard示意图](https://github.com/Bytom-Community/Bytom-Tech-Book/blob/master/Chapter02_pic/pic_03.png)
​	On Bytom, transactions are divided into two types: simple transactions and advanced transactions. Here we create a simple transaction. More details about advanced transactions will be introduced in the following chapters.

> 在比原链中，新建交易分为两种，一种是简单交易，一种是高级交易。本节中使用简单交易发送一笔交易。高级交易会在后面章节进行详细讲解。

​	From this page, we can get following information:

​	There are `8254800 NEU` sent to `bm1q5p9d4gelfm4cc3zq3slj7vh2njx23ma2cf866j` from local wallet named Derek, and `500000 NEU` among them are gas. Then input the password to submit the transaction. Here is the result :
![图 2-4 dashboard示意图](https://github.com/Bytom-Community/Bytom-Tech-Book/blob/master/Chapter02_pic/pic_04.png)

> 从图中看出，我们从本地钱包的derek账户下的BTM资产往bm1q5p9d4gelfm4cc3zq3slj7vh2njx23ma2cf866j地址中打入了8254800 NEU，其中gas的手续费估算是500000个NEU。然后输入钱包密码并提交交易。得到结果如下：

​	Transaction ID is 3ebe89ddb64f3a7ef1742e..., and it is being broadcasted in the main network. This transaction will be done once it is confirmed. We usually think a transaction is irreversible after being validated by 6 blocks.

> 交易ID为3ebe89ddb64f3a7ef1742e...目前正在往主网中广播，待到交易确认后则说明这笔交易最终交易成功。一般我们认为这笔交易经过6个区块的验证以后则说明该笔交易最终无法逆转。

### 2.3.2 使用dashboard开启挖矿模式

​	In 1.3.10, we use Bytomcli to start the node mining. Here, we can also use dashboard  to open or close mining.
![图 2-5 dashboard示意图](https://github.com/Bytom-Community/Bytom-Tech-Book/blob/master/Chapter02_pic/pic_05.png)

> 在1.3.10小节中我们使用bytomcli命令行交互的方式开启节点的挖矿模式，同样我们也可以使用dashboard来启用或关停挖矿模式。

![图 2-6 dashboard示意图](https://github.com/Bytom-Community/Bytom-Tech-Book/blob/master/Chapter02_pic/pic_06.png)

​	Click the gear button on the upper left and it will show the status and configuration of the node. We can start mining here, which is closed by default.

> 点击dashboard左上角齿轮按钮，点击核心状态，显示节点的配置信息。默认情况下挖矿选项是关闭状态，在这里我们可以启用节点的挖矿模式。



## **2.4** 本章总结

​	In this chapter, we give more details about the user interaction layer, such as the analysis of bytomcli and dashboard. About bytomcli, here introduces Cobra with an application example and that makes readers more clear about the process of creating CLI flag. Apart from that , the RPC interaction between bytomcli and bytomd is also analyzed by code. About dashboard, here shows its usage through the example of sending a transaction.

> 本章对用户交互层做了详细介绍。对bytomcli命令行和dashboard做了应用和分析。在bytomcli章节中对cobra库的使用做了详细的用例。让读者对cobra如何生成cli下flag的流程更加透彻。从源码的角度深入分析了bytomcli与bytomd的RPC交互过程。以dashboard发送一笔交易为例，展示了dashboard使用方式。
