---
layout: doc_page
---

# Rackspace 云文件

## 深存储

[Rackspace 云文件](http://www.rackspace.com/cloud/files/)是另一个深存储选择。

|属性|可能的值|描述|默认|
|--------|---------------|-----------|-------|
|`druid.storage.type`|cloudfiles||Must be set.|
|`druid.storage.region`||Rackspace Cloud Files region.|Must be set.|
|`druid.storage.container`||Rackspace Cloud Files container name.|Must be set.|
|`druid.storage.basePath`||Rackspace Cloud Files base path to use in the container.|Must be set.|
|`druid.storage.operationMaxRetries`||Number of tries before cancel a Rackspace operation.|10|
|`druid.cloudfiles.userName`||Rackspace Cloud username|Must be set.|
|`druid.cloudfiles.apiKey`||Rackspace Cloud api key.|Must be set.|
|`druid.cloudfiles.provider`|rackspace-cloudfiles-us,rackspace-cloudfiles-uk|Name of the provider depending on the region.|Must be set.|
|`druid.cloudfiles.useServiceNet`|true,false|Whether to use the internal service net.|true|

## Firehose

#### StaticCloudFilesFirehose

这个Firehose摄取事件与StaticAzureBlobStoreFirehose相似，但是它是从Rackspace云文件摄取。
数据是换行分隔符，每行有一个JSON对象，而且按照每个 `InputRowParser`配置解析。

blobs与Racksapce云文件深存储函数共享一个存储帐号,但是blobs可以在一个不同的容器里。

与Azure blobstore一样,如果扩展以.gz结尾，则假定为gzipped。

示例规范：

```json
"firehose" : {
    "type" : "static-cloudfiles",
    "blobs": [
        {
          "region": "DFW"
          "container": "container",
          "path": "/path/to/your/file.json"
        },
        {
          "region": "ORD"
          "container": "anothercontainer",
          "path": "/another/path.json"
        }
    ]
}
```

|属性|可能的值|描述|默认|
|--------|-----------|-------|---------|
|type|This should be "static-cloudfiles".|N/A|yes|
|blobs|JSON array of Cloud Files blobs.|N/A|yes|

云文件Blobs:

|属性|可能的值|描述|默认|
|--------|-----------|-------|---------|
|container|Name of the Cloud Files container|N/A|yes|
|path|The path where data is located.|N/A|yes|
