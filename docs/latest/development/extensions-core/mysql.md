---
layout: doc_page
---

# MySQL 元数据存储

请确保[包含](../../operations/including-extensions.html) `mysql-metadata-storage`作为一个扩展。
## 安装MySQL

1. 安装 MySQL

  选择你最喜欢的软件管理器安装msql，如:
  - 在Ubuntu/Debian使用apt `apt-get install mysql-server`
  - 在OS X，使用 [Homebrew](http://brew.sh/) `brew install mysql`

  另外,下载并按照安装说明安装MySQL
  社区服务器在:
  [http://dev.mysql.com/downloads/mysql/](http://dev.mysql.com/downloads/mysql/)

2. 创建一个druid数据库和用户

  从安装机器连接MySQL。
  ```bash
  > mysql -u root
  ```

 粘贴下面一段到mysql命令行：
  ```sql
  -- create a druid database, make sure to use utf8 as encoding
  CREATE DATABASE druid DEFAULT CHARACTER SET utf8;

  -- create a druid user, and grant it all permission on the database we just created
  GRANT ALL ON druid.* TO 'druid'@'localhost' IDENTIFIED BY 'diurd';
  ```

3. 配置你的druid元数据存储扩展：

  添加下面参数到你的Druid配置，用数据库的本地（主机名和端口）代替`<host>`。
  
  ```properties
  druid.extensions.loadList=["mysql-metadata-storage"]
  druid.metadata.storage.type=mysql
  druid.metadata.storage.connector.connectURI=jdbc:mysql://<host>/druid
  druid.metadata.storage.connector.user=druid
  druid.metadata.storage.connector.password=diurd
  ```

  注意：元数据存储扩展并不包含在Druid软件包中；它是打包在分离的安装包里，可以从 [这里](http://druid.io/downloads.html)下载。
  你也可以用 [pull-deps](../../operations/pull-deps.html)得到这个扩展，或者你可以从源码中构建，查阅[从资源中构建](../build.html)。