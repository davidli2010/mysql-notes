# 插件API组件

服务器插件实现由若干组件组成。

## SQL语句

- `INSTALL PLUGIN`在`mysql.plugin`表中注册插件并加载插件代码。
- `UNINSTALL PLUGIN`从`mysql.plugin`表中注销差价并卸载插件代码。
- 在创建全文索引时`WITH PRASER`子句将一个全文解析器插件与给定的全文索引关联起来。
- `SHOW PLUGINS`显示服务器插件的信息。

## 命令行选项与系统变量

- `--plugin-load`选项可以在服务器启动时加载插件。
- `plugin_dir`系统变量说明插件必须被安装的目录位置。这个变量的值可以在服务器启动时通过`--plugin_dir=dir_name`选项指定。

## 插件相关的表

- `INFORMATION_SCHEMA.PLUGINS`表包含插件信息。
- `mysql.plugin`表列出了通过`INSTALL PLUGIN`安装的每一个插件。


客户端插件实现更简单。

想要调查MySQL如何实现插件，可以查阅下列源码文件：
- 在`include/mysql`目录下，`plugin.h`暴露了公开的插件API。任何想要编写插件库的人都应该查看这个文件。`plugin_xxx.h`文件提供了关于特定类型插件的额外的信息。
- 在`sql`目录下，`sql_plugin.h`和`sql_plugin.cc`包含了内部插件实现。`sql_acl.cc`是服务器使用认证插件的地方。插件开发者不需要关注这些文件。
