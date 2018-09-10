# MySQL Notes

## Reference
- [MySQL Internals Manual](https://dev.mysql.com/doc/internals/en/)
- [MySQL 5.7 Reference Manual](https://dev.mysql.com/doc/refman/5.7/en/)

## MySQL Internals

- [MySQL Threads](mysql-threads.md)
- [The MySQL Plugin API](mysql-plugin-api.md)
    - [Types of Plugins](types-of-plugin.md)
    - [Plugin API Characteristics](plugin-api-characteristics.md)
    - [Plugin API Components](plugin-api-components.md)
    - [Writing Plugins](writing-plugins.md)
    - [Plugin Data Structures](plugin-data-structures.md)
- [MySQL Services for Plugins](mysql-services-for-plugins.md)
- [Writing a Custom Storge Engine](writing-a-custom-storage-engine.md)
    - [Creating the handlerton](creating-the-handlerton.md)
    - [Handling Handler Instantiation](handling-handler-instantiation.md)
    - [Defining Filename Extensions](defining-filename-extensions.md)
    - [Creating Tables](creating-tables.md)
    - [Opening a Table](opening-a-table.md)
    - [Closing a Table](closing-a-table.md)
    - [Implementing Basic Table Scanning](implementing-basic-table-scanning.md)
    - [Adding Support for INSERT](adding-support-for-insert.md)
    - [Adding Support for UPDATE](adding-support-for-update.md)
    - [Adding Support for DELETE](adding-support-for-delete.md)