title: iOS数据库开发封装、设计与实战
date: 2015-12-16 22:17:14
categories: 码农之路
tags: [iOS Dev]
---

>在我们平常的项目中，经常会碰到数据本地持久化的问题，一般情况，我们是直接调Cocoa的API以普通文件形式进行本地存储。这样的做法对于一般的数据是没有问题的，但是，当我们遇到量比较大的数据时，这种做法将会产生大量IO操作，这是非常影响性能的。并且，这样的做法查询数据十分繁琐，对需要频繁操作数据的开发者十分不友好。所以，当我们需要在应用中频繁操作数据时，我们应该另辟蹊径。这里我选择讲讲数据库和使用设计模式对数据库框架进行封装。

## Overview
1. iOS上轻量级数据库介绍：SQLite3、LevelDB
2. iOS数据库开发神器：FMDB
3. 工厂设计模式简介
4. 对FMDB二次封装并带入项目实战：GRDB
5. SQLite3本地调试
6. 性能讨论



-------
## 一、iOS上轻量级数据库介绍：SQLite3、LevelDB
### 1.1 SQLite3简介
    	SQLite是一个非常轻量级自包含(lightweight and self-contained)的DBMS，它可移植性好，很容易使用，很小，高效而且可靠。SQLite嵌入到使用它的应用程序中，它们共用相同的进程空间，而不是单独的一个进程。从外部看，它并不像一个RDBMS，但在进程内部，它却是完整的，自包含的数据库引擎。
    	嵌入式数据库的一大好处就是在你的程序内部不需要网络配置，也不需要管理。因为客户端和服务器在同一进程空间运行。SQLite 的数据库权限只依赖于文件系统，没有用户帐户的概念。SQLite 有数据库级锁定，没有网络服务器。它需要的内存，其它开销很小，适合用于嵌入式设备。你需要做的仅仅是把它正确的编译到你的程序。
    基于这样的考虑，在可移动终端上，我们选用数据库一般就是SQLite3。
    
### 1.2 在iOS开发中使用SQLite3
1. 在项目中导入包`libsqlite3`(Xcode已自带，但需自己导入)。
2. 在需要使用数据库的类中导入头文件`#import "sqlite3.h"`
3. 实例代码
	
		- (void)testSQL
       {
      		NSArray *paths = NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES);
   			NSString *documentsDirectory = [paths objectAtIndex:0];
    		NSString *path = [documentsDirectory stringByAppendingPathComponent:@"MyDB.sqlite3"]; // 声明数据库路径
    
    		sqlite3 *db; // 声明数据库
    		int status = sqlite3_open(path.UTF8String, &db); // 打开数据库
    		if (status == SQLITE_OK) { // 表示打开成功
        		const char *sql = "CREATE TABLE IF NOT EXISTS MYTABLE (ID INTEGER PRIMARY KEY AUTOINCREMENT, DATA TEXT)"; // 编辑sql代码，这是一个创表操作。
        		char *errorMsg = NULL;
        		sqlite3_exec(db, sql, NULL, NULL, &errorMsg); // 执行非查询语句
        		if(errorMsg){
            
        		}
        		else{
        	
      	    	}
    		}
    
    		const char *query = "SELECT ID, DATA FROM FIELDS ORDER BY ROW"; // 查询操作语句（插入相同）
    
    		sqlite3_stmt *statement;
    
    		int status2 = sqlite3_prepare_v2(db, query, -1, &statement, NULL); // 执行查询操作，statement中包含查询结果
    		if(status2 == SQLITE_OK)
    		{
        		while (sqlite3_step(statement) == SQLITE_ROW) {
            		const unsigned char *data = sqlite3_column_text(statement, 1); // 执行一次，移动一次step指针 。得到第一列的第一个结果数据。
        		}
        		sqlite3_finalize(statement);
    		}
    
    		//关闭数据库
    		sqlite3_close(db);
    
		} 
  
### 1.3 LevelDB简介
		对于LevelDB，其实我也没用过，但是早闻大名。这是google开源的一个轻量级数据库。使用了key-value键值对的方式查询和插入，这种方式再配合上了树结构，使它有很高的效率。此外，这是个跨平台的库，OC平台也有封装。
		举个例子，我们在需要查询一个县，那么LevelDB会维护一棵树，root是中国，则会根据省、市、县一个个节点进行查询查到这个县。但是，很明显它有个缺点，就是它的快速查询是建立在对key的了如指掌（就像我们在使用二叉树查询时，对二叉树的结构有特定的规则，所以查询时可以达到O(logn)的效率）。

### 参考链接：[sqlite3基本使用](http://blog.csdn.net/xingxing513234072/article/details/24426307)、 [LevelDB基本介绍与使用](http://www.tanhao.me/pieces/1397.html/)

## 二、iOS数据库开发神器：FMDB

### 2.1 FMDB简介
		FMDB以OC的方式封装了SQLite的C语言API，大大提高开发数据库效率。相对于Core Data,其具有更高的效率更轻量级，相对于SQLite3，使用起来更加面向对象，省去了很多麻烦、冗余的C语言代码。还有特别好的一点是提供了多线程安全的数据库操作方法，有效地防止数据混乱（数据库的原子性操作是数据安全的核心）。
		
### 2.2 核心类介绍与基本使用

#### 2.2.1 FMDatabase
	这是对SQLite3数据进行抽象的类。一个对象代表一个数据库。使用姿势如下：
	
	    FMDatabase *db = [[FMDatabase alloc] initWithPath:path]; // 创建数据库
	    [db open]; // 打开

#### 2.2.2 FMResultSet
	表示使用FMDatabase执行查询后的结果集,类似于前面的sqlite3_stmt。 更删改查姿势如下：
	
		FMResultSet *set = [db executeQuery:query]; // 查询
    	[db executeUpdate:sql]; // 除查询外操作
    	[db executeUpdateWithFormat:@"%@", ....]; // 带format查询
    	
    	while(set.next){
    		NSString *name = [set stringForColumn:@"查询字段"]; // 其他详细用法可以看头文件，讲得很详细
    	}
	 
#### 2.2.3 FMDatabaseQueue
	一个队列类，每个操作放于队列中一次执行，保证了线程安全。
	
		FMDatabaseQueue *queue = [FMDatabaseQueue databaseQueueWithPath:dbPath];
		[queue inDatabase:^(FMDatabase *db){
			// 操作在此进行
		}
		
### 参考链接：[FMDB简单使用](http://www.cnblogs.com/wendingding/p/3871848.html)


## 三、 工厂设计模式简介

### 3.1 为什么还要封装？

* 上面我已经看到了，经过FMDB的封装，数据库操作已经非常友好，我们可以快速简单地进行操作。可是，FMDB只提供了比较基本的封装，要真正在项目中使用还是远远不够的，我想没有人会喜欢频繁地写SQL语句，虽然语法简单但使用起来也是又长又臭。基于编程核心：懒！我们需要对FMDB进行二次封装，使其能够更加简便得使用，更加面向对象。

* 我们进行框架设计并不是空穴来风，很重要的几点是使用方便、可拓展性强、高内聚低耦合。在我们没有经验的前提下，我们可以借鉴一些经典的设计模式。这里，我们来看看，我们需要封装一个怎样的数据库（目的驱动需求）。

* 首先，对SQL语句再次封装，不再显式调用而是通过API执行操作。

* 接着，考虑到需求，我们可能需要在一个数据库中建立多个表来存储不同类目的数据，每次创建表和使用表应该尽量简洁并不对外暴露实现细节。

* 考虑的需求的多变，我们的数据库表结构应该是多变的，表的增加和减少应该是灵活的，这就要求我们的框架易于拓展。

* 综合以上，我选择使用工厂模式。

### 3.2 什么是工厂模式

* 为什么要使用工厂模式呢？首先我们先来介绍下它。

* 先看看下图：
![Factory](https://github.com/wwhisdavid/MD_Pictures/blob/master/FactoryMethod/FactoryMethod.png?raw=true)

* 这样有什么好处呢？

* 首先，客户端只与工厂进行交互，而不用考虑工厂背后的实现。只要告诉工厂需要什么，工厂会调用子工厂进行生产。对外界屏蔽了创建对象的细节。

* 其次，面向接口编程。对于对象的具体功能，可以在对应的协议中进行规定，统一修改。

* 增加新的具体工厂和产品族很方便，无须修改已有系统，符合“开闭原则”。

* 最重要的是，统一管理对象的创建，当对象的种类增多时，有比较好的代码可读性。

* 这里，我们可能还看不出工厂模式在应用中的价值，可能你更加习惯直接指定你需要创建的类和对象。但是当你需要创建一群相似类的对象时，在维护时，你可能需要不断在各类头文件中跳来跳去，而使用工厂后，你只要在抽象工厂中排查各个工厂的实现细节。

* 下面，我们开始讲讲在数据库中实现这样的思路来展现工厂的魅力。



## 四、 对FMDB二次封装并带入项目实战：GRDB

### 4.1 前言
* 首先，我对FMDB的封装是基于我的一位前辈的框架[GreedDB](https://github.com/greedlab/GreedDB)。大家可以去git下载，多多星星，或者进入我的github主页，进入Organizations也可以看到我们小队的几个项目。

### 4.2 封装思路与原理

* 本节的DEMO地址 : [https://github.com/wwhisdavid/DBDemo](https://github.com/wwhisdavid/DBDemo)

* 下面还是先来看张图：

![demo](https://github.com/wwhisdavid/MD_Pictures/blob/master/FactoryMethod/DBDemo.png?raw=true)

* 什么意思呢？

* 首先是BaseDao，这是一个为数据库设计的基本协议，用于实现数据库对于表的通用操作。具体方法见协议。

* 接着，分别有JsonDao和JsonFilterDao两个基础自BaseDao的协议。分别指定数据库对于两种类型的表的操作细节。注意，这一层其实相当于前面我们讲工厂时对于各个工厂的细节约束。

* 再下来，有两个队列，可以想到，这两个队列充当的角色就是具体执行数据库的操作方，对于其公共父类已经做了FMDatabaseQueue的封装。这两个队列就相当于具体来实现功能的工厂。

* 左边最上则是我们的抽象工厂了，这里我们发现，抽象工厂与子工厂中还隔了点东西。这里使用了OC特有的分类，让工厂的拓展性淋漓尽致，做到了真正的非侵入式拓展数据库子工厂。为抽象工厂的Category添加一些构造方法，创建遵守了对应数据库协议的队列，让队列与表名一一对应进行管理。将表的管理归结到了对队列的管理，并且由工厂统一派发。
 
* 如果你还是觉得云里雾里（我的表述有问题）。来看看workflow。
	
		 // 工厂调用分类方法，获取DemoDao对应的队列JsonQueue，接着调用在对应协议中声明的队列操作方法，保存模型到数据库的表中。具体实现，大家可以在队列中和对应协议中看到，不做赘述。
		 [[DaoFactory getDemoDao] saveWithModel:model];
	
		// 前段同上，获取到ObjectDao对应的队列（可以认为是表）。接着直接通过key就可以取出对应的模型对应的字典。
		 [[DaoFactory getObjectDao] getDictByKey:key];
		
* 这里，我知道你不禁好奇，存储的机制是什么呢？这里需要说明，该框架是可以做到存储基本数据类型和自定义对象的。因为这里的存储机制是依靠一个类GRDatabaseDefaultModel来实现的。我们将这个需要存储对象的标识符作为唯一主键（key）赋予GRDatabaseDefaultModel对象的key属性。将这个对象的值通过操作转换为JsonString作为value存进了对应的value属性。等于是讲，需要存储的数据进行一次模型封装。

* 这样的做法基于两点考虑：

  1. 更加面向对象。
  2. 使用对象作为存储中介提高了对数据约束的拓展性。比如，我们需要为表加一个字段做约束，这个字段可能是uid来约束这些数据指展示给对应的用户，那么我们可以为这个model类添加一个字段filter，在对应的创建表操作中稍做修改即可。此外，如果需要表示数据的排序（存储需要排序的数据），我们还可以增加一个字段sort来指定升降序。这些实现已经加在了这个类中。

* 实例代码来一发：
 		
 		GRDatabaseDefaultModel *model = [[GRDatabaseDefaultModel alloc] init];
    	for (int i = 0; i < 10; i ++) {
       		NSDictionary *tempDict = [self getRandomDict]; // 模拟一个字典
        	model.value = tempDict; 
        	model.key = [NSNumber numberWithInt:i];
        	// filter 和 sort 可以暂时不考虑
        	[[DaoFactory getDemoDao] saveWithModel:model];
    	}


* 或许你还不理解这个机制的简介与强大，我们来再看看一个表：

	key | value | filter | sort
------------ | ------------- | ------------  	| ----------
1 | JsonString  | guoran  | 1  
2 | JsonString  | jieshen 	| 1 	

* 以上就是这个数据库表的实例结构了。我们在存储时只关心key-value和其他字段。对于value的真正值我们不需要关心。（当然，那个model的解析自然会有框架去做啦）

### 4.3 实战演练

* 接下来我手把手教一发。

* 比如现在你要做一个学生管理系统。学生信息有学号和姓名和年纪。你需要存储带本地数据库中。
	1. 在`DaoFactory+NPN`中添加方法 `+ (id<JsonDao>)getStudentDao`。在对于.m文件中实现方法：
		
		
			+ (id<JsonDao>)getObjectDao
			{
    			static id<JsonDao> dao = nil;
    			static dispatch_once_t onceToken;
    			dispatch_once(&onceToken, ^{
    		   		dao = [[JsonFMDBQueue alloc] initWithTableName:@"Student"];
   		 		});
   		 		return dao;
			}
			
			// 这样，当你在对应的地方
			Student *s = [Student new];
			s.name = tianming;
			s.age = 25;
			s.id = 110;
			
			GRDatabaseDefaultModel *model = [[GRDatabaseDefaultModel alloc] init];
			model.key = s.id;
			model.value = s;
			
			[[DaoFactory getStudentDao] saveWithModel:model];
			// 通过这步，你就完成了向一张名为Student_json的表中插入了一个学生对象，model的key是唯一的学号，value学生真是信息的JsonString。
			
	2. 如果你需要取数据。在对应的地方：
		
			NSDictionary *dict = [[DaoFactory getStudentDao] getDictByKey:key];
			// 即可取出对应学生对应字典（如果要直接取出对应对象，还需要封装）
			
* `更多使用方法可以参考我的demo或者框架的demo。（我和框架的思路不太一样，我为了讲工厂方法所以这样封装，其实用它本来的demo做法也是可以的，读者可自行选择使用策略，个人偏向自己的做法。）`

## 五、 SQLite3本地调试

* 上面说了怎么在iOS中使用数据库。现在说说怎么调试数据库。在我们使用数据中过程中，我们会遇见很多坑，比如，一句sql语句的语法错误，你可能很难定位。这里框架已经做了处理，由于SQLite3在错误时会抛出错误信息，所以可以在对应的地方打印。

* 更多的时候，我们会遇见表中数据不符，或者我们也想看看数据库中的内容，这时候怎么办呢？

* 幸运的是mac内置了对于SQLite3的操作命令。下面来简单介绍下几个常用的调试。

		sqlite3 DBName // 在数据库目录下使用，如果存在该数据库，就进入，不存在就创建。
		
		ps1：在sqlite3中的操作命令基本以 . 开头。
		ps2：可以在命令行直接写sql语句操作表。（如果这块忘了或比较薄弱可以看看SQL语法，比较简单）
		
		.exit // 退出数据库
		.help //查看帮助,针对命令
		.database 显示数据库信息,包含当前数据库的位置
		.tables 或者 .table 显示数据库所有表名称,没有表则不显示
		.schema 命令可以查看创建数据对象时的SQL命令
		.schema 表名 查看表字段
		
		select * from 表名; // 查看该表所以数据
		...省略，太多啦
		
#### 参考链接：[SQLite3语法](http://blog.csdn.net/linchunhua/article/details/7184439)
			
			
## 六、 性能讨论

* 说到数据库的性能，其实对于客户端与服务端是没有可比性的。服务端的数据需要面对的高并发和sql注入等问题是客户端基本不会碰到的。所以我们这里谈性能，并不是说数据库本身的负载性能和查询效率。我们讨论的是使用数据库的客户端程序的性能问题。

* 那么这个问题究竟是什么问题呢？

* 细心的同学可能已经想到了，就是IO问题。对于移动端这点可怜的内存，大量数据库操作带来的IO会有一定压力的。当然，我们平常是碰不到什么大数据频繁存储的APP，毕竟我们自己的APP也是没有怎么用数据库的。

* 但是还是想和大家讨论下这个问题，数据库在持久化方面毕竟有它的好处：约束灵活、操作数据灵活等。

* 关于怎么减少IO对客户端的性能影响，很多人讨论过，不阻塞主线程是基本原则。其他的话，我也想到暂时只能从存取数据的流程上和存储形式上进行优化。这是大家比较容易想到的，如果有其他想法欢迎和我联系[wwhisdavid@163.com](mailto:wwhisdavid@163.com)。

* demo其实写了一半，还有关于多filter的队列还没有时间写，其实JsonQueue已经能满足大部分需求了，如果对相关项目有任何疑问或者bug欢迎联系。

* 特别感谢GreedDB作者`Bell`曾经的教导。附上他的邮箱[bell@greedlab.com]()。