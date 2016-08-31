---
layout: doc_page
---

# Deep Storage存储

deep存储是Segment存储的地方。这是Druid不支持的存储机制。这个deep存储基础设置定义你数据的耐久性水平，只要Druid节点可以看到这个存储基础设置和得到Segment存储，不管你丢失多少Druid节点，你都不会丢失数据。
如果Segment从这个存储层消失，那么你将会失去这些片段代表的任何数据。
## 本地安装

一个本地安装也可以被用来存储Segment。只要你的本地文件系统或者任何其他的可以本地安装像NFS，Ceph等等。这个是默认deep存储的。

为了让deep存储使用本地安装，你需要在通用配置设置下面配置。
|属性|可能的值|描述|默认|
|--------|---------------|-----------|-------|
|`druid.storage.type`|local||Must be set.|
|`druid.storage.storageDirectory`||Directory for storing segments.|Must be set.|

注意，你应该设置`druid.storage.storageDirectory`与`druid.segmentCache.locations` 和 `druid.segmentCache.infoDir`不同。

如果你正在本地模式下使用Hadoop程序，那么只要给它一个本地文件作为你的输出目录，它将开始工作。
## S3-compatible

查阅[druid-s3-extensions 扩展文档](../development/extensions-core/s3.html)
## HDFS

查阅[druid-hdfs-storage 扩展文档](../development/extensions-core/hdfs.html)
## 额外的deep存储

对于额外的deep存储，请看我们的[扩展列表](../development/extensions.html)