---
layout: doc_page
---

# Lookups

<div class="note caution">
Lookups are an <a href="../development/experimental.html">experimental</a> feature.
</div>

查找是维值(可选)替换为新值的Druid的一个概念。
查看[dimension specs](../querying/dimensionspecs.html)更多信息。
对于这些文档的目的，一个"key"是指一个维度值匹配，和一个"value"是指其替代品。
如果你想把 `appid-12345`重命名为`Super Mega Awesome App`，然后键成为`appid-12345`值成为 `Super Mega Awesome App`

值得注意的是，查找支持使用情况下键映射到唯一的值(injective)如一个国家编号和一个国家的名字，
还支持使用多个id映射到相同的值的情况下，如多个app-ids属于一个客户经理。

查找是没有历史的。他们总是使用当前数据。这意味着，如果首席客户经理特定的app-id更改，你发出查询与存储app-id查找客户经理的关系，
它将返回当前客户经理，app-id无论你查询的时间范围。


如果您需要敏感查找的数据时间范围，目前这样的用例是不支持动态查询，这样的数据用于Druid属于原始非正规数据。
非常小的查找(键的计算顺序几十到几百)可以通过在查询时做为"map"
查找根据[dimension specs](../querying/dimensionspecs.html)。
静态查找是在`runtime.properties`中定义而不是嵌入查询，请看示例[namespaced lookup extension](../development/extensions-core/namespaced-lookup.html).
对于额外的查找，请看我们的[extensions list](../development/extensions.html)。