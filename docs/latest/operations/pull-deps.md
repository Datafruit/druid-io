---
layout: doc_page
---

# pull-deps 工具

`pull-deps`是一个可以拉下依赖本地存储库,把依赖项根据需要扩展目录的工具。
`pull-deps`有几个命令行选择，如下：
`-c` 或者 `--coordinate`（可以指定多个时间）
扩展坐标下拉,紧随其后的是一个maven坐标,如io.druid.extensions:mysql-metadata-storage
`-h` or `--hadoop-coordinate` （可以指定多个时间）
Hadoop依赖拉下来,后跟一个maven坐标,如org.apache.hadoop:hadoop-client：2.4.0

`--no-default-hadoop`

不要拉下默认hadoop坐标,即org.apache.hadoop:hadoop-client:2.3.0。如果提供`- h`选项,那么默认的hadoop坐标不会被下载。
`--clean`
    
下拉依赖之前删除现有的扩展和hadoop依赖性目录。
`-l` 或者 `--localRepository`

 Maven将使用本地repostiry来下载文件。然后pull-deps将这些文件根据需要扩展目录。
`-r` 或者 `--remoteRepository`

远程存储库添加到默认的远程存储库列表，包括 https://repo1.maven.org/maven2/ 和 https://metamx.artifactoryonline.com/metamx/pub-libs-releases-local
`-d` 或者 `--defaultVersion`
版本用于扩展协调,没有版本信息。例如，如果扩展坐标是 `io.druid.extensions:mysql-metadata-storage`，默认版本是`0.9.0`，那么这个坐标将被看作是 `io.druid.extensions:mysql-metadata-storage:0.9.0`。

为了运行`pull-deps`，你应该

1）指定 `druid.extensions.directory` 和 `druid.extensions.hadoopDependenciesDir`，这两个属性告诉`pull-deps`把扩展放到哪儿。
如果你不指定它们，默认值将代替，查阅 [配置](../configuration/index.html)。

2）告诉`pull-deps`使用`-c`或者`-h`选择下载什么，这个后面跟着一个maven坐标。

示例：

假设你想用指定版本下载```druid-rabbitmq```, ```mysql-metadata-storage```和 ```hadoop-client```( 2.3.0 和 2.4.0)， 
你可以运行 `pull-deps`，输入以下命令 `-c io.druid.extensions:druid-examples:0.9.0`, `-c io.druid.extensions:mysql-metadata-storage:0.9.0`, `-h org.apache.hadoop:hadoop-client:2.3.0`和 `-h org.apache.hadoop:hadoop-client:2.4.0`。
示例如下：

```
java -classpath "/my/druid/library/*" io.druid.cli.Main tools pull-deps --clean -c io.druid.extensions:mysql-metadata-storage:0.9.0 -c io.druid.extensions.contrib:druid-rabbitmq:0.9.0 -h org.apache.hadoop:hadoop-client:2.3.0 -h org.apache.hadoop:hadoop-client:2.4.0
```

因为提供`--clean`,该命令将先删除指定在`druid.extensions.directory` 和 `druid.extensions.hadoopDependenciesDir`的目录,然后重新创建它们,开始下载扩展。完成下载后,如果你扩展指定目录,你会看到

```
tree extensions
extensions
├── druid-examples
│   ├── commons-beanutils-1.8.3.jar
│   ├── commons-digester-1.8.jar
│   ├── commons-logging-1.1.1.jar
│   ├── commons-validator-1.4.0.jar
│   ├── druid-examples-0.9.0.jar
│   ├── twitter4j-async-3.0.3.jar
│   ├── twitter4j-core-3.0.3.jar
│   └── twitter4j-stream-3.0.3.jar
└── mysql-metadata-storage
    ├── jdbi-2.32.jar
    ├── mysql-connector-java-5.1.34.jar
    └── mysql-metadata-storage-0.9.0.jar
```

```
tree hadoop-dependencies
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

请注意,如果您指定`--defaultVersion`,你不需要把版本信息放到坐标上。例如,如果你想要`druid-rabbitmq`和`mysql-metadata-storage`使用版本`0.9.0`,你可以改变上面的命令

```
java -classpath "/my/druid/library/*" io.druid.cli.Main tools pull-deps --defaultVersion 0.9.0 --clean -c io.druid.extensions:mysql-metadata-storage -c io.druid.extensions.contrib:druid-rabbitmq -h org.apache.hadoop:hadoop-client:2.3.0 -h org.apache.hadoop:hadoop-client:2.4.0
```

<div class="note info">
请注意使用pull-deps工具你必须知道Maven groupId,artifactId,版本的扩展。
Druid的社区扩展列表<a href="../development/extensions.html">这里</a>,groupId是"io.druid.extensions.contrib"和artifactId是扩展名。
</div>
