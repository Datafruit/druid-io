---
layout: doc_page
---

# 命令行Hadoop索引器

运行：

```
java -Xmx256m -Duser.timezone=UTC -Dfile.encoding=UTF-8 -classpath lib/*:<hadoop_config_dir> io.druid.cli.Main index hadoop <spec_file>
```

## Options

- "--Coordinate" - 提供一个Hadoop版本使用。这个属性将覆盖默认的Hadoop Coordinate。一旦指定，Druid将通过从指定`druid.extensions.hadoopDependenciesDir`位置查找那些Hadoop依赖关系。
- "--no-default-hadoop" - 不要破坏默认的hadoop版本
## Spec file

规范文件需要包含一个JSON对象，内容与Hadoop索引任务的“spec”字段相同。
另外，下面的字段需要增加到ioConfig：

```
      "ioConfig" : {
        ...
        "metadataUpdateSpec" : {
          "type":"mysql",
          "connectURI" : "jdbc:mysql://localhost:3306/druid",
          "password" : "diurd",
          "segmentTable" : "druid_segments",
          "user" : "druid"
        },
        "segmentOutputPath" : "/MyDirectory/data/index/output"
      },
```    

还有下面的字段需要增加到tuningConfig:

```
  "tuningConfig" : {
   ...
    "workingPath": "/tmp",
    ...
  }
```    

#### 元数据更新工作说明

这个是一个属性规范说明，讲述怎么更新元数据，如Druid集群将输出Segment和加载他们。

|字段|类型|描述|要求|
|-----|----|-----------|--------|
|type|String|"metadata" is the only value available.|yes|
|connectURI|String|A valid JDBC url to metadata storage.|yes|
|user|String|Username for db.|yes|
|password|String|password for db.|yes|
|segmentTable|String|Table to use in DB.|yes|

这些属性应该模仿[Coordinator](../design/coordinator.html)的配置。

#### segmentOutputPath Config

|字段|类型|描述|要求|
|-----|----|-----------|--------|
|segmentOutputPath|String|the path to dump segments into.|yes|

#### workingPath Config

|字段|类型|描述|要求|
|-----|----|-----------|--------|
|workingPath|String|the working path to use for intermediate results (results between Hadoop jobs).|no (default == '/tmp/druid-indexing')|

请注意，命令行Hadoop索引器没有索引服务的锁定权限，如果你选择使用它，你必须注意不能覆盖real-time处理创建的Segment（如果你安装了real-time流）。