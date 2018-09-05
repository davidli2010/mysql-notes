# 插件类型

插件API可以创建实现了以下功能的插件：
- 存储引擎（storage engines）
- 全文解析器（full-text parsers）
- 后台（daemons）
- INFORMATION_SCHEMA表
- 半同步复制（semisynchronous replication）
- 审计（auditing）
- 认证（authentication）
- 密码验证与长度检查（password validation and strength checking）
- 协议跟踪（protocol tracing）
- 查询重写（query rewriting）
- 安全密钥环存储和检索（secure keyring storage and retrieval）

## 存储引擎（storage engines）

MySQL服务器使用的可插拔存储引擎架构使得存储引擎可以实现为插件，并且在运行的服务器上加载或卸载。

对于如何使用插件API来编写存储引擎的信息，可以查阅[MySQL Internals: Writing a Custom Storage Engine](https://dev.mysql.com/doc/internals/en/custom-engine.html)。