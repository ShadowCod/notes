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


- redis中的set类型

  set中的值不能重复,且是无序的

  set类型的命令开头都是s

  ```bash
  #向set类型中存入值
  sadd myset "hello"
  #查看set类型中的值
  smembers myset
  #查询set类型中是否存在指定值
  sismember myset 'hello'
  
  #获取set类型拥有值的数量
  scard myset
  #移除set类型中的某个值
  srem myset 'hello'
  
  #随机抽取set类型中的一个成员
  srandmember myset
  #随机抽取set类型中的二个成员
  srandmember myset 2
  
  #随机弹出一个成员（随机删除）
  spop myset
  #将一个指定的成员移动到另外一个set中
  smove myset myset2 "hello"
  
  #数字集合类
  #	-差集:sdiff set1 set2
  #	-交集:sinter set1 set2
  #	-并集:sunion set1 set2
  ```

  

- redis中的hash类型

  想象成map集合，key-<key,value>,key-map,值是一个map集合,和string类型没太大的区别，只是值变成了map，建议和string类型对比学习

  hash类型的命令都是以h开头的

  应用：①存储变更数据（用户信息） ②更适合对象的存储 ③string更适合字符串的存储

  ```bash
  #向hash类型key中添加值
  hset myhash filed1 hello
  #获取hash类型key中存储的值
  hget myhash filed1
  #同时向hash类型key中添加多个值
  hmset myhash filed1 golang filed2 world
  #同时获取hash类型中key的多个值
  hmget myhash filed1 filed2
  #获取hash某一个key中的所有值
  hgetall myhash;----->filed1 golang filed2 world
  
  #删除hash类型key中指定字段，对应的值也会删除
  hdel myhash filed1
  #获取hash类型key的长度(一共有多少个键值对)
  hlen myhash
  #判断hash类型key中包含某个字段
  hexists myhash filed1
  
  #只获取hash类型中key中的所有字段
  hkeys myhash
  #只获取hash类型中key中的所有值
  hvals myhash
  
  #hash类型key指定自增hincr、hincrby
  hset test fild 1
  hincrby test fild 2
  
  #hash类型key指定自减hdecr、hdecrby
  hdecrby test fild 1
  
  #hsetnx向hash类型的key中存入值，存在则失败
  hsetnx test fild1 5
  ```

  

- redis中的Zset（有序集合）类型

  在set[set k v]的基础上，增加了一个值[zset k 排序标志 v]

  应用场景：存储班级成绩、工资表、带权重的判断、排行榜

  ```bash
  #向zset类型中添加值
  zadd myzset 1 one
  zadd myzset 2 two 3 three
  
  #查看zset中的值
  zrange myzset 0 -1;#查看全部内容
  zrange myzset 0 0;#查看第一个内容
  
  #给zset类型中的值进行排序
  zadd salary 2500 'xiaohong'
  zadd salary 5000 'xiaoming'
  zadd salary 200 'zhangsan'
  
  #按中间值排序（从小到大） 语法：zrangebyscore key min max
  zrangebyscore salary -inf +inf;#将所有的值进行排序
  zrangebyscore salary -inf +inf withscores;#显示值和分数
  zrangebyscore salary -inf 2500 withscores;#只排序负无穷到2500内的并显示键和值
  
  #按score排序（从大到小） 语法：zrevrange key max min
  zrevrangescore salary +inf -inf;
  
  #移除zset类型中的值
  zrem salary 'xiaohong';
  
  #查看zset类型中有多少个元素
  zcard salary;
  
  #获取zset类型中指定区间的成员数量
  zcount salary 0 5000;
  ```

  

- reids中的geospatial类型(地理位置)

  ```bash
  #添加地理位置 语法：geoadd key 经度 纬度 名称 经度 纬度 名称
  #有效经度：-180至180	有效纬度：-85至85
  geoadd china:city 116.40 39.90 '北京'
  geoadd china:city 121.47 31.23 '上海'
  
  #查询指定地点的经纬度		语法：geopos key member [member]
  geopos china:city '北京'
  geopos china:city '北京' '上海'
  
  #返回两个指定位置之间的距离		语法：geodist key member member 距离单位
  geodist china:city '北京' '上海' km
  
  #以指定的经纬度为中心，找出某一半径内的元素	语法：georadius key 经度 纬度 半径 单位 count 显示数量
  georadius china:city 110 30 1000 km
  
  #找出给定位置指定范围内的元素	语法：georadiusbymember key member 半径 单位
  georadiusmember china:city '北京' 500 km
  
  #原理：是利用zset来实现的，因此可以使用zset的操作命令。
  zrange china:city 0 -1;----->查看所有元素
  zrem china:city '北京';------>移除指定元素
  ```

  

- redis中的hyperloglog类型

  基数：不重复的元素，A={1,3,4,5,3,7},基数就是1,3,4,5,7

  该类型的作用：做基数统计的算法，有0.81%的错误率

  优点：占用内存是固定的并且占用很少(12KB)

  应用：统计网页的uv（一个人访问一个网站多次,但是还是只算一次）

  ```bash
  #添加元素
  pfadd mykey a b c d e f g h i
  #统计不重复的元素
  pfcount mykey
  #合并两个key成一个新的key(重复的只算一个)(并集)
  pfmerge mykey3 mykey1 mykey2
  ```

  

- redis中的bitmaps类型

  位存储（0|1），两个状态

  应用场景：签到，打卡

  ```bash
  #添加元素
  setbit sign 0 0
  setbit sign 1 1
  #查看
  getbit sign 1
  #统计(查出值为1的个数)
  bitcount sign
  ```

  

- redis中的事务操作

  redis中单条命令是保证原子性的，但是事务不能保证原子性

  redis事务的本质：一组命令的集合，一个事务中的所有命令都会被序列化，在事务执行过程中，会按照顺序执行

  特性：一次性、顺序性、排他性

  redis中的事务没有隔离级别的概念，事务中的命令不会直接执行！只有发起执行命令的时候才会执行（exec）

  ```bash
  #开启事务（multi）
  multi
  #命令入队
  set key1 vl
  set key2 v2
  get key2
  #执行事务（exec）
  exec
  
  #放弃事务（discard）,队列中的命令全部不执行
  multi
  set mykey 1
  set mykey1 2
  discard
  
  #编译型异常（代码有问题），事务中所有命令都不会执行
  multi
  set k1 v1
  getset k2//错误命令
  set k3 v3
  exec//执行会抛错
  #运行时异常，其他命令正常执行，错误命令抛出异常
  set k 'hello'
  multi
  incr k//只能对数字加一
  set k5 v5
  get k5
  exec
  ```

  

- redis中的乐观锁

  悲观锁：很悲观，认为什么时候都可能出问题，无论做什么都会枷锁！浪费性能

  乐观锁：很乐观，认为什么时候都不会出问题，所以不会上锁！更新数据的时候去判断，在此期间是否有人修改过这个数据。

  ​				获取version

  ​				更新时比较version

  ```bash
  #正常情况下
  set money 100
  set out 0
  watch money #监视money对象（相当于获取对象现在的值，执行事务时判断对象的值是否改变）
  multi
  decrby money 20
  incrby out 20
  exec #事务正常执行完毕，监视会自动取消
  
  #在第一个线程执行事务操作时，第二个线程修改了监视的对象
  watch money
  multi
  decrby money 10
  incrby out 10
  #第二个线程修改money
  set money 1000
  #第一个线程执行事务
  exec  #结果是nil，事务执行失败
  
  #放弃监视对象（如果监视时事务执行失败，可以先解除监视，然后重新监视再进行事务操作）
  unwatch money
  ```

  

- redis配置文件详解

  redis的启动是根据配置文件执行的

  ```bash
  #配置文件unit单位        对大小不敏感
  #可以包含其他redis配置文件
  bind 127.0.0.1  #绑定ip地址
  protected-mode yes  #保护模式是否开启
  port 6379   #端口
  daemibuze yes   #是否用守护模式启动，默认是no
  pidfile /var/run/redis_6379.pid   #如果以后台的方式运行，指定pid进程文件
  
  #日志
  #debug:大量的信息，用于开发、测试
  #verbose
  #notice：部分重要信息，用于生产环境
  #warning：非常重要的信息
  loglevel notice 
  logfile ''   #日志的文件位置名称
  
  database 16   #默认的数据库数量
  always-show-logo yes #是否显示logo
  
  #快照:持久化，在规定的时间内，执行了多少次操作，则会持久化到文件.rdb  .aof
  #注意：redis是内存数据库，如果没有持久化数据就消失了（断电即失）
  save 900 1  #在900S内，如果有一个key进行了修改，就进行持久化操作
  save 300 10  #在300S内，如果有十个key进行了修改，就进行持久化操作
  save 60 10000  #在60S内，如果有一万个key进行了修改，就进行持久化操作
  
  stop-writes-on-bgsave-err yes   #持久化出错后是否还继续让redis工作
  
  rdbcompression yes   #是否压缩rdb文件，需要消耗一些cpu资源
  
  rdbchecksum yes   #保存rdb文件时是否检查校验
  
  dir ./  #rdb文件的保存目录
  
  #SECURITY安全
  requirepass 123456  #设置密码（配置文件设置）
  config set requirepass 123456   #命令设置密码
  #注意：设置了密码后需要登陆才能操作
  auth 123456
  
  #限制
  maxclients 10000  #设置能连接上redis客户端的最大数量
  maxmemory <bytes>   #redis配置最大的内存容量
  maxmemory-policy noeviction #内存到达上限后的处理策略
  #策略
  #volatile-lru：只对设置了过期时间的key进行LRU
  #allkeys-lru：删除Lru算法的key
  #volatile-random：随机删除即将过期的key
  #allkeys-random：随机删除
  #volatile-ttl：删除即将过期的
  #noeviction：永不过期，返回错误
  
  #append only模式  aof配置
  appendonly no   #默认不开启，默认使用rdb持久化方式。在大部分情况下rdb完全够用了
  appendfilename 'appendonly.aof'    #持久化文件的名称
  
  appendfsync always    #每次修改都会同步，消耗性能
  appendfsync everysec  #每秒执行一次同步，但是可能会丢失这1s的数据
  appendfsync no        #不执行同步，操作系统自己同步数据，速度最快
  ```

  

- redis持久化RDB

  ```bash
  #原理：父进程fork一个子线程来确保持久化，而主线程只管数据库
  
  #rdb保存的文件默认是dump.rdb,可以在配置文件中修改
  #修改了redis的配置文件，执行shutdown，exit然后重启reids服务
  #使用flushall会自动触发持久化功能生成.rdb文件
  
  #触发规则
  #1.配置的save规则满足
  #2.执行了flushall命令
  #3.退出redis
  
  #如何恢复rdb文件
  #只需要将rdb文件放在redis启动目录下即可，redis启动时会自动检查
  #查看需要存放的位置
  config get dir;------>/usr/local/bin
  
  #优点：1.适合大规模的数据恢复	2.适合对数据完整性要求不高的情况
  #缺点：1.需要一定的时间间隔进行操作，如果redis意外宕机，最后一次修改的数据就丢失了
  #	  2.fork进程的时候会占用一定的内存空间
  ```

  

- redis持久化AOF

  将执行的所有命令都记录下来，恢复只需要将文件全部执行一遍即可

  ```bash
  #原理：父进程fork一个子进程，子进程将所有命令写入aof文件
  
  #aof保存的是appendonly.aof文件
  #默认是不开启的，需要手动修改配置表开启，重启redis服务就可以生效了
  
  #oppendonly.aof可能会被破坏，被破坏后无法启动redis。可以使用redis-check-aof --fix 文件名 检测工具修复aof文件
  redis-check-aof -- fix appendonly.aof
  
  #优点：1.每次修改都同步，文件完整性更好	2.默认配置为每秒同步一次，只可能丢失1S的数据
  #缺点：1.相对于数据文件来说，aof文件远远大于rdb文件，修复的速度也比rdb慢
  #	  2.aof运行效率比rdb慢
  
#如果aof文件大于64M，就会fork一个新的子进程将文件进行重写
  ```
  
  

