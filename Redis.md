### Redis

**1. Nosql概述**

- Nosql发展

  **90年代单机时代**

  ​	App----->DAL----->Mysql

  出现问题：

  - 数据量如果太大，一个机器放不下
  - mysql（B+Tree）一个表数据超过300W条就需要创建索引，一个机器内存也放不下
  - 访问量（读写混合），一个服务器承受不了

  **Memacahed(缓存) + Mysql + 垂直拆分（读写分离）**

  ​	App----->DAL----->Mysql(多个Mysql，其中一个负责写，写的时候同步给其他数据库，其他的负责读)

  ​	发展过程：优化数据结构和索引----->文件缓存(IO)----->Memcached(当时最热门的技术)

  ​	一个网站大部分数据库操作是读，为了减轻数据库的压力，因此可以使用缓存技术，将读过的数据存放到缓存中，基本解决了读问题

  **分库分表 + 水平拆分 + Mysql集群**

  ​	本质：数据库（读、写）

  ​	MyISAM引擎：表锁（查表中一条数据会将整个表锁起来，十分影响效率，高并发下出现严重的锁问题）

  ​	Innodb引擎：行锁（只锁当前行）

  ​	慢慢的开始使用分库分表来解决写问题

  ​	Mysql的集群很好的满足了哪个年代的所有需求

  **最近年代**

  ​	Mysql等关系型数据库不够用了，数据量大，变化很快~

  ​	Mysql有的人使用来存储比较大的文件，数据表很大，效率就低了，为啥不将这些抽取出现呢，如果有一种数据库专门来处理这种数据，mysql的压力就会变得十分小。大数据压力下，表的设计会很久很久

- 为什么使用NoSql

  用户个人信息、社交网络、地理信息、用户自己产生的数据、用户日志等待爆发式增长

  这时候就需要NoSQL数据库，NoSQL可以很好的处理上述问题

- 什么是NoSQL
  - NoSQl=not only sql
  - 关系型数据库：表格、行、列
  - 泛指非关系型数据库，随着web2.0互联网的诞生，传统的关系型数据库很难对付，尤其是超大规模的高并发的社区，暴露出很多难以克服的问题，NoSQL发展的很快，redis是必须要掌握的
  - 使用键值对来存储，无需关系存储的内容格式

- NoSQl的特点

  1. 方便扩展（数据之间没有关系，很好扩展）

  2. 大数据量高性能（Redis一秒可以写8W次，读取11W次，NoSQL的缓存记录级是一种细粒度的缓存，性能比较高）

  3. 数据类型时多样型的！（不需要事先设计数据库！随取随用！）

  4. 传统RDBMS 和 NoSQL

     ```
     传统RDBMS										NoSQL
     - 结构化组织								 - 没有固定的查询语言
     - SQL									- 存储方式很多
     - 数据和关系都存在单独的表中					- 最终一致性 
     - 数据操作，数据定义语言					 -  CAP定理和BASE（异地多活）
     - 严格的一致性							- 高性能、高可用、高可扩展
     - 基础的事务操作							- ......
     - ......
     ```

  5. 要学会NoSQL + RDBMS

- NoSQL的四大分类

  - KV键值对：redis
  - 文档型数据库（BSON|二进制json）：MongoDB
    - 基于分布式文件存储的数据库，c++编写，主要用来处理大量的文档
    - 是一个介于关系型数据库和非关系型数据库中间的产品，是非关系型数据库中最想关系型数据库的

  - 列存储数据库：HBase、分布式文件系统
  - 图形关系数据库：Neo4j、InfoGrid     是用来存关系的，不是用来存图片的

**Redis入门**

官网:https://redis.io/

- 安装

1. 在window下安装
   - 下载安装包：https://github.com/dmajkic/redis/releases
   - 下载完毕得到压缩包
   - 解压到redis的包（redis十分的小）
   - 开启redis，双击运行服务即可，redis默认端口6379
   - 使用redis客户端连接redis，双击客户端.exe即可,输入ping，如果返回pong即连接成功
2. 在linux下安装

   - 下载安装包：https://redis.io/
   - 将安装包放到linux上并解压（程序一般放在opt下）
   - 进入解压后的文件，有个.conf文件，就是redis的配置文件
   - 安装环境 yum install gcc-c++    查看版本gcc -v
   - 将需要的文件配置上，使用：make  && make install
   - 默认redis安装路径：usr/local/bin
   - 将redis配置文件复制到bin目录下
   - redis默认不是后台启动的，需要对配置文件进行修改，将daemonize改为yes
   - 启动redis服务：在usr/local/bin中运行redis-server 配置文件路径
   - 使用redis客户端连接：redis-cli -h 127.0.0.1 -p 6379
   - 查看redis进程是否正常：ps -ef | grep redis
   - 关闭redis服务：在客户端使用shutdown断开连接，然后exit退出客户端

- redis-benchmark

  是一个压力测试工具，官方自带的性能测试工具

  使用方式：redis-benchmark  命令参数

  ![image-20210415163337767](C:\Users\ll\AppData\Roaming\Typora\typora-user-images\image-20210415163337767.png)

  简单测试：

  ```bash
  #测试:100个并发连接，每个并发10W请求
  #在usr/local/bin中执行命令
  redis-benchmark -h locahost -p 6379 -c 100 -n 100000
  ```

- redis基本知识

  - redis默认有16个数据库----->redis.conf中database关键字----->默认使用第零个数据库---->可以使用select进行切换

    ```bash
    #切换使用第几个数据库
    select 数据库编号
    #查看数据库大小，仅查看使用的数据库
    dbsize
    #查看当前数据库中所有的key
    keys *
    #清空数据库①清空当前②清空所有
    flushdb			flushall
    ```

  - 为什么redis默认端口号是6379?----->一个意大利女歌手名称缩写的按键数字

  - redis是单线程的！

    ```bash
    redis是很快的，官方表示，redis是基于内存操作，cpu不是redis性能瓶颈，redis的瓶颈是部署机器的内存和网络带宽，能使用单线程就使用单线程，所以是单线程
    
    #为什么单线程还这么快?
    reids是C语言编写的，官方提供的数据为10W+QPS，说明不比同样使用key-value的Memecache差
    #误区
    1.高性能的服务器一定是多线程的----->不一定
    2.多线程一定比单线程效率高----->多线程cpu上下文切换（耗时操作）
    #核心
    redis是所有的数据全部放在内存上的，所以使用单线程去操作效率高，对于内存系统来说，如果没有上下文切换操作效率最高
    ```


- redis-key

  ```bash
  #设置key
  set name lili
  #获得key的值
  get 键名
  #判断某个key是否存在
  exists 键名  ---->存在返回1，不存在返回0
  #移除key
  move 键名 1   ----->1代表当前数据库
  #设置key过期时间
  expire 键名 秒数
  #查看key剩余时间
  ttl 键名
  #查看key存储的类型
  type 键名
  ```


- redis中的string类型

  string类型的使用场景：value除了是字符串还可以是数字

  - 计数器
  - 统计多单位的数量
  - 对象缓存存储

  ```bash
  #查看key是否存在
  exists 键名
  #向字符串中追加元素，如果当前key不存在就相当于set key
  append 键名 追加的值
  #获取key中存储的值
  get 键名
  
  #获取字符串长度
  strlen 键名
  #自动加一，类似i++
  incr 键名
  #自动减一，类似i--
  decr 键名
  #步长，自定义每次加多少，类似i+=
  incrby 键名 每次增加数量
  #自定义每次件多少，类似i-=
  decrby 键名 每次减少数量
  #字符串范围(截取字符串)[开始，结束]
  getrange 键名 开始位置 结束位置   eg.set key1 "hello,golang";  getrange key1 0 3;-->"hell"
  #查看全部字符串
  getrange 键名 0 -1
  #替换！
  setrange 键名 开始位置 替换字符  eg.set key2 "abcdef"; setrange key2 1 xx;--->"axxdef"
  
  #setex(set with expire),设置过期时间
  setex key3 30 "hello"
  #setnx(set if not exist)，不存在再设置   在分布式锁中常使用
  setnx key4 "redis";  #如果key4不存在，创建key4，如果存在就创建失败
  #批量设置值
  mset k1 v1 k2 v2
  #批量获取值
  mget k1 k2
  #不存在设置，存在设置失败(原子性操作，如果有一个不能创建就全部失败)
  msetnx k1 v1 k2 v2
  
  #设置对象
  set user:1 {name:zhangsan,age:3};#---->设置一个user1对象，值为json字符串来保存一个对象
  #进阶写法(这里的key是一个巧妙的设计：user:{id}:{filed},如此设计在redis中是可以的)
  mset user:1:name zhangsan user:1:age:12;
  #获取值
  mget user:1:name user:1:age
  
  #组合命令（先get再set）:显示原来的值，再将值进行修改
  getset db "redis";----->nil
  get db;----->redis
  getset db "MongoDB";----->redis
  get db;----->MongoDB
  ```


- redis中的list类型

  基础数据类型：列表

  在redis里面可以用list实现：栈、队列、阻塞队列

  所有的list命令都是l开头的

  **注意**：实际上list类型是一个链表

  ```bash
  #向list中存值，插入到列表的头部(左进)
  lpush list one
  lpush list two
  lpush list three
  #获取list中的值
  lrange list 0 -1;----->获取list中的全部值：three、two、one
  #获取list中指定的值
  lrange list 0 1;----->three two
  
  #向list中存值，插入到列表的尾部（右进）
  rpush list four
  lrange list 0 -1;----->three、two、one、four
  #移除list中的值，移除头部，一次只会移除一个（左|第一个）
  lpop list
  #移除list中的值，移除尾部，一次只会移除一个（右|最后一个）
  rpop list
  lrange list 0 -1;----->two one
  #获取指定索引的值(索引从0开始)
  lindex list 1;----->one
  
  #获取list的长度
  llen list;----->2
  
  #移除list中指定个数的指定值（精确匹配）
  lpush list three
  lrange list 0 -1;----->three two one 
  rpush list three;
  lrange list 0 -1;----->three two one three
  
  lrem list 1 one;---->three two three   #移除list中一个one
  lrem list 2 three;---->two  #移除list中2个three
  
  #截取一部分list中的值(通过下标)
  rpush list1 'hello1'
  rpush list1 'hello2'
  rpush list1 'hello3'
  lrange list1 0 -1;----->hello1 hello2 hello3
  trim list 0 1
  lrange list1 0 -1;----->hello1 hello2
  
  #移除list中的最后一个元素移动的另外一个列表中
  rpoplpush list1 list2
  lrange list2 0 -1;----->hello2
  lrange list1 0 -1;----->hello1
  
  #判断list是否存在
  exists mylist;  #判断mylist这个列表是否存在
  lpush mylist v1
  #将列表中指定位置的元素替换掉(如果指定的位置本来没有值会报错)|更新操作
  lset mylist 0 item
  lrange mylist 0 0;----->item
  
  #向list中的前面或者后面插入指定的值
  linsert mylist before item other;#在mylist的item前面添加一个other
  lrange mylist 0 -1;----->other item 
  linsert mylist after other v2;#在mylist中的other后面添加一个v2
  lrange mylist 0 -1;----->other v2 item 
  ```

  