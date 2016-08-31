---
layout: doc_page
---

# 批数据摄取

Druid可以通过下面描述的各种方法从静态文件加载数据。

## 基于Hadoop的批摄取

在Druid基于Hadoop批量摄取是通过一个Hadoop摄取任务支持的。这些任务可以发布到运行在Druid的实例 [overlord](../design/indexing-service.html)。一个简单的示例如下：
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

|属性|描述|要求？|
|--------|-----------|---------|
|type|The task type, this should always be "index_hadoop".|yes|
|spec|A Hadoop Index Spec. See [Batch Ingestion](../ingestion/batch-ingestion.html)|yes|
|hadoopDependencyCoordinates|A JSON array of Hadoop dependency coordinates that Druid will use, this property will override the default Hadoop coordinates. Once specified, Druid will look for those Hadoop dependencies from the location specified by `druid.extensions.hadoopDependenciesDir`|no|
|classpathPrefix|Classpath that will be pre-appended for the peon process.|no|

### DataSchema

这个字段是必须的。

查阅 [Ingestion](../ingestion/index.html)

### IOConfig

这个字段是必须的。

|字段|类型|描述|要求|
|-----|----|-----------|--------|
|type|String|This should always be 'hadoop'.|yes|
|inputSpec|Object|a specification of where to pull the data in from. See below.|yes|
|segmentOutputPath|String|the path to dump segments into.|yes|
|metadataUpdateSpec|Object|a specification of how to update the metadata for the druid cluster these segments belong to.|yes|

#### InputSpec specification

有多个inputSpecs类型：
##### `static`

数据文件所在的静态路径是一种inputSpec类型。

|字段|类型|描述|要求|
|-----|----|-----------|--------|
|paths|Array of String|A String of input paths indicating where the raw data is located.|yes|

例如，使用静态输入路径：
```
"paths" : "s3n://billy-bucket/the/data/is/here/data.gz, s3n://billy-bucket/the/data/is/here/moredata.gz, s3n://billy-bucket/the/data/is/here/evenmoredata.gz"
```

##### `granularity`

指定路径格式的预期数据是一种inputSpec类型。特别地，它预期日期将会使用这个目录形式`y=XXXX/m=XX/d=XX/H=XX/M=XX/S=XX`（日期是由小写字母表示，时间是大写字母表示）。

|字段|类型|描述|要求|
|-----|----|-----------|--------|
|dataGranularity|String|specifies the granularity to expect the data at, e.g. hour means to expect directories `y=XXXX/m=XX/d=XX/H=XX`.|yes|
|inputPath|String|Base path to append the expected time path to.|yes|
|filePattern|String|Pattern that files should match to be included.|yes|
|pathFormat|String|Joda date-time format for each directory. Default value is `"'y'=yyyy/'m'=MM/'d'=dd/'H'=HH"`, or see [Joda documentation](http://www.joda.org/joda-time/apidocs/org/joda/time/format/DateTimeFormat.html)|no|

例如，如果简单的配置2012-06-01/2012-06-02期间运行，它将在路径下expect数据
```
s3n://billy-bucket/the/data/is/here/y=2012/m=06/d=01/H=00
s3n://billy-bucket/the/data/is/here/y=2012/m=06/d=01/H=01
...
s3n://billy-bucket/the/data/is/here/y=2012/m=06/d=01/H=23
```

##### `dataSource`

读取Druid segments。查阅[这里](../ingestion/update-existing-data.html)了解更多信息。

##### `multi`

读多个数据源。查阅[这里](../ingestion/update-existing-data.html)了解更多信息。
### TuningConfig

tuningConfig是可选的，而且如果没有指定的tuningConfig将使用默认参数。

|字段|类型|描述|要求|
|-----|----|-----------|--------|
|workingPath|String|The working path to use for intermediate results (results between Hadoop jobs).|no (default == '/tmp/druid-indexing')|
|version|String|The version of created segments.|no (default == datetime that indexing starts at)|
|partitionsSpec|Object|A specification of how to partition each time bucket into segments, absence of this property means no partitioning will occur.More details below.|no (default == 'hashed')|
|maxRowsInMemory|Integer|The number of rows to aggregate before persisting. This number is the post-aggregation rows, so it is not equivalent to the number of input events, but the number of aggregated rows that those events result in. This is used to manage the required JVM heap size.|no (default == 5 million)|
|leaveIntermediate|Boolean|Leave behind intermediate files (for debugging) in the workingPath when a job completes, whether it passes or fails.|no (default == false)|
|cleanupOnFailure|Boolean|Clean up intermediate files when a job fails (unless leaveIntermediate is on).|no (default == true)|
|overwriteFiles|Boolean|Override existing files found during indexing.|no (default == false)|
|ignoreInvalidRows|Boolean|Ignore rows found to have problems.|no (default == false)|
|useCombiner|Boolean|Use hadoop combiner to merge rows at mapper if possible.|no (default == false)|
|jobProperties|Object|A map of properties to add to the Hadoop job configuration.|no (default == null)|
|buildV9Directly|Boolean|Whether to build v9 index directly instead of building v8 index and convert it to v9 format|no (default = false)|
|numBackgroundPersistThreads|Integer|The number of new background threads to use for incremental persists. Using this feature causes a notable increase in memory pressure and cpu usage, but will make the job finish more quickly. If changing from the default of 0 (use current thread for persists), we recommend setting it to 1.|no (default == 0)|

### 分块规范

Segment总是基于timestamp分块的（根据granulartySpec)而且可能根据分区类型以一些其他的方式分区。
Druid支持两种分块策略类型："hashed"（基于每一行所有维度的hash），和"dimension" （基于单维度的范围）。

大部分情况下建议使用hash分区，因为它将提高索引性能和创建更多与单维度分区相关联的同等大小的数据Segment。

#### 基于hash的分块

```json
  "partitionsSpec": {
     "type": "hashed",
     "targetPartitionSize": 5000000
   }
```

hash分块首先选择大量的Segment，然后根据每行所有维度的hash，在这些Segment分块行。
Segment的数量是自动基于输入的基数和分块大小来决定。

配置选项是：

|属性|描述|要求|
|--------|-----------|---------|
|type|type of partitionSpec to be used |"hashed"|
|targetPartitionSize|target number of rows to include in a partition, should be a number that targets segments of 500MB\~1GB.|either this or numShards|
|numShards|specify the number of partitions directly, instead of a target partition size. Ingestion will run faster, since it can skip the step necessary to select a number of partitions automatically.|either this or targetPartitionSize|

#### 单维度分块

```json
  "partitionsSpec": {
     "type": "dimension",
     "targetPartitionSize": 5000000
   }
```

单维度首先选择一个维度分块，然后分离配置范围内的维度。
每个Segment将包含所有范围内维度值的行。例如，你的Segment可能使用范围“a.example.com”到“f.example.com”然后“f.example.com”到“z.example.com”，在维度“host”分块。
默认地，维度都是自动决定使用哪个的，尽管你可以通过指定一个维度来覆盖这项功能。

配置选项是：

|属性|描述|要求|
|--------|-----------|---------|
|type|type of partitionSpec to be used |"dimension"|
|targetPartitionSize|target number of rows to include in a partition, should be a number that targets segments of 500MB\~1GB.|yes|
|maxPartitionSize|maximum number of rows to include in a partition. Defaults to 50% larger than the targetPartitionSize.|no|
|partitionDimension|the dimension to partition on. Leave blank to select a dimension automatically.|no|
|assumeGrouped|assume input data has already been grouped on time and dimensions. Ingestion will run faster, but can choose suboptimal partitions if the assumption is violated.|no|

### 远程Hadoop集群

如果你有一个远程Hadoop集群，确保包含你的配置 `*.xml`文件的文件夹在你的Druid`_common`配置文件夹。
如果你的Hadoop和Druid编译版本有依赖性问题，请参阅 [这些文档](../operations/other-hadoop.html)。

### 使用弹性的 MapReduce

如果你的集群是运行在Amazon web Services，你可以使用Elastic MapRedure（EMR）从S3去索引数据。做到这些：

- 创建一个长久的， [long-running 集群](http://docs.aws.amazon.com/ElasticMapReduce/latest/ManagementGuide/emr-plan-longrunning-transient.html).
- 当创建你的集群时，输入下面配置。如果你正在使用wizard，这个应该是"Edit software settings"下的高级模版。
```
classification=yarn-site,properties=[mapreduce.reduce.memory.mb=6144,mapreduce.reduce.java.opts=-server -Xms2g -Xmx2g -Duser.timezone=UTC -Dfile.encoding=UTF-8 -XX:+PrintGCDetails -XX:+PrintGCTimeStamps,mapreduce.map.java.opts=758,mapreduce.map.java.opts=-server -Xms512m -Xmx512m -Duser.timezone=UTC -Dfile.encoding=UTF-8 -XX:+PrintGCDetails -XX:+PrintGCTimeStamps,mapreduce.task.timeout=1800000]
```

- 按照 "[为数据加载配置Hadoop](../tutorials/cluster.html#configure-cluster-for-hadoop-data-loads)" 说明
从你的EMR`/etc/hadoop/conf`使用XML文件。
#### EMR从S3加载
- 在 `jobProperties`文件的Hadoop索引任务的 `tuningConfig`部分，增加：

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

注意,这个方法使用Hadoop的S3装入文件系统,而不是Amazon的EMRFS,而且不兼容Amazon-specific的S3加密等特性和一致的观点。
如果你需要使用这些功能,您需要提供Amazon EMR Hadoop jar，通过Druid中描述在[(使用其他Hadoop发行版)](#using-other-hadoop-distributions)部分的机制。

## 使用其他Hadoop发行版

Druid可以运行许多Hadoop发行版。

如果你的Druid和Hadoop版本有依赖性冲突，你可以试着在 [Druid 用户组](https://groups.google.com/forum/#!forum/druid-user)搜索解决方法，
或者阅读Druid [不同的Hadoop版本](../operations/other-hadoop.html)文档。

## 命令行Hadoop索引器

如果你不想使用一个完整的索引服务去用Hadoop获取数据到Druid，你也可以使用独立的命令行Hadoop索引器。
查阅 [这里](../ingestion/command-line-hadoop-indexer.html) 了解更多信息。

## 基于索引任务的批处理摄取

如果你不想依赖Hadoop批处理摄取，您还可以使用索引任务。这个任务会慢得多,也不如Hadoop-based方法可扩展。查阅 [这里](../ingestion/tasks.html)了解更多信息。
有问题？
----------------
用户第一次获取数据到Druid是非常困难的。如有问题，请在我们的IRC频道或者在我们的[google 页面](https://groups.google.com/forum/#!forum/druid-user)向我们提出。