# MySQL线程

> 参考[MySQL Threads](https://dev.mysql.com/doc/refman/5.7/en/mysql-threads.html)

MySQL服务器创建以下线程：
- 连接管理线程处理服务器监听的网络接口上的客户端连接请求。在所有平台上，一个管理线程处理TCP/IP连接请求。在Unix上，这个管理线程也处理Unix套接字文件连接请求。在Windows上，一个管理线程处理共享内存连接请求，同时另一个管理线程处理命名管道连接请求。服务器不会创建线程去处理它不监听的接口。例如，没有启用支持命名管道连接的Windows服务器不会创建线程处理这类请求。
- 连接管理线程为每一个客户端连接关联一个线程来处理连接的认证和请求。管理线程在必要时创建一个新线程，但是它会通过首先咨询线程缓存是否有可用线程来尝试避免这样做。当一个连接结束时，如果线程缓存未满，它的线程会返回到线程缓存中。
- 在主复制服务器（master replication server）上，来自从服务器上的连接与客户端连接一样处理：每一个连接的从服务器一个线程。
- 在从复制服务器（slave replication server）上，一个I/O线程被启动以连接主服务器，并且读取更新。一个SQL线程被启动以应用从主服务器上读到的更新。这两个线程独立运行，并且可以独立的启动和停止。
- 一个信号线程处理所有的信号。这个线程也处理告警，调用`process_alarm()`来强制空闲太长时间的连接超时。
- 如果使用了InnoDB，默认会有额外的读写线程。这些线程的数量由`innodb_read_io_threads`和`innodb_write_io_threads`参数控制。
- 如果服务器启动时指定参数`--flush_time=val`，相应的会创建一个线程，每隔val秒刷新一次所有的表。
- 如果事件调度器激活了，则调度器会有一个线程，每个运行的事件会有一个线程。
