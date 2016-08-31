---
layout: doc_page
---

# Druid 例子

## TwitterSpritzerFirehose

这个消防带直接关联twitter spritzer数据流。
示例规格：

```json
"firehose" : {
    "type" : "twitzer",
    "maxEventCount": -1,
    "maxRunMinutes": 0
}
```

|属性|描述|默认|要求|
|--------|-----------|-------|---------|
|type|This should be "twitzer"|N/A|yes|
|maxEventCount|max events to receive, -1 is infinite, 0 means nothing is delivered; use this to prevent infinite space consumption or to prevent getting throttled at an inconvenient time.|N/A|yes|
|maxRunMinutes|maximum number of minutes to fetch Twitter events.  Use this to prevent getting throttled at an inconvenient time. If zero or less, no time limit for run.|N/A|yes|
