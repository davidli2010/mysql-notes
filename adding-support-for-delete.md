# 添加对DELETE的支持

MySQL采用与UPDATE相同的方法来执行DELETE语句：使用`rnd_next()`方法找到要删除的行，然后调用`handler::delete_row()`方法来删除行：
```
int ha_foo::delete_row(const byte *buf)
```
`*buf`参数包含了要删除的行的内容。