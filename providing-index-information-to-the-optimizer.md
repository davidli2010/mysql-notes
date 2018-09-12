# 提供索引信息给优化器

为了有效使用索引，存储引擎需要提供关于表和索引的信息给优化器。这些信息被用于选择是否使用索引，如果是的话使用哪个索引。

## 实现`info()`方法

优化器通过`handler::info()`方法来请求更新表的信息。`info()`方法没有返回值，相反它期望存储引擎设置多种公共变量供服务器需要时读取。这些值也被用于构成`SHOW`的输出，像`SHOW TABLE STATUS`和对`INFORMATION_SCHEMA`的查询。

所有的变量都是可选的，但是可能的话还是应该填写：
- records: 表中行的数量。如果不能快速提供一个精确的数字，应该设为大于1的值，那样优化器不会执行对0或1行记录的表的优化。
- deleted：表中删除的行的数量。用于标识表的碎片化。
- data_file_length：数据文件的大小，单位是字节。帮助优化器计算读取的成本。
- index_file_length：索引文件的大小，单位是字节。帮助优化器计算读取的成本。
- mean_rec_length：行的平均长度，单位是字节。
- scan_time：执行全表扫描的I/O搜索的开销。
- delete_length
- check_time

当计算值的时候，速度比精确更重要。没理由花较长的时间来告诉优化器哪个方法会最快。数量级的估算一般就足够好了。

## 实现`records_in_range`方法

优化器调用`records_in_range()`方法以帮助查询或联结选择使用哪个索引。定义如下：
```
ha_rows ha_foo::records_in_range(uint inx, key_range *min_key, key_range *max_key)
```

`inx`参数是要检查的索引。`*min_key`和`*max_key`参数是`key_range`结构体，用来表明范围的最低和最高端。`key_range`结构体如下所示：
```
typedef struct st_key_range
{
  const byte *key;
  uint length;
  key_part_map keypart_map;
  enum ha_rkey_function flag;
} key_range;
```

`key_range`的成员使用如下：
- key：指向key缓存的指针。
- length：key的长度。
- keypart_map：说明key中传递的keypart的位图。
- flag：说明key是否包含在范围内。对于`min_key`和`max_key`其值是不同的。

    `min_key.flag`可以取下面中的一个值：
    - HA_READ_KEY_EXACT：key在范围内
    - HA_READ_AFTER_KEY：key不包含在范围内 
 
    `max_key.flag`可以取下面中的一个值：
    - HA_READ_BEFORE_KEY：key不包含在范围内
    - HA_READ_AFTER_KEY：在范围内包含所有的'end_key'

下面的返回值是允许的：
- 0：在给定范围内没有匹配的键
- number>0：在范围内大约有*number*数量的匹配的行
- HA_POS_ERROR：索引树出错
