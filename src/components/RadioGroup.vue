<template>
    <span>
        <span v-for="(option, key) in options" :key="key" class="ui-radio-group">
            <label class="ui-radio" :class="{ 'checked' : selected_radio === option.value}">
                <input type="radio" v-model="radio_model" :value="option.value" @change="radio_change(option.value)"/>
            </label>
            {{ option.name }}
        </span>
    </span>
</template>

<script>
    export default {
        name: 'RadioGroup',
        data() {
            return {
                radio_model: '',
                selected_radio: ''
            }
        },
        props: {
            options: {
                type: [Array, Object]
            },
            selected: [],
        },
        mounted: function () {
            this.selected_radio = this.selected;
        },
        destroyed: function () {
        },
        methods: {
            radio_change: function (val) {
                // console.log('radio val:',val)
                this.selected_radio = val
                this.$emit('radio_change', this.selected_radio);
            }
        }
    }
</script>

<style>
    .ui-radio-group {
        padding-right: 5px;
        display: inline-flex;
        line-height: 16px;
    }

    .ui-radio {
        width: 14px;
        height: 14px;
        border-radius: 50%;
        border: 1px solid #6ac13c;
        display: inline-flex;
        position: relative;
    }

    .ui-radio.disabled {
        border-color: #a8a8a8;
    }

    .ui-radio::after {
        content: '';
        width: 10px;
        height: 10px;
        border-radius: 50%;
        display: inline-flex;
        position: absolute;
        left: 50%;
        top: 50%;
        margin: -5px 0 0 -5px;
        background-color: #6ac13c;
        transition: all .2s ease;
        opacity: 0;
        transform: scale(0);
    }

    .ui-radio.disabled::after {
        background-color: #a8a8a8;
    }

    .ui-radio.checked::after {
        opacity: 1;
        transform: scale(1);
    }

    .ui-radio input[type=radio] {
        opacity: 0;
        margin: 0;
    }
</style>
