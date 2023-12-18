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
### 1.3.项目内容
1.Home界面：实现本地天气的查看，城市数据的检索，对本地数据的查看、管理。<br>
2.Search界面：天气系统的查看。<br>
![Weather项目](https://github.com/G1Ser/Vue3-Weather/blob/main/Image/%E9%A1%B9%E7%9B%AE%E5%B1%95%E7%A4%BA.gif "Weather项目")
## 2.项目部署
Weather项目可分为两个部位，一个公共的TopHeader.vue，一个主界面，主界面主要是路由界面实现的，一个静态路由Home Page、一个动态路由Search Page.
```
//路由设置
import { createRouter, createWebHistory } from 'vue-router'
import Home from '@/views/Home.vue'
const router = createRouter({
  history: createWebHistory(import.meta.env.BASE_URL),
  routes: [
    {
      path: '/',
      name: 'home',
      component: Home
    },
    {
      path: '/weather/:adecode',
      name: 'weather',
      component: () => import('@/views/Search.vue')
    }
  ]
})

export default router
```
```
//每当点击查看和搜索的时候，将adcode和city传入参数中
route.push(`/weather/${adcode}?search=${city}`)
//Search.vue进行接收
watch([() => route.params.adecode, () => route.query.search],([newAdcode, newCity]) => {
    adcode.value = newAdcode;
    city.value = newCity;
}, { immediate: true });
```
其中Home.vue和Search.vue都存在一个天气预报图表，本次项目基于Vue-Echarts进行绘制，根据第三个接口获取出预报天气，通过设置图表的option来进行绘制，本次将图表区域设置成公共部件CommonChart.vue,将接口数据使用Pinia进行保存。
```
<template>
    <div class="weatherInfo">
        <div class="forecasts">
            <div class="weather" v-for="list in WeatherStore.WeatherLists" :key="list.date">
                <span>{{ list.week }}</span>
                <span>{{ list.date }}</span>
                <span>{{ list.weather }}</span>
                <span>风力 {{ list.power }}级</span>
            </div>
        </div>
        <div class="chart">
            <v-chart :option="option"></v-chart>
        </div>
    </div>
</template>
<script setup>
const props = defineProps({
    city: String,
    adcode: String
});
function renderChart(data) {
    option.value = {
        xAxis: {
            type: 'category',
            data: data.date,
            axisTick: {
                alignWithLabel: true
            }
        },
        yAxis: {
            type: 'value',
            interval: 0
        },
        grid: {
            left: 0,
            top: 0,
            right: 0,
            bottom: 0
        },
        series: [
            {
                name: '白天温度',
                type: 'line',
                data: data.daytemp,
                smooth: true,
                label: {
                    show: true,
                    position: 'bottom', // 标签位置
                    formatter: function (params) {
                        return `白${params.value}℃`;
                    },
                    backgroundColor: 'transparent',
                    color: '#FFF',
                    offset: [-25, 2]
                }
            },
            {
                name: '夜间温度',
                type: 'line',
                data: data.nighttemp,
                smooth: true,
                label: {
                    show: true,
                    position: 'top', // 标签位置
                    formatter: function (params) {
                        return `晚${params.value}℃`;
                    },
                    backgroundColor: 'transparent',
                    color: '#FFF',
                    offset: [25, 2]
                }
            }
        ]
    }
}
</script>
```
这样CommonChart.vue只需监听传进来的props参数获取天气预报数据，对图表进行渲染了。
## 3.项目链接
