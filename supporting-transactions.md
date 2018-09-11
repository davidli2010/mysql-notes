# 支持事务

## 概述

事务并不是在存储引擎层面显式开始的，而是通过调用`start_stmt()`或`external_lock()`隐式开始的。

存储引擎在每个连接的内存中存储事务的信息，并且也将事务注册到MySQL服务器中，以便运行服务器以后执行`COMMIT`和`ROLLBACK`操作。

伴随着操作的执行，存储引擎需要实现某种形式的版本或日志以允许回滚事务内执行的所有操作。

工作完成后，MySQL服务器会调用在存储引擎的handlerton中定义的`commit()`方法或`rollback()`方法。

## 开始一个事务

事务通过存储引擎中`start_stmt()`或`external_lock()`方法的调用开启。

如果没有活动的事务，存储引擎必须开始一个新事务并在MySQL服务器中注册这个事务，从而可以在后面执行ROLLBACK或COMMIT。

### 通过调用`start_stmt()`开始事务

第一个可以开始事务的方法调用是`start_stmt()`方法。

下面的示例展示了存储引擎如何注册一个事务：
```
int my_handler::start_stmt(THD *thd, thr_lock_type lock_type)
{
  int error= 0;
  my_txn *txn= (my_txn *) thd->ha_data[my_handler_hton.slot];

  if (txn == NULL)
  {
    thd->ha_data[my_handler_hton.slot]= txn= new my_txn;
  }
  if (txn->stmt == NULL && !(error= txn->tx_begin()))
  {
    txn->stmt= txn->new_savepoint();
    trans_register_ha(thd, FALSE, &my_handler_hton);
  }
  return error;
}
```
`THD`是当前客户端连接，它持有当前客户端的状态相关的数据，例如身份、网络连接和其它连接的数据。

`thd->ha_data[my_handler_hton.slot]`是在thd上的存储引擎的特定连接的数据的指针。在这个例子中用来存储事务上下文。

### 通过`external_lock()`方法开始事务

MySQL在每个语句的开始为每个将要使用的表调用`handler::external_lock()`。因此，如果这个表是第一个使用，就会隐式地开始一个事务。

注意因为预先加锁，在语句的开始和结束之间所有可能被使用的表都会在语句执行之前被锁定，`handler::external_lock()`会在所有这些表上被调用。即是说，如果一个INSERT触发了触发器，这个触发器调用了一个存储过程，而这个存储过程调用了一个存储的方法，那么所有在触发器、存储过程和方法使用的表都会在INSERT开始时被锁定。另外，如果有一个类似这样的结构：
```
IF
.. use one table
ELSE
.. use another table
```
这两个表都会被锁定。

同样，如果用户调用了`LOCK TABLES`，MySQL只会调用`handler::external_lock`一次。在这个例子中，MySQ会在语句开始前调用`handler::start_stmt()`。

下面的例子展示了存储引擎如何开始一个事务并考虑锁请求：
```
int my_handler::external_lock(THD *thd, int lock_type)
{
  int error= 0;
  my_txn *txn= (my_txn *) thd->ha_data[my_handler_hton.slot];

  if (txn == NULL)
  {
    thd->ha_data[my_handler_hton.slot]= txn= new my_txn;
  }

  if (lock_type != F_UNLCK)
  {
    bool all_tx= 0;
    if (txn->lock_count == 0)
    {
      txn->lock_count= 1;
      txn->tx_isolation= thd->variables.tx_isolation;

      all_tx= test(thd->options & (OPTION_NOT_AUTOCOMMIT | OPTION_BEGIN | OPTION_TABLE_LOCK));
    }

    if (all_tx)
    {
      txn->tx_begin();
      trans_register_ha(thd, TRUE, &my_handler_hton);
    }
    else
    if (txn->stmt == 0)
    {
      txn->stmt= txn->new_savepoint();
      trans_register_ha(thd, FALSE, &my_handler_hton);
    }
  }
  else
  {
    if (txn->stmt != NULL)
    {
      /* Commit the transaction if we're in auto-commit mode */
      my_handler_commit(thd, FALSE);

      delete txn->stmt; // delete savepoint
      txn->stmt= NULL;
    }
  }

  return error;
}
```

每个存储引擎在每次开始事务时都必须调`trans_register_ha()`。`trans_register_ha()`方法将事务注册到MySQL服务器中，并允许在将来COMMIT和ROLLBACK。

## 实现ROLLBACK

在事务的两个主要操作中，ROLLBACK实现起来更复杂。在事务期间发生的所有操作都必须倒退回去，这样所有的行从事务开始之前没有发生变化。

要支持ROLLBACK，创建一个匹配下面定义的方法：
```
int (*rollback)(THD *thd, bool all);
```
这个方法列在`handlerton`中。

`THD`参数用来标识需要回滚的事务，而`bool all`参数用来说明要回滚整个事务还是只是最后一个语句。

## 实现COMMIT

在提交操作时，事务导致的所有变更都会被持久化，回滚操作是不可能的了。根据事务的隔离级别，可能这是第一次这些变更对其它线程可见。

要支持COMMIT，创建一个匹配下面定义的方法：
```
int (*commit)(THD *thd, bool all);
```
这个方法列在`handlerton`中。

`THD`参数用来标识需要提交的事务，而`bool all`参数用来说明要提交的是整个事务还是只是最后一个语句。

如果服务器是auto-commit模式，存储引擎应该自动提交所有只读语句，例如SELECT。

在存储引擎中，“auto-commit”通过对锁计数来实现。每次调`external_lock()`时增加计数，当`external_lock()`被带着参数`F_UNLCK`调用时减少计数。当计数降回0，触发提交。
