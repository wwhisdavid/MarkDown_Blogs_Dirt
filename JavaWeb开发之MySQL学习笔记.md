title: JavaWeb开发之MySQL学习笔记
date: 2015-12-30 11:44:04
categories: 码农之路
tags: [Java Web Dev]
---

> 毕设的题目定了，要做一个简单的web应用。刚好最近在学习Java Web的知识，所学即可用，实在让我欣喜。目前的进度是学习完了基本的基于JSP和Servlet的应用搭建。这俩天在学MySQL，所以今天把学习成果做下笔记，以后忘了也可以看看。特别感谢传智播客的徐仕峰老师的讲解，视频讲得很好。

> 本文就不再赘述MySQL数据库的优缺点和安装了，这些网上很多资料。主要介绍下基本使用。
> 
> 本文MarkDown格式有点问题，暂时在解决中...

## Overview
1. 从最基本的CRUD讲起
2. 灵活的数据约束
3. 数据库设计原则
4. 关联查询
5. 函数式操作--存储过程
6. 我是一个观察者--触发器
7. 权限与备份

### 一、从最基本的CRUD讲起

> 所谓CRUD即 增（Create)/查(Retrieve)/改(Update)/删(Delete) 这四种数据库的操作，我们的使用数据库也是基于这四个操作进行的。

> 以下操作默认在已经安装MySQL的命令行。

> SQL语句不区分大小写，一般大写。

#### 1.1 数据库的CRUD
> 要使用MySQL，需要在电脑上启动其服务器。Mac可以在系统偏好设置中选择MySQL打开服务器。

* 登陆数据库：打开命令行，首先，我们需要使用账号登陆MySQL，对应的账号有对应的权限，我们在安装MySQL时已经注册了账号，这是个最大权限的root账号，这里使用它登陆即可。
		
        mysql -u 账号 -p // 输入完后继续输入密码即可进入MySQL

*  查看数据库：

		show databases; // 可以查看数据库，结果如下：
							（我们看到，这里系统默认帮我们创建了几个数据库）
		+--------------------+
		| Database           |		
		+--------------------+
		| information_schema |     // MySQL的元数据和基础数据
		| mysql              |		// 存储登陆账号的相关权限信息
		| performance_schema |		// MySQL的运行数据和日志信息
		| test               |		// 测试用，空
		+--------------------+
		
* 创建数据库：
		
		create database student; // 创建一个student数据库
		create database teacher default character set utf8; // 创建一个teacher数据库，默认采用utf8编码的默认字符集。
		show create database teacher; // 可以查看数据库默认字符集。
* 删除数据库：

		drop database student;
* 修改数据库：
	
		alter database 数据库名 属性 值; 
		//示例
		alter database teacher default character set gbk; // 修改数据库为gbk字符集编码。 
		
#### 1.2 表的CURD
> 表结构存在于数据库中，在操作表前，首先要选中数据库，使用`use 数据库名`选中对应数据库，再进行表操作。

* 查看表：
		
		show tables; // 显示数据库的表，这里我创建了一个school数据库use了，这个语句后，提示 Empty set (0.08 sec)，说明数据库为空。

* 创建表：
		
		create table 表名 (字段1 类型, 字段2 类型, ····);
		
		示例： 
		mysql> create table student( // 表名为student
    	-> sid int,					  // 学号字段sid，类型为整形
    	-> sname varchar(20),		  // 姓名字段sname，类型为字符，长度20字节 
    	-> sage int
    	-> );
    	// 这里的格式建议如上，每次写字段或者其他单一操作就换行，结构比较清晰，MySQL语句会识别对应的';'作为语句的结尾。
    	
    	// 这里再回来看看这时候show tables的结果。
    	+------------------+
		| Tables_in_school |
		+------------------+
		| student          |
		+------------------+
		
* 查看表具体结构：

		desc student;
		
		+-------+-------------+------+-----+---------+-------+
		| Field | Type        | Null | Key | Default | Extra |
		+-------+-------------+------+-----+---------+-------+
		| sid   | int(11)     | YES  |     | NULL    |       |
		| sname | varchar(20) | YES  |     | NULL    |       |
		| sage  | int(11)     | YES  |     | NULL    |       |
		+-------+-------------+------+-----+---------+-------+
		
* 删除表：

		drop table student;
		
* 修改表：（以下操作的`column`均可省略）

		// 添加字段
		alter table student add column sgender varchar(2); // 为student表添加一个字符型字段sgender。
		
		// 删除字段 
		alter table student drop column sgender; // 删除表的sgender字段。
		
		// 修改字段类型
		 alter table student modify column sname varchar(100); // 修改表的sname字段类型为 varchar(100)。
		 
		// 修改字段名称
		alter table student change column sgender gender varchar(2); // 修改表的sgenser字段为gender。
		
		// 修改表名称
		alter table student rename to teacher; // 修改student表的名字为teacher。
		
#### 1.3 数据的CURD

* 增加数据：

		// 插入所有字段。一定依次按顺序插入,注意不能少或多字段值.		INSERT INTO student VALUES(1,'David',20); // 插入了一条数据，分别对应sid、sname、sage。		// 插入部分字段		INSERT INTO student(sid,sname) VALUES(2,'Jack');
 
* 删除数据

		// 删除所有数据（建议少用）		DELETE FROM student;		// 带条件的删除(推荐使用)		DELETE FROM student WHERE id=2;		// 另一种方式
		TRUNCATE TABLE student;
		
		// DELETE 和 TRUNCATE比较		// delete from: 可以全表删除:
		// 1)可以带条件删除  
		// 2）只能删除表的数据，不能删除表的约束     
		// 3)使用delete from删除的数据可以回滚（事务）		
		// truncate table: 可以全表删除
		// 1）不能带条件删除 
		// 2）即可以删除表的数据，也可以删除表的约束 
		// 3）使用truncate table删除的数据不能回滚
		
* 查询数据

		// 1.查询所有列(字段)
		SELECT * FROM student;		// 2.查询指定列
		SELECT sid,sname FROM student; // 查询所有学生的学号和姓名。
		
		// 3.查询时添加常量列
		SELECT sid,sname,sage,'是' AS '是否应届生'  FROM student; // 查询时为表添加一个字段‘是否应届生’,其值均为常量‘是’。
		
		// 4.查询时合并列
		SELECT sid,sname,(math+chinese) AS '总成绩' FROM student; // 查询时为表添加字段‘总成绩’，其值由math（数学成绩）加上chinese(语文成绩)。
		// 注意：以上合并的math和chinese必须都是数值类型。
		
		// 5.查询时去除重复记录(DISTINCT)
		SELECT DISTINCT sgender FROM student; // sgender字段重复的数据不会显示。
		SELECT DISTINCT(sgender) FROM student; // 等价以上
		
		// 6.条件查询
		
		// 6.1 逻辑条件：and(与)、or(或)
		SELECT * FROM student WHERE sid=2 AND sname='David'; // 查找学号为2且名字为David的学生。
		SELECT * FROM student WHERE sid=2 OR sname='David'; // 查找学号为2或名字为David的学生。
	
		// 6.2 比较条件：>、<、>=、<=、=、<>(不等于)、between and (等价于>= 且 <=)
		SELECT * FROM student WHERE math>60; // 查找数学成绩大于60的学生。
		SELECT * FROM student WHERE math BETWEEN 75 AND 90; // 查找数学成绩大于等于75小于等于90的学生。
		
		// 6.3 判空条件：(null 空字符串)：
			// is null(是否空）
			// is not null(是否不为空)
			// =''(是否是长度0的字符串)
			// <>''(是否不是长度为0的字符串)
			
		// 6.4 模糊条件：like
			// 通常使用以下替换标记：				// % : 表示任意个字符				// _ : 表示一个字符
				
		SELECT * FROM student WHERE sname LIKE '李%'; // 查找姓‘李’的学生。
		SELECT * FROM student WHERE sname LIKE '李_'; // 查找姓为‘李’，名只有一个字的学生。
		
		// 7.聚合查询（使用内置的聚合函数）
		
		// 7.1 SUM(字段)：用于加和
		SELECT SUM(math) AS 'math的总成绩' FROM student; // 所有学生的数学成绩之和。
		
		// 7.2 AVG(字段)：用于平均
		SELECT AVG(math) AS 'math的平均分' FROM student; // 所有学生的数学成绩均值。
		
		// 7.3 MIN(字段)、MAX(字段)：求最值
		SELECT MAX(math) AS '最高分' FROM student;
		SELECT MIN(math) AS '最低分' FROM student;
		
		// 7.4 COUNT(字段)：求数量
		SELECT COUNT(*) FROM student; // 统计各字段最多的一个作为返回 		SELECT COUNT(sid) FROM student; // 指定字段数据量，忽略null
		
		// 8.分页查询
		SELECT * FROM student LIMIT 0,2; // 查询到的数据中选取从第0条开始的俩条数据。
		
		// 9.查询排序
		// 语法 ：order by 字段 asc/desc		// asc: 顺序，正序。数值：递增，字母：自然顺序（a-z）。		// desc: 倒序，反序。数值：递减，字母：自然反序(z-a)。
		
		SELECT * FROM student ORDER BY sid ASC; // 顺序，默认也是这个顺序。
		
		// 10.分组查询
		SELECT sgender,COUNT(*) FROM student GROUP BY sgender; // 将学生按性别分组，并输出对应性别人数。
		
		// 11.分组查询后筛选
		SELECT sgender,COUNT(*) FROM student WHERE GROUP BY sgender HAVING COUNT(*)>2; // 分组查询后的条件查询使用‘HAVING’关键字。
		
### 二、灵活的数据约束

#### 2.1 默认值
* 作用：当用户对使用默认值的字段`不插入`值的时候，就使用默认值。
* 注意： 1）对默认值字段插入null是可以的。2）对默认值字段可以插入非null。

		CREATE TABLE student( // 在创建表的时候指定。		id INT,		name VARCHAR(20),		address VARCHAR(20) DEFAULT '武汉大学'  // DEFAULT指定默认值。		);
		
#### 2.2 非空
* 作用： 限制字段必须赋值* 注意：1）非空字符必须赋值。		2）非空字符不能赋null。
		
		CREATE TABLE student( // 在创建表的时候指定。		id INT,		name VARCHAR(20),		address VARCHAR(20) NOT NULL // 地址不能为空		);	#### 2.3 唯一
* 作用：对字段的值不能重复* 注意：1）唯一字段可以插入null。2）唯一字段可以插入多个null。

		CREATE TABLE student(		id INT UNIQUE, // 指定id唯一		name VARCHAR(20)		);#### 2.4 主键
* 作用：非空+唯一* 注意：1）通常情况下，每张表都会设置一个主键字段。用于标记表中的每条记录的唯一性。		2）建议不要选择表的包含业务含义的字段作为主键，建议给每张表独立设计一个非业务含义的id字段。		CREATE TABLE student(		id INT PRIMARY KEY, // 主键		name VARCHAR(20)		);
		
#### 2.5 自增长
* 作用：自增长字段可以不赋值，自动递增。

		CREATE TABLE student(		id INT(4) ZEROFILL PRIMARY KEY AUTO_INCREMENT, // 自增长，从0开始。ZEROFILL，零填充。		name VARCHAR(20)		);
		
#### 2.6 外键
* 作用：1）约束两种表的数据。2）解决数据冗余高问题：独立出一张表

* 问题描述：见下图，是一个员工表，我们可以看出如果有很多员工都是研发部，那么，每个员工都要存储部门，造成数据冗余，占用较多容量。

![1](https://github.com/wwhisdavid/MD_Pictures/blob/master/MySQL/MySQL1.png?raw=true)

* 为了解决这样一个数据冗余问题，我们可以将这个表拆为两个表，如下：

![2](https://github.com/wwhisdavid/MD_Pictures/blob/master/MySQL/MySQL2.png?raw=true)

这样，员工表的部门全部由部门表的depart_id替代，就解决了以上问题。这里我们称部门表为主表，员工表为副表。主表的主键是副表某一字段的外键。

* 代码：

		// 部门表（主表）		CREATE TABLE dept(		depart_id INT PRIMARY KEY,		depart_name VARCHAR(20)		);		// 修改员工表（副表/从表）		CREATE TABLE employee(		id INT PRIMARY KEY,		name VARCHAR(20),		depart_id INT, // 把部门名称改为部门ID		// 声明一个外键约束		CONSTRAINT emlyee_dept_fk FOREIGN KEY(depart_id) REFERENCES dept(depart_id)			
		//           外键名称                  外键               参考表(参考字段)		);		##### 注意：* 被约束的表称为副表，约束别人的表称为主表，外键设置在副表上的！！！* 主表的参考字段通用为主键！* 添加数据： 先添加主表，再添加副表* 修改数据： 先修改副表，再修改主表* 删除数据： 先删除副表，再删除主表#### 2.7 级联操作

* 问题：当有了外键约束的时候，必须先修改或删除副表中的所有关联数据，才能修改或删除主表！但是，我们希望`直接修改或删除主表数据`，从而影响副表数据。可以使用级联操作实现！！！

		CREATE TABLE employee(		id INT PRIMARY KEY,		name VARCHAR(20),		depart_id INT, // 把部门名称改为部门ID		// 声明一个外键约束		CONSTRAINT emlyee_dept_fk FOREIGN KEY(depart_id) REFERENCES dept(depart_id) ON UPDATE CASCADE ON DELETE CASCADE  // ON CASCADE UPDATE ：级联修改			//          外键名称                  外键               参考表(参考字段)		);		
		注意： 级联操作必须在外键基础上使用

### 三、 数据库设计原则
> 数据库设计:
> 
> 业务模型/实体模型 - > 数据模型 (硬盘)，即，先将各种实例进行抽象，再建立对应的表来描述这些抽象。
#### 三大范式

* 第一范式： 要求表的`每个字段必须是不可分割的独立单元`	id  | name
	------------- | -------------
	1  | 王维恒&David
以上违反第一范式，字段name包含了中文名和英文名，不利于查询。

	正确的是：

	id  | name | englishName
	------------- | ------------- | ----------
	1  | 王维恒 | David* 第二范式： 在第一范式的基础上，要求每张表只表达一个意思。表的每个字段都和表的主键有依赖。					
	id  | name | math | girlfriend
	------------- | ------------- | ---------- | -------
	1  | 王维恒 | 99 | Fang
以上违反第二范式，表及表达了成绩又表达了家人。应该拆成俩个表进行表述。
					      * 第三范式： 在第二范式基础，要求`每张表的主键之外的其他字段都只能和主键有直接决定依赖关系。`（反例参考外键约束）
### 四、关联查询（多表查询）
> 多表查询规则：
> 
> 1）确定查询哪些表   
> 2）确定哪些哪些字段   
> 3）表与表之间连接条件 (规律：连接条件数量是表数量-1)

#### 4.1 交叉连接查询（出现笛卡尔积现象）

		// 对于一个员工表（employee），有员工姓名（empName）；对于一个部门表（dept），有部门名字（deptName）。
		
		SELECT empName,deptName FROM employee,dept; // 若有m个员工、n个部门，则返回m * n的结果。

#### 4.2 内连接查询：只有满足条件的结果才会显示(使用最频繁)

		SELECT empName,deptName       		// 2）确定哪些哪些字段			FROM employee,dept    			// 1）确定查询哪些表			WHERE employee.deptId=dept.id;  // 3）表与表之间连接条件
			// (这里指员工表中的部门id与部门表的部门id相等为连接条件，同时符合这个要求的员工表的empName和部门表的deptName将被选出)			// 内连接的另一种语法		SELECT empName,deptName			FROM employee			INNER JOIN dept			ON employee.deptId=dept.id;
			
#### 4.2 左[外]连接查询： 使用左边表的数据去匹配右边表的数据，如果符合连接条件的结果则显示，如果不符合连接条件则显示null		// 注意： 左外连接：左表的数据一定会完成显示！		SELECT d.deptName,e.empName // 这里的e、d为对应表的别名。			FROM dept d			LEFT OUTER JOIN employee e			ON d.id=e.deptId;#### 4.3 右外连接：定义和用法大体同上。

#### 4.4 自连接查询：在本表中进行匹配。

		SELECT e.empName,b.empName // 查询员工和上级的对应数据			FROM employee e 			LEFT OUTER JOIN employee b			ON e.bossId=b.id;
			

### 五、 函数式操作--存储过程

#### 5.1 什么是存储过程> 存储过程，就是带有逻辑的SQL语句。即可以对SQL进行一些逻辑操作和函数封装。

#### 5.2 存储过程特点* 执行效率非常快！存储过程是在数据库的服务器端执行的！！！
* 移植性很差！不同数据库的存储过程是不能移植。

#### 5.3 存储过程语法

* 创建存储过程

		DELIMITER $       // 声明存储过程的结束符		CREATE PROCEDURE pro_test() // 存储过程名称(参数列表)		BEGIN             // 开始			// 可以写多个sql语句;  			SELECT * FROM employee; // sql语句+流程控制		END $            // 结束 结束符

* 执行存储过程		
		// CALL 存储过程名称(参数);
		CALL pro_test();  
		
		参数：			IN：  表示输入参数，可以携带数据带存储过程中			OUT： 表示输出参数，可以从存储过程中返回结果			INOUT： 表示输入输出参数，既可以输入功能，也可以输出功能
			
* 删除存储过程		
		DROP PROCEDURE pro_test;* 存储过程示例

	1. 带输入参数：
		
			DELIMITER $			CREATE PROCEDURE pro_findById(IN eid INT)  // IN: 输入参数			BEGIN				SELECT * FROM employee WHERE id=eid;			END $ 
		
			// 调用
			CALL pro_findById(4);
			
	2. 带有输出参数的存储过程

			DELIMITER $			CREATE PROCEDURE pro_testOut(OUT str VARCHAR(20))  // OUT：输出参数			BEGIN				SET str='hellojava'; // 给参数赋值			END $
			
	3. MySQL的变量：这里在讲带输出参数的存储过程前，我们先来介绍下MySQL中的变量，明显，带有输出参数的函数需要一个变量来接收。这就要用到MySQL的变量了。
					    1.全局变量（内置变量）：mysql数据库内置的变量 （所有连接都起作用）
		    	 查看所有全局变量： show variables				 查看某个全局变量： select @@变量名
				 修改全局变量： set 变量名=新值	        	 例如：
	        	 	 character_set_client: mysql服务器的接收数据的编码        			 character_set_results：mysql服务器输出数据的编码        			2. 会话变量： 只存在于当前客户端与数据库服务器端的一次连接当中。如果连接断开，那么会话变量全部丢失！        		定义会话变量: set @变量=值        		查看会话变量： select @变量        			3. 局部变量： 在存储过程中使用的变量就叫局部变量。只要存储过程执行完毕，局部变量就丢失！！
			
			4. 示例：
			 	1)定义一个会话变量name
			 	2)使用name会话变量接收存储过程的返回值
					CALL pro_testOut(@name);
					SELECT @name; // 查看变量值

	4. 带有输入输出参数的存储过程			
			DELIMITER $			CREATE PROCEDURE pro_testInOut(INOUT n INT)  -- INOUT： 输入输出参数			BEGIN
  	 			SELECT n; // 查看变量   				SET n =500;			END $			SET @n=10;			CALL pro_testInOut(@n);  // 调用			SELECT @n; // n=500
	5. 带有条件判断的存储过程
	
			DELIMITER $			CREATE PROCEDURE pro_testIf(IN num INT,OUT str VARCHAR(20))			BEGIN				IF num=1 THEN					SET str='星期一';				ELSEIF num=2 THEN					SET str='星期二';				ELSEIF num=3 THEN					SET str='星期三';				ELSE					SET str='输入错误';				END IF;			END $			CALL pro_testIf(4,@str); 			SELECT @str;
 	6. 带有循环功能的存储过程
 	
 			DELIMITER $			CREATE PROCEDURE pro_testWhile(IN num INT,OUT result INT)			BEGIN				DECLARE i INT DEFAULT 1; // 定义一个局部变量，类似于for循环的i				DECLARE vsum INT DEFAULT 0;				WHILE i<=num DO	      			SET vsum = vsum+i;	      			SET i=i+1;				END WHILE;				SET result=vsum;			END $			CALL pro_testWhile(100,@result);			SELECT @result;	7. 使用查询的结果赋值给变量（INTO）
			
			DELIMITER $			CREATE PROCEDURE pro_findById2(IN eid INT,OUT vname VARCHAR(20) )			BEGIN				SELECT empName INTO vname FROM employee WHERE id=eid; // 把查询结果赋给vname传出。			END $			CALL pro_findById2(1,@NAME);			SELECT @NAME;
			
### 六、我是一个观察者--触发器

#### 6.1 触发器的作用
> 当操作了某张表时，希望同时触发一些动作/行为，可以使用触发器完成！！

#### 6.2 使用

* 创建触发器(添加)			
		// 触发器名：tri_empAdd         
		// 指定触发动作(CRUD均可)：AFTER INSERT/DELETE/UPDATE
		// 指定触发时机（每行）: FOR EACH ROW
		CREATE TRIGGER tri_empAdd AFTER INSERT ON employee FOR EACH ROW 
			INSERT INTO test_log(content) VALUES('员工表插入了一条记录');  // 这里是触发器触发后行为。模拟一个日志操作。
		
### 七、权限与备份

> MySQL数据库权限问题：root（即一开始创建的账户）拥有所有权限（可以干任何事情）。>
>  权限账户，只拥有部分权限（CURD）例如，只能操作某个数据库的某张表。

#### 7.1 数据库用户权限管理和密码修改

* 权限管理：前面我们提到，在我们使用MySQL时，系统自动生成了几个数据库，其中有一个MySQL就是用于管理数据库用户的。

		在命令行进入MySQL后。
		use mysql; 
		SELECT * FROM user;
		我们可以看到一堆乱码，其实如果你有专门的数据库查看软件，就可以看到，这里有一条记录。大致如下：
	
Host  | User  |  Password | ·····
------------- | ------------- | -------- |---------
localhost  | root | 28wos988s1928hsh192g7e9s.. | ·····		
		以上这条就是我们的root账号在数据库中存在的具体表达啦，这里指明了访问的host、用户名、密码、各类操作权限等。这里说明下，这个密码是经过单向MD5加密的结果。
		操作这个表，我们就能够管理用户了。
		
* 分配权限账户：
		GRANT SELECT ON company.employee TO 'David'@'localhost' IDENTIFIED BY '123456';
		// 为用户名David、地址localhost的用户在company数据库下的employee表中赋予了SELECT权限，这里还需给出对应账号密码’123456‘。		
		
* 密码修改：
	
		// 在上表下，进行下面操作，这里对新密码需要调用内置函数PASSWORD()进行MD5加密。
		UPDATE USER SET PASSWORD=PASSWORD('123456') WHERE USER='root'; // 密码改为’123456‘。

#### 7.2 数据库备份

* 备份

		mysqldump -u root -p 数据库名 > 备份路径;
	
* 恢复

		mysql -u root -p 现在存在的数据库名 < 存在的备份路径;		 
## 最后
> 如果发现本文有误，请联系我[wwhisdavid@163.com](mailto:wwhisdavid@163.com)，谢谢支持。