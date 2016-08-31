---
layout: doc_page
---

### 从源代码中构建

你可以直接从源代码构建Druid。请注意这些指令是构建最新的稳定的Druid。
为了构建最新的编码，请按照指定[这里](https://github.com/druid-io/druid/blob/master/docs/content/development/build.md)操作。

构建Druid有以下要求：
- [JDK 7](http://www.oracle.com/technetwork/java/javase/downloads/jdk7-downloads-1880260.html)
  或者 [JDK 8](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)
- [Maven version 3.x](http://maven.apache.org/download.cgi)

为此，运行下面命令：

```
git clone git@github.com:druid-io/druid.git
cd druid
mvn clean package
```

这将在`distribution/target/druid-VERSION-bin.tar.gz`下面编译项目并创建Druid二进制发行包。

这个也将在`distribution/target/mysql-metadata-storage-bin.tar.gz`下创建一个包含 `mysql-metadata-storage`扩展的原始码。
如果你想要Druid加载 `mysql-metadata-storage`，你可以先解压 `druid-VERSION-bin.tar.gz`，然后到```druid-<version>/extensions```，
解压`mysql-metadata-storage-bin.tar.gz` 。现在只要在`druid.extensions.loadList`里指定`mysql-metadata-storage`，Druid就会将器打包起来。
查阅[包含扩展](../operations/including-extensions.html)了解更多信息。