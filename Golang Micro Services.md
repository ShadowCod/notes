### Golang Micro Services

 ###### 什么是微服务

- 一个服务仅用于某个特定的业务功能，相互独立
- 总而言之，微服务的目标是通过将应用程序分解成较小的可组合的部分，以便在需要时可以独立部署、升级、删除或缩放，从而简化构建、维护和管理应用程序

###### 单体式架构和微服务的区别

- 单体式：所有的业务处理都在一个服务上，一个业务处理崩溃就可以导致这个服务崩溃（soa按功能模块划分）
  - ==复杂性==随着开发==越来越高==，遇到问题解决困难
  - ==耦合度高==，维护困难
  - 开发周期较长
  - 技术选型成本高，风险大
  - ==扩展性==较差
- 微服务：每个服务只处理一种业务，多个服务共同实现一个大的服务
  - 优点：职责单一、轻量级通讯、迭代开发方便、独立性高
  - 缺点：运维成本高、部署复杂度高、接口成本高、重复性劳动、业务分离困难

###### RPC协议

- 什么是RPC
  - 远程过程调用，属于应用层的协议，使用的tcp实现

- 理解RPC
  - 像调用本地函数一样去调用远程函数（通过rpc协议，传递函数名、参数调用远端函数，得到返回值）

- 为什么要使用RPC协议
  - 每个为服务都是一个独立的进程，且可以使用不同的语言开发，只要都遵循RPC协议就能通讯

###### RPC使用入门

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
    >rpc.ResgisterName("rpc名称"，回调函数)
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

  

