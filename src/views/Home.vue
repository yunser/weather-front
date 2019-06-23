<template>
    <my-page :title="title" :page="page">
        <div class="common-container container">
            <ui-text-field v-model="location" hintText="输入你要查找的城市" label="城市名" />
            <div class="btns">
                <ui-raised-button label="查询" primary @click="search" />
            </div>
            <div class="weather-box" v-if="now">
                <div class="weather-card">
                    <img class="icon" :src="now.icon">
                    <!-- <div class="icon">{{  }}</div> -->
                    <div class="right">
                        <div class="tem">{{ now.tem }}°</div>
                        <div class="weather">{{ now.weather }}，相对湿度：{{ now.humidity }}，降水量：{{ now.precipitation }}，风力：{{ now.windPower }}</div>
                    </div>
                    <!-- <ui-icon-button class="more" icon="more_horiz" /> -->
                </div>
                <!--<div class="today-date">{{ data.date }}</div>-->
                <!-- <div class="info" :style="{color: color}">空气质量：{{ getPm25() }}</div> -->
                <ul class="weather-list">
                    <li class="item" v-for="item in forecasts">
                        <div class="date">{{ item.date }}</div>
                        <div>{{ item.dayTemp }} ~ {{ item.nightTemp }} ℃</div>
                        <div>{{ item.weather }}</div>
                        <div>风力：{{ item.dayWindPower }}</div>
                        <div>降水率：{{ item.pp }}%</div>
                        <div class="img">
                            <div v-if="item.dayWeather === item.nightWeather">{{ item.dayWeather }}</div>
                            <div v-if="item.dayWeather !== item.nightWeather">{{ item.dayWeather }} 转 {{ item.nightWeather }}</div>
                            <!-- <img :src="item.dayPictureUrl" title="白天"/>
                            <img :src="item.nightPictureUrl" title="晚上"/> -->
                        </div>
                    </li>
                </ul>
                <ui-article>
                    <!-- <h2>温馨建议</h2> -->
                    <ul>
                        <!-- <li v-for="item in data.results[0].index">
                            {{ item.title }}：{{ item.des }}
                        </li> -->
                    </ul>
                </ui-article>
            </div>

            <div class="weather-box" v-if="data2">
                <!--<div class="today-date">{{ data.date }}</div>-->
                <!-- <div class="info" :style="{color: color}">空气质量：{{ getPm25() }}</div> -->
                <ul class="weather-list">
                    <li class="item" v-for="item in data.forecasts">
                        <div class="date">{{ item.date }}</div>
                        <div>{{ item.dayTemp }} ~ {{ item.nightTemp }} ℃</div>
                        <div>{{ item.weather }}</div>
                        <div>风力 {{ item.dayWindPower }}</div>
                        <div class="img">
                            <div v-if="item.dayWeather === item.nightWeather">{{ item.dayWeather }}</div>
                            <div v-if="item.dayWeather !== item.nightWeather">{{ item.dayWeather }} 转 {{ item.nightWeather }}</div>
                            <!-- <img :src="item.dayPictureUrl" title="白天"/>
                            <img :src="item.nightPictureUrl" title="晚上"/> -->
                        </div>
                    </li>
                </ul>
                <ui-article>
                    <!-- <h2>温馨建议</h2> -->
                    <ul>
                        <!-- <li v-for="item in data.results[0].index">
                            {{ item.title }}：{{ item.des }}
                        </li> -->
                    </ul>
                </ui-article>
            </div>

            <div class="detail-box" v-if="false">

            </div>
        </div>
    </my-page>
</template>

<script>
    /* eslint-disable */
    export default {
        data () {
            return {
                title: '天气',
                location: '',
                now: null,
                // now: {
                //     icon: '/static/weather/101.png',
                //     tem: 34,
                //     weather: '多云'
                // },
                forecasts: [
                    {
                        date: '10-20',
                        dayTemp: 10,
                        nightTemp: 20,
                        weather: '多云',
                        dayWindPower: '4',

                    }
                ],
                data: null,
                color: '#333',
                page: {
                    menu: [
                        {
                            type: 'icon',
                            icon: 'apps',
                            href: 'https://app.yunser.com?utm_source=weather',
                            target: '_blank',
                            title: '应用'
                        }
                    ]
                }
            }
        },
        mounted() {
            this.init()
        },
        methods: {
            init() {
                this.getMyIp()
                // this.query()
            },
            getMyIp() {
                let url = `https://phpapi.yunser.com/ip.php`
                this.$http.get(url).then(
                    response => {
                        let data = response.data
                        console.log('IP信息', data)
                        this.getWeather(data)
                    },
                    response => {
                        console.log(response)
                    })
            },
            getWeather(ip) {
                let location = ip
                this.$http.get(`/weather/now?location=${location}`).then(
                    response => {
                        let data = response.data
                        console.log('IP信息', data)
                        let now = data.HeWeather6[0].now
                        this.now = {
                            // icon: `https://a.hecdn.net/img/plugin/190516/icon/c/${now.cond_code}d.png`,
                            icon: `/static/weather/${now.cond_code}.png`,
                            tem: now.tmp,
                            weather: now.cond_txt,
                            humidity: now.hum,
                            precipitation: now.pcpn,
                            windPower: now.wind_sc,
                        }
                        this.location = data.HeWeather6[0].basic.location
                        this.get3Days(location)
                    },
                    response => {
                        console.log(response)
                    })
            },
            get3Days(location) {
                this.$http.get(`/weather/forecast?location=${location}`).then(
                    response => {
                        let data = response.data
                        console.log('IP信息2', data)
                        let forecast = data.HeWeather6[0].daily_forecast
                        this.forecasts = forecast.slice(0, 3).map((item, index) => {
                            return {
                                date: ['今天', '明天', '后天'][index],
                                dayTemp: item.tmp_max,
                                nightTemp: item.tmp_min,
                                weather: `${item.cond_txt_d}/${item.cond_txt_n}`,
                                dayWindPower: item.wind_sc,
                                pp: item.pop,
                            }
                        })
                    },
                    response => {
                        console.log(response)
                    })
            },
            search() {
                if (!this.location) {
                    this.$message({
                        type: 'danger',
                        text: '请输入城市名'
                    })
                    return
                }
                this.getWeather(this.location)
            },
            query() {
//                const cacheTime = 30 * 60 // 缓存半小时数据
//                let lastTime = this.$storage.get('cacheTime')
//                let time = new Date().getTime()
                let date = new Date().getHours()
                if (date === this.$storage.get(this.location + 'date') && false) {
                    console.log('从缓存获取')
                    this.data = this.$storage.get(this.location + 'data')
                    if (!this.data) {
                        this.$storage.set(this.location + 'date', null)
                        this.query()
                    }
                    console.log(this.data)
                } else {
                    this.$http.get('/weather?location=' + this.location).then(
                        response => {
                            let data = response.data
                            console.log(data)
                            this.data = data
                            this.$storage.set(this.location + 'date', new Date().getHours())
                            this.$storage.set(this.location + 'data', data)
                        },
                        response => {
                            console.log(response)
                        })
                }
                this.data = this.parser(this.data)
                this.title = this.data.results[0].currentCity + '天气'
            },
            getPm25() {
//                let grades= [50, 100, 150, 200, 300, 500]
                let colors = ['#6acd06', '#fbd12a', '#fe8800', '#ff0000', '#cc0000', '#960453', '#62001e']
                let grade = 0
                let pm25 = this.data.results[0].pm25
                if (pm25 <= 50) {
                    grade = 0
                    return '优'
                } else if (pm25 <= 100) {
                    grade = 1
                    return '良'
                } else if (pm25 <= 150) {
                    grade = 2
                    return '轻度污染'
                } else if (pm25 <= 200) {
                    grade = 3
                    return '中度污染'
                } else if (pm25 <= 300) {
                    grade = 4
                    return '重度污染'
                } else if (pm25 <= 500) {
                    grade = 5
                    return '严重污染'
                }
                this.color = colors[grade]
                console.log(this.color)
                return this.data.results[0].pm25
            },
            parser(data) {
                data.results[0].weather_data[0].date = '今天'
                return data
            }
        }
    }
</script>

<style lang="scss" scoped>
    @import "../scss/var";
    .container {
        max-width: 480px;
    }
    .btns {
        margin-bottom: 24px;
    }
    .today-date {
        margin-bottom: 16px;
    }
    .info {
        margin-bottom: 16px;
    }
    .weather-card {
        position: relative;
        display: flex;
        padding: 40px;
        margin-bottom: 24px;
        box-shadow: 0 1px 6px rgba(0,0,0,.117647), 0 1px 4px rgba(0,0,0,.117647);
        .icon {
            width: 56px;
            height: 56px;
            margin-right: 16px;
        }
        .tem {
            font-size: 40px;
        }
        .more {
            position: absolute;
            bottom: 16px;
            right: 16px;
        }
    }
    .weather-box {
    }
    .weather-list {
        @include clearfix;
        .item {
            float: left;
            padding: 16px;
            margin-right: 16px;
            margin-bottom: 16px;
            background-color: #fff;
            box-shadow: 0 1px 6px rgba(0,0,0,.117647), 0 1px 4px rgba(0,0,0,.117647);
        }
        .date {
            margin-bottom: 16px;
            font-size: 24px;
        }
        .img {
            margin-top: 16px;
            img {
                margin-right: 8px;
            }
        }
    }
    @media all and (max-width: 750px){
        .weather-list {
            @include clearfix;
            .item {
                float: none;
            }
        }
    }
    .detail-box {
        position: fixed;
        top: 0;
        left: 0;
        right: 0;
        bottom: 0;
        z-index: 10000;
        background-color: #fff;
    }
</style>
