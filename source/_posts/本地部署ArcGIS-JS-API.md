---
title: 本地部署ArcGIS-JS-API
date: 2020-06-16 20:05:43
tags:
- ArcGIS
- ArcGIS JS API
- CORS
categories:
- ArcGIS JS API
---

网上有很多的本地部署ArcGIS API for JavaScript教程。

我翻译并简单修改了官网的部署教程（API解压后的install.html），整理如下。
<!-- more -->
## 准备工作

官网下载压缩包，然后解压后放置到要部署的web服务器上，可能是IIS、Tomcat或Nginx。

## 修改代码

1. 打开文件`/var/www/html/arcgis_js_api/library/arcgis_js_api/library/4.15/dojo/dojo.js` 并检索 `[HOSTNAME_AND_PATH_TO_JSAPI]`, 然后用`www.example.com/arcgis_js_api/library/4.15/`替换.
2. 打开文件`/var/www/html/arcgis_js_api/library/arcgis_js_api/library/4.15/init.js` 并检索  `[HOSTNAME_AND_PATH_TO_JSAPI]`, 然后用`www.example.com/arcgis_js_api/library/4.15/`替换.

## 测试

现在应该能够从你的web服务器使用以下URL访问ArcGIS API for JavaScript：

```
<script src="https://www.example.com/arcgis_js_api/library/4.15/dojo/dojo.js"></script>
```

为了测试是否部署成功，可以使用下面的测试页面。

```html
<!DOCTYPE html>
<html>
  <head>
    <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
    <meta name="viewport" content="initial-scale=1, maximum-scale=1,user-scalable=no" />
    <title>Test Map</title>
    <link rel="stylesheet" href="https://www.example.com/arcgis_js_api/library/4.15/dijit/themes/claro/claro.css" />
    <link rel="stylesheet" href="https://www.example.com/arcgis_js_api/library/4.15/esri/themes/light/main.css" />
    <style>
      html,
      body,
      #viewDiv {
        margin: 0;
        padding: 0;
        width: 100%;
        height: 100%;
      }
    </style>
    <script src="https://www.example.com/arcgis_js_api/library/4.15/dojo/dojo.js"></script>
    <script>
      var myMap, view;
      require([
        "esri/Basemap",
        "esri/layers/TileLayer",
        "esri/Map",
        "esri/views/MapView"
      ], function (Basemap, TileLayer, Map, MapView){
        // --------------------------------------------------------------------
        // If you do not have public internet access then use the Basemap class
        // and point this URL to your own locally accessible cached service.
        //
        // Otherwise you can just use one of the named hosted ArcGIS services.
        // https://services.arcgisonline.com/ArcGIS/rest/services/World_Street_Map/MapServer
        // --------------------------------------------------------------------
        var layer = new TileLayer({
          url: "https://www.example.com/arcgis/rest/services/Folder/Custom_Base_Map/MapServer"
        });
        var customBasemap = new Basemap({
          baseLayers: [layer],
          title: "Custom Basemap",
          id: "myBasemap"
        });
        myMap = new Map({
          basemap: customBasemap
        });
        view = new MapView({
          center: [-111.87, 40.57], // long, lat
          container: "viewDiv",
          map: myMap,
          zoom: 6
        });
      });
    </script>
  </head>
  <body class="claro">
    <div id="viewDiv"></div>
  </body>
</html>
```

> *注意*：1. 修改页面中的链接；2. 使用http或https协议，请勿使用file协议访问页面。

## 使用HTTP

如果你希望使用`http`来使用API，那么在修改代码步骤中，用`http`代替`https`，如，

修改前: `baseUrl:"https://www.example.com/arcgis_js_api/library/4.15/dojo"`

修改后: `baseUrl:"http://www.example.com/arcgis_js_api/library/4.15/dojo"`

## 字体跨域

在开发模式下，可能会出现字体跨域，导致微件上的图标无法显示。下面列举在不同服务器上通过允许跨域来处理。

### IIS

1. 打开IIS管理器

2. 选中部署API的网站，找到***HTTP响应标头***和***MIME类型***

3. 配置MIME类型
   有下列的类型需要注册：

   | extension | `MIME/type`                | Description                                                  |
   | --------- | -------------------------- | ------------------------------------------------------------ |
   | `.ttf`    | `application/octet-stream` | True Type Fonts                                              |
   | `.wasm`   | `application/wasm`         | [WebAssembly](http://webassembly.org/)                       |
   | `.woff`   | `application/font-woff`    | [Web Open Font Format](https://developer.mozilla.org/en-US/docs/Web/Guide/WOFF) |
   | `.woff2`  | `application/font-woff2`   | [WOFF File Format 2.0](https://www.w3.org/TR/WOFF2/)         |
   | `.wsv`    | `application/octet-stream` | Supports `SceneView`'s stars visualization                   |

   点击***MIME类型***，右键选择**打开功能**，然后右键选择**添加**。

4. 配置HTTP响应标头
   点击***HTTP响应标头***，右键选择**打开功能**，然后右键选择**添加**。
   添加名称：`Access-Control-Allow-Origin` ，值：`*`。

### Tomcat

修改Tomcat配置文件，安装目录下的`conf/web.xml`。

按照[官网说明](http://tomcat.apache.org/tomcat-8.0-doc/config/filter.html#CORS_Filter)在`<!-- ================== Built In Filter Definitions ===================== -->`之后增加一段配置。

```xml
<filter>
    <filter-name>CorsFilter</filter-name>
    <filter-class>org.apache.catalina.filters.CorsFilter</filter-class>
    <init-param>
        <param-name>cors.allowed.origins</param-name>
        <param-value>*</param-value>
    </init-param>
    <init-param>
        <param-name>cors.allowed.methods</param-name>
        <param-value>GET,POST,HEAD,OPTIONS,PUT</param-value>
    </init-param>
    <init-param>
        <param-name>cors.allowed.headers</param-name>
        <param-value>Content-Type,X-Requested-With,accept,Origin,Access-Control-Request-Method,Access-Control-Request-Headers</param-value>
    </init-param>
    <init-param>
        <param-name>cors.exposed.headers</param-name>
        <param-value>Access-Control-Allow-Origin,Access-Control-Allow-Credentials</param-value>
    </init-param>
    <init-param>
        <param-name>cors.support.credentials</param-name>
        <param-value>false</param-value>
    </init-param>
    <init-param>
        <param-name>cors.preflight.maxage</param-name>
        <param-value>10</param-value>
    </init-param>
</filter>
<filter-mapping>
    <filter-name>CorsFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

> 注意：当cors.allowed.origins设置为*时，cors.support.credentials不能设置为true。

### Nginx

修改Nginx配置文件，

增加一行`add_header Access-Control-Allow-Origin *;`如，

```conf
location /arcgis/api {
	add_header Access-Control-Allow-Origin *;
	alias C:/Resource/arcgis/api;
	index dojo/dojo.js;
}
```

