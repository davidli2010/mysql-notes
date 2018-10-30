# 数据类型

## 数值类型

- BIT[(M)]

  位值类型。M表示每个值的位数，从1到64，默认是1。

  内部类型：MYSQL_TYPE_BIT

- TINYINT[(M)] [UNSIGNED][ZEROFILL]

  非常小的整数。有符号型的范围是-128到127，无符号型的范围是0到255。

  内部类型：MYSQL_TYPE_TINY

- BOOL, BOOLEAN

  这两个类型是TINYINT(1)的同义词。0值为false，非0值为true。

  内部类型：MYSQL_TYPE_TINY

- SMALLINT[(M)] [UNSIGNED][ZEROFILL]

  小整数。有符号型的范围是-32768到32767，无符号型的范围是0到 65535。

  内部类型：MYSQL_TYPE_SHORT

- MEDIUMINT[(M)] [UNSIGNED][ZEROFILL]

  中等大小的整数。有符号类型的范围是-8388608到8388607，无符号型的范围是0到 16777215。

  内部类型：MYSQL_TYPE_INT24

- INT[(M)] [UNSIGNED][ZEROFILL]

  正常大小的整数。有符号型的范围是-2147483648到2147483647，无符号型的范围是0到4294967295。

  内部类型：MYSQL_TYPE_LONG

- INTEGER[(M)] [UNSIGNED][ZEROFILL]

  这是INT的同义词。

- BIGINT[(M)] [UNSIGNED][ZEROFILL]

  大整数。有符号型的范围是-9223372036854775808到 9223372036854775807，无符号型的范围是0到 18446744073709551615。

  内部类型：MYSQL_TYPE_LONGLONG

- DECIMAL[(M[,D])] [UNSIGNED][ZEROFILL]

  精确的定点数。M是数字的总数，D是小数点后数字的数量。如果D是0，那么久没有小数点或小数部分。M的最大值是65。支持的D的最大值是30。如果D被忽略，默认是0。如果M被忽略，默认是10。

  内部类型：MYSQL_TYPE_DECIMAL，MYSQL_TYPE_NEWDECIMAL

- DEC[(M[,D])] [UNSIGNED][ZEROFILL]

  这是DECIMAL的同义词。

- FLOAT[(M,D)] [UNSIGNED][ZEROFILL]

  单精度浮点数。允许的值是-3.402823466E+38到-1.175494351E-38，0，和1.175494351E-38到3.402823466E+38。这是基于IEEE标准的理论上的限制，实际范围可能会稍微小一点，这依赖硬件或操作系统。

  M是总的位数，D是小数点后的位数。

  内部类型：MYSQL_TYPE_FLOAT

- DOUBLE[(M[,D])] [UNSIGNED][ZEROFILL]

  双精度浮点数。允许的值是-1.7976931348623157E+308到-2.2250738585072014E-308，0，和2.2250738585072014E-308到1.7976931348623157E+308。

  M是总的位数，D是小数点后的位数。

  内部类型：MYSQL_TYPE_DOUBLE

## 日期与时间类型

- DATE

  日期，支持的范围从'1000-01-01'到'9999-12-31'。以'YYYY-MM-DD'的格式显示，但是允许使用字符串或数字赋值DATE列。

  内部类型：MYSQL_TYPE_DATE，MYSQL_TYPE_NEWDATE

- TIME[(fsp)]

  时间。范围是'-838:59:59.000000'到'838:59:59.000000'。以'HH:MM:SS[.fraction]'的格式显示，但是允许以字符串或数字赋值TIME列。

  内部类型：MYSQL_TYPE_TIME，MYSQL_TYPE_TIME2

- DATETIME[(fsp)]

  日期和时间的组合。支持的范围是'1000-01-01 00:00:00.000000'到'9999-12-31 23:59:59.999999'。以'YYYY-MM-DD HH:MM:SS[.fraction]'的格式显示，但是允许以字符串或数字赋值DATETIME列。

  内部类型：MYSQL_TYPE_DATETIME，MYSQL_TYPE_DATETIME2

- TIMESTAMP[(fsp)]

  时间戳，范围是'1970-01-01 00:00:01.000000' UTC到'2038-01-19 03:14:07.999999' UTC。TIMESTAMP的值以自'1970-01-01 00:00:00' UTC以来的秒数存储。TIMESTAMP不能表示'1970-01-01 00:00:00'，因为这等于0秒，而0秒被保留表示'0000-00-00 00:00:00'，TIMESTAMP的零值。

  内部类型：MYSQL_TYPE_TIMESTAMP，MYSQL_TYPE_TIMESTAMP2

- YEAR[(4)]

  4位数格式的年。值显示为1901到2155，以及0000。

  内部类型：MYSQL_TYPE_YEAR

## 字符串类型

- [NATIONAL] CHAR[(M)] [CHARACTER SET charset_name] [COLLATE collation_name]

  定长字符串，存储时总是向右填充空格到指定长度。M表示列的长度，范围从0到255。如果M被忽略，则长度为1。

  > 当检索CHAR值时，尾部的空格被移除。除非启动`PAD_CHAR_TO_FULL_LENGTH`SQL模式。

  内部类型：MYSQL_TYPE_VAR_STRING， MYSQL_TYPE_STRING

- [NATIONAL] VARCHAR(M) [CHARACTER SET charset_name] [COLLATE collation_name]

  变长字符串。M表示列的最大字符长度。M的范围从0到65535。

  > MySQL遵循标准SQL规范，不从VARCHAR值中删除尾部的空格。

  内部类型：MYSQL_TYPE_VARCHAR

- BINARY[(M)]

  与CHAR类型类似，但是存储的是二进制字节而不是非二进制的字符。M表示列的字节长度，默认是1。

  内部类型：MYSQL_TYPE_VAR_STRING， MYSQL_TYPE_STRING

- VARBINARY(M)

  与VARCHAR类型类似，但是存储的是二进制字节而不是非二进制的字符。M表示列的字节最大长度。

  内部类型：MYSQL_TYPE_VARCHAR

- TINYBLOB

  最大长度为255字节的BLOB列。每个TINYBLOB使用一个字节的长度前缀说明字节数量。

  内部类型：MYSQL_TYPE_BLOB

- TINYTEXT [CHARACTER SET charset_name] [COLLATE collation_name]

  最大长度为255字符的TEXT列。

  内部类型：MYSQL_TYPE_BLOB

- BLOB[(M)]

  最大长度为65535个字节的BLOB列。每个BLOB使用2个字节的长度前缀说明字节数量。

  内部类型：MYSQL_TYPE_BLOB

- TEXT[(M)] [CHARACTER SET charset_name] [COLLATE collation_name]

  最大长度为65535个字符的TEXT列。

  内部类型：MYSQL_TYPE_BLOB

- MEDIUMBLOB

  最大长度为16777215字节的BLOB列。每个MEDIUMBLOB使用3个字节的长度前缀说明字节数量。

  内部类型：MYSQL_TYPE_BLOB

- MEDIUMTEXT [CHARACTER SET charset_name] [COLLATE collation_name]

  最大长度为16777215个字符的TEXT列。

  内部类型：MYSQL_TYPE_BLOB

- LONGBLOB

  最大长度为4GB字节的BLOB列。每个LONGBLOB使用4个字节的长度前缀说明字节数量。

  内部类型：MYSQL_TYPE_BLOB

- LONGTEXT [CHARACTER SET charset_name] [COLLATE collation_name]

  最大长度为4GB个字符的TEXT列。

  内部类型：MYSQL_TYPE_BLOB

- ENUM('value1','value2',...) [CHARACTER SET charset_name] [COLLATE collation_name]

  枚举。只能有一个值的字符串对象，从值列表'value1','value2',...,NULL选择或者特殊的''错误值。ENUM值在内部表示为整数。

  ENUM列最大可以有个65535个不重复的元素（实际的限制是少于3000）。将ENUM和SET列看作一组，一个表可以有不大于255个唯一的元素列表。

  内部类型：MYSQL_TYPE_STRING，MYSQL_TYPE_ENUM(real_type)

- SET('value1','value2',...) [CHARACTER SET charset_name] [COLLATE collation_name]

  集合。可以有零或多个字符串对象，每一个都必须从值列表'value1','value2',...中选择。SET值在内部表示为整数。

  SET列最大可以有64个唯一的成员。

  内部类型：MYSQL_TYPE_STRING，MYSQL_TYPE_SET(real_type)

## 空间数据类型

内部类型：MYSQL_TYPE_GEOMETRY

单一值类型：
- GEOMETRY
- POINT
- LINESTRING
- POLYGON

值集合：
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
