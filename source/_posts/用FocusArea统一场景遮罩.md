---
title: 用FocusArea统一场景遮罩
date: 2026-10-08 12:19:54
tags:
- FocusArea
categories:
- ArcGIS JS API
---
三维场景里同时挂了动态图层、切片图层和要素图层，要只展示划定范围内的内容。

按图层类型分别处理就得维护三套逻辑，最后用 FocusArea 一个 API 统一了。
<!-- more -->
## 背景

需求是只展示划定范围内的数据和服务。

场景里的图层有三类：动态图层（map-image）、切片图层（tile）和要素图层（feature）。

ArcGIS 对不同图层给的遮罩手段不一样，所以下面几条路都试过或者查过。

> 注：ArcGIS Maps SDK for JavaScript 5.1。

## 考虑

### FeatureEffect

要素图层官方有 [FeatureEffect](https://developers.arcgis.com/javascript/latest/references/core/layers/support/FeatureEffect/)，配一个 `FeatureFilter` 就能只显示范围内的要素：

```js
layer.featureEffect = new FeatureEffect({
  filter: new FeatureFilter({
    geometry: filterGeometry,
    spatialRelationship: "intersects",
    distance: 3,
    units: "miles",
  }),
});
```

但是官方写明 FeatureEffect 只在二维 MapView 生效，不支持三维 SceneView。

所以场景里的要素图层用不了，这条路在三维下就断了。

### clippingArea

三维场景另有 [SceneView.clippingArea](https://developers.arcgis.com/javascript/latest/references/core/views/SceneView/)，能裁掉范围外的部分。

它的值就是一个 `Extent`，没有 `ClippingArea` 这个类，也没有 intersect、exclude 之类的参数：

```js
const clippingExtent = {
  // autocasts as new Extent()
  xmin: -10932882,
  ymin: 4432667,
  xmax: -10834217,
  ymax: 4493918,
  spatialReference: {
    wkid: 3857,
  },
};

view.clippingArea = clippingExtent;
```

限制有两条：只在 `viewingMode` 为 `local` 的局部场景里生效，全局场景下取值为 `null`。

另一条是范围只能是矩形，多边形选区做不了。

### 服务端的clipping

动态图层可以把遮罩交给服务端，[export](https://developers.arcgis.com/rest/services-reference/enterprise/export-map/) 操作有一个 `clipping` 参数。

`MapImageLayer` 上没有 `clippingArea` 属性，参数靠 `customParameters` 附加到请求 URL 上：

```js
const layer = new MapImageLayer({
  url: serviceUrl,
  customParameters: {
    clipping: JSON.stringify({
      geometryType: "esriGeometryPolygon",
      geometry: polygon,
    }),
  },
});
```

`clipping` 是 10.8 才有的参数，只有 ArcGIS Pro 发布的地图服务支持，服务还得声明 `supportsClipping`。

> 注：`customParameters` 官方只说是附加到请求 URL 上的自定义参数，值必须是字符串。而且这条路只解决动态图层一类，其余两类还得各写一套。

### 覆写fetchTile

切片图层官方没有遮罩 API，能想到的办法是继承 `BaseTileLayer`，覆写 `fetchTile`。

取回瓦片图片后画到 canvas 上再返回，遮罩就在画之前做。

照着官方 [Custom TileLayer](https://developers.arcgis.com/javascript/latest/sample-code/layers-custom-tilelayer/) 示例改的骨架：

```js
const MaskTileLayer = BaseTileLayer.createSubclass({
  properties: {
    urlTemplate: null,
  },
  getTileUrl(level, row, col) {
    return this.urlTemplate.replace("{z}", level).replace("{x}", col).replace("{y}", row);
  },
  fetchTile(level, row, col, options) {
    const url = this.getTileUrl(level, row, col);
    return esriRequest(url, {
      responseType: "image",
      signal: options && options.signal,
    }).then((response) => {
      const image = response.data;
      const width = this.tileInfo.size[0];
      const height = this.tileInfo.size[0];
      const canvas = document.createElement("canvas");
      const context = canvas.getContext("2d");
      canvas.width = width;
      canvas.height = height;
      // 遮罩写在这里：先按 mask 多边形 clip，再画瓦片
      context.drawImage(image, 0, 0, width, height);
      return canvas;
    });
  },
});
```

这条路最后没有实现，因为 FocusArea 对切片图层同样生效。

### 倾斜的mask

倾斜摄影有专门的 API，`IntegratedMeshLayer.modifications` 里加一个 `type` 为 `mask` 的 `SceneModification`：

```js
meshLayer.modifications = new SceneModifications([
  new SceneModification({ geometry: polygon, type: "mask" }),
]);
```

只绘制多边形内的部分，几何只支持 Polygon。

跟前面几条一样，它只管倾斜网格一类图层。

## FocusArea

[FocusArea](https://developers.arcgis.com/javascript/latest/references/core/effects/FocusArea/) 是 4.33 加的，挂在 `Map` 的 `focusAreas` 上。

它不是裁掉几何，而是把范围外压暗或者提亮。

属于渲染层的处理，所以对场景里所有图层都生效，三类图层一个 API 就盖住了。

创建一个聚焦区并挂到 map 上：

```js
const focusArea = new FocusArea({
  title: "Focus Area",
  id: "focusarea-0",
  outline: { color: [255, 128, 128, 0.55] },
  geometries: new Collection([polygon]),
});

map.focusAreas.areas.add(focusArea);
map.focusAreas.style = "bright";
```

`style` 有 `bright` 和 `dark` 两个取值，作用在所有启用的聚焦区之外。

单个聚焦区的开关用 `enabled`：

```js
map.focusAreas.areas.at(0).enabled = false;
```

配合 `SketchViewModel` 可以让用户自己画范围，改完把几何塞回去：

```js
sketchViewModel.on("update", (event) => {
  focusArea.geometries = new Collection([event.graphics.at(0).geometry]);
});
```

参数说明：

+ `geometries`— 聚焦区的几何，只支持 Polygon，Z 值会被忽略.
+ `outline`— 聚焦区的边框，画在地面上.
+ `enabled`— 是否启用该聚焦区.
+ `style`— 聚焦区外的渲染样式，`bright` 或 `dark`，默认 `bright`.

> 注：官方示例是地图组件写法，属性挂在 `<arcgis-scene>` 元素上，等价于 `map.focusAreas`。

## 坑

- `focusAreas` 挂在 `Map` 上，但只在三维 `SceneView` 里生效，二维 MapView 没有这个属性。
- FocusArea 只做视觉弱化，数据没有被裁掉，图层查询和统计仍然会返回范围外的要素。
- `geometries` 的几何只支持 Polygon。
- `clippingArea` 的值只是 `Extent`，范围只能是矩形，而且只在局部场景生效。
- 服务端的 `clipping` 参数要求 ArcGIS Pro 发布的服务，并且服务要支持 `supportsClipping`。

官方示例里的效果，聚焦区内保持原色，范围外被提亮：

![focus-area-demo](https://gitee.com/GeoDaoyu/PicGo/raw/master/blog/focus-area-demo.png)

## 参考链接

https://developers.arcgis.com/javascript/latest/sample-code/focus-area/

https://developers.arcgis.com/javascript/latest/sample-code/scene-local/

https://developers.arcgis.com/javascript/latest/sample-code/featureeffect-geometry/

https://developers.arcgis.com/javascript/latest/sample-code/layers-integratedmeshlayer-modification/

https://developers.arcgis.com/javascript/latest/sample-code/layers-custom-tilelayer/

https://developers.arcgis.com/rest/services-reference/enterprise/export-map/
