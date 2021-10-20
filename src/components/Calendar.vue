<template>
    <div class="calendar">
        <div class="cal-title">
            <div class="title-border fl">
                <a id="prev" @click="prev">&lt;</a>
            </div>
            <div class="title-content">
                <div class="green fm" id="cal-ctitle">{{ ctitle }}</div>
                <div class="green fs" id="cal-cyear">{{ cyear }}</div>
            </div>
            <div class="title-border fl">
                <a id="next" @click="next">&gt;</a>
            </div>
        </div>
        <div class="cal-body">
            <div class="lightgrey body-list">
                <ul>
                    <li>MON</li>
                    <li>TUE</li>
                    <li>WED</li>
                    <li>THU</li>
                    <li>FRI</li>
                    <li>SAT</li>
                    <li>SUN</li>
                </ul>
            </div>
            <div class="darkgrey body-list">
                <ul id="days" v-for="cdate in cdates" :key="cdate.key">
                    <li :class="cdate.day_class" @click="change_sel_day(cdate)"> {{ cdate.day }}</li>
                </ul>
            </div>
        </div>
    </div>
</template>
<script>
    import * as R from 'ramda'
    import dateformat from 'dateformat';

    /**
     * 获取当月第一天
     * @param date
     * @returns {Date}
     */
    const first_day = (date) => {
        let d = new Date(date);
        d.setDate(1)
        return d
    }

    /**
     * 获取当月最后一天
     * @param date
     * @returns {Date}
     */
    const last_day = (date) => {
        let d = new Date(date);
        d.setMonth(d.getMonth() + 1)
        d.setDate(0)
        return d
    }

    export default {
        name: 'Calendar',
        props: {
            period_data: {
                type: Array
            }
        },
        data() {
            return {
                month_olympic: [31, 29, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31],
                month_normal: [31, 28, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31],
                month_name: ["1月", "2月", "3月", "4月", "5月", "6月", "7月", "8月", "9月", "10月", "11月", "12月"],
                my_date: new Date(),
                ctitle: '', // 英文月份显示
                cyear: '', // 年份显示
                cdates: [], // 日期显示
                sel_day: new Date().getDate()
            }
        },
        watch: {
            period_data: {
                handler(new_val) {
                    // console.log('watch period_data new_val',new_val)
                    this.period_data = new_val;
                    this.refresh_date() //重置颜色
                    this.refresh_period_date()
                }
            }
        },
        computed: {
            my_year: {
                get: function () {
                    return this.my_date.getFullYear()
                },
                set: function (newVal) {
                    this.my_date = new Date(newVal, this.my_month, this.my_day)
                }
            },
            my_month: {
                get: function () {
                    return this.my_date.getMonth()
                },
                set: function (newVal) {
                    this.my_date = new Date(this.my_year, newVal, this.my_day)
                }
            },
            my_day: {
                get: function () {
                    return this.my_date.getDate()
                }
                // ,
                // set: function (newVal) {
                //     this.my_date = new Date(this.my_year, this.my_month, newVal)
                // }
            }
        },
        mounted() {
            console.log('mounted')
            this.refresh_date()
            if(this.period_data.length > 0){
                this.refresh_period_date()

            }
        },
        methods: {
            /**
             * 获取某年某月第一天是星期几
             * @param month
             * @param year
             * @returns {number}
             */
            dayStart(month, year) {
                let tmpDate = new Date(year, month, 1);
                return (tmpDate.getDay());
            },
            /**
             * 计算某年是不是闰年，通过求年份除以4的余数即可
             * @param month
             * @param year
             * @returns {*}
             */
            daysMonth(month, year) {
                let tmp = year % 4;
                if (tmp == 0) {
                    return (this.month_olympic[month]);
                } else {
                    return (this.month_normal[month]);
                }
            },
            /**
             * 生成月份显示
             */
            refresh_date() {
                this.cdates = []
                this.ctitle = ''
                this.cyear = ''

                let curr_date = new Date()
                let totalDay = this.daysMonth(this.my_month, this.my_year); //获取该月总天数
                let firstDay = this.dayStart(this.my_month, this.my_year); //获取该月第一天是星期几
                let myclass;
                for (let i = 1; i < firstDay; i++) {
                    this.cdates.push({day: '', day_class: '', key: 'blank_' + i})
                }
                for (let i = 1; i <= totalDay; i++) {
                    if ((i < this.my_day && this.my_year == curr_date.getFullYear() && this.my_month == curr_date.getMonth())
                        || this.my_year < curr_date.getFullYear()
                        || (this.my_year == curr_date.getFullYear() && this.my_month < curr_date.getMonth())) {
                        myclass = 'lightgrey' //当该日期在今天之前时，以浅灰色字体显示

                    } else if (i == this.my_day && this.my_year == curr_date.getFullYear() && this.my_month == curr_date.getMonth()) {
                        myclass = 'green greenbox' //当天日期以绿色背景突出显示
                    } else {
                        myclass = 'darkgrey' //当该日期在今天之后时，以深灰字体显示
                    }
                    this.cdates.push({day: i, day_class: myclass, key: 'date_' + i})
                }
                this.ctitle = this.month_name[this.my_month]; //设置英文月份显示
                this.cyear = this.my_year; //设置年份显示
            },
            /**
             * 生成月份显示--经期
             */
            refresh_period_date() {
                let start_dates = R.filter(i => i.is_start)(this.period_data)
                let end_dates = R.filter(i => i.is_end)(this.period_data)

                // console.log('start_dates',start_dates)
                // console.log('end_dates',end_dates)

                if (start_dates.length > end_dates.length) {
                    if (start_dates[0] && end_dates[0] && new Date(start_dates[0].start_date).getDate() > new Date(end_dates[0].end_date).getDate()) {
                        end_dates.unshift({
                            id: start_dates[0].start_date,
                            temperature: null,
                            is_make_love: false,
                            is_record: true,
                            is_start: false,
                            is_end: true,
                            pain_level: null,
                            flow_level: null,
                            color_level: null,
                            state: [],
                            start_date: null,
                            end_date: dateformat(last_day(start_dates[0].start_date),'yyyy-mm-dd'),
                            extra: true // 因为结束日期数量和开始日期数量不相等，而额外补充的数据，加上这个extra字段
                        })
                    } else {
                        end_dates.push({
                            id: start_dates[0].start_date,
                            temperature: null,
                            is_make_love: false,
                            is_record: true,
                            is_start: false,
                            is_end: true,
                            pain_level: null,
                            flow_level: null,
                            color_level: null,
                            state: [],
                            start_date: null,
                            end_date: dateformat(last_day(start_dates[0].start_date),'yyyy-mm-dd'),
                            extra: true // 因为结束日期数量和开始日期数量不相等，而额外补充的数据，加上这个extra字段
                        })
                    }
                }else{
                    start_dates.unshift({
                        id: end_dates[0].end_date,
                        temperature: null,
                        is_make_love: false,
                        is_record: true,
                        is_start: true,
                        is_end: false,
                        pain_level: null,
                        flow_level: null,
                        color_level: null,
                        state: [],
                        start_date: dateformat(first_day(end_dates[0].end_date),'yyyy-mm-dd'),
                        end_date: null,
                        extra: true // 因为开始日期在上个月，而额外补充的数据，加上这个extra字段
                    })
                }

                if (start_dates.length > 0 && end_dates.length > 0) {
                    if (new Date(start_dates[0].start_date).getDate() > new Date(end_dates[0].end_date).getDate()) {
                        start_dates.unshift({
                            id: end_dates[0].end_date,
                            temperature: null,
                            is_make_love: false,
                            is_record: true,
                            is_start: true,
                            is_end: false,
                            pain_level: null,
                            flow_level: null,
                            color_level: null,
                            state: [],
                            start_date: dateformat(first_day(end_dates[0].end_date),'yyyy-mm-dd'),
                            end_date: null,
                            extra: true // 因为开始日期在上个月，而额外补充的数据，加上这个extra字段
                        })
                    }
                }

                if (start_dates.length > end_dates.length) {
                    if (new Date(start_dates[0].start_date).getDate() > new Date(end_dates[0].end_date).getDate()) {
                        end_dates.unshift({
                            id: start_dates[0].start_date,
                            temperature: null,
                            is_make_love: false,
                            is_record: true,
                            is_start: false,
                            is_end: true,
                            pain_level: null,
                            flow_level: null,
                            color_level: null,
                            state: [],
                            start_date: null,
                            end_date: dateformat(last_day(start_dates[0].start_date),'yyyy-mm-dd'),
                            extra: true // 因为结束日期数量和开始日期数量不相等，而额外补充的数据，加上这个extra字段
                        })
                    } else {
                        end_dates.push({
                            id: start_dates[0].start_date,
                            temperature: null,
                            is_make_love: false,
                            is_record: true,
                            is_start: false,
                            is_end: true,
                            pain_level: null,
                            flow_level: null,
                            color_level: null,
                            state: [],
                            start_date: null,
                            end_date: dateformat(last_day(start_dates[0].start_date),'yyyy-mm-dd'),
                            extra: true // 因为结束日期数量和开始日期数量不相等，而额外补充的数据，加上这个extra字段
                        })
                    }

                }

                for (let i = 0; i < start_dates.length; i++) {
                    let curr_start = start_dates[i]
                    let curr_start_date = new Date(curr_start.start_date)
                    let curr_start_day = curr_start_date.getDate()

                    let start_index = R.findIndex(R.propEq('day', curr_start_day))(this.cdates)
                    if (start_index !== -1) {
                        if (curr_start.extra) {
                            this.cdates[start_index].day_class = 'lightred'
                        } else {
                            this.cdates[start_index].day_class = 'darkred'
                        }

                        if (curr_start_day == this.my_day && this.my_year == curr_start_date.getFullYear() && this.my_month == curr_start_date.getMonth()) {
                            this.cdates[start_index].day_class = 'darkred greenbox' //当天日期以绿色背景突出显示
                        }
                    }

                    let curr_end = end_dates[i]
                    let curr_end_date = new Date(curr_end.end_date)
                    let curr_end_day = curr_end_date.getDate()

                    let end_index = R.findIndex(R.propEq('day', curr_end_day))(this.cdates)
                    if (end_index !== -1) {
                        if (curr_end.extra) {
                            this.cdates[end_index].day_class = 'lightred'
                        } else {
                            this.cdates[end_index].day_class = 'darkred'
                        }
                        if (curr_end_day == this.my_day && this.my_year == curr_end_date.getFullYear() && this.my_month == curr_end_date.getMonth()) {
                            this.cdates[end_index].day_class = 'darkred greenbox' //当天日期以绿色背景突出显示
                        }
                    }

                    for (let k = curr_start_day + 1; k < curr_end_day; k++) {
                        let curr_middle_index = R.findIndex(R.propEq('day', k))(this.cdates)
                        if (curr_middle_index !== -1) {
                            this.cdates[curr_middle_index].day_class = 'lightred'
                        }

                        if (k == this.my_day && this.my_year == curr_end_date.getFullYear() && this.my_month == curr_end_date.getMonth()) {
                            this.cdates[curr_middle_index].day_class = 'lightred greenbox' //当天日期以绿色背景突出显示
                        }
                    }
                }
            },
            /**
             * 跳转到上个月
             */
            prev() {
                this.my_month--;
                if (this.my_month < 0) {
                    this.my_year--;
                    this.my_month = 11;
                }
                this.sel_day = new Date().getDate()
                // 向父组件传值
                this.$emit('chang_month_event', {
                    year: this.my_year,
                    month: this.my_month,
                    day: this.my_day,
                    sel_day: this.sel_day
                })
                // 刷新日历
                this.refresh_date()
                this.refresh_period_date()
            },
            /**
             * 跳转到下个月
             */
            next() {
                this.my_month++;
                if (this.my_month > 11) {
                    this.my_year++;
                    this.my_month = 0;
                }
                this.sel_day = new Date().getDate()
                // 向父组件传值
                this.$emit('chang_month_event', {
                    year: this.my_year,
                    month: this.my_month,
                    day: this.my_day,
                    sel_day: this.sel_day
                })
                // 刷新日历
                this.refresh_date()
                this.refresh_period_date()
            },
            change_sel_day(val) {
                this.sel_day = val.day
                // 向父组件传值
                this.$emit('chang_sel_day_event', {
                    year: this.my_year,
                    month: this.my_month,
                    day: this.my_day,
                    sel_day: this.sel_day
                })
                // 刷新日历
                // this.refresh_date()
                // this.refresh_period_date()
                // 更改选中日期的颜色，将前一个取消选中的样式改为未选中的样式
                this.change_day_color(val)
            },
            /**
             * 更改日期显示样式
             * @param val
             */
            change_day_color(val) {
                // let old_cdate = R.find(R.propEq('day_class', 'green greenbox'))(this.cdates)
                let old_cdate = R.find(R.pipe(R.prop('day_class'), R.test(/greenbox$/)))(this.cdates)
                console.log('old_cdate', old_cdate)
                if (!R.isNil(old_cdate)) {
                    if (R.test(/^darkred/, old_cdate.day_class)) {
                        old_cdate.day_class = 'darkred'
                    } else if (R.test(/^lightred/, old_cdate.day_class)) {
                        old_cdate.day_class = 'lightred'
                    } else {
                        if (old_cdate.day < this.my_day) {
                            old_cdate.day_class = 'lightgrey' //当该日期在今天之前时，以浅灰色字体显示
                        } else {
                            old_cdate.day_class = 'darkgrey' //当该日期在今天之后时，以深灰字体显示
                        }
                    }
                }
                if (R.test(/^darkred/, val.day_class)) {
                    val.day_class = 'darkred greenbox'
                } else if (R.test(/^lightred/, val.day_class)) {
                    val.day_class = 'lightred greenbox'
                } else {
                    val.day_class = 'green greenbox'
                }
            }
        }
    }
</script>
<style>
    @media (max-width: 800px) {
        .calendar {
            width: calc(100% - 30px);
        }
    }

    @media (min-width: 800px) {
        .calendar {
            width: calc(50% - 30px);
        }
    }

    .calendar {
        margin-top: 30px;
        background: #fff;
        box-shadow: 0px 1px 1px rgba(0, 0, 0, 0.1);
    }

    .cal-title {
        display: flex;
        flex-direction: row;
        justify-content: space-between;
        align-items: center;
    }

    .title-border {
        width: 10%;
        text-align: center;
    }

    .title-content {
        width: 80%;
    }

    .cal-title a {
        text-decoration: none;
        color: #6ac13c;
    }

    .cal-title a:hover {
        text-decoration: none;
        color: #70E44B;
        cursor: pointer;
    }

    .body-list ul {
        width: 100%;
        font-family: arial;
        font-weight: bold;
        font-size: 14px;
        padding-inline-start: 0;
    }

    .body-list ul li {
        width: 14.28%;
        height: 36px;
        line-height: 36px;
        list-style-type: none;
        display: block;
        box-sizing: border-box;
        float: left;
        text-align: center;
    }

    .lightgrey {
        color: #a8a8a8; /*浅灰色*/
    }

    .darkgrey {
        color: #565656; /*深灰色*/
    }

    .lightred {
        color: #FFCDD2;
    }

    .darkred {
        color: #EF9A9A;
    }

    .green {
        color: #6ac13c; /*绿色*/
    }

    .greenbox {
        border: 1px solid #6ac13c;
        background: #e9f8df; /*浅绿色背景*/
    }

    .fl {
        font-size: 2em;
        font-weight: bold;
    }

    .fm {
        font-size: 1.5em;
        font-weight: bold;
    }

    .fs {
        font-size: 1em;
        font-weight: bold;
    }
</style>
