# 定义文件名扩展

存储引擎被要求提供给MySQL服务器一个由存储引擎在给定表的数据和索引上使用的扩展名的列表。

扩展名预期是null结束的字符串数组的形式。下面是CSV引擎使用的数组：
```
static const char *ha_tina_exts[] = {
    ".CSV",
    NullS
};
```

这个数组由`handler::bas_ext()`方法返回：
```
const char **ha_tina::bas_ext() const
{
    return ha_tina_exts;
}
```

提供了扩展名之后可以省略实现`DROP TABLE`功能，因为MySQL服务器会实现关闭表和删除所有给定扩展名的文件。
