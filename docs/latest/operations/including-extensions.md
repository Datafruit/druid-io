---
layout: doc_page
---
# 包括扩展名

Druid使用一个模块系统,允许在运行时添加扩展。核心扩展于是与Druid tarball绑定的。
社区扩展可以在本地下载通过[pull-deps](../operations/pull-deps.html)工具。
## 下载扩展名

核心Druid扩展已经捆绑在Druid释放压缩文件。你可以通过在[druid.io](http://druid.io/downloads.html)下载压缩文件得到扩展名。
解压压缩文件：你将看到一个```扩展```文件夹包含所有的核心扩展，还有一个```hadoop-dependencies```文件夹包含所有的hadoop扩展。每个扩展将有自己的文件夹，包含扩展jars。
然而，因为未批准，所以我们没有打包 mysql-metadata-storage 扩展到扩展文件夹。为了得到这个扩展，你可以从[druid.io](http://druid.io/downloads.html)自行下载，然后移动到```扩展```目录。

可选地,您可以使用“pull-deps”工具下载你想要的扩展。
查阅[pull-deps](../operations/pull-deps.html)了解完整例子。

## 加载扩展

Druid加载扩展有两种方式。
### 从类路径加载

如果你在运行时将您的扩展jar添加到类路径中，Druid将把它加载到系统。
这种机制相对容易推出,但这也意味着你必须确保所有依赖jar文件的类路径中是兼容的。
也就是说,Druid使用时没有规定这种方法保持类加载隔离,所以你必须确保类路径上的jars是相互兼容的

### 从扩展目录加载

如果你不想混淆类路径，你可以让Druid从扩展目录加载扩展。
为了让Druid加载你的扩展，请跟下面的步骤操作

**告诉Druid你的扩展在哪儿**
指定 `druid.extensions.directory` （根目录包含Druid扩展）。查阅 [配置](../configuration/index.html)。
这个属性的值应该设置为包含所有的扩展文件夹的绝对路径。
一般来讲，你应该简单的重复释放压缩文件的扩展目录（即，```扩展```）。

示例：
支持你指定 `druid.extensions.directory=/usr/local/druid_tarball/extensions`
然后在```extensions```下，结构应该是这样的，

```
extensions/
├── druid-kafka-eight
│   ├── druid-kafka-eight-0.7.3.jar
│   ├── jline-0.9.94.jar
│   ├── jopt-simple-3.2.jar
│   ├── kafka-clients-0.8.2.1.jar
│   ├── kafka_2.10-0.8.2.1.jar
│   ├── log4j-1.2.16.jar
│   ├── lz4-1.3.0.jar
│   ├── metrics-core-2.2.0.jar
│   ├── netty-3.7.0.Final.jar
│   ├── scala-library-2.10.4.jar
│   ├── slf4j-log4j12-1.6.1.jar
│   ├── snappy-java-1.1.1.6.jar
│   ├── zkclient-0.3.jar
└── mysql-metadata-storage
    ├── mysql-connector-java-5.1.34.jar
    └── mysql-metadata-storage-0.9.0.jar
```

正如你所看到的,```扩展```下面有两个子目录``` druid-kafka-eight```和``` mysql-metadata-storage```。
每个子目录表示一个扩展,Druid可以负载。

**告诉Druid加载什么扩展**
使用`druid.extensions.loadList`(查阅 [配置](../configuration/index.html) )指定通过Druid加载的一个扩展名列表。

例如，`druid.extensions.loadList=["druid-kafka-eight", "mysql-metadata-storage"]` 命令Druid加载`druid-kafka-eight` 和`mysql-metdata-storage`扩展。
也就是说，你在列表里指定的名应该与它扩展文件夹名一致。

如果你指定 `druid.extensions.loadList=[]`，Druid不会从文件系统加载任何扩展。
如果你没有指定`druid.extensions.loadList`，Druid将通过`druid.extensions.directory`指定目录下加载所有的扩展。