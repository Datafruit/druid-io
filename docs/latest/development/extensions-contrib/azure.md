---
layout: doc_page
---

# Microsoft Azure

## 深存储
 
[Microsoft Azure Storage](http://azure.microsoft.com/en-us/services/storage/) 是深存储的另一个选择。
这个要求一些额外的Druid配置。

|属性|可能的值|描述|默认|
|--------|---------------|-----------|-------|
|`druid.storage.type`|azure||Must be set.|
|`druid.azure.account`||Azure Storage account name.|Must be set.|
|`druid.azure.key`||Azure Storage account key.|Must be set.|
|`druid.azure.container`||Azure Storage container name.|Must be set.|
|`druid.azure.protocol`|http or https||https|
|`druid.azure.maxTries`||Number of tries before cancel an Azure operation.|3|

查阅[Azure Services](http://azure.microsoft.com/en-us/pricing/free-trial/) 了解更多。

## Firehose

#### StaticAzureBlobStoreFirehose
这个Firehose摄取事件与 StaticS3Firehose相似，但它是从Azure Blob Store摄取。

数据是换行分隔符，每行有一个JSON对象，而且按照每个 `InputRowParser`配置解析。

blobs与Azure深存储函数共享一个存储帐号,但是blobs可以在一个不同的容器里。

与S3 blobstore一样,如果扩展以.gz结尾，则假定为gzipped。
示例规范：

```json
"firehose" : {
    "type" : "static-azure-blobstore",
    "blobs": [
        {
          "container": "container",
          "path": "/path/to/your/file.json"
        },
        {
          "container": "anothercontainer",
          "path": "/another/path.json"
        }
    ]
}
```

|属性|描述|默认|要求？|
|--------|-----------|-------|---------|
|type|This should be "static-azure-blobstore".|N/A|yes|
|blobs|JSON array of [Azure blobs](https://msdn.microsoft.com/en-us/library/azure/ee691964.aspx).|N/A|yes|

Azure Blobs:

|属性|描述|默认|要求？|
|--------|-----------|-------|---------|
|container|Name of the azure [container](https://azure.microsoft.com/en-us/documentation/articles/storage-dotnet-how-to-use-blobs/#create-a-container)|N/A|yes|
|path|The path where data is located.|N/A|yes|
