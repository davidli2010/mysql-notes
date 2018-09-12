# 实现索引的方法

## 通过`index_init()`为使用索引做准备

`index_init()`方法在索引被使用之前调用，使存储引擎可以执行任何必要的准备或优化。
```
int ha_foo::index_init(uint keynr, bool sorted)
```

大部分存储引擎不需要做特别的准备，如果存储引擎没有显式地实现该方法，则会使用一个默认实现。
```
int handler::index_init(uint idx) { active_index=idx; return 0; }
```

## 使用`index_end()`进行清理

`index_end()`方法与`index_init()`方法相对应。`index_end()`方法的目前是清理任何由`index_init()`方法所做的准备工作。

如果存储引擎没有实现`index_init()`，它就没必要实现`index_end()`。

## 实现`index_read()`方法

`index_read()`方法用来按键搜索一行：
```
int ha_foo::index_read(byte * buf, const byte * key,
                       ulonglong keypart_map,
                       enum ha_rkey_function find_flag)
```
`*buf`参数是字节数组，由存储引擎填入匹配`*key`指定的索引键的行。`keypart_map`参数是说明`key`参数中包含哪些键的位图。`find_flag`参数是用来说明搜索行为的枚举值。

使用的索引是之前在`index_init()`调用中定义的，并存储在handler的active_index变量中。

以下`find_flag`的值是允许的：
```
HA_READ_AFTER_KEY
HA_READ_BEFORE_KEY
HA_READ_KEY_EXACT
HA_READ_KEY_OR_NEXT
HA_READ_KEY_OR_PREV
HA_READ_PREFIX
HA_READ_PREFIX_LAST
HA_READ_PREFIX_LAST_OR_PREV
```

存储引擎必须将`*key`参数转换成存储引擎特定的格式，使用它根据find_flag找到匹配的行，然后以MySQL内部行格式填充`*buf`。

除了返回匹配的行，存储引擎还必须设置游标以支持索引顺序读。

如果`*key`参数是null，存储引擎应该读取索引中的第一个键。

## 实现`index_read_idx()`方法

`index_read_idx()`方法等同于`index_read()`方法，除了`index_read_idx()`接受一个额外的`keynr`参数。

```
int ha_foo::index_read_idx(byte * buf, uint keynr, const byte * key,
                           ulonglong keypart_map,
                           enum ha_rkey_function find_flag)
```

`keynr`参数指定用来读取的索引，反之`index_read()`的索引已经提前设定了。

与`index_read()`方法一样，存储引擎必须返回按照`find_flag`找到的匹配的行并且设置游标用于将来读取。

> MySQL 5.7已调整接口，handler::index_read_idx_map()调用handler::index_read_map()，不需要专门实现。

## 实现`index_read_last()`方法

`index_read_last()`方法与`index_read()`工作方式一样，但是返回当前键值的最后一行或前一行。
```
int ha_foo::index_read_last(byte * buf, const byte * key,
                            key_part_map keypart_map)
```

`index_read_last()`在对类型下面的语句优化掉`ORDER BY`子句时使用：
```
SELECT * FROM t1 WHERE a=1 ORDER BY a DESC,b DESC;
```

## 实现`index_next()`方法

`index_next()`方法用于索引扫描：
```
int ha_foo::index_next(byte * buf)
```
`*buf`参数填入内部游标中的下一个匹配键值的行，游标由存储引擎在`index_read()`和`index_first()`之类的操作期间设置。

## 实现`index_prev()`方法

`index_prev()`方法用于反向索引扫描：
```
int ha_foo::index_prev(byte * buf)
```
`*buf`参数填入内部游标中的前一个匹配键值的行，游标由存储引擎在`index_read()`和`index_first()`之类的操作期间设置。

## 实现`index_first()`方法

`index_first()`方法用于索引扫描：
```
int ha_foo::index_first(byte * buf)
```
`*buf`参数填入索引中的第一个键值的行。

## 实现`index_last()`方法

`index_last()`方法用于反向索引扫描：
```
int ha_foo::index_last(byte * buf)
```
`*buf`参数填入索引中的最后一个键值的行。
