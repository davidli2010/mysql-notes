# 数据类型

## 数值类型

- BIT
- TINYINT [UNSIGNED][ZEROFILL]
- BOOL, BOOLEAN
- SMALLINT [UNSIGNED][ZEROFILL]
- MEDIUMINT [UNSIGNED][ZEROFILL]
- INT [UNSIGNED][ZEROFILL]
    - INTEGER [UNSIGNED][ZEROFILL]
- BIGINT [UNSIGNED][ZEROFILL]
- DECIMAL [UNSIGNED][ZEROFILL]
    - DEC [UNSIGNED][ZEROFILL]
- FLOAT [UNSIGNED][ZEROFILL]
- DOUBLE [UNSIGNED][ZEROFILL]

## 日期与时间类型

- DATE
- DATETIME
- TIMESTAMP
- TIME
- YEAR

## 字符串类型

- [NATIONAL] CHAR[(M)] [CHARACTER SET charset_name] [COLLATE collation_name]
- [NATIONAL] VARCHAR(M) [CHARACTER SET charset_name] [COLLATE collation_name]
- BINARY
- VARBINARY
- TINYBLOB
- TINYTEXT [CHARACTER SET charset_name] [COLLATE collation_name]
- BLOB
- TEXT[(M)] [CHARACTER SET charset_name] [COLLATE collation_name]
- MEDIUMBLOB
- MEDIUMTEXT [CHARACTER SET charset_name] [COLLATE collation_name]
- LONGBLOB
- LONGTEXT [CHARACTER SET charset_name] [COLLATE collation_name]
- ENUM('value1','value2',...) [CHARACTER SET charset_name] [COLLATE collation_name]
- SET('value1','value2',...) [CHARACTER SET charset_name] [COLLATE collation_name]

## 空间数据类型

- GEOMETRY
- POINT
- LINESTRING
- POLYGON
- MULTIPOINT
- MULTILINESTRING
- MULTIPOLYGON
- GEOMETRYCOLLECTION

## JSON数据类型

Native JSON data type defined by [RFC7158](https://tools.ietf.org/html/rfc7159).

通过在字符串列中存储JSON格式的字符串，JSON数据类型提供了这些优势：
- 自动校验JSON列中存储的JSON文档。非法的文档会产生错误。
- 优化的存储格式。JSON列中存储的JSON文档被转换成内部格式，允许快速读取文档元素。当服务器需要读取存储在二进制格式中的JSON值时，不需要从文本描述中解析。二进制格式的结构使得服务器可以按键或数组索引直接查找子对象或嵌套的值而不需要读取文档中前后的所有值。

存储JSON文档的空间的要求与LONGBLOB或LONGTEXT相同。

JSON列不能有默认值。

对于JSON数据类型，有一组SQL函数可以在JSON值上操作，比如创建、操纵和搜索。

JSON列与其它的二进制类型的列一样不能直接索引。可以在从JSON列中取出的标量值生成的列上创建索引。

MySQL优化器会寻找匹配JSON表达式的虚拟列上的兼容的索引。

`JSON_TYPE()`函数返回utf8mb4字符串说明JSON值的类型。下面列出了可能的返回值：
- Purely JSON types:
    - OBJECT: JSON objects
    - ARRAY: JSON arrays
    - BOOLEAN: The JSON true and false literals
    - NULL: The JSON null literal 
- Numeric types:
    - INTEGER: MySQL TINYINT, SMALLINT, MEDIUMINT and INT and BIGINT scalars
    - DOUBLE: MySQL DOUBLE FLOAT scalars
    - DECIMAL: MySQL DECIMAL and NUMERIC scalars 
- Temporal types:
    - DATETIME: MySQL DATETIME and TIMESTAMP scalars
    - DATE: MySQL DATE scalars
    - TIME: MySQL TIME scalars 
- String types:
    - STRING: MySQL utf8 character type scalars: CHAR, VARCHAR, TEXT, ENUM, and SET 
- Binary types:
    - BLOB: MySQL binary type scalars: BINARY, VARBINARY, BLOB
    - BIT: MySQL BIT scalars 
- All other types:
    - OPAQUE (raw bits) 
