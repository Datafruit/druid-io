---
layout: doc_page
---
# insert-segment-to-db 工具

`insert-segment-to-db`是一个可以插入段到Druid元数据存储的工具。它是被使用在人们手动的把段从一个地方迁移到另一个地方后，在元数据存储里更新段表。
它也可以被用来将缺失的段插入Druid,甚至通过告诉它段存储的位置恢复元数据存储。
注意:此工具期望用户让Druid集群在一个“安全”模式下运行,没有积极干预的任务段被插入。用户可以选择降低集群确保100%肯定没有干扰。

为了让它正常工作,用户必须提供元数据存储证书和深存储类型通过Java JVM参数或运行时属性文件。具体来说,这个工具我们需要了解
`druid.metadata.storage.type`

`druid.metadata.storage.connector.connectURI`

`druid.metadata.storage.connector.user`

`druid.metadata.storage.connector.password`

`druid.storage.type`

除了上面的属性中,您还需要指定段的位置存储和你是否想要更新descriptor.json。这两个可以通过命令行参数提供。

`--workingDir` (必须)

    The directory URI where segments are stored. This tool will recursively look for segments underneath this directory
    and insert/update these segments in metdata storage.
    Attention: workingDir must be a complete URI, which means it must be prefixed with scheme type. For example,
    hdfs://hostname:port/segment_directory

`--updateDescriptor` (可选)

    if set to true, this tool will update `loadSpec` field in `descriptor.json` if the path in `loadSpec` is different from
    where `desciptor.json` was found. Default value is `true`.

注意:你还需要加载不同的Druid扩展你使用的每个元数据和深存储。
例如,如果你使用`mysql`作为元数据存储和`HDFS`深存储,你应该加载`mysql-metadata-storage` 和 `druid-hdfs-storage`扩展。

示例：

假设你的元数据存储是`mysql`而且你需要把一些段迁移到HDFS的一个目录,目录看起来像这样,
```
Directory path: /druid/storage/wikipedia

├── 2013-08-31T000000.000Z_2013-09-01T000000.000Z
│   └── 2015-10-21T22_07_57.074Z
│       └── 0
│           ├── descriptor.json
│           └── index.zip
├── 2013-09-01T000000.000Z_2013-09-02T000000.000Z
│   └── 2015-10-21T22_07_57.074Z
│       └── 0
│           ├── descriptor.json
│           └── index.zip
├── 2013-09-02T000000.000Z_2013-09-03T000000.000Z
│   └── 2015-10-21T22_07_57.074Z
│       └── 0
│           ├── descriptor.json
│           └── index.zip
└── 2013-09-03T000000.000Z_2013-09-04T000000.000Z
    └── 2015-10-21T22_07_57.074Z
        └── 0
            ├── descriptor.json
            └── index.zip
```

加载所有这些段到`mysql`,您可以使用下面的命令,
```
java 
-Ddruid.metadata.storage.type=mysql 
-Ddruid.metadata.storage.connector.connectURI=jdbc\:mysql\://localhost\:3306/druid 
-Ddruid.metadata.storage.connector.user=druid 
-Ddruid.metadata.storage.connector.password=diurd 
-Ddruid.extensions.loadList=[\"mysql-metadata-storage\",\"druid-hdfs-storage\"] 
-Ddruid.storage.type=hdfs
-cp $DRUID_CLASSPATH 
io.druid.cli.Main tools insert-segment --workingDir hdfs://host:port//druid/storage/wikipedia --updateDescriptor true
```

在这个例子中,`mysql`和深存储类型是通过Java JVM参数提供的,你可以把它们都放在运行期间属性文件里和包含在druid类路径里。
注意,我们还包括“mysql-metadata-storage” 　　和“druid-hdfs-storage”扩展列表。
运行此命令后,段表的`mysql`应该为我们刚插入的每个段存储新的位置。