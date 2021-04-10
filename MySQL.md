### MySQL

**第一部分：基础**

- mysql的配置文件my.ini在安装目录中，可以自己去设置port、存储路径、字符集等等
- 启动mysql：
  - 可以进服务里面找到mysql服务名称右击启动或者关闭
  - 也可以用管理员身份运行控制台:net stop mysql服务器、net start mysql服务进行

- 登录和退出mysql
  - 使用mysql自带的客户端---mysql command line client---->直接输入root账户密码（不建议使用）
  - 退出使用exit或者ctrl+c
  - 也可以用管理员身份运行控制台:mysql -h localhost -P 3306  -u root -p 回车然后输入密码(推荐使用)
  - 注意：最后的-p和密码之间不能有空格

**第二部分：命令**

- 查看数据库版本--->在mysql客户端：select version();--->在windows终端：mysql --version

- 显示mysql中有那些数据库：show databases;

- 打开具体数据库：use 数据库名称;----->use test;

- 查看打开的数据库中有那些表:show tables;
- 在一个数据库中查看另一个数据库中的表show tables from 数据库名称;--->show tables from mysql;
- 查看自己在那个数据库里面：select database();

语法规范：

- 不区分大小写，建议关键字大写
- 每条命令最好用;号结束
- 每条命令根据需要，可以进行缩进或者换行
- 注释使用---->单行：#注释文字|--  注释文字----->多行：/* 注释文字 */

**第三部分：DQL语言**

- 基础查询：

  - 注意：在做关于某个数据库的任何操作之前都需要先进入该数据库

  ```sql
  /*
  语法：select 查询列表 from 表名;
  
  1.查询列表可以是：表中的字段、常量值、表达式、函数
  2.查询的结果是一个虚拟的表格
  */
  #进入指定数据库
  use employess;
  
  #查询表中的单个字段
  select last_name from employess;
  
  #查询表中的多个字段
  select last_name,salary,email  from employess;
  
  #查询表中的所有字段
  select * from employess;
  
  #查询常量值   注意：字符型和日期型的常量必须用单引号包括
  select 100;
  select 'john';
  
  #查询表达式
  select 100*98;
  
  #查询函数
  select version();
  
  #起别名:①便于理解	②查询的字段有重复时可以区分
  /*
  方式一：使用as
  */
  select 100*98 as 结果;
  select last_name as 姓,first_name as 名 from employees;
  
  /*
  方式二：可以省略as
  */
  select　last_name 姓,first_name 名 from employess;
  #注意：如果别名中间有特殊符号[out put]需要使用‘’
select salary as 'out put' from employess;
  
  #去重:关键字distinct   注意：不能有多个字段
  select distinct department_id from employess;
  /*
  +号的作用
  仅仅只有一个功能：运行符
  select 10+9;两个操作数都为数值型，则作加法运行
  select "123"+9;其中一个为字符型，试图将字符型转换为数值型进行加法运行
  select "jhon"+1;转换失败则将字符型转换为0
  select null+1;只要一方为null结果肯定为null
  */
  
  /*
  实现字符型链接需要使用concat
  */
  select concat(last_name,first_name) as '姓名' from employess;
  
  /*
  使用ifnull(可能为null的字段，期望输出什么)
  */
  select concat(job_id,',',ifnull(commission_pct,0)) as 'info' from employess;
  ```
  
  **第四部分：条件查询**
  
  ```sql
  /*
  语法：
  	select 查询列表 from 表明 where 筛选条件
  	
  分类：
  	1、按条件表达式筛选
  		条件运算符：> < = != <> >= <=
  	2、按逻辑表达式筛选
  		逻辑运算符：&& || !	and or not
  	3、模糊查询：	like | between and | in | is null
  */
  /*按条件表达式筛选*/
  select * from employess where salary >12000;
  select last_name,department_id from emp where department !=90;
  
  /*按逻辑表达式筛选*/
  select last_name from emp where salary>=10000 and salary <=20000
  
  select * from emp where dep<90 or dep >100 or salary>12000
  select * from emp where not(dep>=90 and dep <=100) or salary >10000
  
  /*
  模糊查询
  	like特点：
  	①一般和通配符搭配使用:%任意多个字符、_任意单个字符
  	②可以使用转义字符
  	③可以使用escape指定转义字符
  */
  select * from emp where last_name like '%a%';
  select * from emp where last_name like '__e_a%';
  select * from emp where last_neme like '_\_%';
  select * from emp where dep_id like '1__';查询编号1开头有三位的值
  
  select * from emp where last_name like '_$_%' escape '$';#指定'$'是转义符
  
  /*
  between and
  注意：包含临界值
  */
  select * from emp where emp_id>100 and emp_id <120;
  select * from emp where emp_id between 100 and 120;
  
  /*
  in拥有判断某字段的值是否属于in列表中的某一项，列表中的值类型必须统一或者兼容
  */
  select * from emp where job_id ='it' or job_id ='ad_vp';
  select * from emp where job_id in('it','ad_vp');
  
  /*
  is null：仅仅只能判断null值
  =或者<>不能判断是否为null值
  */
  select last_name,salary from emp where com_pct is null;
  select last_name,salary from emp where com_pct is not null;
  
  /*
  安全等于：<=>
  不仅可以判断为null，也可以判断普通值
  */
  select last_name,salary from emp where com_pct <=> null;
  ```
  

**第五部分：排序查询**

- 语法：select 查询列表 from 表名 [where 筛选条件] order by 排序列表 [asc|desc]

- 特点：asc代表升序，desc代表降序，不写默认为asc

- order by后面可以支持单个、多个字段、表达式、函数、别名

- order by执行顺序：from--->where--->select--->order by

  ```sql
  /*
  基础用法
  */
  select * from emp order by salary desc; 
  select * from emp order by salary asc;
  
  select * from emp where dep_id >= 90 order by hiredate asc;
  
  /*按表达式排序*/
  select *,salary*12*(1+ifnull(commission_pct,0)) as '年薪' from emp order by salary*12*(1+ifnull(commission_pct,0));
  
  /*按别名排序*/
  select *,salary*12*(1+ifnull(commission_pct,0)) as '年薪' from emp order by '年薪';
  
  /*按函数排序*/
  select length('jhon');
  select length(last_name) '字节长度',last_name,salary from emp order by length(last_name) desc;

  /*先按照某个条件排序再按照另一个条件排序*/
  select * from emp order by salary asc,emp_id desc;
  ```
  
  

**第五部分：常见函数**

- 功能：类似于java中的方法

- 调用：select 函数名(实参列表) from 表;

- 分类：单行函数(concat、length、ifnull等)、分组函数别名：统计函数、聚合函数、组函数(做统计的)

  ```sql
  /*
  单行函数：
  	字符函数
  		length:获取参数的字节个数
  		concat：拼接字符串
  		upper:将参数转换为大写
  		lower:将参数转换为小写
  		substr|substring:截取字符(索引从1开始)
  		instr:返回子串在主串第一次出现的索引位置，找不到返回0
  		trim:去除前后空格----->规定去掉指定内容trim(指定字符 from 目标字符串)
  		lpad:用指定的字符实现左填充指定长度---->指定长度没有字符串长度长就返回指定长度个字符
  		rpad:用指定的字符实现右填充指定长度
  		replace:替换
  */
  select length('jhon');
  select length('张三丰');
  
  select concat(last_name,"_",first_name)  from emp;
  
  select upper('lili');
  select lower('LILI');
  
  select concat(upper(first_name),lower(last_name)) from emp;
  
  select substr('李莫愁',2) as '称号';---->截取从索引出后面所有字符
  select substr('小龙女是李莫愁的师妹',1,3);---->截取从指定索引处指定字符长度的字符
  
  select concat(upper(substr(last_name,1,1)),lower(substring(last_name,2)),'_',lower(first_name)) as '名称' from emp
  
  select instr("小龙女",'龙女') as out_put;
  
  select trim('    张无忌    ');
  select trim('a' from 'aaaa杨aaaaaaa过aaaaaaa');
  
  select lpad("赵敏",10,'&');
  
  select replace("张无忌爱上了周芷若",'周芷若','赵敏');
  
/*
  数学函数：
  	round：四舍五入,支持负数
  	ceil:向上取整,返回大于等于参数的最小整数
  	floor:向下取整，返回小于等于参数的最大整数
  	truncate:截断
  	mod:取模
  */
  select round(1.65);
  select round(1.3456,2);---->四舍五入且保留小数后俩位
  
  select ceil(1.2);
  
  select floor(-1.2);
  
  select truncate(1.69999,1);----->小数后保留1位
  
  select mod(-10,3);
  
  /*
  日期函数：
  	now():返回当前系统日期+时间
  	curdate():返回当前系统日期，不包含时间
  	curtime():返回当前时间，不包含日期
  	year、month、day、hour、minute、second:获取年份、月份、日、时、分、秒
  	str_to_date:将日期格式的字符转换成指定格式的日期
  	date_format:将日期转换成字符
  */
  select year(now());
  select year('1995-8-29');
  
  select month(now());
  select monthname(now());
  
  select day(now())
  
  select str_to_date('1995-8-20','%Y-%c-%d');
  
  select * from emp where hiredate = str_to_date('4-3 1995','%c-%d %Y');
  
  select date_format(now(),'%Y年%m月%d日');
  
  select date_format(hiredate,'%Y年/%m月/%d日') from emp where com_pct is not null;
  
  /*
  其他函数
  	version:版本号
  	database:查看当前数据库
  	user：查看当前用户
  */
  
  
  /*
  流程控制函数
  	if函数：if else的效果 类似三元运算符
  	case:流程控制结构
  		①case 要判断的字段或者表达式
  		when 常量1 then 要显示的值或者语句
  		when 常量2 then 要显示的值或者语句
  		else 默认指定的值
  		end
  		注意：then后面是值不需要;，但是是语句就需要;
  		②case
  		when 条件1 then 要显示的值
  		when 条件2 then 要显示的值
  		else 默认值
  		end
  */
  select if(10>5,'大','小');
  
  select salary 原始工资,
  case dep_id
  when 30 then salary*1.1
  when 40 then salary*1.2
  when 50 then salary*1.3
  else salary
  end as 新工资
  from emp;
  
  
  select salary 原始工资,
  case 
  when salary>20000 then 'A'
  when salary>15000 then 'B'
  when salary>10000 then 'C'
  else 'D'
  end as 级别
  from emp;
  ```
  
  

