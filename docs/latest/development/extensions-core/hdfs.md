---
layout: doc_page
---

# HDFS

请确保[包含](../../operations/including-extensions.html) `druid-hdfs-storage` 作为一个扩展。

## 深存储 

### 配置

|属性|可能的值|描述|默认|
|--------|---------------|-----------|-------|
|`druid.storage.type`|hdfs||Must be set.|
|`druid.storage.storageDirectory`||Directory for storing segments.|Must be set.|

如果你正在使用Hadoop索引，设置你的输出目录为Hadoop上的位置，然后它将开始工作。