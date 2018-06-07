

## 背景

大部分 App 项目中的通讯录数据一般会选择放在本地，一般也会选择增量更新(每条数据都记录一个时间戳，每次只更新比时间戳大的数据)的方式来同步本地的数据,这样有一个问题就是当用户的通讯录数据较多。第一次登录某个新客户端的时候，就要同步通讯录所有的数据。网络拉去大量的数据，拉完了就要插入本地数据库。如果这时候sql 插入比较慢的时候，就会给用户明显的卡顿等待的感觉。我们项目中登录完成会返回用户加入的所有的圈子，然后根据圈子id 去请求圈子成员，插入数据库。当圈子成员达到100  圈子成员达到10万条数据的时候，感觉非常卡。下面就说说采坑 和 出坑的过程。

### 网络会是瓶颈吗？

同时要发送一百多个 request 应该网络会有一点拥堵，但是老大说网络应该不是瓶颈。

### 是在子线中插入的吗？

是的 每次 response 回来的时候 都是在子线程里面做插入操作

### 是在事务中批量插入数据的吗？

是的 我直接调用 FMDBqueue 的inTransaction 在一个事务中 循环 insert 数据, sqlite 在事务中插入 能成倍 提高插入的速度

### 最后

那为什么主线程还是被卡住了，看代码发现每插入一条数据都会更新 NSUserDefault 中的时间戳，难道sqlite在事务中出了数据库 之外不能操作其他的文件？去掉之后 改用其他方式实现之后 果然没那么卡 插入速度也快了一些

又 google 一圈 发现加入下面两句代码 又快了一写 基本可以接受，但是加上这个配置之后，可能在在操作数据库的时候断电了 可能会造成数据丢失 但是这个概率 太小了

```
 [[self getDBQueue] inDatabase:^(FMDatabase *db) {
        [db executeUpdate:@"PRAGMA synchronous = OFF"];
        [db executeUpdate:@"PRAGMA journal_mode = MEMORY"];
    }];

```

### 参考链接

[https://stackoverflow.com/questions/1711631/improve-insert-per-second-performance-of-sqlite?utm_medium=organic&utm_source=google_rich_qa&utm_campaign=google_rich_qa](https://stackoverflow.com/questions/1711631/improve-insert-per-second-performance-of-sqlite?utm_medium=organic&utm_source=google_rich_qa&utm_campaign=google_rich_qa)