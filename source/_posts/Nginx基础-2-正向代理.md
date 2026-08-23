---
title: 'Nginx基础-2:正向代理'
date: 2020-06-03 20:44:02
tags:
- Nginx
categories: 
- Nginx
---

正向代理，简单来说，就是你想请求A，但是不能直接访问A，所以你托Nginx帮你请求A，再将请求的结果返回给你。一般用来处理跨域、翻墙等。

下面以处理跨域来举例。
<!-- more -->
## 配置

```
server {
    listen       8088;
    server_name  localhost;
    charset utf-8;
    location / {
        root   html;
        index  index.html index.htm;
    }
    location /juhe/ {
        proxy_pass http://v.juhe.cn/;
    }
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   html;
    }
}
```

前端往`http://localhost:8088/juhe/`上发送的请求，都会代理到`http://v.juhe.cn/`上。根据API接口的地址往请求url上追加即可。

如：想请求` http://v.juhe.cn/weather/index`，则访问` http://localhost:8088/juhe/weather/index`。

## proxy_pass说明

在NGINX中配置proxy_pass代理转发时，如果在proxy_pass后面的url加 / ，表示绝对根路径；如果没有 / ，表示相对路径，把匹配的路径部分也给代理。

下面举4种情况来说明，注意看proxy_pass中末尾有无 / 。

1. 访问地址：` http://localhost:8088/juhe/weather/index`

   ```
   location /juhe/ {
       proxy_pass http://v.juhe.cn/;
   }
   ```

   真实地址：`http://v.juhe.cn/`+` weather/index`=` http://v.juhe.cn/weather/index`

2. 访问地址：` http://localhost:8088/weather/index`

   ```
   location /weather/ {
       proxy_pass http://v.juhe.cn;
   }
   ```

   真实地址：`http://v.juhe.cn`+` /weather/index`=` http://v.juhe.cn/weather/index`

3. 访问地址：` http://localhost:8088/juhe/index`

   ```
   location /juhe/ {
       proxy_pass http://v.juhe.cn/weather/;
   }
   ```

   真实地址：` http://v.juhe.cn/weather/`+` index`=` http://v.juhe.cn/weather/index`

4. 访问地址：` http://localhost:8088/juhe/ther/index`

   ```
   location /juhe/ {
       proxy_pass http://v.juhe.cn/wea;
   }
   ```

   真实地址：`http://v.juhe.cn/wea`+` ther/index`=` http://v.juhe.cn/weather/index`

## 补充

参数都在ajax的data中，会自动带到地址的url中。

参数中如果有中文，可能需要编码或解码。

## 请求示例

~~~ html
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>Document</title>
  <script src="https://cdn.staticfile.org/jquery/1.10.2/jquery.min.js"></script>
  <style>
    input {
      width: 400px;
    }
  </style>
</head>

<body onload="bindEvent()">
  url: <input type="text" id="url" value="http://localhost:8088/juhe/weather/index" /><br>
  key: <input type="text" id="key" value="b2cf66d157ca387b7b32455ce282427e" /><br>
  city: <input type="text" id="city" value="北京" /><br>

  <button id="get">get</button><br>
  <button id="post">post</button><br>
  <textarea cols="100" rows="50"></textarea>

  <script>
    function bindEvent() {
      $("button").on("click", async function () {
        const url = $("#url").val();
        const key = $("#key").val();
        const cityname = $("#city").val();
        const type = $(this).attr('id');
        const params = {
          key,
          cityname
        };
        const response = await queryData(url, params, type);
        printResponse(response);
      });
    }
    async function queryData(url, params, type) {
      return $.ajax({
        type,
        data: params,
        url: url,
        dataType: "json"
      });
    }
    function printResponse(response) {
      $("textarea").val(JSON.stringify(response, null, "  "));
    }
  </script>
</body>

</html>
~~~