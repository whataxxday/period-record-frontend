<template>
    <div class="container">
        <calendar @chang_sel_day_event="get_sel_date" @chang_month_event="get_change_date"
                  :period_data="period_data"></calendar>
        <div class="record">
            <form>
                <div class="form-item">
                    <label>
                        体温：<input type="text" width="50px" v-model="form.temperature"><span style="margin-left: 5px;">&#176;C&nbsp;&nbsp;&nbsp;</span>
                    </label>
                </div>
                <div class="form-item">
                    是否性生活？
                    <checkbox-group :options="is_make_love_options"
                                    @checkbox_change="get_is_make_love"></checkbox-group>
                    &nbsp;&nbsp;
                </div>
                <div class="form-item">
                    是否记录经期？
                    <checkbox-group :options="is_record_options" @checkbox_change="get_is_record"></checkbox-group>
                </div>
                <div v-if="form.is_record">
                    <div class="form-item" v-if="!form.is_start">
                        经期开始了吗？
                        <checkbox-group :options="is_start_options"
                                        @checkbox_change="get_is_start"></checkbox-group>
                    </div>
                    <div class="form-item red" v-else>
                        经期今天开始&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
                    </div>

                    <div class="form-item" v-if="!form.is_end">
                        经期结束了吗？
                        <checkbox-group :options="is_end_options" @checkbox_change="get_is_end"></checkbox-group>
                    </div>
                    <div class="form-item red" v-else>
                        经期今天结束&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
                    </div>

                    <div class="sub-form">
                        <div class="form-item"></div>
                        不适症状：
                        <div class="sub-form-item">
                            痛经：
                            <radio-group :options="pain_level_options" :selected="form.pain_level"
                                         @radio_change="get_pain_level"></radio-group>
                        </div>
                        <div class="sub-form-item">
                            流量：
                            <radio-group :options="flow_level_options" :selected="form.flow_level"
                                         @radio_change="get_flow_level"></radio-group>
                        </div>
                        <div class="sub-form-item">
                            血色：
                            <radio-group :options="color_level_options" :selected="form.color_level"
                                         @radio_change="get_color_level"></radio-group>
                        </div>
                        <div>（血色参考：
                            <span class="color-span light-color">浅色</span>
                            <span class="color-span lighter-color">较浅</span>
                            <span class="color-span normal-color">正常</span>
                            <span class="color-span deeper-color">较深</span>
                            <span class="color-span deep-color">深色</span>）
                        </div>
                        <div class="sub-form-item">
                            状态：
                            <checkbox-group :options="state_options"
                                            @checkbox_change="get_state"></checkbox-group>
                        </div>
                    </div>
                </div>
                <div class="form-item" style="text-align: center;">
                    <button class="button" type="button" @click="submit_form">保存</button>
                    <button class="button del-btn" type="button" style="margin-left: 5px;" @click="del_data">删除</button>
                </div>
                <div class="form-item" style="text-align: center; color: red;">
                    <div> {{ message }}</div>
                </div>
            </form>
        </div>
    </div>
</template>
<script>
    import Calendar from './Calendar.vue'
    import RadioGroup from "./RadioGroup";
    import CheckboxGroup from "./CheckboxGroup";
    import * as R from 'ramda';
    import dateformat from 'dateformat';

    // const BASE_URL = 'http://localhost:8888'
    const BASE_URL = 'http://192.168.192.179:32771'

    export default {
        name: 'Main',
        components: {
            CheckboxGroup,
            RadioGroup,
            Calendar
        },
        data() {
            return {
                period_data: [],
                message: '',
                year: new Date().getFullYear(),
                month: new Date().getMonth(),
                date: new Date().getDate(),
                form: {
                    // id: new Date(new Date().getFullYear(), new Date().getMonth(), new Date().getDate()).getTime(),
                    id: dateformat(new Date(), 'yyyymmdd'),
                    temperature: null,
                    is_make_love: false,
                    is_record: false,
                    is_start: false,
                    is_end: false,
                    pain_level: null,
                    flow_level: null,
                    color_level: null,
                    state: [],
                    start_date: null,
                    end_date: null,
                },
                is_make_love_options: [
                    {name: '', value: 1, checked: false}
                ],
                is_record_options: [
                    {name: '', value: 1, checked: false}
                ],
                is_start_options: [
                    {name: '', value: 1, checked: false}
                ],
                is_end_options: [
                    {name: '', value: 1, checked: false}
                ],
                pain_level_options: [
                    {name: '轻', value: 1},
                    {name: '中', value: 2},
                    {name: '重', value: 3}
                ],
                flow_level_options: [
                    {name: '少', value: 1},
                    {name: '正常', value: 2},
                    {name: '多', value: 3}
                ],
                color_level_options: [
                    {name: '浅色', value: 1},
                    {name: '较浅', value: 2},
                    {name: '正常', value: 3},
                    {name: '较深', value: 4},
                    {name: '深色', value: 5}
                ],
                state_options: [
                    {name: '异味', value: 1, checked: false},
                    {name: '血块', value: 2, checked: false},
                    {name: '渣状', value: 3, checked: false}
                ]
            }
        },
        mounted() {
            this.get_init_data()
            this.get_period_data()
        },
        methods: {
            get_is_make_love(data) {
                this.form.is_make_love = data[0].checked
            },
            get_is_record(data) {
                this.form.is_record = data[0].checked

                if (!this.form.is_record) {
                    this.form.is_start = false
                    this.form.is_end = false
                    this.form.pain_level = null
                    this.form.flow_level = null
                    this.form.color_level = null
                    this.form.state = []
                    this.is_start_options = [
                        {name: '', value: 1, checked: false}
                    ]
                    this.is_end_options = [
                        {name: '', value: 1, checked: false}
                    ]
                    this.state_options = [
                        {name: '异味', value: 1, checked: false},
                        {name: '血块', value: 2, checked: false},
                        {name: '渣状', value: 3, checked: false}
                    ]
                }
            },
            get_is_start(data) {
                this.form.is_start = data[0].checked
                if (this.form.is_start) {
                    this.form.start_date = new Date(this.year, this.month, this.date)
                }
            },
            get_is_end(data) {
                this.form.is_end = data[0].checked
                if (this.form.is_end) {
                    this.form.end_date = new Date(this.year, this.month, this.date)
                }
            },
            get_pain_level(data) {
                this.form.pain_level = data
            },
            get_flow_level(data) {
                this.form.flow_level = data
            },
            get_color_level(data) {
                this.form.color_level = data
            },
            get_state(data) {
                this.form.state = R.pipe(R.filter(i => i.checked), R.pluck('value'))(data)
            },
            get_sel_date(data) {
                // console.log('get_sel_date')
                this.reset_data()
                this.year = data.year
                this.month = data.month
                this.date = data.sel_day
                console.log('data', data)
                this.form.id = dateformat(new Date(data.year, data.month, data.sel_day), 'yyyymmdd')
                this.get_init_data()
            },
            get_change_date(data) {
                // console.log('get_change_date')
                this.reset_data()
                this.year = data.year
                this.month = data.month
                this.date = data.day
                this.form.id = dateformat(new Date(data.year, data.month, data.day), 'yyyymmdd')
                this.get_init_data()
                this.get_period_data()
            },
            reset_data() {
                this.form = {
                    id: dateformat(new Date(this.year, this.month, this.date), 'yyyymmdd'),
                    temperature: null,
                    is_make_love: false,
                    is_record: false,
                    is_start: false,
                    is_end: false,
                    pain_level: null,
                    flow_level: null,
                    color_level: null,
                    state: [],
                    start_date: null,
                    end_date: null
                }
                this.is_make_love_options = [
                    {name: '', value: 1, checked: false}
                ]
                this.is_record_options = [
                    {name: '', value: 1, checked: false}
                ]
                this.is_start_options = [
                    {name: '', value: 1, checked: false}
                ]
                this.is_end_options = [
                    {name: '', value: 1, checked: false}
                ]
                this.state_options = [
                    {name: '异味', value: 1, checked: false},
                    {name: '血块', value: 2, checked: false},
                    {name: '渣状', value: 3, checked: false}
                ]
            },
            submit_form() {
                // console.log('submit_form')
                // console.log('form:', this.form)
                this.message = ''
                const data = this.form
                this.axios.post(BASE_URL + '/save-or-update', data).then(res => {
                    this.reset_data()
                    this.get_init_data()
                    this.get_period_data()
                    if (res && res.data && res.data.code !== 0) {
                        this.message = '保存数据失败'
                    }
                }).catch(e => {
                    console.log(e)
                    this.message = '保存数据失败'
                })
            },
            del_data() {
                this.message = ''
                this.axios.delete(BASE_URL + `/delete/${this.form.id}`).then(res => {
                    this.reset_data()
                    this.get_init_data()
                    this.get_period_data()
                    if (res && res.data && res.data.code !== 0) {
                        this.message = '删除数据失败'
                    }
                }).catch(e => {
                    console.log(e)
                    this.message = '删除数据失败'
                })
            },
            get_init_data() {
                // console.log('get_init_data')
                // console.log('form:', this.form)
                this.message = ''
                this.axios.get(BASE_URL + `/find-one/${this.form.id}`).then(res => {
                    if (res && res.data && res.data.code === 0 && res.data.data !== null) {
                        if (res.data.data.is_make_love !== null) {
                            this.is_make_love_options[0].checked = res.data.data.is_make_love
                        }
                        if (res.data.data.is_record !== null) {
                            this.is_record_options[0].checked = res.data.data.is_record
                        }
                        if (res.data.data.is_start !== null) {
                            this.is_start_options[0].checked = res.data.data.is_start
                        }
                        if (res.data.data.is_end !== null) {
                            this.is_end_options[0].checked = res.data.data.is_end
                        }
                        if (res.data.data.state !== null) {
                            let state = R.map(i => parseInt(i), R.split('|', R.prop('state')(res.data.data)))
                            this.state_options = R.map(i => {
                                if (R.includes(i.value, state)) {
                                    i.checked = true
                                }
                                return i
                            })(this.state_options)
                            this.form = R.set(R.lensProp('state'), state)(res.data.data)
                        }
                        this.form = R.omit(['createdAt', 'updatedAt'])(this.form)
                        // console.log('get_init_data this.form:', this.form)
                    }
                    if (res && res.data && res.data.code !== 0) {
                        this.message = '获取数据失败'
                    }
                }).catch(e => {
                    console.log(e)
                    this.message = '初始化数据失败'
                })
            },
            /**
             * 获取经期数据--前后三个月内数据
             */
            get_period_data() {
                // console.log('get_period_data')
                // console.log('this.date:',this.date)
                this.axios.get(BASE_URL + `/find-all/${new Date(this.year, this.month, this.date)}`).then(res => {
                    if (res && res.data && res.data.code === 0) {
                        this.period_data = res.data.data
                    }
                }).catch(e => {
                    console.log(e)
                    this.message = '获取经期数据失败'
                })
            }
        }
    }
</script>
<style>
    @media (max-width: 800px) {
        .record {
            width: calc(100% - 20px);
        }
    }

    @media (min-width: 800px) {
        .record {
            width: calc(50% - 20px);
        }
    }

    .container {
        height: 100%;
        width: 100%;
        color: #565656;
        display: flex;
        flex-direction: column;
        align-items: center;
    }

    .record {
        padding: 10px;
        height: calc(50% - 20px);
        display: flex;
        flex-direction: column;
        align-items: center;
    }

    .form {
        width: 100%;
        display: flex;
        flex-direction: column;
        align-items: flex-start;
    }

    .form-item {
        margin-top: 10px;
    }

    .sub-form {
        display: flex;
        flex-direction: column;
        align-items: flex-start;
    }

    .sub-form-item {
        margin-top: 5px;
    }

    .form-item input[type=text] {
        width: 50px;
        border: none;
        border: 1px solid #6ac13c;
    }

    .form-item input[type=text]:focus {
        outline-color: #6ac13c;
        border: none;
        border: 1px solid #6ac13c;
    }

    .color-span {
    }

    .light-color {
        color: #ffffff;
        background-color: #FFCDD2;
    }

    .lighter-color {
        color: #ffffff;
        background-color: #EF9A9A;
    }

    .normal-color {
        color: #ffffff;
        background-color: #F44336;
    }

    .deeper-color {
        color: #ffffff;
        background-color: #D32F2F;
    }

    .deep-color {
        color: #ffffff;
        background-color: #B71C1C;
    }

    .button {
        border: none;
        border: 1px solid #6ac13c;
        background-color: #e9f8df;
        color: #6ac13c;
        font-size: 16px;
        cursor: pointer;
    }

    .del-btn {
        background-color: #EF9A9A;
        color: #ffffff;
        border-color: #FFCDD2;
    }

    .green {
        color: #6ac13c;
    }

    .red {
        color: #EF9A9A;
    }
</style>
