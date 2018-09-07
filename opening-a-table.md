# 打开表

在表上执行任何读写操作之前，MySQL服务器会调用`handler::open()`方法打开表的数据和索引文件（如果存在的话）。
```
int open(const char *name, int mode, uint test_if_locked);
```
第一个参数是要打开的表的名字。第二个参数决定文件打开的方式和要采用的操作，它的值定义在`my_base.h`中。

最后一个参数说明handler在打开表之前是否应该检查表上的锁。