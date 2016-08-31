---
layout: doc_page
---
# 使用不同版本的Hadoop

## 下载Hadoop的依赖性
 
```hadoop-client:2.3.0``` 已经捆绑在Druid释放压缩文件。你可以通过在[druid.io](http://druid.io/downloads.html)下载压缩文件得到它。
解压压缩文件：你将看到 ```hadoop-dependencies```文件夹，包含所有的Hadoop依赖。每个依赖将有包含着Hadoop jars的文件夹。

您还可以使用`pull-deps`工具下载其他你想要的Hadoop依赖。
查阅[pull-deps](../operations/pull-deps.html)了解完整例子。

## 加载Hadoop依赖

有两种不同的方式让Druid获取Hadoop版本,选择一个适合你需求的。
### 从Hadoop依赖目录加载Hadoop依赖

您可以创建一个Hadoop依赖目录,告诉Druid在此处加载您的Hadoop的依赖性。

这项工作,遵循以下步骤

**告诉druid,你的Hadoop依赖关系在哪儿**

指定 `druid.extensions.hadoopDependenciesDir`（Hadoop相对依赖的绝对目录）查阅 [配置](../configuration/index.html)。

这个属性的值应该设置为包含所有Hadoop的依赖性文件夹的绝对路径。
一般来讲，你应该重复利用压缩文件的 ```hadoop-dependencies```目录。

示例：:

假设你指定`druid.extensions.hadoopDependenciesDir=/usr/local/druid_tarball/hadoop-dependencies`，而且你已经下载了`hadoop-client` 2.3.0 and 2.4.0。

那么在 ```hadoop-dependencies```下面，目录结构应该是这样的：
```
hadoop-dependencies/
└── hadoop-client
    ├── 2.3.0
    │   ├── activation-1.1.jar
    │   ├── avro-1.7.4.jar
    │   ├── commons-beanutils-1.7.0.jar
    │   ├── commons-beanutils-core-1.8.0.jar
    │   ├── commons-cli-1.2.jar
    │   ├── commons-codec-1.4.jar
    ..... lots of jars
    └── 2.4.0
        ├── activation-1.1.jar
        ├── avro-1.7.4.jar
        ├── commons-beanutils-1.7.0.jar
        ├── commons-beanutils-core-1.8.0.jar
        ├── commons-cli-1.2.jar
        ├── commons-codec-1.4.jar
    ..... lots of jars
```

正如你所看到的，在 ```hadoop-client```下面，有两个子目录，每个代表一个 ```hadoop-client```版本。

**告诉Druid加载那个Hadoop版本**

在[Hadoop索引任务](../ingestion/batch-ingestion.html)使用 `hadoopDependencyCoordinates`指定你想Druid加载的Hadoop依赖。
例如，在你的Hadoop索引任务规范文件，有
`"hadoopDependencyCoordinates": ["org.apache.hadoop:hadoop-client:2.4.0"]`

这个指示Druid在处理任务时加载hadoop-client 2.4.0。
Druid首先寻找`druid.extensions.hadoopDependenciesDir`下名为 ```hadoop-client``` 的文件夹，然后查找```hadoop-client```下名为```2.4.0```的文件夹，成功定位这些文件夹后,加载hadoop-client 2.4.0 。

### 追加Hadoop jars 到Druid类路径

如果你不喜欢上面的方法,你只是想用一个特定的Hadoop的版本,而且不想让Druid使用不同的Hadoop版本,你可以
（1）设置`druid.indexer.task.defaultHadoopCoordinates=[]`。 `druid.indexer.task.defaultHadoopCoordinates`指定Druid使用默认的Hadoop协调。默认值为`["org.apache.hadoop:hadoop-client:2.3.0"]`。
通过设置为空的列表，Druid将不加载任何其他Hadoop依赖除了指定在类路径的。

（2）添加您的Hadoop jar文件到类路径,Druid将其加载到系统中。这种机制相对容易推出,但这也意味着你必须确保所有依赖jar文件的类路径中是兼容的。
也就是说，Druid并没有规定在使用这个方法来维护类装入器隔离,所以你必须确保类路径上的jars是相互兼容的。

#### Hadoop 2.x

Hadoop绑定Druid的默认版本是2.3。
覆盖默认的Hadoop版本,Hadoop索引任务和独立的Hadoop索引器支持参数`hadoopDependencyCoordinates`(查阅 [索引Hadoop任务](../ingestion/tasks.html))。
你可以通过这个参数终止另一组Hadoop坐标(例如,您可以为Hadoop 2.4.0指定坐标 `["org.apache.hadoop:hadoop-client:2.4.0"]`)),这将覆盖Druid使用的默认Hadoop坐标。
Hadoop索引任务需要这个参数的一部分任务JSON而且独立的Hadoop索引器将该参数作为一个命令行参数。

如果你仍然有问题,包括所有相关你的索引或者历史节点的类路径的hadoop jars。

#### CDH
CDH和Druid使用的Jackson版本之间的社区成员报告依赖的冲突。目前,我们最好的解决方法是在您的Hadoop版本和重新编译Druid里编辑Druid的pom.xml依赖性匹配Jackson的版本。

更多关于Druid，请查阅[Building Druid](../development/build.html)。
另一个解决方案是使用[sbt](http://www.scala-sbt.org/)建立一个定制的Druid jar，
手动排除所有Jackson依赖性的冲突,然后把这个jar放进开始霸王索引服务的命令类路径中。
为此,请按照下列步骤。

（1）下载并且安装sbt。
（2）新建一个目录命名为“druid_build”。
（3）进入‘druid_build’使用[这里](./use_sbt_to_build_fat_jar.html)内容创建sbt文件。
你可以添加更多的建设目标或删除你不需要的。
（4）在相同的目录中创建一个新目录命名为'project'。
（5）把Druid的源代码放在'druid_build /project'。
（6）使用下面的内容创建一个文件 'druid_build/project/assembly.sbt'。
```
addSbtPlugin("com.eed3si9n" % "sbt-assembly" % "0.13.0")
```

（7）在'druid_build'目录，运行'sbt assembly'。
（8）在 'druid_build/target/scala-2.10'文件夹，你可以看到刚刚新建的jar。
（9）确保你上传的jars已经完全移除。HDFS目录默认是 '/tmp/druid-indexing/classpath'。
（10）当你开始索引服务时类路径将包含jar。确保你已经从你的类路径移除了 'lib/*'，因为现在的jar包含了你需要的东西。
## 使用Hadoop 1.x和更老的版本

我们建议重新编译Druid和你的特定版本的Hadoop通过改变在Druid的依赖关系pom.xml文件。
确保也可以覆盖代码里默认的 `hadoopDependencyCoordinates` 或通过您的Hadoop版本作为索引的一部分。