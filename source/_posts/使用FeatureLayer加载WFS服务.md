---
title: 使用FeatureLayer加载WFS服务
date: 2020-07-02 20:03:29
tags:
- XML
- WFS
- FeatureLayer
- OGC
- ArcGIS JS API 
categories:
- ArcGIS JS API
---
ArcGIS JS API 在当前（4.15版本）提供有WebTileLayer、WMSLayer、WMTSLayer等图层类，但并没有提供加载WFS服务的图层类。所以使用FeatureLayer来实现加载。

> WFS，Web Feature Service，网络要素服务
<!-- more -->
## 思路

手动构造FeatureLayer的source 属性。

```flow
st=>start: Start
req=>operation: WFS请求
xml=>operation: 解析XML
feature=>operation: 实例化FeatureLayer
e=>end
st->req->xml->feature->e
```

## WFS请求

 ### GetCapabilities 

参数说明：

| 参数                    | 是否必须 | 默认值 |
| ----------------------- | -------- | ------ |
| SERVICE                 | Y        | WFS    |
| REQUEST=GetCapabilities | Y        |        |

请求示例：
```
http://www.someserver.com/wfs?

SERVICE=WFS&

REQUEST=GetCapabilities
```

### GetFeature 

参数说明：

| 参数               | 是否必须 | 默认值                      |
| ------------------ | -------- | --------------------------- |
| VERSION            | Y        | 1.1.0                       |
| SERVICE            | Y        | WFS                         |
| REQUEST=GetFeature | Y        |                             |
| TYPENAME           | Y        |                             |
| OUTPUTFORMAT       |          | text/xml; subtype=gml/3.1.1 |
| BBOX               |          |                             |
| FILTER             |          |                             |
| SORTBY             |          |                             |
| MAXFEATURES        |          |                             |
| PROPERTYNAME       |          |                             |
| SRSNAME            |          |                             |
| FEATUREID          |          |                             |
| EXPIRY             |          |                             |
| RESULTTYPE         |          | results                     |
| FEATUREVERSION     |          |                             |

请求示例：

```
http://www.someserver.com/wfs？

SERVICE=WFS&

VERSION=1.1.0&

REQUEST=GetFeature&

PROPERTYNAME=InWaterA_1M/wkbGeom,InWaterA_1M/tileId&

TYPENAME=InWaterA_1M&

FILTER=InWaterA_1M/wkbGeom
```

## 解析XML

从返回的XML中解析数据，构造Features数组。

解析过程示例代码：

```javascript
const parser = new DOMParser();
const xml = parser.parseFromString(response, 'text/xml');
const htmlCollection = xml.getElementsByTagName('wfs:FeatureCollection');
for (let i = 0, len = htmlCollection.length; i < len; i++) {
  console.log(htmlCollection[i].innerHTML);
}
```

Features数组：

``` javascript
const features = [
 {
   geometry: {
     type: "point",
     x: -100,
     y: 38
   },
   attributes: {
     ObjectID: 1,
     DepArpt: "KATL",
     MsgTime: Date.now(),
     FltId: "UAL1"
   }
 },
 ...
];
```

## 实例化FeatureLayer

```javascript
const layer = new FeatureLayer({
  source: features,  // autocast as a Collection of new Graphic()
  objectIdField: "ObjectID"
});
```
