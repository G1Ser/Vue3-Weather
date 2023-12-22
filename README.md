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
| 图表绘制     | vue-echarts |
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
这样CommonChart.vue只需监听传进来的props参数获取天气预报数据，对图表进行渲染了。<br>
由于接口获取的数据不满足需求，我们需要在pinia中对接口数据进行处理。
```
  const Weather = ref({
    weather: '',
    temperature: '',
    winddirection: '',
    windpower: ''
  })
  const WeatherLists = ref(null)
  const WeatherInfo = ref({
    date: [],
    daytemp: [],
    nighttemp: []
  })
  const getWeather = async (city, key, extensions) => {
    const res = await axios.get(`https://restapi.amap.com/v3/weather/weatherInfo?city=${city}&key=${key}&extensions=${extensions}`)
    if (extensions === 'base') {
      Weather.value = {
        weather: res.data.lives[0].weather,
        temperature: res.data.lives[0].temperature,
        winddirection: res.data.lives[0].winddirection,
        windpower: res.data.lives[0].windpower
      }
    }
    if (extensions === 'all') {
      const forecasts = res.data.forecasts[0].casts
      WeatherLists.value = formatcasts(forecasts)
      getChartData(forecasts)
    }
  }
  function formatcasts(casts) {
    const today = new Date();
    const tomorrow = new Date();
    const currentHour = today.getHours()
    tomorrow.setDate(today.getDate() + 1);
    const weekDays = ["周日", "周一", "周二", "周三", "周四", "周五", "周六"]
    casts.forEach(cast => {
      //格式化week
      const castDate = new Date(cast.date);
      if (castDate.toDateString() === today.toDateString()) {
        cast.week = "今天";
      } else if (castDate.toDateString() === tomorrow.toDateString()) {
        cast.week = "明天";
      } else {
        cast.week = weekDays[castDate.getDay()];
      }
      //格式化date
      const date = cast.date.split('-');
      cast.date = date[1] + '-' + date[2];
      //格式化weather和power
      if (currentHour >= 8 && currentHour < 18) {
        cast.weather = cast.dayweather;
        cast.power = cast.daypower;
      } else {
        cast.weather = cast.nightweather;
        cast.power = cast.nightpower;
      }
    })
    return casts
  }
  function getChartData(casts) {
    WeatherInfo.value.date = [];
    WeatherInfo.value.daytemp = [];
    WeatherInfo.value.nighttemp = [];
    casts.forEach(cast => {
      WeatherInfo.value.date.push(cast.date)
      WeatherInfo.value.daytemp.push(cast.daytemp)
      WeatherInfo.value.nighttemp.push(cast.nighttemp)
    })
  }
```
## 3.项目链接
该项目已经部署线上[weather](https://weather-1322830973.cos-website.ap-beijing.myqcloud.com/)
