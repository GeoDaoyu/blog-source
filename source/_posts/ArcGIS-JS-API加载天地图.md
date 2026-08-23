---
title: ArcGIS-JS-API加载天地图
date: 2020-07-06 21:27:58
tags:
- WMTS
- 天地图
- TileInfo
- SpatialReference
- WKT
categories:
- ArcGIS JS API
---
## 背景

在[天地图官网服务](<http://lbs.tianditu.gov.cn/server/MapService.html>)可以看到，天地图提供了多种底图服务，分别有两种投影类型，CGCS2000和Web Mercator。

![1584498518305](https://gitee.com/GeoDaoyu/PicGo/raw/master/blog/1584498518305.png)
<!-- more -->
## WMTSLayer加载失败

使用WMTSLayer进行加载

``` js
const wmtsLayer = new WMTSLayer({
  id: "wmts",
  url: "http://t0.tianditu.gov.cn/vec_c/wmts",
  customParameters: {
    tk: "b854fdb3a3b2625bd6c8353e83f7cca3"
  },
  serviceMode: "KVP",
});
```

结果：无报错，不上图。

查看源码得知：

WMTSLayer先请求元数据，然后构造tileInfo，最后用WebTileLayer加载。

## 问题排查

查看天地图和Esri的切片方案。

Esri的切片方案是从第0级开始，天地图的切片方案从第1级开始。

切片方案，附在文末附件中。

## WebTileLayer加载

4490和4326差别很小，可以叠加，所以CGCS2000的服务，采用4326来加载。

tileInfo中dpi，参考一篇[博文](<https://www.cnblogs.com/cehui0303/p/10978011.html>)，设置为`layers/support/wmtsUtils.js`中的90.71428571428571。

lods采用4326的切片方案，levelValue设置为天地图的level。

size和origin按天地图的切片方案设置。

spatialReference设置为4326。（如果设置为4490，也可以上图，但是view的center不生效。）

通过构造TileInfo，使用WebTileLayer来加载。

```typescript
import WebTileLayer from "esri/layers/WebTileLayer";
import TileInfo from "esri/layers/support/TileInfo";
import SpatialReference from "esri/geometry/SpatialReference";

const tileInfoWebMercator = new TileInfo({
  dpi: 90.71428571427429,
  lods: [{
    level: 0,
    scale: 591657527.591555,
    resolution: 156543.033928
  },
  {
    level: 1,
    scale: 295828763.795777,
    resolution: 78271.5169639999
  },
  {
    level: 2,
    scale: 147914381.897889,
    resolution: 39135.7584820001
  },
  {
    level: 3,
    scale: 73957190.948944,
    resolution: 19567.8792409999
  },
  {
    level: 4,
    scale: 36978595.474472,
    resolution: 9783.93962049996
  },
  {
    level: 5,
    scale: 18489297.737236,
    resolution: 4891.96981024998
  },
  {
    level: 6,
    scale: 9244648.868618,
    resolution: 2445.98490512499
  },
  {
    level: 7,
    scale: 4622324.434309,
    resolution: 1222.99245256249
  },
  {
    level: 8,
    scale: 2311162.217155,
    resolution: 611.49622628138
  },
  {
    level: 9,
    scale: 1155581.108577,
    resolution: 305.748113140558
  },
  {
    level: 10,
    scale: 577790.554289,
    resolution: 152.874056570411
  },
  {
    level: 11,
    scale: 288895.277144,
    resolution: 76.4370282850732
  },
  {
    level: 12,
    scale: 144447.638572,
    resolution: 38.2185141425366
  },
  {
    level: 13,
    scale: 72223.819286,
    resolution: 19.1092570712683
  },
  {
    level: 14,
    scale: 36111.909643,
    resolution: 9.55462853563415
  },
  {
    level: 15,
    scale: 18055.954822,
    resolution: 4.77731426794937
  },
  {
    level: 16,
    scale: 9027.977411,
    resolution: 2.38865713397468
  },
  {
    level: 17,
    scale: 4513.988705,
    resolution: 1.19432856685505
  },
  {
    level: 18,
    scale: 2256.994353,
    resolution: 0.597164283559817
  },
  {
    level: 19,
    scale: 1128.497176,
    resolution: 0.298582141647617
  }],
  size: [256, 256],
  origin: {
    x: -20037508.342787,
    y: 20037508.342787
  },
  spatialReference: SpatialReference.WebMercator
});
const tileInfoWGS84 = new TileInfo({
  dpi: 90.71428571427429,
  lods: [
    {
      level: 0,
      levelValue: "1",
      scale: 295828763.79585470937713011037,
      resolution: 0.703125
    },
    {
      level: 1,
      levelValue: "2",
      scale: 147914381.89792735468856505518,
      resolution: 0.3515625
    },
    {
      level: 2,
      levelValue: "3",
      scale: 73957190.948963677344282527592,
      resolution: 0.17578125
    },
    {
      level: 3,
      levelValue: "4",
      scale: 36978595.474481838672141263796,
      resolution: 0.087890625
    },
    {
      level: 4,
      levelValue: "5",
      scale: 18489297.737240919336070631898,
      resolution: 0.0439453125
    },
    {
      level: 5,
      levelValue: "6",
      scale: 9244648.868620459668035315949,
      resolution: 0.02197265625
    },
    {
      level: 6,
      levelValue: "7",
      scale: 4622324.4343102298340176579745,
      resolution: 0.010986328125
    },
    {
      level: 7,
      levelValue: "8",
      scale: 2311162.2171551149170088289872,
      resolution: 0.0054931640625
    },
    {
      level: 8,
      levelValue: "9",
      scale: 1155581.1085775574585044144937,
      resolution: 0.00274658203125
    },
    {
      level: 9,
      levelValue: "10",
      scale: 577790.55428877872925220724681,
      resolution: 0.001373291015625
    },
    {
      level: 10,
      levelValue: "11",
      scale: 288895.2771443893646261036234,
      resolution: 0.0006866455078125
    },
    {
      level: 11,
      levelValue: "12",
      scale: 144447.63857219468231305181171,
      resolution: 0.00034332275390625
    },
    {
      level: 12,
      levelValue: "13",
      scale: 72223.819286097341156525905853,
      resolution: 0.000171661376953125
    },
    {
      level: 13,
      levelValue: "14",
      scale: 36111.909643048670578262952926,
      resolution: 0.0000858306884765625
    },
    {
      level: 14,
      levelValue: "15",
      scale: 18055.954821524335289131476463,
      resolution: 0.00004291534423828125
    },
    {
      level: 15,
      levelValue: "16",
      scale: 9027.977410762167644565738231,
      resolution: 0.000021457672119140625
    },
    {
      level: 16,
      levelValue: "17",
      scale: 4513.9887053810838222828691158,
      resolution: 0.0000107288360595703125
    },
    {
      level: 17,
      levelValue: "18",
      scale: 2256.9943526905419111414345579,
      resolution: 0.00000536441802978515625
    },
    {
      level: 18,
      levelValue: "19",
      scale: 1128.4971763452709555707172788,
      resolution: 0.000002682209014892578125
    }
  ],
  size: [256, 256],
  origin: {
    x: -180,
    y: 90
  },
  spatialReference: SpatialReference.WGS84
});
export default class TianDiTuLayer extends WebTileLayer {
  constructor(props: WebTileLayer) {
    super(props);
    this.tileInfo = this.generateTileInfo(props.urlTemplate);
    this.urlTemplate = this.generateUrlTemplate(props.urlTemplate);
    this.subDomains = ["t0", "t1", "t2", "t3", "t4", "t5", "t6", "t7"];
  }
  
  private generateUrlTemplate(urlTemplate: string): string {
    const [layer, tileMatrixSet] = urlTemplate.split('/')[3].split('_');
    return `http://{subDomain}.tianditu.com/${layer}_${tileMatrixSet}/wmts?SERVICE=WMTS&VERSION=1.0.0&REQUEST=GetTile&LAYER=${layer}&STYLE=default&FORMAT=tiles&TILEMATRIXSET=${tileMatrixSet}&TILEMATRIX={level}&TILEROW={row}&TILECOL={col}&tk=ac0daf56728bbb77d9514ba3df69bcd3`;
  }

  private generateTileInfo(urlTemplate: string): TileInfo {
    const [layer, tileMatrixSet] = urlTemplate.split('/')[3].split('_');
    return tileMatrixSet === 'c' ? tileInfoWGS84 : tileInfoWebMercator;
  }
}
```

```typescript
new TianDiTuLayer({
  urlTemplate: "http://t0.tianditu.com/vec_w/wmts"
} as TianDiTuLayer);
new TianDiTuLayer({
  urlTemplate: "http://t0.tianditu.com/img_c/wmts",
  spatialReference: SpatialReference.WGS84
} as TianDiTuLayer);
```

## 附件

### 天地图-经纬度投影-切片方案

~~~ xml
<?xml version="1.0" encoding="UTF-8"?>
<Capabilities
    xsi:schemaLocation="http://www.opengis.net/wmts/1.0 http://schemas.opengis.net/wmts/1.0.0/wmtsGetCapabilities_response.xsd"
    version="1.0.0" xmlns="http://www.opengis.net/wmts/1.0"
    xmlns:ows="http://www.opengis.net/ows/1.1"
    xmlns:gml="http://www.opengis.net/gml"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xlink="http://www.w3.org/1999/xlink">
    <ows:ServiceIdentification>
        <ows:Title>在线地图服务</ows:Title>
        <ows:Abstract>基于OGC标准的地图服务</ows:Abstract>
        <ows:Keywords>
            <ows:Keyword>OGC</ows:Keyword>
        </ows:Keywords>
        <ows:ServiceType codeSpace="wmts"/>
        <ows:ServiceTypeVersion>1.0.0</ows:ServiceTypeVersion>
        <ows:Fees>none</ows:Fees>
        <ows:AccessConstraints>none</ows:AccessConstraints>
    </ows:ServiceIdentification>
    <ows:ServiceProvider>
        <ows:ProviderName>国家基础地理信息中心</ows:ProviderName>
        <ows:ProviderSite>http://www.tianditu.com</ows:ProviderSite>
        <ows:ServiceContact>
            <ows:IndividualName>Mr Liu</ows:IndividualName>
            <ows:PositionName>Software Engineer</ows:PositionName>
            <ows:ContactInfo>
                <ows:Phone>
                    <ows:Voice>010-88187700</ows:Voice>
                    <ows:Facsimile>010-88187700</ows:Facsimile>
                </ows:Phone>
                <ows:Address>
                    <ows:DeliveryPoint>北京市海淀区莲花池西路28号</ows:DeliveryPoint>
                    <ows:City>北京市</ows:City>
                    <ows:AdministrativeArea>北京市</ows:AdministrativeArea>
                    <ows:Country>中国</ows:Country>
                    <ows:PostalCode>101399</ows:PostalCode>
                    <ows:ElectronicMailAddress>tianditu.com</ows:ElectronicMailAddress>
                </ows:Address>
                <ows:OnlineResource xlink:type="simple" xlink:href="http://www.tianditu.com"/>
            </ows:ContactInfo>
        </ows:ServiceContact>
    </ows:ServiceProvider>
    <ows:OperationsMetadata>
        <ows:Operation name="GetCapabilities">
            <ows:DCP>
                <ows:HTTP>
                    <ows:Get xlink:href="http://t0.tianditu.com/vec_c/wmts?">
                        <ows:Constraint name="GetEncoding">
                            <ows:AllowedValues>
                                <ows:Value>KVP</ows:Value>
                            </ows:AllowedValues>
                        </ows:Constraint>
                    </ows:Get>
                </ows:HTTP>
            </ows:DCP>
        </ows:Operation>
        <ows:Operation name="GetTile">
            <ows:DCP>
                <ows:HTTP>
                    <ows:Get xlink:href="http://t0.tianditu.com/vec_c/wmts?">
                        <ows:Constraint name="GetEncoding">
                            <ows:AllowedValues>
                                <ows:Value>KVP</ows:Value>
                            </ows:AllowedValues>
                        </ows:Constraint>
                    </ows:Get>
                </ows:HTTP>
            </ows:DCP>
        </ows:Operation>
    </ows:OperationsMetadata>
    <Contents>
        <Layer>
            <ows:Title>vec</ows:Title>
            <ows:Abstract>vec</ows:Abstract>
            <ows:Identifier>vec</ows:Identifier>
            <ows:WGS84BoundingBox>
                <ows:LowerCorner>-180.0 -90.0</ows:LowerCorner>
                <ows:UpperCorner>180.0 90.0</ows:UpperCorner>
            </ows:WGS84BoundingBox>
            <ows:BoundingBox>
                <ows:LowerCorner>-180.0 -90.0</ows:LowerCorner>
                <ows:UpperCorner>180.0 90.0</ows:UpperCorner>
            </ows:BoundingBox>
            <Style>
                <ows:Identifier>default</ows:Identifier>
            </Style>
            <Format>tiles</Format>
            <TileMatrixSetLink>
                <TileMatrixSet>c</TileMatrixSet>
            </TileMatrixSetLink>
        </Layer>
        <TileMatrixSet>
            <ows:Identifier>c</ows:Identifier>
            <ows:SupportedCRS>urn:ogc:def:crs:EPSG::4490</ows:SupportedCRS>
            <TileMatrix>
                <ows:Identifier>1</ows:Identifier>
                <ScaleDenominator>2.958293554545656E8</ScaleDenominator>
                <TopLeftCorner>90.0 -180.0</TopLeftCorner>
                <TileWidth>256</TileWidth>
                <TileHeight>256</TileHeight>
                <MatrixWidth>2</MatrixWidth>
                <MatrixHeight>1</MatrixHeight>
            </TileMatrix>
            <TileMatrix>
                <ows:Identifier>2</ows:Identifier>
                <ScaleDenominator>1.479146777272828E8</ScaleDenominator>
                <TopLeftCorner>90.0 -180.0</TopLeftCorner>
                <TileWidth>256</TileWidth>
                <TileHeight>256</TileHeight>
                <MatrixWidth>4</MatrixWidth>
                <MatrixHeight>2</MatrixHeight>
            </TileMatrix>
            <TileMatrix>
                <ows:Identifier>3</ows:Identifier>
                <ScaleDenominator>7.39573388636414E7</ScaleDenominator>
                <TopLeftCorner>90.0 -180.0</TopLeftCorner>
                <TileWidth>256</TileWidth>
                <TileHeight>256</TileHeight>
                <MatrixWidth>8</MatrixWidth>
                <MatrixHeight>4</MatrixHeight>
            </TileMatrix>
            <TileMatrix>
                <ows:Identifier>4</ows:Identifier>
                <ScaleDenominator>3.69786694318207E7</ScaleDenominator>
                <TopLeftCorner>90.0 -180.0</TopLeftCorner>
                <TileWidth>256</TileWidth>
                <TileHeight>256</TileHeight>
                <MatrixWidth>16</MatrixWidth>
                <MatrixHeight>8</MatrixHeight>
            </TileMatrix>
            <TileMatrix>
                <ows:Identifier>5</ows:Identifier>
                <ScaleDenominator>1.848933471591035E7</ScaleDenominator>
                <TopLeftCorner>90.0 -180.0</TopLeftCorner>
                <TileWidth>256</TileWidth>
                <TileHeight>256</TileHeight>
                <MatrixWidth>32</MatrixWidth>
                <MatrixHeight>16</MatrixHeight>
            </TileMatrix>
            <TileMatrix>
                <ows:Identifier>6</ows:Identifier>
                <ScaleDenominator>9244667.357955175</ScaleDenominator>
                <TopLeftCorner>90.0 -180.0</TopLeftCorner>
                <TileWidth>256</TileWidth>
                <TileHeight>256</TileHeight>
                <MatrixWidth>64</MatrixWidth>
                <MatrixHeight>32</MatrixHeight>
            </TileMatrix>                
			    <TileMatrix>
                <ows:Identifier>7</ows:Identifier>
                <ScaleDenominator>4622333.678977588</ScaleDenominator>
                <TopLeftCorner>90.0 -180.0</TopLeftCorner>
                <TileWidth>256</TileWidth>
                <TileHeight>256</TileHeight>
                <MatrixWidth>128</MatrixWidth>
                <MatrixHeight>64</MatrixHeight>
            </TileMatrix>
            <TileMatrix>
                <ows:Identifier>8</ows:Identifier>
                <ScaleDenominator>2311166.839488794</ScaleDenominator>
                <TopLeftCorner>90.0 -180.0</TopLeftCorner>
                <TileWidth>256</TileWidth>
                <TileHeight>256</TileHeight>
                <MatrixWidth>256</MatrixWidth>
                <MatrixHeight>128</MatrixHeight>
            </TileMatrix>
            <TileMatrix>
                <ows:Identifier>9</ows:Identifier>
                <ScaleDenominator>1155583.419744397</ScaleDenominator>
                <TopLeftCorner>90.0 -180.0</TopLeftCorner>
                <TileWidth>256</TileWidth>
                <TileHeight>256</TileHeight>
                <MatrixWidth>512</MatrixWidth>
                <MatrixHeight>256</MatrixHeight>
            </TileMatrix>
            <TileMatrix>
                <ows:Identifier>10</ows:Identifier>
                <ScaleDenominator>577791.7098721985</ScaleDenominator>
                <TopLeftCorner>90.0 -180.0</TopLeftCorner>
                <TileWidth>256</TileWidth>
                <TileHeight>256</TileHeight>
                <MatrixWidth>1024</MatrixWidth>
                <MatrixHeight>512</MatrixHeight>
            </TileMatrix>
            <TileMatrix>
                <ows:Identifier>11</ows:Identifier>
                <ScaleDenominator>288895.85493609926</ScaleDenominator>
                <TopLeftCorner>90.0 -180.0</TopLeftCorner>
                <TileWidth>256</TileWidth>
                <TileHeight>256</TileHeight>
                <MatrixWidth>2048</MatrixWidth>
                <MatrixHeight>1024</MatrixHeight>
            </TileMatrix>
            <TileMatrix>
                <ows:Identifier>12</ows:Identifier>
                <ScaleDenominator>144447.92746804963</ScaleDenominator>
                <TopLeftCorner>90.0 -180.0</TopLeftCorner>
                <TileWidth>256</TileWidth>
                <TileHeight>256</TileHeight>
                <MatrixWidth>4096</MatrixWidth>
                <MatrixHeight>2048</MatrixHeight>
            </TileMatrix>
            <TileMatrix>
                <ows:Identifier>13</ows:Identifier>
                <ScaleDenominator>72223.96373402482</ScaleDenominator>
                <TopLeftCorner>90.0 -180.0</TopLeftCorner>
                <TileWidth>256</TileWidth>
                <TileHeight>256</TileHeight>
                <MatrixWidth>8192</MatrixWidth>
                <MatrixHeight>4096</MatrixHeight>
            </TileMatrix>
            <TileMatrix>
                <ows:Identifier>14</ows:Identifier>
                <ScaleDenominator>36111.98186701241</ScaleDenominator>
                <TopLeftCorner>90.0 -180.0</TopLeftCorner>
                <TileWidth>256</TileWidth>
                <TileHeight>256</TileHeight>
                <MatrixWidth>16384</MatrixWidth>
                <MatrixHeight>8192</MatrixHeight>
            </TileMatrix>
            <TileMatrix>
                <ows:Identifier>15</ows:Identifier>
                <ScaleDenominator>18055.990933506204</ScaleDenominator>
                <TopLeftCorner>90.0 -180.0</TopLeftCorner>
                <TileWidth>256</TileWidth>
                <TileHeight>256</TileHeight>
                <MatrixWidth>32768</MatrixWidth>
                <MatrixHeight>16384</MatrixHeight>
            </TileMatrix>
            <TileMatrix>
                <ows:Identifier>16</ows:Identifier>
                <ScaleDenominator>9027.995466753102</ScaleDenominator>
                <TopLeftCorner>90.0 -180.0</TopLeftCorner>
                <TileWidth>256</TileWidth>
                <TileHeight>256</TileHeight>
                <MatrixWidth>65536</MatrixWidth>
                <MatrixHeight>32768</MatrixHeight>
            </TileMatrix>
            <TileMatrix>
                <ows:Identifier>17</ows:Identifier>
                <ScaleDenominator>4513.997733376551</ScaleDenominator>
                <TopLeftCorner>90.0 -180.0</TopLeftCorner>
                <TileWidth>256</TileWidth>
                <TileHeight>256</TileHeight>
                <MatrixWidth>131072</MatrixWidth>
                <MatrixHeight>65536</MatrixHeight>
			</TileMatrix>
            <TileMatrix>
                <ows:Identifier>18</ows:Identifier>
                <ScaleDenominator>2256.998866688275</ScaleDenominator>
                <TopLeftCorner>90.0 -180.0</TopLeftCorner>
                <TileWidth>256</TileWidth>
                <TileHeight>256</TileHeight>
                <MatrixWidth>262144</MatrixWidth>
                <MatrixHeight>131072</MatrixHeight>
            </TileMatrix>
			<TileMatrix>
                <ows:Identifier>19</ows:Identifier>
                <ScaleDenominator>1128.4994333441375</ScaleDenominator>
                <TopLeftCorner>90.0 -180.0</TopLeftCorner>
                <TileWidth>256</TileWidth>
                <TileHeight>256</TileHeight>
                <MatrixWidth>524288</MatrixWidth>
                <MatrixHeight>262144</MatrixHeight>
            </TileMatrix>
        </TileMatrixSet>
    </Contents>
</Capabilities>
~~~

### 天地图-球面墨卡托投影-切片方案

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Capabilities
    xsi:schemaLocation="http://www.opengis.net/wmts/1.0 http://schemas.opengis.net/wmts/1.0.0/wmtsGetCapabilities_response.xsd"
    version="1.0.0" xmlns="http://www.opengis.net/wmts/1.0"
    xmlns:ows="http://www.opengis.net/ows/1.1"
    xmlns:gml="http://www.opengis.net/gml"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xlink="http://www.w3.org/1999/xlink">
    <ows:ServiceIdentification>
        <ows:Title>在线地图服务</ows:Title>
        <ows:Abstract>基于OGC标准的地图服务</ows:Abstract>
        <ows:Keywords>
            <ows:Keyword>OGC</ows:Keyword>
        </ows:Keywords>
        <ows:ServiceType codeSpace="wmts"/>
        <ows:ServiceTypeVersion>1.0.0</ows:ServiceTypeVersion>
        <ows:Fees>none</ows:Fees>
        <ows:AccessConstraints>none</ows:AccessConstraints>
    </ows:ServiceIdentification>
    <ows:ServiceProvider>
        <ows:ProviderName>国家基础地理信息中心</ows:ProviderName>
        <ows:ProviderSite>http://www.tianditu.com</ows:ProviderSite>
        <ows:ServiceContact>
            <ows:IndividualName>Mr Liu</ows:IndividualName>
            <ows:PositionName>Software Engineer</ows:PositionName>
            <ows:ContactInfo>
                <ows:Phone>
                    <ows:Voice>010-88187700</ows:Voice>
                    <ows:Facsimile>010-88187700</ows:Facsimile>
                </ows:Phone>
                <ows:Address>
                    <ows:DeliveryPoint>北京市海淀区莲花池西路28号</ows:DeliveryPoint>
                    <ows:City>北京市</ows:City>
                    <ows:AdministrativeArea>北京市</ows:AdministrativeArea>
                    <ows:Country>中国</ows:Country>
                    <ows:PostalCode>101399</ows:PostalCode>
                    <ows:ElectronicMailAddress>tianditu.com</ows:ElectronicMailAddress>
                </ows:Address>
                <ows:OnlineResource xlink:type="simple" xlink:href="http://www.tianditu.com"/>
            </ows:ContactInfo>
        </ows:ServiceContact>
    </ows:ServiceProvider>
    <ows:OperationsMetadata>
        <ows:Operation name="GetCapabilities">
            <ows:DCP>
                <ows:HTTP>
                    <ows:Get xlink:href="http://t0.tianditu.com/vec_w/wmts?">
                        <ows:Constraint name="GetEncoding">
                            <ows:AllowedValues>
                                <ows:Value>KVP</ows:Value>
                            </ows:AllowedValues>
                        </ows:Constraint>
                    </ows:Get>
                </ows:HTTP>
            </ows:DCP>
        </ows:Operation>
        <ows:Operation name="GetTile">
            <ows:DCP>
                <ows:HTTP>
                    <ows:Get xlink:href="http://t0.tianditu.com/vec_w/wmts?">
                        <ows:Constraint name="GetEncoding">
                            <ows:AllowedValues>
                                <ows:Value>KVP</ows:Value>
                            </ows:AllowedValues>
                        </ows:Constraint>
                    </ows:Get>
                </ows:HTTP>
            </ows:DCP>
        </ows:Operation>
    </ows:OperationsMetadata>
    <Contents>
        <Layer>
            <ows:Title>vec</ows:Title>
            <ows:Abstract>vec</ows:Abstract>
            <ows:Identifier>vec</ows:Identifier>
            <ows:WGS84BoundingBox>
                <ows:LowerCorner>-20037508.3427892 -20037508.3427892</ows:LowerCorner>
                <ows:UpperCorner>20037508.3427892 20037508.3427892</ows:UpperCorner>
            </ows:WGS84BoundingBox>
            <ows:BoundingBox>
                <ows:LowerCorner>-20037508.3427892 -20037508.3427892</ows:LowerCorner>
                <ows:UpperCorner>20037508.3427892 20037508.3427892</ows:UpperCorner>
            </ows:BoundingBox>
            <Style>
                <ows:Identifier>default</ows:Identifier>
            </Style>
            <Format>tiles</Format>
            <TileMatrixSetLink>
                <TileMatrixSet>w</TileMatrixSet>
            </TileMatrixSetLink>
        </Layer>
        <TileMatrixSet>
            <ows:Identifier>w</ows:Identifier>
            <ows:SupportedCRS>urn:ogc:def:crs:EPSG::900913</ows:SupportedCRS>
            <TileMatrix>
                <ows:Identifier>1</ows:Identifier>
                <ScaleDenominator>2.958293554545656E8</ScaleDenominator>
                <TopLeftCorner>20037508.3427892 -20037508.3427892</TopLeftCorner>
                <TileWidth>256</TileWidth>
                <TileHeight>256</TileHeight>
                <MatrixWidth>2</MatrixWidth>
                <MatrixHeight>2</MatrixHeight>
            </TileMatrix>
            <TileMatrix>
                <ows:Identifier>2</ows:Identifier>
                <ScaleDenominator>1.479146777272828E8</ScaleDenominator>
                <TopLeftCorner>20037508.3427892 -20037508.3427892</TopLeftCorner>
                <TileWidth>256</TileWidth>
                <TileHeight>256</TileHeight>
                <MatrixWidth>4</MatrixWidth>
                <MatrixHeight>4</MatrixHeight>
            </TileMatrix>
            <TileMatrix>
                <ows:Identifier>3</ows:Identifier>
                <ScaleDenominator>7.39573388636414E7</ScaleDenominator>
                <TopLeftCorner>20037508.3427892 -20037508.3427892</TopLeftCorner>
                <TileWidth>256</TileWidth>
                <TileHeight>256</TileHeight>
                <MatrixWidth>8</MatrixWidth>
                <MatrixHeight>8</MatrixHeight>
            </TileMatrix>
            <TileMatrix>
                <ows:Identifier>4</ows:Identifier>
                <ScaleDenominator>3.69786694318207E7</ScaleDenominator>
                <TopLeftCorner>20037508.3427892 -20037508.3427892</TopLeftCorner>
                <TileWidth>256</TileWidth>
                <TileHeight>256</TileHeight>
                <MatrixWidth>16</MatrixWidth>
                <MatrixHeight>16</MatrixHeight>
            </TileMatrix>
            <TileMatrix>
                <ows:Identifier>5</ows:Identifier>
                <ScaleDenominator>1.848933471591035E7</ScaleDenominator>
                <TopLeftCorner>20037508.3427892 -20037508.3427892</TopLeftCorner>
                <TileWidth>256</TileWidth>
                <TileHeight>256</TileHeight>
                <MatrixWidth>32</MatrixWidth>
                <MatrixHeight>32</MatrixHeight>
            </TileMatrix>
            <TileMatrix>
                <ows:Identifier>6</ows:Identifier>
                <ScaleDenominator>9244667.357955175</ScaleDenominator>
                <TopLeftCorner>20037508.3427892 -20037508.3427892</TopLeftCorner>
                <TileWidth>256</TileWidth>
                <TileHeight>256</TileHeight>
                <MatrixWidth>64</MatrixWidth>
                <MatrixHeight>64</MatrixHeight>
            </TileMatrix>
			      <TileMatrix>
                <ows:Identifier>7</ows:Identifier>
                <ScaleDenominator>4622333.678977588</ScaleDenominator>
                <TopLeftCorner>20037508.3427892 -20037508.3427892</TopLeftCorner>
                <TileWidth>256</TileWidth>
                <TileHeight>256</TileHeight>
                <MatrixWidth>128</MatrixWidth>
                <MatrixHeight>128</MatrixHeight>
            </TileMatrix>
            <TileMatrix>
                <ows:Identifier>8</ows:Identifier>
                <ScaleDenominator>2311166.839488794</ScaleDenominator>
                <TopLeftCorner>20037508.3427892 -20037508.3427892</TopLeftCorner>
                <TileWidth>256</TileWidth>
                <TileHeight>256</TileHeight>
                <MatrixWidth>256</MatrixWidth>
                <MatrixHeight>256</MatrixHeight>
            </TileMatrix>
            <TileMatrix>
                <ows:Identifier>9</ows:Identifier>
                <ScaleDenominator>1155583.419744397</ScaleDenominator>
                <TopLeftCorner>20037508.3427892 -20037508.3427892</TopLeftCorner>
                <TileWidth>256</TileWidth>
                <TileHeight>256</TileHeight>
                <MatrixWidth>512</MatrixWidth>
                <MatrixHeight>512</MatrixHeight>
            </TileMatrix>
            <TileMatrix>
                <ows:Identifier>10</ows:Identifier>
                <ScaleDenominator>577791.7098721985</ScaleDenominator>
                <TopLeftCorner>20037508.3427892 -20037508.3427892</TopLeftCorner>
                <TileWidth>256</TileWidth>
                <TileHeight>256</TileHeight>
                <MatrixWidth>1024</MatrixWidth>
                <MatrixHeight>1024</MatrixHeight>
            </TileMatrix>
            <TileMatrix>
                <ows:Identifier>11</ows:Identifier>
                <ScaleDenominator>288895.85493609926</ScaleDenominator>
                <TopLeftCorner>20037508.3427892 -20037508.3427892</TopLeftCorner>
                <TileWidth>256</TileWidth>
                <TileHeight>256</TileHeight>
                <MatrixWidth>2048</MatrixWidth>
                <MatrixHeight>2048</MatrixHeight>
            </TileMatrix>
            <TileMatrix>
                <ows:Identifier>12</ows:Identifier>
                <ScaleDenominator>144447.92746804963</ScaleDenominator>
                <TopLeftCorner>20037508.3427892 -20037508.3427892</TopLeftCorner>
                <TileWidth>256</TileWidth>
                <TileHeight>256</TileHeight>
                <MatrixWidth>4096</MatrixWidth>
                <MatrixHeight>4096</MatrixHeight>
            </TileMatrix>
            <TileMatrix>
                <ows:Identifier>13</ows:Identifier>
                <ScaleDenominator>72223.96373402482</ScaleDenominator>
                <TopLeftCorner>20037508.3427892 -20037508.3427892</TopLeftCorner>
                <TileWidth>256</TileWidth>
                <TileHeight>256</TileHeight>
                <MatrixWidth>8192</MatrixWidth>
                <MatrixHeight>8192</MatrixHeight>
            </TileMatrix>
            <TileMatrix>
                <ows:Identifier>14</ows:Identifier>
                <ScaleDenominator>36111.98186701241</ScaleDenominator>
                <TopLeftCorner>20037508.3427892 -20037508.3427892</TopLeftCorner>
                <TileWidth>256</TileWidth>
                <TileHeight>256</TileHeight>
                <MatrixWidth>16384</MatrixWidth>
                <MatrixHeight>16384</MatrixHeight>
            </TileMatrix>
            <TileMatrix>
                <ows:Identifier>15</ows:Identifier>
                <ScaleDenominator>18055.990933506204</ScaleDenominator>
                <TopLeftCorner>20037508.3427892 -20037508.3427892</TopLeftCorner>
                <TileWidth>256</TileWidth>
                <TileHeight>256</TileHeight>
                <MatrixWidth>32768</MatrixWidth>
                <MatrixHeight>32768</MatrixHeight>
            </TileMatrix>
            <TileMatrix>
                <ows:Identifier>16</ows:Identifier>
                <ScaleDenominator>9027.995466753102</ScaleDenominator>
                <TopLeftCorner>20037508.3427892 -20037508.3427892</TopLeftCorner>
                <TileWidth>256</TileWidth>
                <TileHeight>256</TileHeight>
                <MatrixWidth>65536</MatrixWidth>
                <MatrixHeight>65536</MatrixHeight>
            </TileMatrix>
            <TileMatrix>
                <ows:Identifier>17</ows:Identifier>
                <ScaleDenominator>4513.997733376551</ScaleDenominator>
                <TopLeftCorner>20037508.3427892 -20037508.3427892</TopLeftCorner>
                <TileWidth>256</TileWidth>
                <TileHeight>256</TileHeight>
                <MatrixWidth>131072</MatrixWidth>
                <MatrixHeight>131072</MatrixHeight>
			</TileMatrix>
            <TileMatrix>
                <ows:Identifier>18</ows:Identifier>
                <ScaleDenominator>2256.998866688275</ScaleDenominator>
                <TopLeftCorner>20037508.3427892 -20037508.3427892</TopLeftCorner>
                <TileWidth>256</TileWidth>
                <TileHeight>256</TileHeight>
                <MatrixWidth>262144</MatrixWidth>
                <MatrixHeight>262144</MatrixHeight>
            </TileMatrix>
			<TileMatrix>
                <ows:Identifier>19</ows:Identifier>
                <ScaleDenominator>1128.4994333441375</ScaleDenominator>
                <TopLeftCorner>90.0 -180.0</TopLeftCorner>
                <TileWidth>256</TileWidth>
                <TileHeight>256</TileHeight>
                <MatrixWidth>524288</MatrixWidth>
                <MatrixHeight>262144</MatrixHeight>
            </TileMatrix>
        </TileMatrixSet>
    </Contents>
</Capabilities>
```

### Esri-WGS84_Geographic_Coordinate_System_V2

```xml
<?xml version="1.0" encoding="utf-8" ?>
<CacheInfo xsi:type='typens:CacheInfo' xmlns:xsi='http://www.w3.org/2001/XMLSchema-instance' xmlns:xs='http://www.w3.org/2001/XMLSchema' xmlns:typens='http://www.esri.com/schemas/ArcGIS/10.0'>
  <TileCacheInfo xsi:type='typens:TileCacheInfo'>
	<SpatialReference xsi:type="typens:GeographicCoordinateSystem">
		<WKT>GEOGCS["GCS_WGS_1984",DATUM["D_WGS_1984",SPHEROID["WGS_1984",6378137.0,298.257223563]],PRIMEM["Greenwich",0.0],UNIT["Degree",0.0174532925199433]]</WKT>
		<XOrigin>-399.99999999999989</XOrigin>
		<YOrigin>-399.99999999999989</YOrigin>
		<XYScale>11258999068426.24</XYScale>
		<ZOrigin>-100000</ZOrigin>
		<ZScale>10000</ZScale>
		<MOrigin>-100000</MOrigin>
		<MScale>10000</MScale>
		<XYTolerance>8.9831528411952133e-009</XYTolerance>
		<ZTolerance>0.001</ZTolerance>
		<MTolerance>0.001</MTolerance>
		<HighPrecision>true</HighPrecision>
		<LeftLongitude>-180</LeftLongitude>
		<WKID>4326</WKID>
	</SpatialReference>
    <TileOrigin xsi:type='typens:PointN'>
      <X>-180.0</X>
      <Y>90.0</Y>
    </TileOrigin>
    <TileCols>256</TileCols>
    <TileRows>256</TileRows>
    <DPI>96</DPI>
    <PreciseDPI>96</PreciseDPI>
    <LODInfos xsi:type="typens:ArrayOfLODInfo">
      <LODInfo xsi:type='typens:LODInfo'>
        <LevelID>0</LevelID>
        <Scale>295828763.79585470937713011037</Scale>
        <Resolution>0.703125</Resolution>
      </LODInfo>
      <LODInfo xsi:type='typens:LODInfo'>
        <LevelID>1</LevelID>
        <Scale>147914381.89792735468856505518</Scale>
        <Resolution>0.3515625</Resolution>
      </LODInfo>
      <LODInfo xsi:type='typens:LODInfo'>
        <LevelID>2</LevelID>
        <Scale>73957190.948963677344282527592</Scale>
        <Resolution>0.17578125</Resolution>
      </LODInfo>
      <LODInfo xsi:type='typens:LODInfo'>
        <LevelID>3</LevelID>
        <Scale>36978595.474481838672141263796</Scale>
        <Resolution>0.087890625</Resolution>
      </LODInfo>
      <LODInfo xsi:type='typens:LODInfo'>
        <LevelID>4</LevelID>
        <Scale>18489297.737240919336070631898</Scale>
        <Resolution>0.0439453125</Resolution>
      </LODInfo>
      <LODInfo xsi:type='typens:LODInfo'>
        <LevelID>5</LevelID>
        <Scale>9244648.868620459668035315949</Scale>
        <Resolution>0.02197265625</Resolution>
      </LODInfo>
      <LODInfo xsi:type='typens:LODInfo'>
        <LevelID>6</LevelID>
        <Scale>4622324.4343102298340176579745</Scale>
        <Resolution>0.010986328125</Resolution>
      </LODInfo>
      <LODInfo xsi:type='typens:LODInfo'>
        <LevelID>7</LevelID>
        <Scale>2311162.2171551149170088289872</Scale>
        <Resolution>0.0054931640625</Resolution>
      </LODInfo>
      <LODInfo xsi:type='typens:LODInfo'>
        <LevelID>8</LevelID>
        <Scale>1155581.1085775574585044144937</Scale>
        <Resolution>0.00274658203125</Resolution>
      </LODInfo>
      <LODInfo xsi:type='typens:LODInfo'>
        <LevelID>9</LevelID>
        <Scale>577790.55428877872925220724681</Scale>
        <Resolution>0.001373291015625</Resolution>
      </LODInfo>
      <LODInfo xsi:type='typens:LODInfo'>
        <LevelID>10</LevelID>
        <Scale>288895.27714438936462610362340</Scale>
        <Resolution>0.0006866455078125</Resolution>
      </LODInfo>
      <LODInfo xsi:type='typens:LODInfo'>
        <LevelID>11</LevelID>
        <Scale>144447.63857219468231305181171</Scale>
        <Resolution>0.00034332275390625</Resolution>
      </LODInfo>
      <LODInfo xsi:type='typens:LODInfo'>
        <LevelID>12</LevelID>
        <Scale>72223.819286097341156525905853</Scale>
        <Resolution>0.000171661376953125</Resolution>
      </LODInfo>
      <LODInfo xsi:type='typens:LODInfo'>
        <LevelID>13</LevelID>
        <Scale>36111.909643048670578262952926</Scale>
        <Resolution>0.0000858306884765625</Resolution>
      </LODInfo>
      <LODInfo xsi:type='typens:LODInfo'>
        <LevelID>14</LevelID>
        <Scale>18055.954821524335289131476463</Scale>
        <Resolution>0.00004291534423828125</Resolution>
      </LODInfo>
      <LODInfo xsi:type='typens:LODInfo'>
        <LevelID>15</LevelID>
        <Scale>9027.977410762167644565738231</Scale>
        <Resolution>0.000021457672119140625</Resolution>
      </LODInfo>
      <LODInfo xsi:type='typens:LODInfo'>
        <LevelID>16</LevelID>
        <Scale>4513.9887053810838222828691158</Scale>
        <Resolution>0.0000107288360595703125</Resolution>
      </LODInfo>
      <LODInfo xsi:type='typens:LODInfo'>
        <LevelID>17</LevelID>
        <Scale>2256.9943526905419111414345579</Scale>
        <Resolution>0.00000536441802978515625</Resolution>
      </LODInfo>
      <LODInfo xsi:type='typens:LODInfo'>
        <LevelID>18</LevelID>
        <Scale>1128.4971763452709555707172788</Scale>
        <Resolution>0.000002682209014892578125</Resolution>
      </LODInfo>
      <LODInfo xsi:type='typens:LODInfo'>
        <LevelID>19</LevelID>
        <Scale>564.24858817263547778535863938</Scale>
        <Resolution>0.0000013411045074462890625</Resolution>
      </LODInfo>
      <LODInfo xsi:type='typens:LODInfo'>
        <LevelID>20</LevelID>
        <Scale>282.12429408631773889267931988</Scale>
        <Resolution>0.00000067055225372314453125</Resolution>
      </LODInfo>
      <LODInfo xsi:type='typens:LODInfo'>
        <LevelID>21</LevelID>
        <Scale>141.06214704315886944633965975</Scale>
        <Resolution>0.000000335276126861572265625</Resolution>
      </LODInfo>
      <LODInfo xsi:type='typens:LODInfo'>
        <LevelID>22</LevelID>
        <Scale>70.531073521579434723169829875</Scale>
        <Resolution>0.0000001676380634307861328125</Resolution>
      </LODInfo>
    </LODInfos>
  </TileCacheInfo>
  <TileImageInfo xsi:type='typens:TileImageInfo'>
    <CacheTileFormat>PNG</CacheTileFormat>
    <CompressionQuality>0</CompressionQuality>
    <Antialiasing>false</Antialiasing>
  </TileImageInfo>
</CacheInfo>
```

### Esri-ArcGIS_Online_Bing_Maps_Google_Maps

``` xml
<?xml version="1.0" encoding="utf-8" ?>
<CacheInfo xsi:type='typens:CacheInfo' 
  xmlns:xsi='http://www.w3.org/2001/XMLSchema-instance' 
  xmlns:xs='http://www.w3.org/2001/XMLSchema' 
  xmlns:typens='http://www.esri.com/schemas/ArcGIS/9.3'>
  <TileCacheInfo xsi:type='typens:TileCacheInfo'>
    <SpatialReference xsi:type='typens:ProjectedCoordinateSystem'>
      <WKT>PROJCS[&quot;WGS_1984_Web_Mercator_Auxiliary_Sphere&quot;,GEOGCS[&quot;GCS_WGS_1984&quot;,DATUM[&quot;D_WGS_1984&quot;,SPHEROID[&quot;WGS_1984&quot;,6378137.0,298.257223563]],PRIMEM[&quot;Greenwich&quot;,0.0],UNIT[&quot;Degree&quot;,0.0174532925199433]],PROJECTION[&quot;Mercator_Auxiliary_Sphere&quot;],PARAMETER[&quot;False_Easting&quot;,0.0],PARAMETER[&quot;False_Northing&quot;,0.0],PARAMETER[&quot;Central_Meridian&quot;,0.0],PARAMETER[&quot;Standard_Parallel_1&quot;,0.0],PARAMETER[&quot;Auxiliary_Sphere_Type&quot;,0.0],UNIT[&quot;Meter&quot;,1.0],AUTHORITY[&quot;EPSG&quot;,3857]]</WKT>
      <XOrigin>-20037700</XOrigin>
      <YOrigin>-30241100</YOrigin>
      <XYScale>148923141.92838538</XYScale>
      <ZOrigin>-100000</ZOrigin>
      <ZScale>10000</ZScale>
      <MOrigin>-100000</MOrigin>
      <MScale>10000</MScale>
      <XYTolerance>0.001</XYTolerance>
      <ZTolerance>0.001</ZTolerance>
      <MTolerance>0.001</MTolerance>
      <HighPrecision>true</HighPrecision>
      <WKID>3857</WKID>
    </SpatialReference>
    <TileOrigin xsi:type='typens:PointN'>
      <X>-20037508.342787</X>
      <Y>20037508.342787</Y>
    </TileOrigin>
    <TileCols>256</TileCols>
    <TileRows>256</TileRows>
    <DPI>96</DPI>
    <LODInfos xsi:type='typens:ArrayOfLODInfo'>
      <LODInfo xsi:type='typens:LODInfo'>
        <LevelID>0</LevelID>
        <Scale>591657527.591555</Scale>
        <Resolution>156543.033928</Resolution>
      </LODInfo>
      <LODInfo xsi:type='typens:LODInfo'>
        <LevelID>1</LevelID>
        <Scale>295828763.795777</Scale>
        <Resolution>78271.5169639999</Resolution>
      </LODInfo>
      <LODInfo xsi:type='typens:LODInfo'>
        <LevelID>2</LevelID>
        <Scale>147914381.897889</Scale>
        <Resolution>39135.7584820001</Resolution>
      </LODInfo>
      <LODInfo xsi:type='typens:LODInfo'>
        <LevelID>3</LevelID>
        <Scale>73957190.948944</Scale>
        <Resolution>19567.8792409999</Resolution>
      </LODInfo>
      <LODInfo xsi:type='typens:LODInfo'>
        <LevelID>4</LevelID>
        <Scale>36978595.474472</Scale>
        <Resolution>9783.93962049996</Resolution>
      </LODInfo>
      <LODInfo xsi:type='typens:LODInfo'>
        <LevelID>5</LevelID>
        <Scale>18489297.737236</Scale>
        <Resolution>4891.96981024998</Resolution>
      </LODInfo>
      <LODInfo xsi:type='typens:LODInfo'>
        <LevelID>6</LevelID>
        <Scale>9244648.868618</Scale>
        <Resolution>2445.98490512499</Resolution>
      </LODInfo>
      <LODInfo xsi:type='typens:LODInfo'>
        <LevelID>7</LevelID>
        <Scale>4622324.434309</Scale>
        <Resolution>1222.99245256249</Resolution>
      </LODInfo>
      <LODInfo xsi:type='typens:LODInfo'>
        <LevelID>8</LevelID>
        <Scale>2311162.217155</Scale>
        <Resolution>611.49622628138</Resolution>
      </LODInfo>
      <LODInfo xsi:type='typens:LODInfo'>
        <LevelID>9</LevelID>
        <Scale>1155581.108577</Scale>
        <Resolution>305.748113140558</Resolution>
      </LODInfo>
      <LODInfo xsi:type='typens:LODInfo'>
        <LevelID>10</LevelID>
        <Scale>577790.554289</Scale>
        <Resolution>152.874056570411</Resolution>
      </LODInfo>
      <LODInfo xsi:type='typens:LODInfo'>
        <LevelID>11</LevelID>
        <Scale>288895.277144</Scale>
        <Resolution>76.4370282850732</Resolution>
      </LODInfo>
      <LODInfo xsi:type='typens:LODInfo'>
        <LevelID>12</LevelID>
        <Scale>144447.638572</Scale>
        <Resolution>38.2185141425366</Resolution>
      </LODInfo>
      <LODInfo xsi:type='typens:LODInfo'>
        <LevelID>13</LevelID>
        <Scale>72223.819286</Scale>
        <Resolution>19.1092570712683</Resolution>
      </LODInfo>
      <LODInfo xsi:type='typens:LODInfo'>
        <LevelID>14</LevelID>
        <Scale>36111.909643</Scale>
        <Resolution>9.55462853563415</Resolution>
      </LODInfo>
      <LODInfo xsi:type='typens:LODInfo'>
        <LevelID>15</LevelID>
        <Scale>18055.954822</Scale>
        <Resolution>4.77731426794937</Resolution>
      </LODInfo>
      <LODInfo xsi:type='typens:LODInfo'>
        <LevelID>16</LevelID>
        <Scale>9027.977411</Scale>
        <Resolution>2.38865713397468</Resolution>
      </LODInfo>
      <LODInfo xsi:type='typens:LODInfo'>
        <LevelID>17</LevelID>
        <Scale>4513.988705</Scale>
        <Resolution>1.19432856685505</Resolution>
      </LODInfo>
      <LODInfo xsi:type='typens:LODInfo'>
        <LevelID>18</LevelID>
        <Scale>2256.994353</Scale>
        <Resolution>0.597164283559817</Resolution>
      </LODInfo>
      <LODInfo xsi:type='typens:LODInfo'>
        <LevelID>19</LevelID>
        <Scale>1128.497176</Scale>
        <Resolution>0.298582141647617</Resolution>
      </LODInfo>
      <LODInfo xsi:type='typens:LODInfo'>
        <LevelID>20</LevelID>
        <Scale>564.248588</Scale>
        <Resolution>0.14929107082380833</Resolution>
      </LODInfo>
      <LODInfo xsi:type='typens:LODInfo'>
        <LevelID>21</LevelID>
        <Scale>282.124294</Scale>
        <Resolution>0.07464553541190416</Resolution>
      </LODInfo>
      <LODInfo xsi:type='typens:LODInfo'>
        <LevelID>22</LevelID>
        <Scale>141.062147</Scale>
        <Resolution>0.03732276770595208</Resolution>
      </LODInfo>
      <LODInfo xsi:type='typens:LODInfo'>
        <LevelID>23</LevelID>
        <Scale>70.5310735</Scale>
        <Resolution>0.01866138385297604</Resolution>
      </LODInfo>
    </LODInfos>
  </TileCacheInfo>
  <TileImageInfo xsi:type='typens:TileImageInfo'>
    <CacheTileFormat>PNG8</CacheTileFormat>
    <CompressionQuality>0</CompressionQuality>
    <Antialiasing>false</Antialiasing>
  </TileImageInfo>
</CacheInfo>
```

