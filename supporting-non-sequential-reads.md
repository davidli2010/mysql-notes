# 支持非顺序读

除了表扫描，存储引擎还要实现非顺序读的方法。某些操作依赖`position()`和`rnd_pos()`。例如多表的UPDATE和`SELECT .. table.blob_column ORDER BY something`）。MySQL服务器对某些排序操作使用这些方法。

## 实现`position()`方法

如果数据需要重新排序，在每次调用`rnd_next()`之后会调用`position()`方法。
```
void ha_foo::position(const byte *record)
```

这将存储一个记录的“位置”到`this->ref`中。“位置”的内容取决于实现，不管是什么内容都会在后面搜索行时被返回。唯一的规则-“位置”或者一行必须包含足够的信息用来在后面搜索这一行。大部分存储引擎会保存某种形式的偏移或者一个主键值。

## 实现`rnd_pos()`方法

`rnd_pos()`方法的行为与`rnd_next()`类似，但是使用额外的参数：
```
int ha_foo::rnd_pos(byte * buf, byte *pos)
```

`*pos`参数包含之前使用`position()`方法记录的位置信息。

存储引擎必须定位到位置对应的行，并通过`*buf`返回MySQL行格式的数据。