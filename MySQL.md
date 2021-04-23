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
  
  --去重--
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
  is not null
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
    	rand：0-1之间的随机数
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
    		①case 要判断的字段或者表达式------>类似switch
    		when 常量1 then 要显示的值或者语句
    		when 常量2 then 要显示的值或者语句
    		else 默认指定的值
    		end
    		注意：then后面是值不需要;，但是是语句就需要;
    		②case----->类似else if
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
    
  /*
  分组函数（聚合函数、统计函数、组函数）：
  	sum：求和
  	avg:平均值
  	max:最大值
  	min:最小值
  	count:计算个数
  	
  分组函数的参数类型：
  	sum和avg一般用于处理数值型,会忽略null
  	max和min支持任意类型，会忽略null
  	
  	count支持任意类型,会忽略null
  	
  分组函数可以和distinct搭配使用
  
  分组函数一同查询的字段限制：
  	group by后的字段，其他都不行
  	
  */
  #简单使用
  select sum(salary) from emp;
  select avg(salary) from emp;
  select max(salary) from emp;
  select min(salary) from emp;
  select count(salary) from emp;
  #多个字段查询
  select sum(salary),avg(salary) from emp;
  #作为其他函数的参数
  select round(avg(salary),2) from emp;
  #和distinct搭配使用
  select sum(distinct salary) from emp;
  /*
  count函数的详细介绍：
  效率：MYISAM存储引擎下，count(*)的效率最高
  INNODB存储引擎下，count(*)和count(1)效率差不多
  */
  #查询表中的总行数
  select count(*) from emp;
  #相当于表中每一行多一个1，用来统计总行数，2也可以
  select count(1) from emp;
  
  --datediff--
  #datediff(日期，日期)得到两个日期之间的天数
  select datediff(now(),'1995-08-20');
  
  /*
  分组函数进阶：
  	group by 字段
  	语法：
  	select 分组函数，列（要求出现在group by后面）
  	from 表名
  	[where 刷新条件]
  	group by 分组列表
  	[order by 子句]
  特点：
  	1.分组查询中的筛选条件分为两类
  				数据源			位置				关键字
  	分组前筛选	原始表			group by前面		where
  	分组后筛选	分组后的结果		group by后面		having
  	
  	分组函数做条件肯定是放在having子句中
  	能用分组前筛选的就放在group by前面
  	
  	group by子句支持单个或者多个字段，多个字段之间用“，”隔开，没有顺序要求，也支持表达式和函数
  	也可以添加排序，放置整个分组排序之后
  */
  #查询每个工种的最高工资
  select max(salary),job_id from emp group by job_id;
  
  select avg(salary),emp_id from emp where email like '%a%' group by emp_id;
  
  select max(salary),manager_id from emp where com_pct is not null group by manager_id;
  /*
  添加分组后的条件查询
  	select 分组函数，列（要求出现在group by后面）
  	from 表名
  	[where 筛选条件]
  	group by 分组列表
  	having 条件
  */
  select count(*),dep_id from emp group by dep_id having count(*)>2
  
  select max(salary),job_id from emp where com_pct is not null group by job_id having max(salary) >12000;
  
  /*
  按表达式或者函数分组
  	group by 和having后面可以跟别名
  */
  --按员工姓名长度分组，查询每一组的员工个数，筛选员工个数>5--
  select count(*),length(last_name) from emp group by length(last_name) having count(*)>5;
  
  --按多个字段分组--
  #查询每个部门每个工种的员工的平均工资
  select avg(salary),job_id,emp_id from emp group by job_id,emp_id;
  
  --分组查询中添加排序--
  #查询每个部门每个工种的员工的平均工资并且按平均工资的高低排序
  select avg(salary),emp_id,job_id from emp where dep_id is not null group by job_id,emp_id having avg(salary)>10000 order by avg(salary) desc;
  --查询员工最高工资和最低工资的差距，使用diffrence--
  select max(salary),min(salary) diffrence from emp 
  
  
  ```

**第六部分：连接查询**

```sql
/*
链接查询    多表查询
	笛卡尔乘积现象：表1有m行，表2有n行
	如何发生的：没有有效的链接条件
	如何避免：添加有效的链接条件
	
	分类：
		按年代分类  sql92标准（仅仅支持内链接）  sql99标准（推荐）
		按功能分类  内连接（等值链接、非等值链接、自链接）、外链接（左外链接、右外链接、全外链接）、交叉链接
*/
select name,boyName from boys,beauty where buauty.boyfriend=boys.id;

/*
sql92标准
	1.等值连接
	2.为表取别名（当两个表的字段名称一样时会出现歧义）
	注意：如果为表取了别名就不能用原表名去限定
	两个表的顺序可以交换
	
特点：
	多表连接的结果为多表的交集部分
	N表连接，至少需要N-1个连接条件
	多表的顺序没有要求
	一般需要为表取别名
	可以搭配排序、分组、筛选
*/
#查询女神名和对应的男神名
select name,boyName from boys,beauty where beauty.boyfriend = boys.id;
#查询员工名和对应的部门名
select last_name,department_name from emp,departments where emp.dep_id = departments.id;
#查询员工名，工种名，工种编号
select last_name,job_id,job_title from emp,jobs where emp.job_id = jobs.job_id;//job_id会出现歧义
select last_name,e.job_id,job_title from emp as e,jobs as j where e.job_id = j.job_id;

--增加筛选条件--
#查询有奖金的员工名、部门名
select last_name,dep_name from emp as e,dep as d where e.dep_id = d.dep_id and e.com_pct is not null;

select dep_name,city from dep as d,locations as l where d.location_id = l.location and city like '_o%';

--增加分组--
#查询每个城市的部门个数
select count(*),city from locations as l,emp as e where l.location_id = e.location_id group by city;
#查询有奖金的每个部门的部门名和部门的领导编号和该部门的最低工资
select dep_name,manager_id,min(salary) from emp as e,dep as d where e.dep_id=d.dep_id and com_pct is not null group by dep_name;

--增加排序--
#查询每个工种的工种名和员工的个数，并且按员工的个数降序
select job_title,count(*) from emp e,jobs j where e.job_id=j.job_id group by job_title order by count(*) desc;

--三表连接--
#查询员工名、部门名和所在的城市
select last_name,dep_name,city from emp e,deps d,locations l where e.dep_id=d.dep_id and d.location_id=l.location_id and city like 's%';

/*
非等值连接
*/
#查询员工的工资和工资级别
select salary,grade_level from emp e,job_grade g where salary between g.lowest_sal and g.highest_sal;

/*
自连接
	将一张表当成多张表使用
	一张表中有两种需要的数据，查找两遍
*/
#查询员工名和上级的名称
select e.last_name,m.last_name from emp e,emp m where e.manager_id=m.emp_id;


/*
sql99语法：
	select 查询列表
	from 表1 别名 [连接类型]
	join 表2 别名 
	on 连接条件
	where 筛选条件
	[group by 分组]
	[having 条件]
	[order by 排序列表]
	
连接类型：
	内连：inner
	外连：左外（left）[outer]、右外（right）[outer]、全外（full）[outer]
	交叉：cross
*/

/*
内连接语法
	select 查询列表
	from 表1 别名 inner
	join 表2 别名
	on 连接条件
	
分类：
	等值
	非等值
	自连接
*/
--查询员工名、部门名--
select last_name,dep_name from emp e inner join deps d on e.dep_id = d.dep_id;

--查询名字中包含e的员工名和工种名（添加筛选）--
select last_name,job_title from emp e inner join jobs j on e.job_id=j.job_id where last_name like '%e%';

--查询部门个数>3的城市名和部门个数(添加筛选和分组)--
select city,count(*) from emp p inner join citys c on p.city_id=c.city_id group by city  having count(*)>3;

--查询那个部门的员工个数>3的部门名和员工个数，并按个数降序--
select dep_name,count(*) from emp e inner join deps d on e.dep_id = d.dep_id group by dep_id having count(*)>3 order by count(*) desc;

--查询员工名、部门名、工种名，并按部门名降序（三表连接：后面的表需要和前面的表有联系）--
select last_name,dep_name,job_name from emp e inner join deps d on e.dep_id=d.dep_id inner join jobs j on e.job_id=j.job_id order by dep_name desc;

--员工的工资级别（非等值）--
select salary,grade_level from emp e join jobs_grade j on e.salary between j.lowest and j.hightest;

--查询员工的名字和领导的名字（自连接）--
select e.last_name,m.last_name from emp e join emp m on e.manager_id=m.emp_id;

/*
外连接
	应用场景：用于查询一个表中有，一个表中没有的场景
	特点：
	①外连接的查询结果为主表中的所有记录，如果有匹配的就显示匹配的值，没有就显示null
	外连接查询的结构=内连接查询结果+主表中有但是从表中没有的记录
	②左外连接，left join左边的表是主表	右外连接，right join右边的表是主表
	③左外和右外交换两个表的顺序可以实现同样的效果
*/
--查询男朋友不在男神表的女神名(左外)--
select b.name from beauty b left outer join boys n on b.b_id=n.id where n.id is null;


--查询那个部门没有员工--
select dep_id from dep d left outer join emp e on e.dep_id=d.id where e.id is null;
--使用右外连接(主表不变)--
select dep_id from emp e right outer join dep d on e.dep_id=d.id where e.id is null;

/*
全外连接=内连接结果+表一中有但表二中没有的+表2中有但表一中没有的	
*/
select b.*,bo.* from beauty b full outer join boys bo on b.b_id=bo.id;
/*
交叉连接
	使用99语法实现笛卡尔乘积
*/
select b.*,bo.* from beauty b cross join boys bo;
```

**第七部分：子查询**

```sql
/*
子查询
	含意：出现在其他语句中的select语句，称为子查询或者内查询
	外部的查询语句，称为主查询或者外查询
分类
按子查询出现的位置：
	select后面（仅仅支持标量子查询）
	from后面（支持表子查询）
	where和having后面（支持标量（单行）、列（多行）、行子查询）
	exists后面（相关子查询）（支持表子查询）
按结果集的行列数不同：
	标量子查询（结果集只有一行一列）
	列子查询（结果集只有一列多行）
	行子查询（结果集有一行多列）
	表子查询（结果集一般为多行多列）
	
*/

/*
where和having后面
1.标量
2.列

3.行
特点：
	①子查询都放在小括号内
	②子查询一般放在条件的右侧
	③标量子查询，一般搭配单行操作符使用（> < >= <= <>）
	④列子查询，一般搭配多行操作符使用（in、any/some、all）
	⑤子查询的执行优先于主查询
*/
#1.标量子查询（如果子查询的结果不是一行一列都是非法的）
--谁的工资比abel高--
#①先查询abel的工资
select salary from emp where last_name like 'abel';
#②查询员工信息，满足salary>①的结果
select * from emp where salary > (select salary from emp where last_name like 'abel');
--返回job_id和141号员工相同，salary比143号员工多的员工信息--
select * from emp where job_id=(select job_id from emp where id=141) and salary >(select salary from emp where id=143);
--返回公司工资最少的员工的last_name、job_id和salary--
select last_name,job_id,salary from emp where salary <= (select min(salary) from emp );
--查询最低工资大于50号部门最低工资的部门ID和其最低工资--
#①查询50号部门最低工资
select min(salary) from emp where dep_id=50
#②查询每个部门的最低工资
select min(salary),dep_id from emp group by dep_id;
#③在②基础上筛选，满足min(salary)>①
select min(salary),dep_id from emp group by dep_id having min(salary)>(select min(salary) from emp where dep_id=50);

#2.列子查询（多行子查询）
--返回location_id是1400或者1700的部门中的所有员工姓名--
#①查询location_id是1400或者1700的部门ID
select dep_id from emp where loction_id in (1400,1700);
#②查询员工姓名，满足部门号是①中的
select last_name from emp where dep_id in (select distinct dep_id from emp where loction_id in (1400,1700))
--返回其他部门中比job_id为‘it_prog’部门任意工资低的员工的：工号、姓名、salary--
#①查询ob_id为‘it_prog’的部门的工资
select distinct salary from emp where job_id like 'it_prog';
#②查询最终结果
select id,last_name,salary from emp where salary < any(select distinct salary from emp where job_id like 'it_prog') and job_id <> 'it_prog';
#上面的也可以写成：
select id,last_name,salary from emp where salary < (select distinct max(salary) from emp where job_id like 'it_prog') and job_id <> 'it_prog';

#3.行值查询（一行多列或者多行多列）
--查询员工编号最小且工资最高的员工信息--
--上面知识完成--
#①查询最小员工编号
select min(id) from emp;
#②查询最高工资
select max(salary) from emp;
#③ 查询员工信息
select * from emp where id=(select min(id) from emp) and salary=(select max(salary) from emp);
--行查询完成--
select * from emp where (id,salary)=(select min(id),max(salary) from emp);

/*
select 后面   只支持标量子查询
*/
--查询每个部门的所有信息+员工的个数--
select d.*,(select count(*) from emp e where e.dep_id=d.id) as '员工个数' from deps d;
--查询员工号=102的部门名称--
select d.dep_name,(select dep_id from emp e where id=102) as "指定部门编号" from deps d where d.dep_id = "指定部门编号";
select (select dep_name from deps d inner join emp e on d.dep_id=e.dep_id where e.id=102) as '部门名'

/*
from 后面
*/
--查询每个部门的平均工资的工资等级--
#①查询每个部门的平均工资
select avg(salary) from emp group by dep_id;
#②连接①的结果集和job_grades表，添加筛选条件
select avg.*,j.grades from (select avg(salary) as av,dep_id from emp group by dep_id) avg inner join jobs j on avg.av between lowest and highest;

/*
exists 后面 （相关子查询）
	语法：exists（完整的查询语句） 效果：判断查询的结果是否有值  结果：1|0
*/
--查询有员工的部门名--
select dep_name from deps d where exists(select * from emp e where e.dep_id=d.dep_id);
--使用in完成上面的功能--
select dep_name from deps d where d.dep_id in (select dep_id from emp);
```



