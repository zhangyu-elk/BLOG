# 多线程下的Sqlite(一)
> 最近在多线程并发执行的情况下, 碰到了`database is lock`导致操作被驳回 

## Sqlite多线程安全吗?
> 在编译时可以设置线程模式, 可以编译为线程安全的

## Sqlite线程模式
> Sqlite的线程模式是在编译时确定的, 通过编译参数设置

* 1 `-DSQLITE_THREADSAFE = 0`: 单线程模式, 所有互斥锁均被禁用多线程使用不安全
* 2 `-DSQLITE_THREADSAFE = 2`: 多线程模式, 只要一个数据库连接不被多个线程同时使用就是安全的
* 3 `-DSQLITE_THREADSAFE = 1`: 串行模式, 保证线程安全

**锁**:`bCoreMutex`和`bFullMutex`, 多线程模式仅开启`bCoreMutex`

&emsp;&emsp;可以使用`int sqlite3_threadsafe(void);`获取线程模式, 模式模式是序列化

## 后两种模式之间可以进行设置
> 如果设置为多线程模式或者串行模式, 那么可以通过`sqlite3_config`或者`sqlite3_open_v2`指定线程模式(两者之1)

```
int sqlite3_config(int, ...);
```
`sqlite`的全局设定, 请注意线程安全; 设置线程模式的参数`SQLITE_CONFIG_SINGLETHREAD`,`SQLITE_CONFIG_MULTITHREAD`,`SQLITE_CONFIG_SERIALIZED`, 注意最好在程序开头执行, 应当在数据库初始化之前执行.

```
int sqlite3_open_v2(
  const char *filename,   /* Database filename (UTF-8) */
  sqlite3 **ppDb,         /* OUT: SQLite db handle */
  int flags,              /* Flags */
  const char *zVfs        /* Name of VFS module to use */
);
```
第三个参数, `SQLITE_OPEN_NOMUTEX`或者`SQLITE_OPEN_FULLMUTEX`分别执行多线程模式和串行模式


## Sqlite的线程安全
&emsp;&emsp; 设置正确的前提下，多线程同时访问SQLite并不会影响数据库的完整性,而是说每个线程的数据库操作都可以正确执行; 两个线程同时操作数据库的话, 有可能直接返回`database is lock`或者`SQLITE_BUSY`驳回请求
&emsp;&emsp; 我们的实际的要求应当是阻塞等待最后执行, 如果线程模式和使用设置不当的话就很有出现错误

## 解决方案
&emsp;&emsp;序列化模式不在我们考虑范围内
* 1 手动加锁
* 2 建立操作队列, 统一操作数据库

### 锁的机制
* 1 当有写操作时，其他读操作会被驳回
* 2 当有写操作时，其他写操作会被驳回
* 3 当开启事务时，在提交事务之前，其他写操作会被驳回
* 4 当开启事务时，在提交事务之前，其他事务请求会被驳回
* 5 当有读操作时，其他写操作会被驳回
* 6 读操作之间能够并发执行

## 扩展: 多进程
&emsp;&emsp;多进程时应当是采用了文件锁, 能够多进程读取但仅单进程写入, 同时需要注意文件系统的问题.


# 多线程下的Sqlite(二)
> 默认为序列化的多线程模型, 本文将对多线程模式下的一些情况做一些测试

## 函数介绍
```
SQLITE_API int sqlite3_open(
  const char *filename,   /* Database filename (UTF-8) */
  sqlite3 **ppDb          /* OUT: SQLite db handle */
);
```
参数1为文件名, 参数2传入指向一个数据库句柄指针的指针; 打开数据库, 如果不存在会创建

```
typedef int (*sqlite3_callback)(void*,int,char**, char**);
SQLITE_API int sqlite3_exec(
  sqlite3*,                                  /* An open database */
  const char *sql,                           /* SQL to be evaluated */
  int (*callback)(void*,int,char**,char**),  /* Callback function */
  void *,                                    /* 1st argument to callback */
  char **errmsg                              /* Error msg written here */
);
```
执行一条SQL语句, 由`sql`该参数传入
**参数1**: 数据库句柄
**参数2**: `sql`语句
**参数3**: 回调函数(暂不做详细介绍)
**参数4**: 给回调函数传入的参数



## 两个线程使用同一个句柄进行操作
```
#include <stdio.h>
#include "sqlite3/sqlite3.h"
#include <thread>

void func1(sqlite3 *db)
{
	int i = 1;

	const char *insert_sql = "INSERT INTO COMPANY (ID,NAME,AGE,ADDRESS,SALARY) "  \
								"VALUES (1, 'zhangsan', 32, 'China', 100) "; 
	const char *update_sql = "UPDATE COMPANY set SALARY = 200 where ID = 2";

	char *errmsg = NULL;
	while(i < 100)
	{
	    errmsg = NULL;
		sqlite3_exec(db, insert_sql, NULL, 0, &errmsg);
		if(errmsg)
		{
			printf("func1 1 %s\n", errmsg);
		}
		errmsg = NULL;
		sqlite3_exec(db, update_sql, NULL, 0, &errmsg);
		if(errmsg)
		{
			printf("func1 2 %s\n", errmsg);
		}
		i++;
	}	
}

void func2(sqlite3 *db)
{
	int i = 1;

	const char *insert_sql = "INSERT INTO COMPANY (ID,NAME,AGE,ADDRESS,SALARY) "  \
								"VALUES (2, 'zhangsan', 32, 'China', 100) "; 
	const char *update_sql = "UPDATE COMPANY set SALARY = 200 where ID = 1";

	char *errmsg = NULL;
	while(i < 100)
	{
	    errmsg = NULL;
		sqlite3_exec(db, insert_sql, NULL, 0, &errmsg);
		if(errmsg)
		{
			printf("func2 1 %s\n", errmsg);
		}
		errmsg = NULL;
		sqlite3_exec(db, update_sql, NULL, 0, &errmsg);
		if(errmsg)
		{
			printf("func2 2 %s\n", errmsg);
		}
		i++;
	}	
}

int main()
{
	int mode = sqlite3_threadsafe();
	sqlite3_config(SQLITE_CONFIG_MULTITHREAD);

	sqlite3 *db; 
	sqlite3_open("E:/debian32.winsf/safe_sqlite/cmake-build-debug/test", &db);

	const char *create_sql = "CREATE TABLE COMPANY("
         						"ID      		INT 	NOT NULL,"
         						"NAME           TEXT,"
         						"AGE            INT,"
         						"ADDRESS        CHAR(50),"
         						"SALARY         INT);";
    sqlite3_exec(db, create_sql, NULL, 0, NULL);

	std::thread t1(func1, db);
	std::thread t2(func2, db);


	t1.join();
	t2.join();

	sqlite3_close(db);
    system("pause");
    return 0;
}

```

**测试结果**
频繁报出错误: `disk I/O error`或者`database disk image is malformed`
说明数据库结构已经出现异常了


# 多线程下的Sqlite(三)

## 在多线程下每个线程使用各自的句柄
```
#include <stdio.h>
#include "sqlite3/sqlite3.h"
#include <thread>
#include <mutex>

std::mutex mutex;

void func1()
{
	int i = 1;

	sqlite3 *db; 
	sqlite3_open("./testdb", &db);

	const char *insert_sql = "INSERT INTO COMPANY (ID,NAME,AGE,ADDRESS,SALARY) "  \
								"VALUES (1, 'zhangsan', 32, 'China', 100) "; 
	const char *update_sql = "UPDATE COMPANY set SALARY = 200 where ID = 2";

	char *errmsg = NULL;
	while(i < 100)
	{
	    errmsg = NULL;
		sqlite3_exec(db, insert_sql, NULL, 0, &errmsg);
		if(errmsg)
		{
			printf("func1 1 %s\n", errmsg);
		}
		errmsg = NULL;
		sqlite3_exec(db, update_sql, NULL, 0, &errmsg);
		if(errmsg)
		{
			printf("func1 2 %s\n", errmsg);
		}
		i++;
	}	
}

void func2()
{
	int i = 1;

	sqlite3 *db; 
	sqlite3_open("./testdb", &db);

	const char *insert_sql = "INSERT INTO COMPANY (ID,NAME,AGE,ADDRESS,SALARY) "  \
								"VALUES (2, 'zhangsan', 32, 'China', 100) "; 
	const char *update_sql = "UPDATE COMPANY set SALARY = 200 where ID = 1";

	char *errmsg = NULL;
	while(i < 100)
	{
	    errmsg = NULL;
		sqlite3_exec(db, insert_sql, NULL, 0, &errmsg);
		if(errmsg)
		{
			printf("func2 1 %s\n", errmsg);
		}
		errmsg = NULL;
		sqlite3_exec(db, update_sql, NULL, 0, &errmsg);
		if(errmsg)
		{
			printf("func2 2 %s\n", errmsg);
		}
		i++;
	}	
}

int main()
{
    int status = sqlite3_config(SQLITE_CONFIG_MULTITHREAD);
	int mode = sqlite3_threadsafe();

	printf("status: %d mode: %d\n", status, mode);

	sqlite3 *db; 
	sqlite3_open("./testdb", &db);

	const char *create_sql = "CREATE TABLE COMPANY("
         						"ID      		INT 	NOT NULL,"
         						"NAME           TEXT,"
         						"AGE            INT,"
         						"ADDRESS        CHAR(50),"
         						"SALARY         INT);";
    sqlite3_exec(db, create_sql, NULL, 0, NULL);

    sqlite3_close(db);

	std::thread t1(func1);
	std::thread t2(func2);


	t1.join();
	t2.join();

	
    //system("pause");
    return 0;
}

```

**测试结果**:
频繁: `database is locked`错误


## 序列化模型下使用唯一的数据库连接呢?

**测试结果**: 代码差别不大就不贴了,  在序列化模型中使用同一个数据库连接能够保证每条SQL语句正确执行
ps: **多个句柄会出现`database is locked`错误**

## 结论
&emsp;&emsp;`In this mode, SQLite can be safely used by multiple threads provided that no single database connection is used simultaneously in two or more threads.`在多线程模型下官方指明的线程安全指的是应当不会出现`磁盘错误`这种的问题(导致数据库出问题), 而不是说能够保证SQL语句正常执行(会被驳回请求)

&emsp;&emsp;必须明确的一点是: 多线程模式下不应当在不同的线程中使用相同的句柄, 如果不加锁的话甚至可能导致数据库出现问题

&emsp;&emsp;对于序列化来说, 无论怎么样都不会出现 `database disk image is malformed`这样的错误, 但如果在多线程中使用多个数据库句柄还是有可能会被驳回请求

&emsp;&emsp;首先, 如果对性能有要求的话那么不太可能使用序列化模型; 其次,对于多线程来说保证SQL语句执行也是需要加锁的; 个人认为比较好的方式还是使用单线程模型自行加锁吧!



## `bCoreMutex`和`bFullMutex`
> 在多线程模型下仅开启`bCoreMutex`, 那么我们可以猜测一下这两个锁的用途?

`bCoreMutex`: 该锁保证了不同数据库句柄不会同时对数据库修改(多线程模型多连接情况仅仅会被驳回请求)
`bFullMutex`: 保证了单个数据库句柄中不会多个SQL语句同时对数据库修改(序列化模型但数据库连接被多个线程使用仅仅会被驳回请求)



# SQlite事务
> 对于事务的概念想必操作过数据库的人应该都是明白的; 在一个事务中可能存在多个SQL语句, 在最后`COMMIT`时一次性提交, 也可以在出现错误时回滚.
> Sqlite同样支持事务, 且除了`SELECT`以外的所有命令实际都是自动开启了一个新的事务

## 开启事务再进行插入的速度差距？
* 不开启事务, 插入10000条数据, 时间: 94203ms
* 开启事务, 插入数据后一次提交, 时间: 63ms

ps: 代码十分简单, 就不贴上来了
## 结论
&emsp;&emsp;再大并发情况下, 是否开启事务的效率差距非常大

# Sqlite事务锁

## Sqlite事务中锁的状态

* 1 UNLOCKED: 表示数据库并未开启任何读写, 使用`BEGIN`开启一个事务时就是该状态
* 2 SHARED:	  `Sqlite`可以同时多线程读取, 再进行`SELECT`操作就会获得该锁, 同时可以被多线程事务持有
* 3 RESERVED: 表示数据库准备写入数据库, 同一时间只能被一个持有, 但是并不禁止持有`SHARED`锁
* 4 PENDING: 在真正写入之前等待其他`SHARED`的释放
* 5 EXCLUSIV: 表示可以写入数据库了, 其他线程不再能对数据库进行任何读写操作了


## 一个事务
```
BEGIN(UNLOCKED)
SELECT...(SHARED)
INSERT...(RESERVED)
COMMIT(PENDING)
```
&emsp;&emsp;一个事务大致会经过如上流程进行一个修改数据库操作, 但是我们可以看到: 在`COMMIT`时需要等待其他线程释放`SHARED`锁, 那么假设另外一个事务进行了读取操作下一步等待进行插入操作, 那么就进入了死锁

```
//线程1				//线程2
BEGIN(UNLOCKED)		
					BEGIN(UNLOCKED)
SELECT...(SHARED)
					SELECT...(SHARED)
INSERT...(RESERVED)	
COMMIT(PENDING)...
					INSERT...
```
&emsp;&emsp;`COMMIT`和`INSERT`各自等待对方的锁释放, 造成了死锁


## `BEGIN`起始状态

* 1 `BEGIN` 不获取任何锁
* 2 `BEGIN IMMEDIATE`: 开启时获取`RESERVED`锁, 
* 3 `BEGING EXCLUSIVE`: 开启时获取`EXCLUSIV`锁

&emsp;&emsp;`RESERVED`同时不允许多个获取, 所以不允许同时开启两个`IMMEDIATE`方式的事务; `EXCLUSIVE`方式来说, 即使使用`BEGIN`开启了另外一个事务, 也无法进行任何查看更改操作了

