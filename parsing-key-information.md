# 解析键的信息

许多索引方法传递一个名为`*key`的字节数组来标识按标准格式读取的索引条目。存储引擎需要取出存储在键中的信息并转换为其内部的索引格式来标识与索引相关的行。

通过遍历键来获取键的信息，键的格式与`table->key_info[index]->key_part[part_num]`中的定义相同。

连同键一起，handler方法传递一个`keypart_map`参数来说明在`key`参数中有哪些键。`keypart_map`是一个每个key part一位的`ulonglong`位图：1是keypart[0]，2是keypart[1]，4是keypart[2]，等等。如果keypart_map中的某一位被设置了，那么这个key part的值就在key的缓存中。最后一个key part之后的位并无关系，因此~0可以用于所有的keyparts。目前，只支持key prefixes。即是说，`assert((keypart_map + 1) & keypart_map == 0)`。

`keypart_map`是`records_in_range()`使用的`key_range`结构体的一部分，`keypart_map`的值被直接传给`index_read()`、`index_read_idx()`和`index_read_last()`方法。
