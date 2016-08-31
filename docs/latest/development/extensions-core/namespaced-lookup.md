---
layout: doc_page
---

# 命名空间查找

<div class="note caution">
查找是一个 <a href="../development/experimental.html">试验</a> 特性。
</div>

请确保[包含](../../operations/including-extensions.html) `druid-namespace-lookup`作为一个扩展。
## Configuration配置

|属性|描述|默认|
|--------|-----------|-------|
|`druid.lookup.snapshotWorkingDir`| Working path used to store snapshot of current lookup configuration, leaving this property null will disable snapshot/bootstrap utility|null|

命名空间适当地查找由于其尺寸不能在查询时通过, 或在查询时不期望通过,因为数据是驻留在Druid和被Druid服务器处理的。
可以指定命名空间为运行属性文件的一部分。属性是这个页面每部分描述的一个命名空间列表。例如:

 ```json
 druid.query.extraction.namespace.lookups=
   [
     {
       "type": "uri",
       "namespace": "some_uri_lookup",
       "uri": "file:/tmp/prefix/",
       "namespaceParseSpec": {
         "format": "csv",
         "columns": [
           "key",
           "value"
         ]
       },
       "pollPeriod": "PT5M"
     },
     {
       "type": "jdbc",
       "namespace": "some_jdbc_lookup",
       "connectorConfig": {
         "createTables": true,
         "connectURI": "jdbc:mysql:\/\/localhost:3306\/druid",
         "user": "druid",
         "password": "diurd"
       },
       "table": "lookupTable",
       "keyColumn": "mykeyColumn",
       "valueColumn": "MyValueColumn",
       "tsColumn": "timeColumn"
     }
   ]
 ```

适当的名称空查找功能需要加载以下扩展代理,劳工,和历史的节点:
`druid-namespace-lookup`

## 缓存设置

查找是本地历史节点的缓存。下面的设置是使用了设置命名空间（代理，劳工，历史）的服务查询节点

|属性|描述|默认|
|--------|-----------|-------|
|`druid.query.extraction.namespace.cache.type`|Specifies the type of caching to be used by the namespaces. May be one of [`offHeap`, `onHeap`]. `offHeap` uses a temporary file for off-heap storage of the namespace (memory mapped files). `onHeap` stores all cache on the heap in standard java map types.|`onHeap`|

缓存的填充方式取决于下面的设置。一般来说,大多数名称空间在结尾使用`pollPeriod`，他们调查远程资源的更新时间。

# 支持查找

对于额外的查找，请查阅我们的[扩展列表](../extensions.html)。
## URI 命名空间更新

每个名称空间查找重新映射值可以由每个json指定

```json
{
  "type":"uri",
  "namespace":"some_lookup",
  "uri": "s3://bucket/some/key/prefix/",
  "namespaceParseSpec":{
    "format":"csv",
    "columns":["key","value"]
  },
  "pollPeriod":"PT5M",
  "versionRegex": "renames-[0-9]*\\.gz"
}
```

|属性|描述|要求|默认|
|--------|-----------|--------|-------|
|`namespace`|The namespace to define|Yes||
|`pollPeriod`|Period between polling for updates|No|0 (only once)|
|`versionRegex`|Regex to help find newer versions of the namespace data|Yes||
|`namespaceParseSpec`|How to interpret the data at the URI|Yes||

`pollPeriod`值指定在检查更新之间ISO 8601格式的期间。如果查找的资源可以提供时间戳，查找将只有在改变`pollPeriod`优先权后才更新。
值为0时，当作缺省参数，或者`null`意思是填充一次,不要试图更新。
当更新时，更新系统将查找一个最近的时间戳并假设是最近的数据的文件。

`versionRegex`值指定一个正则表达式来确定当试图找到最新版本时，uri的父路径文件名是否应该考虑。
省略此设置或设置它等于`null`将匹配所有文件能找到(相当于使用`".*"`)。搜索是uri中最重要的“目录”。

`namespaceParseSpec`可以是一个数值。下面的例子将重命名foo为bar,baz为bat，还有buck为truck。
所有parseSpec类型假设每个输入由一个新行分隔。见下文parseSpec类型的支持。

### csv lookupParseSpec

|参数|描述|要求|默认|
|---------|-----------|--------|-------|
|`columns`|The list of columns in the csv file|yes|`null`|
|`keyColumn`|The name of the column containing the key|no|The first column|
|`valueColumn`|The name of the column containing the value|no|The second column|

*输入示例*

```
bar,something,foo
bat,something2,baz
truck,something3,buck
```

*namespaceParseSpec示例*

```json
"namespaceParseSpec": {
  "format": "csv",
  "columns": ["value","somethingElse","key"],
  "keyColumn": "key",
  "valueColumn": "value"
}
```

### tsv lookupParseSpec

|Parameter|Description|Required|Default|
|---------|-----------|--------|-------|
|`columns`|The list of columns in the csv file|yes|`null`|
|`keyColumn`|The name of the column containing the key|no|The first column|
|`valueColumn`|The name of the column containing the value|no|The second column|
|`delimiter`|The delimiter in the file|no|tab (`\t`)|

*输入示例*

```
bar|something,1|foo
bat|something,2|baz
truck|something,3|buck
```

*namespaceParseSpec示例*

```json
"namespaceParseSpec": {
  "format": "tsv",
  "columns": ["value","somethingElse","key"],
  "keyColumn": "key",
  "valueColumn": "value",
  "delimiter": "|"
}
```

### 自定义Json lookupParseSpec

|Parameter|Description|Required|Default|
|---------|-----------|--------|-------|
|`keyFieldName`|The field name of the key|yes|null|
|`valueFieldName`|The field name of the value|yes|null|

*输入示例*

```json
{"key": "foo", "value": "bar", "somethingElse" : "something"}
{"key": "baz", "value": "bat", "somethingElse" : "something"}
{"key": "buck", "somethingElse": "something", "value": "truck"}
```

*namespaceParseSpec示例*

```json
"namespaceParseSpec": {
  "format": "customJson",
  "keyFieldName": "key",
  "valueFieldName": "value"
}
```


### simpleJson lookupParseSpec

`simpleJson` lookupParseSpec不需要任何参数。它是简单的一个行分隔的json文件，其字段的位置是键,和字段的值是值。

*输入示例*
 
```json
{"foo": "bar"}
{"baz": "bat"}
{"buck": "truck"}
```

*namespaceParseSpec示例*

```json
"namespaceParseSpec":{
  "format": "simpleJson"
}
```

## JDBC 命名空间查找

JDBC查找将选择数据库填充它的本地缓存。如果设置了`tsColumn`,则它必须能够接受`'2015-01-01 00:00:00'`格式。
例如，下面必须是有效的表格sql语句 `SELECT * FROM some_lookup_table WHERE timestamp_column >  '2015-01-01 00:00:00'`。
如果设置了`tsColumn`，缓存服务将尝试只选择最后同步的写为*after*的值。如果没有设置`tsColumn`，则每次都会拉动整个表格。

|参数|描述|要求|默认|
|---------|-----------|--------|-------|
|`namespace`|The namespace to define|Yes||
|`connectorConfig`|The connector config to use|Yes||
|`table`|The table which contains the key value pairs|Yes||
|`keyColumn`|The column in `table` which contains the keys|Yes||
|`valueColumn`|The column in `table` which contains the values|Yes||
|`tsColumn`| The column in `table` which contains when the key was updated|No|Not used|
|`pollPeriod`|How often to poll the DB|No|0 (only once)|

```json
{
  "type":"jdbc",
  "namespace":"some_lookup",
  "connectorConfig":{
    "createTables":true,
    "connectURI":"jdbc:mysql://localhost:3306/druid",
    "user":"druid",
    "password":"diurd"
  },
  "table":"some_lookup_table",
  "keyColumn":"the_old_dim_value",
  "valueColumn":"the_new_dim_value",
  "tsColumn":"timestamp_column",
  "pollPeriod":600000
}
```
