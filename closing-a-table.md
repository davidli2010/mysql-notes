# 关闭表

当MySQL服务器在表上完成操作之后，它会调用`handler::close()`方法来关闭表的指针并释放其它所有资源。

像CSV引擎这样使用共享访问方法的存储引擎必须将自己从共享结构中删除：
```
int ha_tina::close(void)
{
    int rc= 0;
    DBUG_ENTER("ha_tina::close");
    rc= mysql_file_close(data_file, MYF(0));
    DBUG_RETURN(free_share(share) || rc);
}
```

使用自己的共享管理系统的存储引擎应该使用必要的方法为在handler中打开的表将handler实例从共享中删除。