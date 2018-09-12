# 在`CREATE TABLE`操作期间获取索引信息

对于支持索引的存储引擎来说，最好在`CREATE TABLE`操作期间提供索引信息并保存以供将来使用。背后的原因是索引信息在表和索引创建时最容易读取到，而之后不易找回。

表的索引信息包含在`create()`方法的`TABLE`参数的`key_info`结构体中。

在`key_info`结构体中有一个`flag`定义了索引的行为：
```
#define HA_NOSAME             1  /* Set if not duplicated records   */
#define HA_PACK_KEY           2  /* Pack string key to previous key */
#define HA_AUTO_KEY           16
#define HA_BINARY_PACK_KEY    32 /* Packing of all keys to prev key */
#define HA_FULLTEXT          128 /* For full-text search            */
#define HA_UNIQUE_CHECK      256 /* Check the key for uniqueness    */
#define HA_SPATIAL          1024 /* For spatial search              */
#define HA_NULL_ARE_EQUAL   2048 /* NULL in key are cmp as equal    */
#define HA_GENERATED_KEY    8192 /* Automatically generated key     */
```

除了flag，还有一个名为`algorithm`的枚举指定了索引类型：
```
enum ha_key_alg {
  HA_KEY_ALG_UNDEF=     0,  /* Not specified (old file)     */
  HA_KEY_ALG_BTREE=     1,  /* B-tree, default one          */
  HA_KEY_ALG_RTREE=     2,  /* R-tree, for spatial searches */
  HA_KEY_ALG_HASH=      3,  /* HASH keys (HEAP tables)      */
  HA_KEY_ALG_FULLTEXT=  4   /* FULLTEXT (MyISAM tables)     */
};
```

除了`flag`和`algorithm`，还有一个`key_part`元素组成的数组，描述了组合键的每一个部分。
