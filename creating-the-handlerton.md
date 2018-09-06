#  创建handlerton

`handlerton`(handler singleton的简写)定义了存储引擎，并包含全局使用的方法指针。与之相对的是在每个表上使用的方法。

handlerton有40多个元素，只有少数几个是强制要求的。
1. 定制的存储引擎db_type值为`DB_TYPE_UNKNOWN`。
2. slot。每个存储引擎在`thd`上有自己的内存区域（实际是一个指针），存储每个连接的信息。通过`thd->ha_data[handlerton.slot]`访问。slot的值在存储引擎初始化后被初始化。

**MySQL 5.7与文档描述不同，不再需要显式地创建handlerton全局变量，而是在插件的init函数中初始化handlerton的字段**
