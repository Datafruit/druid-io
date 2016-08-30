---
layout: doc_page
---

# 近似直方图聚合器

确保 [包含](../../operations/including-extensions.html) `druid-histogram`作为扩展。
这个聚合器是基于
[http://jmlr.org/papers/volume11/ben-haim10a/ben-haim10a.pdf](http://jmlr.org/papers/volume11/ben-haim10a/ben-haim10a.pdf)
计算近似直方图，修改以下：

- 一些权衡精度的速度(见下文)
- 描述保持准确的原始数据的数量只要不同的数据点少于决定的数量(重心的数量)，
  当有少量数据点进行速率时，或者当处理一些离散的数据点时。你可以在[这个网址](https://metamarkets.com/2013/histograms/)找到更多细节。
  
近似直方图任然只是为个人试验的，在使用之前你应该理解当前执行有什么限制。
近似是强烈依靠数据的,这使得它很难给出好的一般的指导方针,所以你应该试验,看看什么参数适合您的数据。

使用前这里有几件事要注意:

- 正如最初的文章中指出,没有正式的误差界限近似。
  在实践中,如果分布是扭曲的，近似将变得更糟。 
  
- 算法是依赖顺序的,所以相同的查询结果可能不同,因为变换顺序的结果会被合并。

- 在一般情况下,该算法只适用于随机的数据分布式(即如果最终数据点排序的列,近似将会变的可怕)

- 我们对聚合速度要求准确性,当添加一些捷径直方图时,如果您的数据是有顺序的,或如果您的分布有长尾从而导致病态情况。
  它应该是更低成本的增加草图的决议来得到你所需要的精度。


那也就是说，当平均水平不够好时这些草图可以有利于一阶近似。假设你段存储中大多数行的数据点少于直方图的决议,
你应该能够使用他们用于监测目的和用数百个重心检测有意义的变化。为了得到第95个百分位数的读数精度很好的数百万行数据,
您可能想要使用几千个重心,特别有长尾的,因为这就是近似变地更糟的地方。

### 在摄入时间创建近似直方图草图


要使用该功能,一个“approxHistogram”或“approxHistogramFold”聚合器必须包含在索引的时间内。
摄入聚合器只能适用于数值。如果你使用“approxHistogram”，那么任何输入行缺失的值都将被认为有一个值为0,同时有“approxHistogramFold”的行将被忽略。

对于结果查询，查询必须包含一个 "approxHistogramFold" 聚合器。

```json
{
  "type" : "approxHistogram or approxHistogramFold (at ingestion time), approxHistogramFold (at query time)",
  "name" : <output_name>,
  "fieldName" : <metric_name>,
  "resolution" : <integer>,
  "numBuckets" : <integer>,
  "lowerLimit" : <float>,
  "upperLimit" : <float>
}
```

|属性|描述|默认|
|-------------------------|------------------------------|----------------------------------|
|`resolution`             |Number of centroids (data points) to store. The higher the resolution, the more accurate results are, but the slower the computation will be.|50|
|`numBuckets`             |Number of output buckets for the resulting histogram. Bucket intervals are dynamic, based on the range of the underlying data. Use a post-aggregator to have finer control over the bucketing scheme|7|
|`lowerLimit`/`upperLimit`|Restrict the approximation to the given range. The values outside this range will be aggregated into two centroids. Counts of values outside this range are still maintained. |-INF/+INF|


### 近似直方图post-aggregators

post-aggregator把不透明的近似直方图草图转换成桶直方图表示,以及计算不同的分布指标,如分位数，最小值和最大值。

#### 平等的桶post-aggregator

计算一个用给定数量大小相等的垃圾箱的可视化近似直方图。
桶的间隔是基于底层数据的范围。

```json
{
  "type": "equalBuckets",
  "name": "<output_name>",
  "fieldName": "<aggregator_name>",
  "numBuckets": <count>
}
```

#### 桶post-aggregator

计算一个给定初始断点、偏移和桶的大小的可视化的视图。
桶的大小决定容器间隔的大小。
偏移决定这些间隔容器的对齐值。
```json
{
  "type": "buckets",
  "name": "<output_name>",
  "fieldName": "<aggregator_name>",
  "bucketSize": <bucket_size>,
  "offset": <offset>
}
```

#### 自定义桶post-aggregator

计算一个根据给定间隙布局的容器近似直方图的可视化视图。

```json
{ "type" : "customBuckets", "name" : <output_name>, "fieldName" : <aggregator_name>,
  "breaks" : [ <value>, <value>, ... ] }
```

#### 最小post-aggregator

返回底层近似直方图聚合器的最小值。

```json
{ "type" : "min", "name" : <output_name>, "fieldName" : <aggregator_name> }
```

#### 最大post-aggregator

返回底层近似直方图聚合器的最大值。
```json
{ "type" : "max", "name" : <output_name>, "fieldName" : <aggregator_name> }
```

#### 分位数post-aggregator

计算一个基于底层近似直方图聚合器的单个分位数
```json
{ "type" : "quantile", "name" : <output_name>, "fieldName" : <aggregator_name>,
  "probability" : <quantile> }
```

#### 分位数post-aggregator

计算基于底层近似直方图聚合器的一组分位数
```json
{ "type" : "quantiles", "name" : <output_name>, "fieldName" : <aggregator_name>,
  "probabilities" : [ <quantile>, <quantile>, ... ] }
```
