# 实现基本的表扫描

最基本的存储引擎实现了只读表扫描。这样的引起可以用来支持日志和来自MySQL之外的其它数据文件的SQL查询。

下面展示了CSV引擎中在一个有9行记录的表上扫描时调用的方法：
```
ha_tina::store_lock
ha_tina::external_lock
ha_tina::info
ha_tina::rnd_init
ha_tina::extra - ENUM HA_EXTRA_CACHE   Cache record in HA_rrnd()
ha_tina::rnd_next
ha_tina::rnd_next
ha_tina::rnd_next
ha_tina::rnd_next
ha_tina::rnd_next
ha_tina::rnd_next
ha_tina::rnd_next
ha_tina::rnd_next
ha_tina::rnd_next
ha_tina::extra - ENUM HA_EXTRA_NO_CACHE   End caching of records (def)
ha_tina::external_lock
ha_tina::extra - ENUM HA_EXTRA_RESET   Reset database to after open
```

## 实现store_lock()方法

`store_lock()`方法在任何读或写操作执行前被调用。

在将锁添加到表锁处理器之前，mysqld带着请求的锁调用存储锁（store lock）。存储锁可以修改锁的级别，例如将阻塞读锁改为非阻塞、忽略锁（如果我们不想使用MySQL的表锁）或者为多张表添加锁（像MERGE时一样）。

当释放锁时，store_lock()也被调用。此时，一般不需要做任何事情。

如果store_lock的参数是TL_IGNORE，这意味着MySQL请求handler存储与上次一样的锁级别。

潜在的锁类型定义在`includes/thr_lock.h`中：
```
enum thr_lock_type
{
  TL_IGNORE=-1,
  TL_UNLOCK,                  /* UNLOCK ANY LOCK */
  TL_READ,                    /* Read lock */
  TL_READ_WITH_SHARED_LOCKS,
  TL_READ_HIGH_PRIORITY,      /* High prior. than TL_WRITE. Allow concurrent insert */
  TL_READ_NO_INSERT,          /* READ, Don't allow concurrent insert */
  TL_WRITE_ALLOW_WRITE,       /*   Write lock, but allow other threads to read / write. */
  TL_WRITE_ALLOW_READ,        /* Write lock, but allow other threads to read / write. */
  TL_WRITE_CONCURRENT_INSERT, /* WRITE lock used by concurrent insert. */
  TL_WRITE_DELAYED,           /* Write used by INSERT DELAYED.  Allows READ locks */
  TL_WRITE_LOW_PRIORITY,      /* WRITE lock that has lower priority than TL_READ */
  TL_WRITE,                   /* Normal WRITE lock */
  TL_WRITE_ONLY               /* Abort new lock request with an error */
};
```

实际的锁处理依赖一个锁的实现，可以选择实现其中一部分锁类型或者都不实现而改用自己实现的合适的方法。下面是对于一个不需要降级锁的存储引擎的最小实现：
```
THR_LOCK_DATA **ha_tina::store_lock(THD *thd,
                                     THR_LOCK_DATA **to,
                                     enum thr_lock_type lock_type)
{
   if (lock_type != TL_IGNORE && lock.type == TL_UNLOCK)
     lock.type=lock_type;
   *to++= &lock;
   return to;
}
```

`ha_myisammrg::store_lock()`有一个更复杂的实现。

## 实现external_lock()方法

`extern_lock()`方法在语句开始或者发出`LOCK TALBES`语句的时候被调用。

以下摘录自InnoDB的`extern_lock()`方法的注释：
> As MySQL will execute an external lock for every new table it uses when it starts to process an SQL statement (an exception is when MySQL calls start_stmt for the handle) we can use this function to store the pointer to the THD in the handle. We will also use this function to communicate to InnoDB that a new SQL statement has started and that we must store a savepoint to our transaction handle, so that we are able to roll back the SQL statement in case of an error.

## 实现rnd_init()方法

`rnd_init()`方法在任何表扫描之前被调用。`rnd_init()`方法用来为表扫描做准备，复位计数器并将指针指向表的起始。

## 实现info(uint flag)方法

在开始表扫描之前，`info()`方法被调用来提供额外的表信息给优化器。

优化器需要的信息不是通过返回值给出，而是通过填入handler的stats成员的某些属性，优化器会在调用`info()`之后读取这些属性。stats成员是`ha_statistics`类的实例。

除了被优化器使用，在`info()`调用时设置的许多值也被`SHOW TABLE STATUS`语句使用。flag参数是位字段，传达了调用info方法的上下文。

标识定义在`include/my_base.h`中：
- HA_STATUS_NO_LOCK：如果能够阻止共享锁表，handler可能使用了过时的信息
- HA_STATUS_TIME：只更新stats->update_time
- HA_STATUS_CONST：更新stats的不可变成员（max_data_file_length，max_index_file_length，create_time, sortkey，ref_length，block_size，data_file_name，index_file_name）
- HA_STATUS_VARIABLE：records，deleted，data_file_length，index_file_length，delete_length, check_time，mean_rec_length
- HA_STATUS_ERRKEY：最后一次错误键相关的状态（errkey and dupp_ref)
- HA_STATUS_AUTO：更新自增字段

公开的属性在`sql/handler.h`中：
```
ulonglong data_file_length;      /* Length off data file */
ulonglong max_data_file_length;  /* Length off data file */
ulonglong index_file_length;
ulonglong max_index_file_length;
ulonglong delete_length;         /* Free bytes */
ulonglong auto_increment_value;
ha_rows records;                 /* Records in table */
ha_rows deleted;                 /* Deleted records */
ulong raid_chunksize;
ulong mean_rec_length;           /* physical reclength */
time_t create_time;              /* When table was created */
time_t check_time;
time_t update_time;
```

对于表扫描，最重要的属性是`records`，表明表中记录的数量。当存储引擎表明有0或1个记录在表中与有2个或更多记录相比，优化器表现不同。因此，当在表扫描之前不知道表中实际的记录数量时，返回2或更大的值。

## 实现extra()方法

在某些操作之前`extra()`方法会被调用，来提供额外的提示告诉存储引擎怎样执行这些操作。

不强制实现extra，大部分存储引擎返回0。

## 实现rnd_next()方法

在表初始化之后，MySQL服务器会为每一行记录调一次handler的`rnd_next()`方法，直到满足了查询条件或者到达了文件结尾返回了` HA_ERR_END_OF_FILE`。

`rnd_next()`方法使用字节数组参数`*buf`，这个参数必须按照MySQL内部格式填入记录的内容。

服务器使用3种数据格式：定长记录、变长记录和带有BLOB指针的变长记录。在每一种格式中，列按照`CREATE TABLE`语句定义的顺序出现。

每种格式都以每个可空列占一位的NULL位图开始。有8个可空列的表有一个1字节的位图；有9~16个可空列的表有一个2字节的位图。定宽的表是个例外，它有额外的起始位，因此有8个可空列的表有一个2字节的位图。

在NULL位图之后是一个一个的列。每个列的大写由**数据类型**决定。在服务器上，列的数据类型定义在`sql/field.cc`文件中。在定长记录格式中，列简单地一个个放在一起。在变长格式中，VARCHAR列被编码为1或2个字节长，后面跟着一个字符串。在带有BLOB列的变长记录中，每个blob有两部分：第一部分是表示BLOB实际大小的整数，然后是一个指向BLOB内存的指针。
