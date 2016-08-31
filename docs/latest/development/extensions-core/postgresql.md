---
layout: doc_page
---

# PostgreSQL 元数据存储

请确保[包含](../../operations/including-extensions.html) `postgresql-metadata-storage`作为一个扩展。
## 安装 PostgreSQL

1. 安装 PostgreSQL

  选择你最喜欢的软件管理器安装PostgreSQL，如：
  - 在Ubuntu/Debian使用apt `apt-get install postgresql`
  - 在OS X，使用 [Homebrew](http://brew.sh/) `brew install postgresql`


2. 创建一个druid数据库和用户

  在安装PostgreSQL的集群上，使用一个适当的Postgresql账户权限：
 
  创建一个druid用户，当提交密码时进入`druid`。
  
  ```bash
  createuser druid -P
  ```

  创建一个属于我们刚刚创建的用户的druid数据库

  ```bash
  createdb druid -O druid
  ```

  *注意：* 在Ubuntu/Debian你可能需要在`sudo -u postgres`添加前缀`createuser`和`createdb`命令为了得到适当的权限。

3. 配置你的druid元数据存储扩展：

  增加下面的参数到你的Druid配置，用本地数据库的（主机名和端口号）代替`<host>`。
 
  ```properties
  druid.extensions.loadList=["postgresql-metadata-storage"]
  druid.metadata.storage.type=postgresql
  druid.metadata.storage.connector.connectURI=jdbc:postgresql://<host>/druid
  druid.metadata.storage.connector.user=druid
  druid.metadata.storage.connector.password=diurd
  ```
