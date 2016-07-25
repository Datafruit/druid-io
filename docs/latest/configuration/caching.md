---
layout: doc_page
---

# 缓存

缓存可以选择在 broker，historical，和 realtime过程中启动。查阅[broker](broker.html#caching)，
[historical](historical.html#caching)，和[realtime](realtime.html#caching)
如何启动不同过程的配置选项。

Druid默认使用本地内存缓存，除非指定不同类型的缓存。
使用`druid.cache.type`配置设置不同类型的缓存。
## 缓存配置

缓存设置是全球性的，所以当Broker和historical节点定义在常见的属性文件中时，相同的配置可以被重复使用。
|属性|可能的值|描述|默认|
|--------|---------------|-----------|-------|
|`druid.cache.type`|`local`, `memcached`, `hybrid`|The type of cache to use for queries. See below of the configuration options for each cache type|`local`|


#### 本地缓存

一个简单的内存LRU缓存。本地缓存是存在JVM堆内存中，所以如果你启用它，确保你相应地增加堆大小。
|属性|描述|默认|
|--------|-----------|-------|
|`druid.cache.sizeInBytes`|Maximum cache size in bytes. Zero disables caching.|0|
|`druid.cache.initialSize`|Initial size of the hashtable backing the cache.|500000|
|`druid.cache.logEvictionCount`|If non-zero, log cache eviction every `logEvictionCount` items.|0|


#### Memcached分布式缓存

使用分布式缓存作为缓存后端。这允许所有节点共享相同的缓存。
|属性|描述|默认|
|--------|-----------|-------|
|`druid.cache.expiration`|Memcached [expiration time](https://code.google.com/p/memcached/wiki/NewCommands#Standard_Protocol).|2592000 (30 days)|
|`druid.cache.timeout`|Maximum time in milliseconds to wait for a response from Memcached.|500|
|`druid.cache.hosts`|Command separated list of Memcached hosts `<host:port>`.|none|
|`druid.cache.maxObjectSize`|Maximum object size in bytes for a Memcached object.|52428800 (50 MB)|
|`druid.cache.memcachedPrefix`|Key prefix for all keys in Memcached.|druid|
|`druid.cache.numConnections`|Number of memcached connections to use.|1|


#### Hybrid
使用任意两个缓存的组合作为二级L1 / L2缓存。　　
这可能是用来结合本地内存缓存和远程memcached缓存。

缓存请求将在检查L2前检查L1缓存。
如果L1缺失而且点击了L2，它也会填充L1。

|属性|描述|默认|
|--------|-----------|-------|
|`druid.cache.l1.type`|type of cache to use for L1 cache. See `druid.cache.type` configuration for valid types.|`local`|
|`druid.cache.l2.type`|type of cache to use for L2 cache. See `druid.cache.type` configuration for valid types.|`local`|
|`druid.cache.l1.*`|Any property valid for the given type of L1 cache can be set using this prefix. For instance, if you are using a `local` L1 cache, specify `druid.cache.l1.sizeInBytes` to set its size.|defaults are the same as for the given cache type.|
|`druid.cache.l2.*`|Prefix for L2 cache settings, see description for L1.|defaults are the same as for the given cache type.|
