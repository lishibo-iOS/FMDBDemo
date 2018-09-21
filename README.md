# FMDBDemo
FMDB的基本使用练习

### 什么是FMDB

很简单，一个iOS中SQLite API的封装库。

- 是对libsqlite3库的封装，使用起来简洁、高效，没有原来的一大堆晦涩难懂、影响开发效率的C语句，更加面向对象
- 非常的轻量化、灵活
- 对于多线程的并发操作进行了处理，是线程安全的（重要特性之一）
- 因为它是OC语言封装的，只能在ios开发的时候使用，所以在实现跨平台操作的时候存在局限性


### FMDB重要（常用）类

- FMDatabase：一个FMDatabase对象就代表一个单独的SQLite数据库（注意并不是表），用来执行SQL语句
- FMResultSet：使用FMDatabase执行查询后的结果集
- FMDatabaseQueue：用于在多线程中执行多个查询或更新,它是线程安全的

### FMDB的使用步骤详解

#### 1.准备步骤（基础操作）

1.项目中导入FMDB [Github地址](https://github.com/ccgus/fmdb)

- Cocoapods导入（推荐），如果你还没有使用Cocoapods，就赶紧学起来吧 
- 手动下载工程，并导入

2.导入libsqlite3框架（一般情况下默认是导入的，所以一般都不用做这一步）,细心的朋友可能会发现我们的库中还有一个libsqlite3.0框架[那么它们俩有什么区别呢](https://www.jianshu.com/p/7d5bce47f97f)

3.在需要用到FMDB的控制器（或模型）地方 `import FMDatabase.h` 或 `#import <FMDB/FMDB.h>`

4.新建一个继承自NSObject的对象，这里我们使用student，接下来数据库中增删改查的对象就用stduent进行操作示例

![pic1](https://raw.githubusercontent.com/lishibo-iOS/pictures/master/blog15pic/1.png)

5.你已经跃跃欲试了吧，别急，本地的.sqlite你现在还无法查看， 现在推荐App Store里面自带的Datum Free（免费版）了，使用起来很方便

![pic2](https://raw.githubusercontent.com/lishibo-iOS/pictures/master/blog15pic/2.png)

#### 2.数据库的创建

![pic3](https://raw.githubusercontent.com/lishibo-iOS/pictures/master/blog15pic/3.png)


``` objectivec

 //1.获取数据库文件的路径
    _docPath = [NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES) lastObject];
    NSLog(@"_docPath==%@",_docPath);
    mark_student = 1;
    //设置数据库名称
    NSString *fileName = [_docPath stringByAppendingPathComponent:@"student.sqlite"];
    
    //2.获取数据库
    _db = [FMDatabase databaseWithPath:fileName];
    if ([_db open]) {
        NSLog(@"打开数据库成功");
    } else {
        NSLog(@"打开数据库失败");
    }

```

这里我们可以打开_docPath的路径，可以看到名为“student.sqlite”的数据库已经创建了

#### 3.表的创建

``` objectivec

    //3.创建表
    BOOL result = [_db executeUpdate:@"CREATE TABLE IF NOT EXISTS t_student (id integer PRIMARY KEY AUTOINCREMENT, name text NOT NULL, age integer NOT NULL, sex text NOT NULL);"];
    if (result) {
        NSLog(@"创建表成功");
    } else {
        NSLog(@"创建表失败");
    }
    
    [_db close];//注意每次操作完数据库后要关闭一下

```

#### 4.添加数据

``` objectivec

for (int i = 0; i < 4; i++) {
        
        //插入数据
        NSString *name = [NSString stringWithFormat:@"测试名字%@",@(mark_student)];
        int age = mark_student;
        NSString *sex = @"男";
        mark_student ++;
        //1.executeUpdate:不确定的参数用？来占位（后面参数必须是oc对象，；代表语句结束）
        BOOL result = [_db executeUpdate:@"INSERT INTO t_student (name, age, sex) VALUES (?,?,?)",name,@(age),sex];
        //2.executeUpdateWithForamat：不确定的参数用%@，%d等来占位 （参数为原始数据类型，执行语句不区分大小写）
        //    BOOL result = [_db executeUpdateWithFormat:@"insert into t_student (name,age, sex) values (%@,%i,%@)",name,age,sex];
        //3.参数是数组的使用方式
        //    BOOL result = [_db executeUpdate:@"INSERT INTO t_student(name,age,sex) VALUES  (?,?,?);" withArgumentsInArray:@[name,@(age),sex]];
        if (result) {
            NSLog(@"插入成功");
        } else {
            NSLog(@"插入失败");
        }

    }

```

多插入几次后，我们可以看到t_student表中添加了多条数据


#### 5.删除数据

``` objectivec

//1.不确定的参数用？来占位 （后面参数必须是oc对象,需要将int包装成OC对象）
    int idNum = 11;
    BOOL result1 = [_db executeUpdate:@"delete from t_student where id = ?",@(idNum)];
    //2.不确定的参数用%@，%d等来占位
    //BOOL result = [_db executeUpdateWithFormat:@"delete from t_student where name = %@",@"王子涵"];
    if (result1) {
        NSLog(@"删除成功");
    } else {
        NSLog(@"删除失败");
    }


```

#### 6.修改数据

``` objectivec

//修改学生的名字
    NSString *newName = @"新名字";
    NSString *oldName = @"测试名字2";
    BOOL result2 = [_db executeUpdate:@"update t_student set name = ? where name = ?",newName,oldName];
    if (result2) {
        NSLog(@"修改成功");
    } else {
        NSLog(@"修改失败");
    }


```

查看sqlite，可以看到测试名字2的姓名已经被修改了

#### 7.查询数据

``` objectivec

//查询整个表
    FMResultSet * resultSet = [_db executeQuery:@"select * from t_student"];
    //根据条件查询
    //FMResultSet * resultSet = [_db executeQuery:@"select * from t_student where id < ?", @(4)];
    //遍历结果集合
    while ([resultSet next]) {
        int idNum = [resultSet intForColumn:@"id"];
        NSString *name = [resultSet objectForColumn:@"name"];
        int age = [resultSet intForColumn:@"age"];
        NSString *sex = [resultSet objectForColumn:@"sex"];
        NSLog(@"学号：%@ 姓名：%@ 年龄：%@ 性别：%@",@(idNum),name,@(age),sex);
    }


```
执行查询之后控制台的打印，可以看到打印出了目前的所有数据

#### 8.表的删除

``` objectivec

    //如果表格存在 则销毁
//    BOOL result = [_db executeUpdate:@"drop table if exists t_student"];
//    if (result) {
//        NSLog(@"删除表成功");
//    } else {
//        NSLog(@"删除表失败");
//    }

```

执行之后，刷新SQLite，可以看到t_student表已经被删除了

#### 9.FMDB的事务

- 事务定义：
事务(Transaction)是并发操作的基本单位，是指单个逻辑工作单位执行的一系列操作序列，这些操作要不都成功，要不就不成功，事务是数据库维护数据一致性的单位，在每个事务结束时，都能保证数据一致性与准确性，通常事务跟程序是两个不同的概念，一个程序中包含多个事务，事务主要解决并发条件下操作数据库，保证数据
- 事务特征：
原子性（Atomic）：事务中包含的一系列操作被看作一个逻辑单元，这个逻辑单元要不全部成功，要不全部失败
一致性（Consistency）：事务中包含的一系列操作，只有合法的数据被写入数据库，一些列操作失败之后，事务会滚到最初创建事务的状态
隔离性（Isolation）：对数据进行修改的多个事务之间是隔离的，每个事务是独立的，不应该以任何方式来影响其他事务
持久性（Durability）事务完成之后，事务处理的结果必须得到固化，它对于系统的影响是永久的，该修改即使出现系统固执也将一直保留，真实的修改了数据库
- 事务语句：

``` objectivec

 transaction:事务 开启一个事务执行多个任务，效率高
 1.fmdb 封装transaction 方法，操作简单
 - (BOOL)beginTransaction;
 - (BOOL)beginDeferredTransaction;
 - (BOOL)beginImmediateTransaction;
 - (BOOL)beginExclusiveTransaction;
 - (BOOL)commit;
 - (BOOL)rollback;
 等等


```

开启事务 ：beginTransaction
回滚事务：rollback
提交事务：commit

- 事务代码

用事务处理一系列数据库操作，省时效率高

``` objectivec

- (void)handleTransaction:(UIButton *)sender {
    NSString *documentPath = NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES)[0];
    NSString *dbPath = [documentPath stringByAppendingPathComponent:@"test1.db"];
    FMDatabase *db = [[FMDatabase alloc]initWithPath:dbPath];
    [db open];
    if (![db isOpen]) {
        return;
    }
    BOOL result = [db executeUpdate:@"create table if not exists text1 (name text,age,integer,ID integer)"];
    if (result) {
        NSLog(@"create table success");
    }
    //1.开启事务
    [db beginTransaction];
    NSDate *begin = [NSDate date];
    BOOL rollBack = NO;
    @try {
       //2.在事务中执行任务
        for (int i = 0; i< 500; i++) {
            NSString *name = [NSString stringWithFormat:@"text_%d",i];
            NSInteger age = i;
            NSInteger ID = i *1000;
            
            BOOL result = [db executeUpdate:@"insert into text1(name,age,ID) values(:name,:age,:ID)" withParameterDictionary:@{@"name":name,@"age":[NSNumber numberWithInteger:age],@"ID":@(ID)}];
            if (result) {
                NSLog(@"在事务中insert success");
            }
        }
    }
    @catch(NSException *exception) {
        //3.在事务中执行任务失败，退回开启事务之前的状态
        rollBack = YES;
        [db rollback];
    }
    @finally {
        //4. 在事务中执行任务成功之后
        rollBack = NO;
        [db commit];
    }
    NSDate *end = [NSDate date];
    NSTimeInterval time = [end timeIntervalSinceDate:begin];
    NSLog(@"在事务中执行插入任务 所需要的时间 = %f",time);
    
}

```

未使用事务执行一系列操作

``` objectivec

- (void)handleNotransaction:(UIButton *)sender {
    NSString *documentPath = NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES)[0];
    NSString *dbPath = [documentPath stringByAppendingPathComponent:@"test1.db"];
    FMDatabase *db = [FMDatabase databaseWithPath:dbPath];
    [db open];
    if (![db isOpen]) {
        return;
    }
    BOOL result = [db executeUpdate:@"create table if not exists text2('ID' INTEGER PRIMARY KEY AUTOINCREMENT,name TEXT,age INTRGER)"];
    if (!result) {
        [db close];
    }
    NSDate *begin = [NSDate date];
    for (int i = 0; i< 500; i++) {
        NSString *name = [NSString stringWithFormat:@"text_%d",i];
        NSInteger age = i;
        NSInteger ID = i *1000;
        
        BOOL result = [db executeUpdate:@"insert into text2(name,age,ID) values(:name,:age,:ID)" withParameterDictionary:@{@"name":name,@"age":[NSNumber numberWithInteger:age],@"ID":@(ID)}];
        if (result) {
            NSLog(@"不在事务中insert success");
        }
    }
    NSDate *end = [NSDate date];
    NSTimeInterval time = [end timeIntervalSinceDate:begin];
    NSLog(@"不在事务中执行插入任务 所需要的时间 = %f",time);
}


```

比较结果如下：

在事务中执行插入任务 所需要的时间 = 0.426221

不在事务中执行插入任务 所需要的时间 = 0.790417
