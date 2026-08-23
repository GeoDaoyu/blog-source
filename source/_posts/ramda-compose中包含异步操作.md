---
title: ramda-compose中包含异步操作
date: 2020-09-20 16:42:18
tags:
- Ramda
categories:
- Ramda
---

在函数组合时，如果有异步操作，那么ramda中compose应该怎么写呢？

在写空间查询例子的时候，就遇到了这个问题，其中doQuery方法是一个异步函数。

解决方法是一个andThen函数。
<!-- more -->
代码如下：

```JavaScript
// highLight上图
let highLightHandler: __esri.Handle = null;
const highLight = (features: Array<Graphic>) => {
  view.whenLayerView(featureLayer).then(layerView => {
    if (highLightHandler) {
      highLightHandler.remove(); // 清空高亮
    }
    highLightHandler = layerView.highlight(features);
  });
}

// 使用查询条件查询返回查询结果
// @ts-ignore
const doQuery = async query => {
  const result = await featureLayer.queryFeatures(query);
  const { features } = result;
  return features;
}

// 使用绘制的图形生成查询条件
const generateQuery = (geometry: Geometry) => new Query({
  returnGeometry: true,
  outFields: ["NAME", "OBJECTID"], // highlight必须要OBJECTID字段
  geometry,
  outSpatialReference: view.spatialReference,
  spatialRelationship: "intersects"
});

// 查询结果展示到控制台
const showInTable = (features: Graphic[]) => {
  const attributes = features.map(feature => feature.attributes);
  console.table(attributes);
}

// 处理查询结果
const showQueryResult = (features: Graphic[]) => {
  showInTable(features);
  highLight(features);
  graphicsLayer.removeAll(); // 清空绘制图形
}

const workFlow = R.compose(
  R.andThen(showQueryResult), // 处理查询结果
  doQuery,                    // 使用查询条件查询返回查询结果      
  generateQuery               // 使用绘制的图形生成查询条件
);

// 2.在绘制结束，拿到绘制的图形
sketch.on("create", event => {
  if (event.state === "complete") {
    const geometry = event.graphic.geometry;
    workFlow(geometry);
  }
});
```

