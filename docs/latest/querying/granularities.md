---
layout: doc_page
---

# 聚合粒度
granularity field决定了数据在时间dimension怎么分布，或它如何被小时，天，分钟aggregated等等。
它可以作为简单的字符串指定granularities或作为任意granularities的对象。
### Simple Granularities

Simple Granularities指定作为string和bucket timestamps根据他们的UTC时间（例如，一天从00：00开始）。
Supported granularity string是：`all`， `none`， `minute`， `fifteen_minute`， `thirty_minute`，`hour` 和 `day`。
* `all` buckets都是一个single bucket 
* `none`并不是bucket data(它实际上使用granularity索引，minimum在这里是指`none` 这意味着millisecond granularity)。使用`none` [TimeseriesQuery](../querying/timeseriesquery.html)目前不推荐（系统为不存在的所有milliseconds生成0 values，这是常见的。）
#### 例如：

假设你下面的数据用millisecond ingestion granularity存储在Druid，
``` json
{"timestamp": "2013-08-31T01:02:33Z", "page": "AAA", "language" : "en"}
{"timestamp": "2013-09-01T01:02:33Z", "page": "BBB", "language" : "en"}
{"timestamp": "2013-09-02T23:32:45Z", "page": "CCC", "language" : "en"}
{"timestamp": "2013-09-03T03:32:45Z", "page": "DDD", "language" : "en"}
```

用`hour`提交一个groupBy query后
``` json
{
   "queryType":"groupBy",
   "dataSource":"my_dataSource",
   "granularity":"hour",
   "dimensions":[
      "language"
   ],
   "aggregations":[
      {
         "type":"count",
         "name":"count"
      }
   ],
   "intervals":[
      "2000-01-01T00:00Z/3000-01-01T00:00Z"
   ]
}
```

你将得到
``` json
[ {
  "version" : "v1",
  "timestamp" : "2013-08-31T01:00:00.000Z",
  "event" : {
    "count" : 1,
    "language" : "en"
  }
}, {
  "version" : "v1",
  "timestamp" : "2013-09-01T01:00:00.000Z",
  "event" : {
    "count" : 1,
    "language" : "en"
  }
}, {
  "version" : "v1",
  "timestamp" : "2013-09-02T23:00:00.000Z",
  "event" : {
    "count" : 1,
    "language" : "en"
  }
}, {
  "version" : "v1",
  "timestamp" : "2013-09-03T03:00:00.000Z",
  "event" : {
    "count" : 1,
    "language" : "en"
  }
} ]
```

注意所有的空buckets是不用的。

如果你改变granularity为 `day`，你将得到
``` json
[ {
  "version" : "v1",
  "timestamp" : "2013-08-31T00:00:00.000Z",
  "event" : {
    "count" : 1,
    "language" : "en"
  }
}, {
  "version" : "v1",
  "timestamp" : "2013-09-01T00:00:00.000Z",
  "event" : {
    "count" : 1,
    "language" : "en"
  }
}, {
  "version" : "v1",
  "timestamp" : "2013-09-02T00:00:00.000Z",
  "event" : {
    "count" : 1,
    "language" : "en"
  }
}, {
  "version" : "v1",
  "timestamp" : "2013-09-03T00:00:00.000Z",
  "event" : {
    "count" : 1,
    "language" : "en"
  }
} ]
```

如果你改变granularity 为`none`，你将得到与设置为ingestion granularity相同的结果。
``` json
[ {
  "version" : "v1",
  "timestamp" : "2013-08-31T01:02:33.000Z",
  "event" : {
    "count" : 1,
    "language" : "en"
  }
}, {
  "version" : "v1",
  "timestamp" : "2013-09-01T01:02:33.000Z",
  "event" : {
    "count" : 1,
    "language" : "en"
  }
}, {
  "version" : "v1",
  "timestamp" : "2013-09-02T23:32:45.000Z",
  "event" : {
    "count" : 1,
    "language" : "en"
  }
}, {
  "version" : "v1",
  "timestamp" : "2013-09-03T03:32:45.000Z",
  "event" : {
    "count" : 1,
    "language" : "en"
  }
} ]
```

有一个query granularity比ingestion granularity小并没有影响，
因为关于更小的granularity 的信息是没有展示在indexed data上的。
所以，如果query granularity比query granularity更小，druid产生的结果相当于对ingestion granularity设置了query granularity。
查看`queryGranularity` 在[Ingestion Spec](../ingestion/index.html).

如果你改变granularity为`all`，你将得到所有的aggregated在一个bucket里，
``` json
[ {
  "version" : "v1",
  "timestamp" : "2000-01-01T00:00:00.000Z",
  "event" : {
    "count" : 4,
    "language" : "en"
  }
} ]
```


### Duration Granularities

Duration granularities被指定为一个精确的时间以毫秒为单位和timestamps以UTC返回。Duration granularity values are in millis
它们还支持指定一个可选的起源、定义从哪里开始计数time buckets(默认为1970 - 01 - 01 - t00:00:00z)。
```javascript
{"type": "duration", "duration": 7200000}
```

每两个小时chunks up
```javascript
{"type": "duration", "duration": 3600000, "origin": "2012-01-01T00:30:00Z"}
```

每半个钟chunks up
#### 例如：

在上一个示例中重用的数据，24小时持续时间提交groupBy query后，
``` json
{
   "queryType":"groupBy",
   "dataSource":"my_dataSource",
   "granularity":{"type": "duration", "duration": "86400000"},
   "dimensions":[
      "language"
   ],
   "aggregations":[
      {
         "type":"count",
         "name":"count"
      }
   ],
   "intervals":[
      "2000-01-01T00:00Z/3000-01-01T00:00Z"
   ]
}
```

你将得到
``` json
[ {
  "version" : "v1",
  "timestamp" : "2013-08-31T00:00:00.000Z",
  "event" : {
    "count" : 1,
    "language" : "en"
  }
}, {
  "version" : "v1",
  "timestamp" : "2013-09-01T00:00:00.000Z",
  "event" : {
    "count" : 1,
    "language" : "en"
  }
}, {
  "version" : "v1",
  "timestamp" : "2013-09-02T00:00:00.000Z",
  "event" : {
    "count" : 1,
    "language" : "en"
  }
}, {
  "version" : "v1",
  "timestamp" : "2013-09-03T00:00:00.000Z",
  "event" : {
    "count" : 1,
    "language" : "en"
  }
} ]
```

如果你把granularity的原始设置改为 `2012-01-01T00:30:00Z`
``` javascript
   "granularity":{"type": "duration", "duration": "86400000", "origin":"2012-01-01T00:30:00Z"}
```

你将得到
``` json
[ {
  "version" : "v1",
  "timestamp" : "2013-08-31T00:30:00.000Z",
  "event" : {
    "count" : 1,
    "language" : "en"
  }
}, {
  "version" : "v1",
  "timestamp" : "2013-09-01T00:30:00.000Z",
  "event" : {
    "count" : 1,
    "language" : "en"
  }
}, {
  "version" : "v1",
  "timestamp" : "2013-09-02T00:30:00.000Z",
  "event" : {
    "count" : 1,
    "language" : "en"
  }
}, {
  "version" : "v1",
  "timestamp" : "2013-09-03T00:30:00.000Z",
  "event" : {
    "count" : 1,
    "language" : "en"
  }
} ]
```

注意每个bucket的timestamp在30分钟时开始。
### Period Granularities

Period granularities作为年，月，周，钟，分，秒任意时间组合（即P2W, P3M, PT1H30M, PT0.750S）在[ISO8601](https://en.wikipedia.org/wiki/ISO_8601)格式。它们支持指定的时区，那些周期边界开始以及时区返回的timestamps.默认情况下，年开始于一月，月开始于每个月的第一天，一周开始于星期一除非原始值被指定过。
时区是可选择的（默认为UTC）。开始是选择的（默认为在给定的时区1970-01-01T00:00:00）
```javascript
{"type": "period", "period": "P2D", "timeZone": "America/Los_Angeles"}
```

这将bucket两天在Pacific timezone上chunks。
```javascript
{"type": "period", "period": "P3M", "timeZone": "America/Los_Angeles", "origin": "2012-02-01T00:00:00-08:00"}
```

This will bucket by 3-month chunks in the Pacific timezone where the three-month quarters are defined as starting from February.
这将bucket3个月在Pacific timezone上chunks，当3个月的季度被定义为从二月开始。
#### 例子

重用的数据在之前的例子中，如果你用1天内提交groupBy查询在Pacific timezone，
``` json
{
   "queryType":"groupBy",
   "dataSource":"my_dataSource",
   "granularity":{"type": "period", "period": "P1D", "timeZone": "America/Los_Angeles"},
   "dimensions":[
      "language"
   ],
   "aggregations":[
      {
         "type":"count",
         "name":"count"
      }
   ],
   "intervals":[
      "1999-12-31T16:00:00.000-08:00/2999-12-31T16:00:00.000-08:00"
   ]
}
```

你将得到

``` json
[ {
  "version" : "v1",
  "timestamp" : "2013-08-30T00:00:00.000-07:00",
  "event" : {
    "count" : 1,
    "language" : "en"
  }
}, {
  "version" : "v1",
  "timestamp" : "2013-08-31T00:00:00.000-07:00",
  "event" : {
    "count" : 1,
    "language" : "en"
  }
}, {
  "version" : "v1",
  "timestamp" : "2013-09-02T00:00:00.000-07:00",
  "event" : {
    "count" : 2,
    "language" : "en"
  }
} ]
```

注意为每个bucket的timestamp已经改变为Pacific time. Row `{"timestamp": "2013-09-02T23:32:45Z", "page": "CCC", "language" : "en"}` 和
`{"timestamp": "2013-09-03T03:32:45Z", "page": "DDD", "language" : "en"}` 放在相同的bucket因为他们是在Pacific time相同的一天。

还要注意`intervals`groupBy查询将不会转换为指定的时区，时区指定粒度只应用于查询结果。

如果你设置粒度原始值为`1970-01-01T20:30:00-08:00`，
``` javascript
   "granularity":{"type": "period", "period": "P1D", "timeZone": "America/Los_Angeles", "origin": "1970-01-01T20:30:00-08:00"}
```

你将得到

``` json
[ {
  "version" : "v1",
  "timestamp" : "2013-08-29T20:30:00.000-07:00",
  "event" : {
    "count" : 1,
    "language" : "en"
  }
}, {
  "version" : "v1",
  "timestamp" : "2013-08-30T20:30:00.000-07:00",
  "event" : {
    "count" : 1,
    "language" : "en"
  }
}, {
  "version" : "v1",
  "timestamp" : "2013-09-01T20:30:00.000-07:00",
  "event" : {
    "count" : 1,
    "language" : "en"
  }
}, {
  "version" : "v1",
  "timestamp" : "2013-09-02T20:30:00.000-07:00",
  "event" : {
    "count" : 1,
    "language" : "en"
  }
} ]
```

注意你指定的`origin`并不会影响时区，它仅仅是用于作为开始点定位第一个granularity bucket。
在这种情况下，Row `{"timestamp": "2013-09-02T23:32:45Z", "page": "CCC", "language" : "en"}`和`{"timestamp": "2013-09-03T03:32:45Z", "page": "DDD", "language" : "en"}`
是不会有相同的bucket。
#### 支持时区
时区支持是 [Joda Time library](http://www.joda.org)提供的，它是用标准的IANA时区。查看[Joda Time supported timezones](http://joda-time.sourceforge.net/timezones.html).