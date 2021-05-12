### 											golang周学习内容

​	1.golang介绍：go被称为21世纪的“C”语言，天生就支持高并发，速度和java差不多，是强类型的编译性语言

​	2.GOPATH和GOROOT和GOBIN：GOPATH是设置工作目录的路径、GOROOT是设置golang安装包的位置、GOBIN是设置放置go编译好的运行程序的路径

​	3.go目录分层：在gopath路径下需要创建三个文件夹：bin、src、pkg，分别存放编译后的运行文件、源码和依赖包

​	4.golang中的helloworld代码详解：

```go
package main//指定文件属于的包
import "fmt"//引入包
func main(){//{不能单独一行，必须跟在函数名后面
    fmt.Println("helloworld")
}
```

​	注意：在golang中每个文件夹都应该有自己所属于的一个包，该文件夹下面的所有.go文件都属于这个包

​	5.golang中变量的定义和声明方式

```go
var a int//第一种方式：先声明再赋值
a = 10
var b string = "hello"//第二种方式：声明的时候进行赋值
var c = "world"//第二种方式的变种：自动推导数据类型
d:=true//第三种方式（使用最多的）：局部变量的声明和赋值
```

​	6.golang中的自增语法

​	golang中使用i++、i--作为自增和自减的语法，但是没有++i和--i的语法

​	注意：在golang中自增、自减的语法都只能单独一行，不能和其他的在一行

​	7.golang中的指针

​	在golang中指针只有两种用法：获取指定变量的指针（内存地址）（&）、通过指针获取该内存地址中记录的值（*），golang中空指针是nil

​	8.golang中不支持的语法

​	在golang中不支持使用三目运算，就算条件后面只有一行代码也不能省去{}

​	9.golang中的string

​	golang中的多行string需要使用到反引号，链接两个string可以直接使用+号，可以使用fmt包中的方法生成string(fmt.Sprintf())

​	10.golang中数组

​	注意：数组在声明时需要声明长度，在golang中数组一般被切片代替，遍历数组可以使用for循环、for range等方式

​	注意2：在数组中声明完毕后就会存储类型零值，数组值的修改只能通过索引，无法想JS一样使用push

```go
var array [5]int//第一种声明方式：声明一个数组和它的容量，后面再赋值，声明以后数组里面存储的都是对应类型的零值
var array1 =[3]string{"hello","golang","nihao"}//第二种声明方式：声明的同时直接赋值使用
array2 ：=[...]bool{true,false,true}//第三种方式：使用...自动推断数组的长度
```

​	11.golang中的for range

​	for range一般用于遍历数组、切片、string数组、管道、map等

```go
var array = [...]int{1,2,3,4,5}
for k,v:=range array{//遍历数组时，k为下标，v为下标索引的值
    fmt.Println(k,v)
}

var str = "helloworld"
for k,v:=range str{//v是字符串中每个元素
    fmt.Println(k,v)
}

var slice = []int{1,2,3}
for k.,v:=range slice{//遍历切片时和数组一致
    fmt.Println(k,v)
}

m:=make(map[int]int,3)
m[1]=2
for k,v:=range m{//遍历map时，k为键，v为值
    fmt.Println(k,v)
}

c：=make(chan int,2)//在使用make声明管道时，不申请容量就是默认无缓冲的
for v:=range c {//遍历管道时只有一个返回值，就是管道中存储的值，使用该方法遍历管道需要关闭管道，使其返回零值，否则会抛错（死锁）
    fmt.Println(v)
}

```

注意：map和chanel的声明必须使用make，不然默认值就是nil无法使用

12. golang中的切片(slice)

    注意：切片的本质是由底层数组切割而来

    ```go
    //切片的声明
    var s []int//第一种方式：先声明在赋值，注意：此处的切片是不能使用的，nil
    var s = []int{1,2,3}//第二种方式：类型推导，这样的声明方式可以直接使用
    s:=[]int{1,2,3}//和第二种一样，只是使用了短声明方式
    s:=make([]int,0)//第三种方式：使用make声明切片并绑定内存，可以正常使用
    //由数组生成切片
    var a = [5]int{1,2,3,4,5}
    s:=a[:]
    s:=a[:3]
    ```

13. golang中向切片中追加元素

    ```go
    var s = []int{}
    s = append(s,1)
    //注意：append是生成了另外一个切片，而不是改变原切片，所以需要将s重新赋值
    ```

14. golang中map

    ```go
    //map类型的声明
    m:=make(map[int]string)
//下面这个方式声明不赋值默认为nil
    var m1 = map[int]string{1:"hello"}
    
    //向map中添加值
    m[2]="world"
    //修改map中的值
    m[2]="golang"
    //删除map中的值
    delete(m,1)
    //查询map中的值
    v,ok:=map[2]
    ```
    
15. golang中的函数

    ```go
    //函数的声明
    func 函数名 (形参 类型，形参 类型) (返回值 类型){
        函数体
    }
    /*
    多个形参类型一致可以使用：形参，形参 类型的方式
    函数也可以作为参数或者返回值使用
    */
    
    //匿名函数
    func (形参 类型)(返回值 类型){
        函数体
    }(实参)
    
    //函数作为参数使用
    func 函数名(形参 func(类型)(类型)){
        函数体
    }
    ```

16. golang中的内存逃逸

    - golang中允许内存逃逸---->栈中的指针作为值返回到堆上  eg:在一个函数中获得了一个指针地址，将该指针地址作为返回值返回给调用者

17. golang中的import

    - import是golang中导入包的关键字

    ```go
    import "fmt"//导入单个包
    //导入多个包
    import(
    	"fmt"
        "net"
    )
    //不使用包中的函数，只需要导入包执行init函数
    import _ "mysql"
    ```

18. golang中的命令行参数

    ```go
    //命令行参数：os.args
    ```

    

    