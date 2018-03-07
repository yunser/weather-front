<template>
    <my-page :title="title">
        <ui-text-field v-model="location" hintText="输入你要查找的城市" />
        <div class="btns">
            <ui-raised-button label="查询" primary @click="query" />
        </div>
        <div class="weather-box" v-if="data">
            <!--<div class="today-date">{{ data.date }}</div>-->
            <div class="info" :style="{color: color}">空气质量：{{ getPm25() }}</div>
            <ul class="weather-list">
                <li class="item" v-for="item in data.results[0].weather_data">
                    <div class="date">{{ item.date }}</div>
                    <div>{{ item.temperature }}</div>
                    <div>{{ item.weather }}</div>
                    <div>{{ item.wind }}</div>
                    <div class="img">
                        <img :src="item.dayPictureUrl" title="白天"/>
                        <img :src="item.nightPictureUrl" title="晚上"/>
                    </div>
                </li>
            </ul>
            <ui-article>
                <h2>温馨建议</h2>
                <ul>
                    <li v-for="item in data.results[0].index">
                        {{ item.title }}：{{ item.des }}
                    </li>
                </ul>
            </ui-article>
        </div>
    </my-page>
</template>

<script>
    export default {
        data () {
            return {
                title: '天气',
                location: '广州',
                data: null,
                color: '#333',
                page: {
                    menu: [
                        {
                            type: 'icon',
                            icon: 'help',
                            to: '/help'
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
                this.query()
            },
            query() {
//                const cacheTime = 30 * 60 // 缓存半小时数据
//                let lastTime = this.$storage.get('cacheTime')
//                let time = new Date().getTime()
                let date = new Date().getHours()
                if (date === this.$storage.get(this.location + 'date')) {
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
    .btns {
        margin-bottom: 24px;
    }
    .today-date {
        margin-bottom: 16px;
    }
    .info {
        margin-bottom: 16px;
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
</style>
