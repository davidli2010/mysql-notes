# 支持索引

存储引擎添加对索引的支持围绕着两个任务：提供信息给优化器并实现索引相关的方法。提供给优化器的信息帮助优化器更好的选择使用哪个索引甚至是跳过使用索引而执行表扫描。

索引方法既可以读取匹配键的行，按照索引顺序扫描行，也可以从索引上直接读取信息。

下面的例子展示了在一次UPDATE的查询中使用索引时调用的方法，比如`UPDATE foo SET ts = now() WHERE id = 1`:
```
ha_foo::index_init
ha_foo::index_read
ha_foo::index_read_idx
ha_foo::rnd_next
ha_foo::update_row
```

除了索取读方法，存储引擎必须支持新索引的创建，在添加、修改、删除行时保持索引数据的更新。
