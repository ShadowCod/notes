### Golang Micro Services

 ##### 什么是微服务

- 一个服务仅用于某个特定的业务功能，相互独立
- 总而言之，微服务的目标是通过将应用程序分解成较小的可组合的部分，以便在需要时可以独立部署、升级、删除或缩放，从而简化构建、维护和管理应用程序

##### 单体式架构和微服务的区别

- 单体式：所有的业务处理都在一个服务上，一个业务处理崩溃就可以导致这个服务崩溃（soa按功能模块划分）
  - ==复杂性==随着开发==越来越高==，遇到问题解决困难
  - ==耦合度高==，维护困难
  - 开发周期较长
  - 技术选型成本高，风险大
  - ==扩展性==较差
- 微服务：每个服务只处理一种业务，多个服务共同实现一个大的服务
  - 优点：职责单一、轻量级通讯、迭代开发方便、独立性高
  - 缺点：运维成本高、部署复杂度高、接口成本高、重复性劳动、业务分离困难

##### RPC协议

- 什么是RPC
  - 远程过程调用，属于应用层的协议，使用的tcp实现

- 理解RPC
  - 像调用本地函数一样去调用远程函数（通过rpc协议，传递函数名、参数调用远端函数，得到返回值）

- 为什么要使用RPC协议
  - 每个为服务都是一个独立的进程，且可以使用不同的语言开发，只要都遵循RPC协议就能通讯

##### RPC使用入门

- 远程---->网络通信

  > 回顾golang的socket通讯
  >
  > server:
  >
  > ​	net.listen()--->listener
  >
  > ​	listener.appect()--->conn
  >
  > ​	conn.read()
  >
  > ​	conn.write()
  >
  > ​	defer conn.close()|listener.close()
  >
  > client:
  >
  > ​	net.Dial()--->conn
  >
  > ​	conn.write()
  >
  > ​	conn.read()
  >
  > ​	defer conn.close()

- 步骤

  - server端：

    >1.注册RPC服务对象，给对象绑定方法（1.定义类 2.绑定类方法）
    >
    >```go
    >rpc.ResgisterName("rpc名称"，实现接口的对象)
    >```
    >
    >2.创建监听器
    >
    >```go
    >listener,err:=net.Listen()
    >```
    >
    >3.建立连接
    >
    >```go
    >conn,err:=listener.Accpet()
    >```
    >
    >4.将连接绑定到RPC服务
    >
    >```go
    >rpc.Server(conn)
    >```

  - client端：

    >1.连接RPC服务
    >
    >```go
    >conn,err:=rpc.Dail()
    >```
    >
    >2.调用远程函数
    >
    >```go
    >conn.Call("服务名.方法名",传入参数,传出参数)
    >```

- Rpc相关函数

  ```go
  //注册rpc服务
  func (server *server) RegisterName(name string,rcvr interface{}) error
  参1:服务名称
  参2:对应的rpc对象
  	1).方法必须是导出的（首写字母大写）
  	2).方法有两个参数
  	3).第二个参数必须是'指针'（传出参数）
  	4).方法只有一个error返回值
  
  参2案例：
  type World struct{
  }
  func (w *World) HelloWorld(name stirng,resp *string) error{
  }
  注册案例：
  rpc.RegisterName("服务名",new(World))
  
  //绑定服务
  func(server *server)ServeConn(conn io.ReadWriteCloser)
  conn:成功建立连接后的socket--->conn
  
  //调用远程函数
  func (c *client) Call(serviceName string,args interface{},reply interface{})error
  serviceName:"服务名.方法名"
  args：传入参数
  reply:传出参数--->定义var变量，&变量名，完成传参
  ```

  

- 代码实现

  ```go
  //服务端
  package main
  
  import 'net/rpc'
  
  type World struct{
      
  }
  func (w *World) HelloWorld(name string,resp *string)error{
      *resp=name+"hello world"
  }
  func main(){
      w:=World{}
      err:=rpc.RegisterName("hello",w)
      listener,err:=net.Listen("tcp","127.0.0.1:8080")
      defer listener.Close()
      conn,err:=listener.Accept()
      defer conn.Close()
      rpc.ServeConn(conn)
  }
  
  //客服端
  package main
  import 'net/rpc'
  
  func main(){
      conn,err:=rpc.Dail("tcp","127.0.0.1:8080")
      var resp string
      err=conn.Call("hell.HelloWorld","张三",&resp)
  }
  ```
  
  
  
- 通信问题
  - 不同的语言之间通信会存在乱码的情况
    - go语言实现rpc使用了特有的数据序列化方式：gob，其他语言不能正常解析
    - 使用通用的序列化技术：json、protobuf

- 使用json格式化rpc数据

  - client端

    ```go
    package main
    import (
    	'net/rpc/jsonrpc'
    )
    func main(){
        conn,err:=jsonrpc.Dail("tcp","127.0.0.1:8080")
        var result string
        conn.Call("hello.HelloWorld","zhangsan",&result)
    }
    ```

  - server端

    ```go
    package main
    
    import 'net/rpc/jsonrpc'
    
    type World struct{}
    func (w *World) HelloWorld(name string,resp *string)error{
        *resp=name+"hello world"
    }
    func main(){
        w:=World{}
        err:=rpc.RegisterName("hello",w)
        listener,err:=net.Listen("tcp","127.0.0.1:8080")
        defer listener.Close()
        conn,err:=listener.Accept()
        defer conn.Close()
        jsonrpc.ServeConn(conn)
    }
    ```
    
  - ==如何绑定方法返回的error不为空，接收的数据就为空==
  
- rpc封装

  - 为什么需要做封装？
    - 如果注册rpc时，传入的对象的方法不符合规范，编译时是不会抛错的，只有在运行时才抛错，需要防止这种情况的出现，使其在编译时就暴露出问题

  ```go
  //server端
  package main
  import 'net/rpc'
  type rpcInterface interface{//定义接口类型
      rpcFunction(arg string,resp *string) error
  }
  
  func serverRegister(name string,i rpcInterface){//实现了接口类型的才能正确使用该函数
      rpc.RegisterName(name,i)
  }
  type stu struct {}
  func (s *stu) rpcFunction (name string,resp *string)error{//实现接口方法
      *resp=name+"hello"
      return nil
  }
  func main(){
      w:=World{}
      serverRegister("hello",w)
      listener,err:=net.Listen("tcp","127.0.0.1:8080")
      defer listener.Close()
      conn,err:=listener.Accept()
      defer conn.Close()
      jsonrpc.ServeConn(conn)
  }
  ```

  

- golang中protobuf语法

  - protobuf是类似json一样的数据描述语言（数据格式）==序列化后体积小==、==反序列化快==
  - protobuf非常适合于rpc数据交换格式

  ```protobuf
  //从创建一个以.proto结尾的文件
  syhtax = 'proto3';//指定版本（默认是proto2）
  
  package pb;//指定所在包包名
  
  //枚举类型
  enum Week{
      Monday = 0;//枚举类型必须从0开始
      Thresday = 1；
  }
  //消息定义中的每个字段都有一个唯一的编号，编号用于标识消息二进制格式的字段（不能使用19000-19999）
  message Student{
      int32 id = 1;//同一个结构体中可以不从1开始，但是不能重复
      string name = 2;
      People p = 3;//支持嵌套
      repeated int32 score = 4;//数组|切片
      Week w =5;//枚举类型
      oneof data{//联合类型
          string teacher = 6;
          int32 class = 7;
      }
  }
  
  message People{
      int32 wight = 1；
  }
  ```

  

- protobuf编译

  > C++编译命令
  >
  > protoc --cpp_out = ./ *.proto ---->xx.pb.cc 和xx.pb.h

  > golang编译命令
  >
  > protoc --go_out = ./ *.proto  ---->xx.pb.go

- protobuf中添加rpc服务

  - 语法：

  ```protobuf
  service 服务名{
  	rpc 函数名(参数：消息体) returns (返回值：消息);//注意：此处是returns，这个s不能少
  }
  
  例子：
  message People{
  	string name = 1;
  }
  message Resp{
  	string mes = 1;
  }
  service hello {
  	rpc HelloWorld(People) returns (Resp);
  }
  ```

  - 默认情况下，protobuf在编译期间不会编译RPC服务，要想编译需要使用GRPC
    - 命令：protoc --go_out = plugins = grpc : ./ *.proto

- Grpc知识

  ```protobuf
  //在pb包中创建person.proto
  syntax="proto3";
  package pb;
  //一个package中不允许定义同名的消息体
  message Teacher{
  	int32 age=1;
  	string name=2;
  }
  //定义服务
  service SayName{
  	rpc SayHello(Teacher) returns (Teacher)
  }
  ```

  ```go
  //服务端grpc
  package main
  import (
      'google.golang.org/grpc'
      'src/pb'
      'context'
      'net'
  )
  
  type M struct{}
  func (m *M) SayHello(c context.Context,t *pb.Teacher)(*pb.Teacher,error){
      t.Name+="chi"
      return t,nil
  }
  func main(){
      //1.初始化一个grpc对象
      grpcServer:=grpc.NewServer()
      //2.注册服务
      pb.RegisterSayNameServer(grpcServer,new(M))
      //3.设置监听
      listener,err:=net.Listen("tcp","127.0.0.1:8800")
      //4.启动服务
      grpcServer.Serve(listener)
  }
  ```

  ```go
  //客户端grpc
  package main
  import ('')
  fun main(){
      //1.连接Grpc服务
      conn，err:=grpc.Dail("127.0.0.1:8800",grpc.WithInsecure())
      //2.初始化grpc客户端
      grpcClient:=pb.NewSayNameClient(conn)
      //创建并初始化一个Teacher
      var teacher pb.Teacher
      teacher.Name="san"
      teacher.Age=20
      //3.调用远程
      t,err:=grpcClient.SayHello(context.TODO(),&teacher)
  }
  ```

  

##### go micro

- micro是一个专注于简化分布式系统开发的微服务生态系统。由开源库和工具组成：go-micro(核心)、go-plugins、micro

- 服务发现：微服务开发中必须的技术

  ```go
  //传统的客户端访问服务端：通过ip:port请求，不同的服务通过不同的ip:port请求，如果服务端的ip:port发生了变化，客户端必须知道并且修改才能继续访问
  
  //服务发现是加在客户端和服务端之间的一层，每个服务启动时会去服务发现注册ip:port和服务名，客户端先向服务发现请求获取目的服务的ip:port，客户端再借助服务发现将请求发向目的服务
  ```

- 服务发现种类
  - consul：常用于go-micro
  - mdns：go-micro默认自带的
  - etcd：k8s内嵌的服务发现
  - zookeeper:java中常用

- consul关键特性：服务发现（服务端主动向其进行注册）、健康检查（心跳检测）、键值存储（一般使用redis）、多数据中心（方便搭建集群）

- consul安装

  ```go
  1.下载安装包：wget https://releases.hashicorp.com/consul/1.5.2/consul_1.5.2_linux_amd64.zip
  2.解压安装包：unzip consul_1.5.2_linux_amd64.zip
  3.将解压包放到：sudo consul /usr/local/bin/
  4.检查是否成功：consul -h
  ```

  

- consul常用命令

  ```go
  consul agent 
  	-dev    于开发者模式（使用默认配置）启动一个consul代理
  	-bind=0.0.0.0	指定consul所在机器的ip地址
  	-http-port=8500		访问端口
  	-client=127.0.0.1	可以访问该consul的机器的ip（0.0.0.0即所有机器均可访问）
  	-config-dir=foo		所有主动注册服务的描述信息存储文件
  	-data-dir=path		存储注册机器的信息信息
  	-node=hostname		服务发现的名称
  	-rejoin				consul启动时，允许加入到集群
  	-server				以服务启动允许其他consul连接，不加则不能连接
  	-ui					可以使用web页面查看服务发现信息
  
  测试案例：
  consul agent -server -bootstrap-expect 1 -data-dir /tmp/consul -node=n1 -bind=192.168.0.1 -ui -rejoin -config-dir=/etc/consul.d/ -client 0.0.0.0
  
  浏览器上：192.168.0.1:8500可以看见详情
  
  consul members:查看集群中的成员
  consul info:查看当前consul的相关信息
  consul leave:优雅的关闭consul
  ```

  

- 注册服务到consul上

  ```go
  //通过提供服务定义文件或者调用http api来注册
  步骤：
  	1.进入配置文件的路径（即-config-dir的配置路径下）
  	2.创建一个json文件
  	3.按json语法填写服务信息
  		{
              "service":{
                  "name":"服务名称",
                  "tags":["别名1","别名2"],
                  "port":8800
              }
          }
  	4.重启consul
  
  查看服务：
  curl -s 127.0.0.1:8500/v1/catalog/service/服务名称
  ```

  

- 健康检查

  ```shell
  在编写的json文件中添加check对象
  		{
              "service":{
                  "name":"服务名称",
                  "tags":["别名1","别名2"],
                  "port":8800,
                  "check":{
                  	"id":"api",
                  	"name":"servier check",
                  	"http":"http://192.168.6.77:8800",
                  	"interval":"10s",
                  	"timeout":"1s"
                  }
              }
          }
  重新加载配置文件的方法：
  	1.关闭consul后重新启动
  	2.在consul启动的情况下使用：consul reload
  	
  除了http实现健康检查外，还可以使用"脚本"、"tpc"、"tcl"方式进行健康检查
  ```

  

##### grpc和consul的结合使用

- 注册使用服务

```shell
整体流程：
	1.创建proto文件，指定rpc服务
	2.启动consul服务
	3.启动server	[1.获取consul对象	2.使用consul对象将server信息注册到consul	3.启动服务]
	4.启动client	[1.获取consul对象	2.使用consul对象获取健康的服务	3.访问服务(grpc远程调用)]
```



```go
//1.创建pb文件夹，在pb中创建一个proto文件
syntax="proto3";
package pb;
message Peroson{
    string name = 1,
    string say = 2
}
service say{
    rpc saySomething(Person) returns (Person)
}
//2.在pb包中编译proto文件
protoc --go_out=plugins=grpc:./*.proto
```

```go
//3.编写服务端的代码
package main

type Stu struct{}
//实现xx.pb.proto中的方法
func (s *Stu)SaySomething(c context.Context,p *pb.Person)(*pb.Person,error){
    p.Say=p.Name+"hello"
    return p,nil
}

func main(){
    //1.初始化grpc对象
    grpc:=grpc.NewServer()
    //2.注册服务
    pb.RegisterSayServer(grpc,new())
    //3.设置监听
    listener,err:=net.Listen("tcp","127.0.0.1:8800")
    //4.启动服务
    grpc.Serve(listener)
}
```

```go
//4.编写客户端代码
package main
func main(){
    //1.连接服务
    grpcConn,err:=grpc.Dail("127.0.0.1:8800",grpc.WithInsecure)
    //2.初始化grpc客户端
    client:= pb.NewClient(grpcConn)
    //3.远程调用函数
    var p pb.Person
    p.Name="zhang"
    p,err:=client.SaySomething(context.TODO(),&Person)
    fmt.Println(p)
}
```

```go
//5.引入consul，把grpc注册到consul上（服务端）
import 'github.com/hashicorp/consul/api'
func main(){
    //-----------------------------------consul-----------------------------
    //1.获取consul配置文件
    consulConfig:=api.DefaultConfig()
    //2.创建consul对象
    consulClient,err:=api.NewClient(consulConfig)
    //3.注册服务的配置信息
    registerService:=api.AgentServiceRegistration{
        ID:"service",
        Tags:[]string{"grpc","consul"},
        Name:"grpc register",
        Address:"127.0.0.1",
        Port:8800,
        Check:&api.AgentServiceCheck{
            Tcp:"192.168.137.130:8800",
            Timeout:"5s",
            Interval:"10s",
        },
    }
    //4.注册服务到consul上
    consulClient.Agent().ServiceRegister(&registerService)
    //--------------------------------------grpc----------------------------
    //1.初始化grpc对象
    grpc:=grpc.NewServer()
    //2.注册服务
    pb.RegisterSayServer(grpc,new())
    //3.设置监听
    listener,err:=net.Listen("tcp","127.0.0.1:8800")
    //4.启动服务
    grpc.Serve(listener)
}
```

```go
//6.引入consul（客户端）
package main
func main(){
    //1.初始化consul配置
    config:=api.DefaultConfig()
    //2.初始化consul对象(可以重新指定consul属性：ip/port)
    client,err:=api.NewClient(config)
    //3.服务发现（从consul上获取健康的服务)
    services,_,err:=client.Health().Service("grpc register","grpc",true,nil)
    //4.组装远程函数地址
    addr:=services[0].Service.Address+":"+strconv.Atoi(service[0].Service.Port)
    //--------------------------------------grpc----------------------------
    //1.连接服务
    grpcConn,err:=grpc.Dail(addr,grpc.WithInsecure)
    //2.初始化grpc客户端
    client:= pb.NewClient(grpcConn)
    //3.远程调用函数
    var p pb.Person
    p.Name="zhang"
    p,err:=client.SaySomething(context.TODO(),&Person)
    fmt.Println(p)
}
```

```go
//使用到的函数
func(h *Health)Service(service,tag string,passingOnly bool,q *QuerOptions)([]*ServiceEntry,*QuerMeta,error){}
//参数
service:服务名称---注册服务时指定
tag:别名
passingOnly：是否通过健康检查
q:查询参数，一般nil
//返回值
serviceEntry:存储服务的切片（可用的服务）
QuerMeta：额外查询返回值
```

- 注销服务

  ```go
  package main
  
  func main(){
      //获取配置
      config:=api.DefatuilConfig()
      //初始化consul对象
      client,err:=api.NewClient(config)
      //注销
      client.Agent().ServiceDeregister("service")
  }
  ```

#### go-micro

- 安装方式：在线安装、docker镜像安装

- 测试：使用micro命令即可

##### go-micro的使用

```go
new
	参数：
	--namespace：命名空间==包名
	--type：为服务类型（srv(微服务)\web(基于微服务的web网站))
```

```go
//创建微服务项目文件
micro new --type srv 项目名称
生成文件介绍：
main:项目入口
handler/：处理grpc实现的接口，对应实现接口的子类都放在该文件中
proto/：预生成的protobuf文件目录
dockerfile：部署微服务使用的dockerfile
makefile：编译文件 --快速编译protobuf文件

//使用make命令编译protobuf文件
make proto------>xx.pb.go和xx.micro.go
```

```go
//生成的main.go文件
func main(){
    //初始化
    service:=mirco.NewService(
        micro.Name("服务名称"),
        micro.Version("版本号"),
    )
    
    //注册服务
    项目名.Register项目名Handler(service.Server(),new(handler.项目名))
    //运行服务
    service.Run()
}
```

- 在micro框架中添加consul

```go
import 'github.com/micro/go-micro/registry/consul'
func main(){
    //初始化服务发现
    consulReg:=consul.NewRegistry()
    //初始化micro
    service:=micro.NewService(
        micro.Name(""),
        micro.Registry(consulReg),
        micro.Version(""),
    )
    //另一种初始化
    service.Init()
}
```

#### gin框架

- 构建一个简单的web

 ```go
package main
import "github.com/gin-gonic/gin"
func main(){
    //1.初始化web引擎(初始化路由)
    router:=gin.Default()
    //2.路由匹配
    router.GET("/index",func(c *gin.Context){c.Writer.WriteString("hello")})
    router.POST()
    router.PUT()
    router.DELETE()
    //3.启动运行
    router.Run(":8800")
}
 ```

- 联合go-micro使用

```go
//1.将编译好的proto文件导入项目--->将pb包复制到新的项目
package main
import (
	'github.com/gin-gonic/gin'
    pb '项目名/proto/proto文件目录'
    'github.com/micro/go-micro/client''
    c2 'context'
)

func Handler(c *gin.Context){
    //1.根据编译报的proto文件中的方法初始化客户端
    client:=pb.New项目名Service("微服务名称",client.DefaultClient)
    //2.调用远程服务
    response,err:=client.Call(c2.TODO(),&pb.Request{name:"lili"})
    //3.查看返回值
    fmt.Println(response)
    c.Writer.WriteString(response.Msg)
}

func main(){
    //1.初始化路由
    router:=gin.Default()
    //2.路由匹配
    router.GET('/',Handler)
    //3.启动
    router.Run("127.0.0.1:8080")
}
```

- 联合g-micro和consul使用

  - mdns服务发现：（组播）支持的服务必须是本地服务，局域网内的服务
  
  ```go
//服务端
  package main
import (
  	'github.com/micro/go-micro/registry/consul'
      'github.com/micro/go-micro/registry'
  )
  
  func main(){
      //1.注册consul
      consulReg:=consul.NewRegistry(func(o *registry.Options){
          o.Addrs = []string{"127.0.0.1:8080"}
      })
      
      service:=micro.NewService(
          micro.Name("rpc"),
          micro.Registry(consulReg),
          micro.Version("last")
      )
      
      service.Run()
  }
  ```
  
  ```go
  //客户端
  //1.将编译好的proto文件导入项目--->将pb包复制到新的项目
  package main
  import (
  	'github.com/gin-gonic/gin'
      pb '项目名/proto/proto文件目录'
      c2 'context'
  )
  
  func Handler(c *gin.Context){
      cr:=consul.NewRegistry()
      service:=micro.NewService(
          micro.Registry(cr),
      )
      //1.根据编译报的proto文件中的方法初始化客户端
      client:=pb.New项目名Service("微服务名称",service.Client())
      //2.调用远程服务
      response,err:=client.Call(c2.TODO(),&pb.Request{name:"lili"})
      //3.查看返回值
      fmt.Println(response)
      c.Writer.WriteString(response.Msg)
  }
  
  func main(){
      //1.初始化路由
      router:=gin.Default()
      //2.路由匹配
      router.GET('/',Handler)
      //3.启动
      router.Run("127.0.0.1:8080")
  }
  ```
  
  ### 项目
  
  http5大错误类型：1xx(请求成功，需要继续发送请求)、2xx(请求被成功接收)、3xx(请求的资源指定到对应的URI上)、4xx(请求端错误)、5xx(服务端错误)
  
  ```go
  //错误常量设置 micro/web/utils/error.go
  package utils
  //错误对应的编号
  const (
      ERCODE_OK	= "0",
  )
  
  var recodeText = map[string]string{
      ERCODE_OK:	"成功",
  }
  //获取map值的方法
  func RecodeText(code string) string{
      str,ok:=recodeText[code]
      if ok{
          return str
      }
      return recodeText[RECODE_UNKNOWERR]
  }
  
  
  ```
  
  ```go
  //入口代码 micro/web/mian.go
  package main
  import (
  	'github.com/gin-gonic/gin'
      'micro/web/controller'
  )
  
  func main(){
      //gin框架第一步：初始化路由
      router:=gin.Default()
      //gin框架第二步：路由匹配
      router.GET("/api/v1.0/session",controller.GetSession)
      router.GET("/api/v1.0/imagecode/:uuid",controller.GetImageCD)
      //gin获取静态资源:第一个参数为访问路径，第二个参数为访问该路径时去那个目录中找静态资源
      router.Static("/","view")
      //gin框架第三步：启动运行绑定端口
      router.Run(":8080")
  }
  ```
  
  ```go
  // 控制函数模块 micro/web/controller/user.go
  package controller
  
  import (
  	'github.com/gin-gonic/gin'
      getCaptcha 'micro/web/proto/getCaptcha'
  )
  //返回json格式的数据
  func GetSession(ctx *gin.Context){
      m:=make(map[string]string)
      m["errno"]=utils.RECODE_SESSIONERR
      m["errmsg"]=utils.RecodeText(utils.RECODE_SESSIONERR)
      ctx.JSON(http.StatusOK,m)
  }
  
  //获取路径中的参数
  func GetImageCD(ctx *gin.Context){
      uuid:=ctx.Param("uuid")
      registry:=consul.NewRegistry()
      service:=micro.NewService(
          micro.Registry(registry)
      )
    client:=getCaptcha.NewGetCaptchaService("go.micro.srv.getCaptcha",service.Client())
      rsp,err:=client.Call(context.TODO(),&getCaptcha.Request{})
      if err!=nil{
          return
      }
      var img capthcha.Image
      json.Unmarshal(rsp.img,img)
      png.Encode(ctx.Writer,img)
  }
  ```
  
  **将验证码功能集成为微服务**
  
  1. 在项目的service目录下生成微服务文件`micro new --type srv micro/service/getCaptcha`
  2. 修改生成的proto文件micro/service/getCaptcha/proto
  
  ```protobuf
  syntax = 'proto3';
  package go.micro.srv.getCaptcha;
  service GetCaptcha{
  	rpc Call(Request) returns (Response){}
  }
  message Request{
  }
  message Reponse{
  //使用切片存储图片信息，用json序列化
  	bytes img = 1;
  }
  ```
  
  3. 编译proto文件`make proto`获得xx.mirco.go和xx.pb.go文件
  4. 修改生成的main
  
  ```go
  package main
  
  func main(){
      registry:=consul.NewRegistry()
      service:=micro.NewService(
          micro.Address("127.0.0.1:8880"),//防止随机生成port
          micro.Name("go.micro.srv.getCaptcha"),
          micro.Registry(registry),
          micro.Version("latest"),
      )
      getCaptcha.RegisterGetCaptchaHandler(service.Server(),new(handler.GetCaptcha))
      if err := service.Run(); err != nil{
          log.Fatal(err)
      }
  }
  ```
  
  5. 修改handler/getCaptcha.go文件
  
  ```go
  package handler
  
  import ...
  
  type GetCaptcha struct{}
  
  func (e *GetCaptcha)Call(ctx context.Context,req *getCaptcha.Requset,rsp *getCaptcha.Response) error{
      uuid:=ctx.Param("uuid")
      cap:=captcha.New()
      cap.SetFont("./conf/comic.ttf")
      cap.SetSize(128,64)
      cap.SetDisturbance(captcha.MEDIUM)
      cap.SetFrontColor(color.RGBA{0,0,0,255})
      cap.SetBkgColor(color.RGBA{100,0,255,255})
      img,str:=cap.Create(4,captcha.NUM)
      err:=SaveImgCode(str,uuid)
      if err !=nil{
          return err
      }
      imgjson,_:=json.Marshal(img)
      rsp.img=imgjson
      return nil
  }
  ```
  
  **go操作redis**
  
  ```go
  package main
  import (
  	 'github.com/gomodule/redigo/redis'
  )
  func main(){
      //连接redis
      conn,err:=redis.Dail("tcp","127.0.0.1:6379")
      defer conn.Close()
      //操作数据
      rep,err:=conn.Do("set","key","value")
      //使用回调助手确认类型
      s,err:= redis.String(rep,err)
      fmt.Println(s,err)
  }
  ```
  
  **在微服务中添加redis** 
  
  ```go
  // micro/service/getCaptcha/model
  package model
  
  import 'github.com/gomodule/redigo/redis'
  
  //存储图片id 到redis数据库
  func SaveImgCode(code,uuid string)error{
      //连接redis
      conn,err:=redis.Dail("tcp","127.0.0.1:6379")
      if err !=nil{
          return err
      }
      defer conn.Close()
      //操作数据,有效时间5分钟
      _,err=conn.Do("setex",uuid,60*5,code)
      return err
  }
  ```
  
  

