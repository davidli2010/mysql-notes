# 创建表

一旦handler实例化，第一个要求的操作可能是创建表。

存储引擎必须实现`create()`虚方法：
```
virtual int create(const char *name, TABLE *form, HA_CREATE_INFO *info)=0;
```
这个方法创建所有必要的文件，但是并不需要打开表。MySQL服务器会在后面打开表。

`*name`参数是表名。`*form`参数是表的结构，定义了表并与MySQL服务器已经创建的`tablename.frm`文件的内容相匹配。存储引擎绝对不能修改`tablename.frm`文件。

`*info`参数包含用来建表的`CREATE TALBE`语句中的信息。这个结构体在`handler.h`文件中定义。

一个基本的存储引擎可以忽略`*form`和`*info`的内容，因为它们实际是用于创建和初始化数据文件的（假设存储引擎是基于文件的）。

