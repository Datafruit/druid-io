---
layout: doc_page
---

# 批量数据接入

Druid能够通过这里介绍的各种方法从静态文件中读取数据。

## 基于Hadoop的批量接入

Druid中基于Hadoop的批量接入通过Hadoop-ingestion任务实现，这些任务可以发送到Druid[overlord](../design/indexing-service.html)的一个运行实例。下面是一个任务的例子：

```json
{
  "type" : "index_hadoop",
  "spec" : {
    "dataSchema" : {
      "dataSource" : "wikipedia",
      "parser" : {
        "type" : "hadoopyString",
        "parseSpec" : {
          "format" : "json",
          "timestampSpec" : {
            "column" : "timestamp",
            "format" : "auto"
          },
          "dimensionsSpec" : {
            "dimensions": ["page","language","user","unpatrolled","newPage","robot","anonymous","namespace","continent","country","region","city"],
            "dimensionExclusions" : [],
            "spatialDimensions" : []
          }
        }
      },
      "metricsSpec" : [
        {
          "type" : "count",
          "name" : "count"
        },
        {
          "type" : "doubleSum",
          "name" : "added",
          "fieldName" : "added"
        },
        {
          "type" : "doubleSum",
          "name" : "deleted",
          "fieldName" : "deleted"
        },
        {
          "type" : "doubleSum",
          "name" : "delta",
          "fieldName" : "delta"
        }
      ],
      "granularitySpec" : {
        "type" : "uniform",
        "segmentGranularity" : "DAY",
        "queryGranularity" : "NONE",
        "intervals" : [ "2013-08-31/2013-09-01" ]
      }
    },
    "ioConfig" : {
      "type" : "hadoop",
      "inputSpec" : {
        "type" : "static",
        "paths" : "/MyDirectory/example/wikipedia_data.json"
      }
    },
    "tuningConfig" : {
      "type": "hadoop"
    }
  },
  "hadoopDependencyCoordinates": <my_hadoop_version>
}
```

|属性|描述|必须?|
|--------|-----------|---------|
|type|任务类型，总是被设定为“index_hadoop”。|是|
|spec|Hadoop Index Spec. 请参考[Batch Ingestion](../ingestion/batch-ingestion.html)|是|
|hadoopDependencyCoordinates|Druid将使用一个Hadoop依赖的coordinates的JSON数组，这个属性将覆盖默认的Hadoop coordinates。一旦被指定，Druid将会从`druid.extensions.hadoopDependenciesDir`指定的地方查找这些Hadoop依赖。 |否|
|classpathPrefix|Classpath将会在peon处理时被预添加 |否|

### 数据模式

这个字段是必须的。

请参考[数据接入](../ingestion/index.html)

### IOConfig

这个字段是必须的。

|字段|类型|描述|必须|
|-----|----|-----------|--------|
|type|String|设置为'hadoop'。|yes|
|inputSpec|Object|指定数据源。参考下文|yes|
|segmentOutputPath|String|输出segments路径。|yes|
|metadataUpdateSpec|Object|指定如何更新segments所属的Druid集群的元数据|yes|

#### InputSpec 规范

inputSpecs有多种类型:

##### `static`

数据文件的静态路径被传递。

|字段|类型|描述|必须|
|-----|----|-----------|--------|
|paths|Array of String|指定原始数据存放的路径|是|

使用静态路径的例子:

```
"paths" : "s3n://billy-bucket/the/data/is/here/data.gz, s3n://billy-bucket/the/data/is/here/moredata.gz, s3n://billy-bucket/the/data/is/here/evenmoredata.gz"
```

##### `granularity`

期望数据以一种特定的形式展现，比如，期望按照`y=XXXX/m=XX/d=XX/H=XX/M=XX/S=XX`这种格式的目录来隔离每天的数据(数据以小写形式展现，时间以大写形式展现)。

|字段|类型|描述|必须|
|-----|----|-----------|--------|
|dataGranularity|String|指定期望的数据粒度,例如，hour表示期望目录`y=XXXX/m=XX/d=XX/H=XX`.|是|
|inputPath|String|基础路径附加期望的时间路径|是|
|filePattern|String|文件应该匹配包含的模式|是|
|pathFormat|String|每个目录的Joda date-time格式。默认为`"'y'=yyyy/'m'=MM/'d'=dd/'H'=HH"`, 或参考[Joda documentation](http://www.joda.org/joda-time/apidocs/org/joda/time/format/DateTimeFormat.html)|否|

例如，如果样子配置的时间间隔为2012-06-01/2012-06-02，则期望的数据在路径中为：

```
s3n://billy-bucket/the/data/is/here/y=2012/m=06/d=01/H=00
s3n://billy-bucket/the/data/is/here/y=2012/m=06/d=01/H=01
...
s3n://billy-bucket/the/data/is/here/y=2012/m=06/d=01/H=23
```

##### `dataSource`

读取Druid segments. 请参考 [这里](../ingestion/update-existing-data.html) for more information.

##### `multi`

读取多数据源 请参考 [这里](../ingestion/update-existing-data.html) for more information.

### TuningConfig

tuningConfig是可选的，当没有tuningConfig被指定的时候将使用默认参数。

|字段|类型|描述|必须|
|-----|----|-----------|--------|
|workingPath|String|用于中间结果的工作路径（Hadoop作业的结果）。|否 (default == '/tmp/druid-indexing')|
|version|String|创建的segments版本号。|否 (default == datetime that indexing starts at)|
|partitionsSpec|Object|指定如何每个时间段的数据分区到segments，没有这个属性意味着不产生分区。下面会详细介绍。 |否 (default == 'hashed')|
|maxRowsInMemory|Integer|指定在持久化前聚合的行数。为聚合后的数值，所以这个值不等于输入的事件数量，但是等于这些事件输出的被聚合的行数。这个被用来管理JVM所需的堆栈大小|否 (default == 5 million)|
|leaveIntermediate|Boolean|无论通过海事失败，在作业完成时，都会在工作路径下留下中间文件(用于调试)。|否 (default == false)|
|cleanupOnFailure|Boolean|当作业失败时清除中间文件(除非leaveIntermediate设置为on)。|否 (default == true)|
|overwriteFiles|Boolean|创建索引期间覆盖已存在的文件|否 (default == false)|
|ignoreInvalidRows|Boolean|忽略有问题的行。|否 (default == false)|
|useCombiner|Boolean|如果可能，在mapper中用hadoop combiner来合并行|否 (default == false)|
|jobProperties|Object|一个用于添加hadoop作业配置的属性图|否 (default == null)|
|buildV9Directly|Boolean|是否直接编译v9索引去替换v8索引，并转化成v9格式。|否 (default = false)|
|numBackgroundPersistThreads|Integer|指定用于增量持久化的新后台线程的数量，使用此功能会明显增加内存压力和CPU的使用率，但是将让作业更快完成。如果需要修改默认值0，我们建议将其设置为1 。|否 (default == 0)|

### 分区规范

Segments总是基于时间戳来分区（根据granularitySpec ），并且根据分区类型，后续的分区可以用其他的方式，Druid提供2种分区策略：“hasded”（每一行都基于所有维度hash）和“dimension”（基于单一维度的范围）。

大多数情况下，我们建议用hash的分区，因为相对于单维度分区，它会增强索引的性能并且创建更统一的数据segments大小

#### 基于hash的分区

```json
  "partitionsSpec": {
     "type": "hashed",
     "targetPartitionSize": 5000000
   }
```

被hash的分区首先选择多个segments，然后根据每一行所有维度的hash值来对这些segments的行进行分区，这些segments基于输入设置和目标分区大小自动分区。

配置选项如下:

|属性|描述|必须?|
|--------|-----------|---------|
|type|被使用的artitionSpec类型 |"hashed"|
|targetPartitionSize|分区包含的目标行数，应该设置为500MB~1GB的目标segments数。|和numShards二选一|
|numShards|直接指定分区的数量，用于替换目标分区大小。数据接入会更快，因为它能够自动跳过选择分组数量的步骤。|和targetPartitionSize二选一|

#### 单维度分区

```json
  "partitionsSpec": {
     "type": "dimension",
     "targetPartitionSize": 5000000
   }
```

单维度分区首先选择一个需要分区的维度，然后将这个维度分离成一个连续的范围。在这个范围内每一个segment将会包含该维度中所有带数值的行。例如，在范围"a.example.com"到"f.example.com"和"f.example.com"到"z.example.com"中，segments可能在“host”维度上被分区。默认情况下，维度的使用是自动决定的，即使它能够被特定的维度覆盖。

配置选项如下:

|属性|描述|必须?|
|--------|-----------|---------|
|type|被使用的artitionSpec类型 |"dimension"|
|targetPartitionSize|分区包含的目标行数，应该设置为500MB~1GB的目标segments数。|是|
|maxPartitionSize|分区包含的最大行数，默认为比targetPartitionSize大50%|否|
|partitionDimension|要分区的维度。留空会自动选择一个维度|否|
|assumeGrouped|假设输入数据已经被及时分组并且设定了维度。这会使数据接入将会更快，但是如果假设无效，会选择次优的分区|否|

### 远程hadoop集群

如果有个一个远程的hadoop集群，请确保Druid`_common`配置文件夹里面包含`*.xml`配置文件的文件夹。
  
关于Hadoop的版本和Druid的编译版本问题，请参考[这些文档](../operations/other-hadoop.html).

### 使用弹性MapReduce

如果集群运行在AWS(Amazon Web Services)上,弹性MapReduce(EMR)能够被用来从S3索引数据:

- 创建持久化, [long-running cluster](http://docs.aws.amazon.com/ElasticMapReduce/latest/ManagementGuide/emr-plan-longrunning-transient.html).
- 创建集群时, 进去下面的配置。如使用向导，这将是"Edit software settings"下的高级模式。

```
classification=yarn-site,properties=[mapreduce.reduce.memory.mb=6144,mapreduce.reduce.java.opts=-server -Xms2g -Xmx2g -Duser.timezone=UTC -Dfile.encoding=UTF-8 -XX:+PrintGCDetails -XX:+PrintGCTimeStamps,mapreduce.map.java.opts=758,mapreduce.map.java.opts=-server -Xms512m -Xmx512m -Duser.timezone=UTC -Dfile.encoding=UTF-8 -XX:+PrintGCDetails -XX:+PrintGCTimeStamps,mapreduce.task.timeout=1800000]
```

- 在EMR master上，依据`/etc/hadoop/conf`下XML文件"[Configure Hadoop for data loads](../tutorials/cluster.html#configure-cluster-for-hadoop-data-loads)"的说明。

#### 用EMR从S3读取数据

- 在Hadoop索引任务的`tuningConfig`章节`jobProperties`字段下，添加:

```
"jobProperties" : {
   "fs.s3.awsAccessKeyId" : "YOUR_ACCESS_KEY",
   "fs.s3.awsSecretAccessKey" : "YOUR_SECRET_KEY",
   "fs.s3.impl" : "org.apache.hadoop.fs.s3native.NativeS3FileSystem",
   "fs.s3n.awsAccessKeyId" : "YOUR_ACCESS_KEY",
   "fs.s3n.awsSecretAccessKey" : "YOUR_SECRET_KEY",
   "fs.s3n.impl" : "org.apache.hadoop.fs.s3native.NativeS3FileSystem",
   "io.compression.codecs" : "org.apache.hadoop.io.compress.GzipCodec,org.apache.hadoop.io.compress.DefaultCodec,org.apache.hadoop.io.compress.BZip2Codec,org.apache.hadoop.io.compress.SnappyCodec"
}
```

请注意，这个方法适用Hadoop的内置S3文件系统而不是亚马逊的EMRFS，并且不兼容Amazon-specific的功能，比如S3加密和consistent views。如果需要使用这些功能，则需要通过[使用其他Hadoop分布式](#using-other-hadoop-distributions)章节所描述的机制让Druid的Amazon EMR Hadoop JARs可用。

## 使用其他Hadoop分布式

Druid支持与多Hadoop分布式。

如果Druid和Hadoop的版本有冲突，可以在[Druid用户组](https://groups.google.com/forum/#!forum/druid-
user)中搜索解决方案，或者查阅Druid文档[不同Hadoop版本](../operations/other-hadoop.html)。

##命令行Hadoop索引

如果不想用完全索引服务(full indexing service)来使用Hadoop获取数据到Druid，还可以使用独立的命令行的Hadoop索引。更多信息请参考[这里](../ingestion/command-line-hadoop-indexer.html)。

## IndexTask-based 批量接入

如果想要批量接入的Hadoop依赖，也可以使用索引任务。这会比基于Hadoop的方法慢很多，扩展性也会下降。更多信息请参考[这里](../ingestion/tasks.html)。

有问题?
----------------

获取数据到Druid对于第一次使用的用户来说肯定很困难。请毫不犹豫地在我们的IRC通道或者[google groups page](https://groups.google.com/forum/#!forum/druid-user)上提问。
