---
title: ramda初体验-数据转换
date: 2020-09-14 20:53:18
tags:
- Ramda
categories:
- Ramda
---
## 背景

前段时间发现了ramda这个函数式的JavaScript的库。一直没机会用。今天遇到个需求，刚好可以使用上。

要求是把后台接口返回的数据格式转化成与echart对接的数据格式，如图：
<!-- more -->
``` json
{
  "2020-07-01": [
    {
      "北京输气管理处": "95"
    },
    {
      "山西输气管理处": "51"
    }
  ],
  "2020-07-02": [
    {
      "北京输气管理处": "98"
    },
    {
      "山西输气管理处": "54"
    }
  ]
}
->
{
  "北京输气管理处": [
    "95",
    "98"
  ],
  "山西输气管理处": [
    "51",
    "54"
  ]
}
```

## 代码

先放上平时的写法：

```html
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>ES6</title>

  <style>
    textarea {
      width: 100%;
    }
  </style>
</head>

<body onload="init()">
  <button onclick="run()">run</button><br><br>
  <textarea cols="100" rows="50"></textarea>
  <script>
    function init() {
      run();
    }

    function run() {
      workFlow()
    };

    function print(str) {
      document.getElementsByTagName("textarea")[0].value = str;
    };

    function stringify(json) {
      return JSON.stringify(json, null, "  ");
    }

    function workFlow(layerUrl, sublayerId, token) {
      const response = getResponse();
      const resultMap = response.result.ResultMap;
      const result = format(resultMap);
      print(stringify(result));
    }

    function getResponse() {
      return {
        "responseCode": "200",
        "result": {
          "ResultMap": {
            "2020-07-01": [{
              "北京输气管理处": "95"
            }, {
              "山西输气管理处": "51"
            }],
            "2020-07-02": [{
              "北京输气管理处": "98"
            }, {
              "山西输气管理处": "54"
            }],
          },
        }
      };
    }

    function format(resultMap) {
      const time = [];
      let data = [];
      for (let key in resultMap) {
        time.push(key);
        data.push(resultMap[key]);
      }
      // 二维数组转化为一维
      data = data.reduce(
        function (a, b) {
          return a.concat(b);
        },
        []
      );
      data = groupBy(data, "key");
      return {
        time,
        data
      };
    }

    function groupBy(objectArray, property) {
      return objectArray.reduce(function (acc, obj) {
        var key = obj[property];
        if (!acc[key]) {
          acc[key] = [];
        }
        acc[key].push(obj);
        return acc;
      }, {});
    }
  </script>
</body>

</html>
```

使用ramda的代码实现如下：

```html
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>Ramda</title>

  <style>
    textarea {
      width: 100%;
    }
  </style>
  <script src="//cdnjs.cloudflare.com/ajax/libs/ramda/0.27.0/ramda.min.js"></script>
</head>

<body onload="init()">
  <button onclick="run()">run</button><br><br>
  <textarea cols="100" rows="50"></textarea>
  <script>
    function init() {
      run();
    }

    function run() {
      workFlow()
    };

    function print(str) {
      document.getElementsByTagName("textarea")[0].value = str;
    };

    function stringify(json) {
      return JSON.stringify(json, null, "  ");
    }

    function getResponse() {
      return {
        "responseCode": "200",
        "result": {
          "ResultMap": {
            "2020-07-01": [{
              "北京输气管理处": "95"
            }, {
              "山西输气管理处": "51"
            }],
            "2020-07-02": [{
              "北京输气管理处": "98"
            }, {
              "山西输气管理处": "54"
            }],
          },
        }
      };
    }

    const objFormat = o => {
      const [key, value] = Object.entries(o)[0];
      return {
        key,
        value
      };
    }
    const valueFn = (acc, { value }) => acc.concat(value);
    const keyFn = ({ key }) => key;
    function workFlow() {
      const response = getResponse();
      const resultMap = response.result.ResultMap;
      const result = R.compose(
        R.reduceBy(valueFn, [], keyFn),  // 对数组进行groupBy
        R.flatten,                       // 抹平数组
        R.values,                        // 取出value作为数组
        R.map(R.map(objFormat))          // 转化为key value的对象
      )(resultMap);
      print(stringify(result));
    }
  </script>
</body>

</html>
```

## 结束语

ramda在进行数据处理、数据转换上，流程似乎更加明晰，代码更加可读。

而且保持了数据不变。

