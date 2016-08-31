---
layout: doc_page
---
# 查询

Druid支持过滤基于原始和约束指定地空间的索引列。
  
# 空间索引

在任何数据规范,提供空间维度的选择。例如,对于一个JSON数据规范、空间维度可以指定如下:

```json
"dataSpec" : {
    "format": "JSON",
    "dimensions": <some_dims>,
    "spatialDimensions": [
        {
            "dimName": "coordinates",
            "dims": ["lat", "long"]
        },
        ...
    ]
}
```

|属性|描述|要求|
|--------|-----------|---------|
|dimName|The name of the spatial dimension. A spatial dimension may be constructed from multiple other dimensions or it may already exist as part of an event. If a spatial dimension already exists, it must be an array of coordinate values.|yes|
|dims|A list of dimension names that comprise a spatial dimension.|no|

# 空间过滤器

空间过滤器的语法如下：

```json
"filter" : {
    "type": "spatial",
    "dimension": "spatialDim",
    "bound": {
        "type": "rectangular",
        "minCoords": [10.0, 20.0],
        "maxCoords": [30.0, 40.0]
    }
}
```

界限
---

### 直角

|属性|描述|要求|
|--------|-----------|---------|
|minCoords|List of minimum dimension coordinates for coordinates [x, y, z, …]|yes|
|maxCoords|List of maximum dimension coordinates for coordinates [x, y, z, …]|yes|

### 半径

|属性|描述|要求|
|--------|-----------|---------|
|coords|Origin coordinates in the form [x, y, z, …]|yes|
|radius|The float radius value|yes|

