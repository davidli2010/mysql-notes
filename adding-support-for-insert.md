# 添加对INSERT的支持

所有的INSERT操作都通过`handler::write_row()`方法处理。
```
int ha_foo::write_row(byte *buf);
```
`*buf`参数包含MySQL内部格式的要插入的行。

写入行的处理流程与读恰好相反：从MySQL内部行格式中取出数据并写入到数据文件中。