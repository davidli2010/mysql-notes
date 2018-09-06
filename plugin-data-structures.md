# 插件数据结构

插件库文件包含描述符信息来说明插件包含的内容。

如果插件库包含任何服务器插件，它必须包含下面的描述符信息：
- 库描述符，说明库使用的通用服务器插件API版本号，并且在库中包含一个通用插件描述符。从`plugin.h`头文件中调用两个宏，来为描述符提供框架：
```
mysql_declare_plugin(name)
    ... one or more server plugin descriptors here ...
mysql_declare_plugin_end;
```
这个宏展开后自动提供了API版本的声明。必须提供插件描述符。
- 在库描述符中，每个通用的服务器插件通过一个`st_mysql_plugin`结构体描述。这个插件描述符结构体包含对每个服务器插件类型通用的信息：说明插件类型的值；插件名称、作者、描述和license类型；指向初始化和析构函数的指针，以及指向插件实现的状态或系统变量的指针。
- 每个在库描述符中的通用服务器插件描述符也包含一个指向特定类型插件描述符的指针。特定类型的描述符在不同插件类型之间是不一样的，因为每一种插件类型都有自己的API。一个特定类型的插件的描述符包含特定类型的API版本号和指向该类型需要实现的函数的指针。

## 服务器插件库和插件描述符

每个包含服务器插件的插件库都必须含有一个包含通用插件描述符的库描述符。

库描述符必须定义两个符号：
- `_mysql_plugin_interface_version_`指定了通用插件框架的版本号。这是由`MYSQL_PLUGIN_INTERFACE_VERSION `符号给出的，定义在`plugin.h`文件中。
- `_mysql_plugin_declarations_`定义了一个插件声明的数组，终止于一个所有成员都是0的声明。每一个声明都是`st_mysql_plugin`结构体的实例（同样定义在`plugin.h`中）。

如果服务器在一个库中没有发现这两个符号，它就不会接受它作为一个合法的插件库。

定义这两个要求的符号的常规方式是使用`plugin.h`文件中的`mysql_declare_plugin()`和`mysql_declare_plugin_end`宏：
```
mysql_declare_plugin(name)
    ... one or more server plugin descriptors here ...
mysql_declare_plugin_end;
```

每一个服务器插件必须有一个通用描述符来提供服务器插件API的信息。所有的插件类型有相同的结构。`plugin.h`文件中的`st_mysql_plugin`定义了这个描述符：
```
struct st_mysql_plugin
{
    int type;                 /* the plugin type (a MYSQL_XXX_PLUGIN value)   */
    void *info;               /* pointer to type-specific plugin descriptor   */
    const char *name;         /* plugin name                                  */
    const char *author;       /* plugin author (for I_S.PLUGINS)              */
    const char *descr;        /* general descriptive text (for I_S.PLUGINS)   */
    int license;              /* the plugin license (PLUGIN_LICENSE_XXX)      */
    int (*init)(void *);      /* the function to invoke when plugin is loaded */
    int (*deinit)(void *);    /* the function to invoke when plugin is unloaded */
    unsigned int version;     /* plugin version (for I_S.PLUGINS)             */
    struct st_mysql_show_var *status_vars;
    struct st_mysql_sys_var **system_vars;
    void * __reserved1;       /* reserved for dependency checking             */
    unsigned long flags;      /* flags for plugin */
};
```

`st_mysql_plugin`描述符结构体成员使用如下。`char *`成员应指定为以null结尾的字符串。
- type：插件类型。必须是`plugin.h`中的插件类型值中的一个：
    ```
    /*
    The allowable types of plugins
    */
    #define MYSQL_UDF_PLUGIN             0  /* User-defined function        */
    #define MYSQL_STORAGE_ENGINE_PLUGIN  1  /* Storage Engine               */
    #define MYSQL_FTPARSER_PLUGIN        2  /* Full-text parser plugin      */
    #define MYSQL_DAEMON_PLUGIN          3  /* The daemon/raw plugin type */
    #define MYSQL_INFORMATION_SCHEMA_PLUGIN  4  /* The I_S plugin type */
    #define MYSQL_AUDIT_PLUGIN           5  /* The Audit plugin type        */
    #define MYSQL_REPLICATION_PLUGIN     6  /* The replication plugin type */
    #define MYSQL_AUTHENTICATION_PLUGIN  7  /* The authentication plugin type */
    ...
    ```
- info：指向插件的类型特定描述符的指针。描述符的结构依赖插件的特定类型。出于版本控制的目的，特定类型描述符的第一个成员是类型的接口版本。在版本号后面，描述符包含了任何需要的成员。
- name：给出插件的名称。这个名称将会在`mysql.plugin`表中列出，可以在像`INSTALL PLUGIN`和`UNINSTALL PLUGIN`这样的语句中使用。插件名称不能以任何服务器选项的名字起始，否则服务器会初始化失败。
- author：插件作者名称。
- desc：提供插件的一般描述。
- license：插件的license类型。值可以是`PLUGIN_LICENSE_PROPRIETARY, PLUGIN_LICENSE_GPL, or PLUGIN_LICENSE_BSD`中的一个。
- init：仅一次的初始化函数，如果没有则为NULL。服务器在加载插件时执行这个函数。
- deinit：仅一次的析构函数，如果没有则为NULL。服务器在卸载插件时执行这个函数。
- version：插件的版本号。插件安装之后，这个值可以在`INFORMATION_SCHEMA.PLUGINS`表中找到。值包括主版本号和次版本号。如果把值写做十六进制，格式是`0xMMNN`，MM和NN分别为主版本号和次版本号。例如，0x0302表示版本3.2。
- status_vars：指向插件的状态变量结构体的指针，如果没有变量则为NULL。当插件安装后，这些变量在`SHOW STATUS`语句中显示。
- system_vars：指向插件的系统变量结构体的指针，如果没有变量则为NULL。这些选项和系统变量可以帮助初始化插件中的变量。当插件安装后，这些变量在`SHOW VARIABLES`语句中显示。  
`system_vars`成员如果不为NULL，则指向`st_mysql_sys_var`结构体数组。
- __reserved1：占位符。应设为NULL。
- flags：插件的标识位。每一位表示不同的标识。值应设为标识位的与操作。标识可以是：
    ```
    #define PLUGIN_OPT_NO_INSTALL   1UL   /* Not dynamically loadable */
    #define PLUGIN_OPT_NO_UNINSTALL 2UL   /* Not dynamically unloadable */
    ```

服务器只在加载和卸载插件时调用通用插件描述符中的init和deinit函数。

## 服务器插件状态和系统变量

服务器插件接口通过使用通用插件描述符中的`status_vars`和`system_vars`成员来暴露状态和系统变量。

`status_vars`成员如果不设置为0，则指向一个`st_mysql_show_var`结构体数组。数组中的每个元素描述一个状态变量，最后一个元素的所有成员都设为0。`st_mysql_show_var`结构体定义如下：
```
struct st_mysql_show_var {
    const char *name;
    char *value;
    enum enum_mysql_show_type type;
};
```

下表展示了允许的状态变量的`type`值以及对应的变量：
|变量类型|含义|
|-------|----|
|SHOW_BOOL|指向一个boolean变量|
|SHOW_INT|指向一个int变量|
|SHOW_LONG|指向一个long int变量|
|SHOW_LONGLONG|指向一个long long int变量|
|SHOW_CHAR|一个字符串|
|SHOW_CHAR_PTR|指向一个字符串|
|SHOW_ARRAY|指向另一个st_mysql_show_var数组|
|SHOW_FUNC|指向一个函数|
|SHOW_DOUBLE|指向一个double|

对于SHOW_FUNC类型，函数被调用并填充它的out参数。函数签名如下：
```
#define SHOW_VAR_FUNC_BUFF_SIZE 1024

typedef int (*mysql_show_var_func) (void *thd,
                                    struct st_mysql_show_var *out,
                                    char *buf);
```

`system_vars`成员如果不为0，则指向一个`st_mysql_sys_var`结构体的数组。数组中的每一个元素描述一个系统变量（也可以通过命令行或配置文件设置）,最后一个元素的所有成员都设为0。`st_mysql_sys_var`结构体定义如下：
```
struct st_mysql_sys_var {
    int flags;
    const char *name, *comment;
    int (*check)(THD*, struct st_mysql_sys_var *, void*, st_mysql_value*);
    void (*update)(THD*, struct st_mysql_sys_var *, void*, const void*);
};
```

为了方便，定义了一组宏以方便创建新的系统变量。
纵观这些宏，可以使用以下字段：
- name：系统变量的标识符。
- varname：静态变量的标识符。没有时，与name字段相同。
- opt：额外使用的标识位。下表展示了允许的标识：

    |标识值|描述|
    |-----|----|
    |PLUGIN_VAR_READONLY|系统变量是只读的|
    |PLUGIN_VAR_NOSYSVAR|系统变量在运行时对用户不可见|
    |PLUGIN_VAR_NOCMDOPT|系统变量不能通过命令行设置|
    |PLUGIN_VAR_NOCMDARG|在命令行上不要求参数（典型应用例如布尔变量）|
    |PLUGIN_VAR_RQCMDARG|在命令行上要求参数（默认）|
    |PLUGIN_VAR_OPCMDARG|在命令行上参数是可选的|
    |PLUGIN_VAR_MEMALLOC|用于字符串变量；表示为存储字符串分配了内存|

- comment：在服务器帮助信息中显示的说明性注解。
- check：check函数，默认为NULL。
- update：update函数，默认为NULL。
- default：默认值。
- mininum：最小值。
- maximum：最大值。
- blocksize：变量块大小。当设置值时，被圆整到最近的blocksize的倍数。

系统变量既可以通过直接使用静态变量来访问，也可以使用`SYSVAR()`访问器宏定义。一般只应该在代码不能直接访问变量时才使用`SYSVAR()`宏。
例如：
```
static int my_foo;
static MYSQL_SYSVAR_INT(foo_var, my_foo,
                        PLUGIN_VAR_RQCMDARG, "foo comment",
                        NULL, NULL, 0, 0, INT_MAX, 0);
 ...
    SYSVAR(foo_var)= value;
    value= SYSVAR(foo_var);
    my_foo= value;
    value= my_foo;
```

会话变量只能通过`THDVAR()`访问宏定义来访问。例如：
```
static MYSQL_THDVAR_BOOL(some_flag,
                         PLUGIN_VAR_NOCMDARG, "flag comment",
                         NULL, NULL, FALSE);
 ...
    if (THDVAR(thd, some_flag))
    {
        do_something();
        THDVAR(thd, some_flag)= FALSE;
    }
```

所有的全局和会话系统变量必须在使用前发布到mysqld。这是通过构建一个以NULL终止的变量数组并连接到插件公开接口完成的。例如：
```
static struct st_mysql_sys_var *my_plugin_vars[]= {
    MYSQL_SYSVAR(foo_var),
    MYSQL_SYSVAR(some_flag),
    NULL
};
mysql_declare_plugin(fooplug)
{
    MYSQL_..._PLUGIN,
    &plugin_data,
    "fooplug",
    "foo author",
    "This does foo!",
    PLUGIN_LICENSE_GPL,
    foo_init,
    foo_fini,
    0x0001,
    NULL,
    my_plugin_vars,
    NULL,
    0
}
mysql_declare_plugin_end;
```

在内部，所有可修改的系统变量和插件系统变量都存储在哈希表中。

插件应将`thd`参数视为只读的。