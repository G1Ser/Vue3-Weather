# Vue3-Weather
本项目是基于vue3、vite制作的天气系统。通过这个项目，我旨在学习Vite项目部署，了解vue3、vue-router、Echarts技术的应用。
## 1.项目说明
### 1.1.技术栈
|   说明  |    技术栈     |
|:--------:|:----------:|
| Js框架 |    Vue3   |
| 状态管理 |    Pinia   |
| 脚手架   |    Vite    |
| 路由     | vue-router |
| 图标     | vue-echarts |
| 网络请求 |    axios   |
| 本次存储 | Localstorage |
### 1.2.接口说明
1.通过ip地址查询本地城市<br>
接口地址：https://restapi.amap.com/v3/ip?parameters<br>
请求参数：参数1:key 高德地图KEY 必填 /参数2:ip ip地址 可选(若不填 则默认取http本地ip)<br>
请求示例：https://restapi.amap.com/v3/ip?key=<你的key><br>
2.通过城市名查询adcode<br>
接口地址：https://restapi.amap.com/v3/geocode/geo?parameters<br>
请求参数：参数1:key 高德地图KEY 必填 /参数2:address 城市名 必填<br>
请求示例：https://restapi.amap.com/v3/geocode/geo?address=<城市名>&key=<你的key><br>
3.通过adcode查询城市天气<br>
接口地址：https://restapi.amap.com/v3/weather/weatherInfo?parameters<br>
请求参数：参数1:key 高德地图KEY 必填 /参数2:city 城市adcode编码 必填 / 参数3:extensions 可选base：返回实况天气 可选all:返回预报天气<br>
请求示例：https://restapi.amap.com/v3/weather/weatherInfo?city=<城市编码>&key=<你的key>&extensions=<'base' or 'all'>
