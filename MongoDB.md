### MongoDB

**第一部分**

1. 数据库是什么？
   
- 按照数据结构来组织、存储和管理数据的仓库
  
2. 数据库的分类
   - 关系型数据库：MySQL、Oracle、DB2、SQL Server...			全是表
   - 非关系型数据库：MongoDB（文档数据库）、Redis...		键值对数据库		

3. SQL是什么？
   
- 结构化查询语言
  
4. MongoDB作用是什么？
   - 是为快速开发互联网Web应用而设计的数据库----->关系型数据库在项目初期需要花费大量数据设计表
   - 设计目标是极简、灵活、作为Web应用栈的一部分
   - 数据模型是面向文档的，文档是一类类似Json的结构----->增强版的Json-----Bson（二进制的json）

5. MongoDB中的三个概念
   - 数据库:一个数据集服务器中可以有多个数据库，在数据库中存放的是集合
   - 集合：集合存储在数据库中，类似于数组，在数组中可以存储文档
   - 文档：文档是数据库中最小的单位，存储于集合中，存储和操作的内容都是文档

6. MongoDB下载：
   
- [MongoDB](https://www.mongodb.org/dl/win32/)----->mongoDB对于32为操作系统支持很差，需要用64位系统
  
7. MongoDB环境配置
   - 找到安装mongoDB的文件夹，进入bin目录，将上边的路径复制到高级环境中的path后面，使用;隔开
   - win+r打开dos控制台，在控制台使用mongod查看是否配置成功
   - 在C盘上新建一个data文件夹，在data文件夹中创建要给db文件夹----->mongoDB存储数据的地方

8. 启动mongoDB
   - 输入cmd打开控制台，输入mongod----->最后一行会展示端口号(默认27017)
   - 将上面的控制台缩小，然后在打开一个控制台，输入mongo链接mongoDB---->出现connecting to:test则表示链接成功

9. 将data文件夹放在其他盘
   
- 打开命令行窗口，使用mongod --dbpath D:\data\db
  
10. 设置端口号
    
- 打开命令行窗口，使用mongod --dbpath D:\data\db --prot 8080
  
11. 数据库组成
    - 数据库服务器----->保存数据------>mongod启动服务器----->选中或者关闭服务器窗口后客户端都无法操作
    - 数据库客户端------>操作服务器，对数据进行增删查改----->mongo启动客户端

12. 将mongodb设置为系统服务，可以自动在后台启动，不需要每次都手动启动 [操作文档](https://docs.mongodb.com/manual/tutorial/install-mongodb-on-windows-unattended/)

    - 在data下面创建一个log文件夹

    - 在mongodb的安装目录的bin目录同级目录中添加一个配置文件（mongod.cfg）

      ```
      //配置文件内容
      systemLog:
      	destination:file
      	path: c:\data\log\mongod.log
      storage:
      	daPath: c:\data\db
      ```

    - 以管理员的身份打开命令行窗口输入

      ```
      sc.exe create MongoDB binPath="\"mongod的bin目录\mongod.exe\" --service --config=\"mongo的安装目录\mongod.cfg\"" DisplayName = "MongoDB" start= "auto"
      ```

13. 如果启动失败
    
    - 在控制台上输入sc delete MongoDB 删除之前的配置服务，然后重新再设置

14. 安装图形化管理工具
    
    - 下载mongodbmanager进行安装

**第二部分**

- 基本概念：
  - 在mongodb中，数据库和集合都不需要手动创建---->当我们创建文档时，如果文档所在的集合和数据库不存在会自动创建数据库和集合

1. 查看数据库服务器中有多少个数据库

   ```sql
   /*
   mongoDB中
   */
   show dbs		show databases
   /*
   MySQL中
   */
   show databases;
   ```

2. 进入某个数据库（use后面可以跟没有的数据库名称，会在第一次插入文档时创建该数据库）

   ```sql
   /*
   mongodb中
   */
   use 数据库名称	use test
   /*
   MySQL中
   */
   use 数据库名称;
   ```

3. 查看当前在那个数据库

   ```sql
   /*
   mongodb中直接使用命令:db
   db相当于一个变量，进入某个数据库是就将值赋予db这个变量
   */
   db
   /*
   MySQL中使用:select database()
   */
   select database();
   ```

4. 查看数据库中有哪些集合

   ```sql
   /*
   mongoDB中
   */
   show collections
   
   /*
   MySQL中
   */
   show tables;
   ```

**第三部分**

- CRUD操作 [操作文档](https://docs.mongodb.com/manual/crud/)

1. 向数据库中插入文档		**增**

   ```sql
   /*
   语法：db.collection.insert(doc)
   eg:向test数据库中的stus集合中插入一个新的学生集合
   */
   db.stus.insert({name:"孙悟空",age:18,gender:"男"});
   
   /*
   插入多个的语法：
   db.collection.insert([doc,doc])
   */
   db.stus.insert([{name:"猪八戒",age:20,gender:"男"},{name:"老沙",age:30,gender:"男"}])
   /*
   如果没有给文档指定_id属性，会自动生成objectId(),该属性用来作为文档的唯一标识
   _id也可以自己指定：{_id:"hello"},必须确保唯一性
   */
   
   /*
   db.collection.insertOne()   ----->插入一个参数只能有一个
   db.collection.insertMany()  ----->插入多个参数必须是数组
   */
   
   
   /*
   向numbers集合中添加2W条数据
   */
   #将操作mongodb的语句放置循环内会执行多次
   for(var i=1;i<20000;i++){
   	db.numbers.insert({num:i});
   }
   #优化：先将数据对象生成，放进一个数组中，在使用插入语句
   var arr=[];
   for(var i=1;i<20000;i++){
   	arr.push({num:i})
   }
   db.numbers.insert(arr);
   ```

2. 查询集合中的所有文档     **查**

   ```sql
   /*
   语法：db.collection.find()----->查找集合中所有符合条件的文档，可以接收一个对象作为条件参数
   结果中的ID和mongodb自动生成的
   */
   db.stus.find();------>mongodb中
   
   select * from stus;----->mysql中
   
   /*
   条件查询
   语法：db.<collection>.find({字段名:值})
   注意：多个条件之间是&&关系
   	find()返回的是一个数组
   */
   db.stus.find({_id:"hello"});----->mongodb中
   
   select * from stus where id = 1;----->mysql中
   
   /*
   查询集合中符合条件的第一个文档
   语法：db.collection.findOne()
   findOne()返回的是一个对象，可以直接获取属性值
   */
   db.stus.findOne({age:18})
   
   /*
   获取查询出来有多少个元素
   db.collection.find().count();
   db.collection.find().length();
   */
   db.stus.find().count();
   
   /*
   条件为内嵌文档的属性，如果要使用内嵌属性来对文档进行查询，属性名必须使用“”
   */
   db.stus.find({hobby.movies:"hero"});//错误的写法
   db.stus.finc({"hobby.movies":"hero"});//正确的写法
   
   /*
   查询条件中使用逻辑运算符：
   	$gt 大于
   	$gte 大于等于
   	$lt 小于
   	$lte 小于等于
   	$ne 不等于
   	$eq 等于
   	$or 满足其中一个条件
   */
   db.numbers.find({num:{$gt:5000}})
   
   db.numbers.find({num:{$gt:30,$lt:40}})
   
   db.numbers.find({num:{$or:[$lt:100,$gt:5000]}});//错误写法
   
   db.numbers.find({$or:[{num:{$lt:1000}},{num:{$gt:5000}}]});//正确写法
   /*
   查看集合中的指定条数据：
   	limit()可以来设置显示数据的上限
   */
   db.numbers.find().limit(10);
   
   /*
   分页查询:
   	skip()用于跳过指定数量的数据
   	
   db.collection.find().skip((页码-1)*每页显示的条数).limit(每页显示的条数)	
   mongodb会自动调整skip和limit的位置
   */
   db.numbers.find().skip(10).limit(10)
   db.numbers.find().skip(20).limit(10)
   
   /*
   查询文档时，默认情况下是按照_id值进行排序（升序）
   sort()用来指定排序的规则,需要传递一个对象指定【排序规则
   	1：升序
   	-1：降序
   */
   db.emp.find().sort({sal:1});
   
   db.emp.find().sort({sal:1,empno:-1});----->先按照sal排序，值一样的再安装empno排序
   
   /*
   再查询时，可以在第二个参数的位置来设置查询结果的  投影
   1显示
   0不显示
   */
   db.emp.find({},{ename:1,_id:0})
   ```

3. 修改文档    **改**

   ```sql
   /*
   语法：db.collection.update(查询条件，新对象,默认设置);------>默认情况下只会修改一个
   	update()默认是使用新对象替换旧对象
   	想要修改而不是替换需要使用“修改操作符”
   		$set 可以用来修改文档中的指定属性
   		$unset  可以用来删除文档中的指定属性
   
   db.collection.updateOne();----->修改第一个查询到的值
   
   db.collection.updateMany();----->修改多个查询到的值
   
   db.collection.replaceOne();----->替换
   */
   db.stus.update({name:"老沙"},{age:20})
   
   db.stus.update({name:"老沙"},{$set:{age:20}})
   
   db.stus.update({name:"老沙"},{$unset:{age:20}})
   
   /*
   向stus集合中name为“老沙”的对象添加一个hobby：{city:["shanghai","beijing"],move:["hero"]}
   */
   db.stus.updata({name:"老沙"},{$set:{hobby:{city:["shanghai","beijing"],move:["hero"]}}});
   
   注意：mongodb的文档的属性值也可以是一个文档，当一个文档的属性值为文档时，我们称添加的这个文档为内嵌文档
   
   /*
   向内嵌文档中的属性字段添加值，及往内嵌属性的数组里面添加元素
   $push向数组中添加一个新的元素
   $addToSet向数组中添加一个新元素---->只会新添加数组中没有的元素，如果数组中已经拥有则不会添加成功
   */
   db.stus.update({name:"老沙"},{$push:{"hobby.movies":"sanguo"}})
   
   /*
   在原来的值上增加
   	%inc
   */
   db.number.update({num:{$lt:200}},{$inc:{num:10}})
   ```

4. 删除文档    **删**

   ```sql
   /*
   语法：
   	db.collection.remove();----->可以根据条件删除文档，传递条件的方式和find()一样，删除符合条件的所有文档，如果只传入{}则会删除集合中的所有文档
   	db.collection.deleteOne();
   	db.collection.deleteMany();
   */
   
   db.stus.remove({_id:"hello"})
   
   db.stus.remove({_id:"hello"},true);----->第二个参数表示只删除一个
   
   /*
   删除集合：
   	db.collection.drop();
   	如果只有一个集合在数据库中，删除该集合就会删除该数据库
   */
   db.stus.drop();
   
   /*
   删除数据库：
   	db.dropDatabase();
   */
   db.dropDatabase();
   ```


**第四部分**

1. 文档之间的关系

   - 一对一（one to one）

     - 在mongodb中可以使用内嵌的方式来体现一对一的关系

       ```sql
       db.wifeAndHusband.insert([{name:"huangrong",husband:{name:"guojing"}}])
       ```

   - 一对多（one to many） 或者   多对一（many to one）

     - 在mongodb中也可以使用内嵌的方式来实现

       ```sql
       db.users.insert([{_id:"1",name:"sunwukong"},{_id:"2",name:"zhubajie"}]);
       db.orders.insert({list:["香蕉","西瓜"],user_id:"1"});
       #查询sunwukong的订单
       var id =db.users.findOne({name:"sunwukong"})._id;
       db.orders.find({user_id:id})
       ```

   - 多对多（many to many）

     - 在mongodb中也是使用内嵌的方式进行实现

       ```sql
       db.teachers.insert([{name:"1"},{name:"2"},{name:"3"}]);
       db.stus.insert([{name:"3",tech_ids:["1","2"]}]);
       ```

**第五部分**

​	根据不同的编程语言选择不同的mongodb库对数据库进行操作。

​	[goalng中使用mongodb](https://www.liwenzhou.com/posts/Go/go_mongodb/)

