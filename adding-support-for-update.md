# 添加对UPDATE的支持

MySQL服务器通过进行（表/索引/范围等）扫描直到定位到匹配WHERE子句的记录来执行UPDATE语句，然后调用`handler::update_row()`方法。
```
int ha_foo::update_row(const byte *old_data, byte *new_data)
```
`*old_data`参数包含更新前的行的数据，而`*new_data`参数包含行的新内容。

执行UPDATE依赖行格式和存储实现。一些存储引擎在原地替换数据，而另一些实现删除存在的行并在数据尾部添加新行。

非索引存储引擎可以忽略`*old_data`参数的内容而只处理`*new_data`。事务性引擎可能需要比较缓存，为了在稍后的回滚时决定发生了哪些变化。

如果被修改的表包含timestamp列，需要在`update_row()`的调用中更新timestamp。