<template>
    <span>
        <span v-for="(option, key) in options" :key="key" class="ui-checkbox-group">
            <label class="ui-checkbox" :class="{ 'checked' : option.checked}">
                <input type="checkbox" v-model="checkbox_model" :value="option.value" @change="checkbox_change(option)"/>
            </label>
            {{ option.name }}
        </span>
    </span>
</template>

<script>
    import * as R from 'ramda'
    export default {
        name: 'CheckboxGroup',
        data() {
            return {
                checkbox_model: ''
            }
        },
        props: {
            options: {
                type: [Array, Object]
            }
        },
        destroyed: function () {
        },
        methods: {
            checkbox_change: function (val) {
                let index = R.indexOf(val,this.options)
                val.checked = R.not(val.checked)
                this.options[index] = val
                // console.log('this.options:',this.options)
                this.$emit('checkbox_change', this.options);
            }
        }
    }
</script>

<style>
    .ui-checkbox-group {
        padding-right: 5px;
        display: inline-flex;
        line-height: 16px;
    }

    .ui-checkbox {
        width: 14px;
        height: 14px;
        border: 1px solid #6ac13c;
        display: inline-flex;
        position: relative;
    }

    .ui-checkbox.disabled {
        border-color: #a8a8a8;
    }

    .ui-checkbox::after {
        content: '';
        width: 10px;
        height: 10px;
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

    .ui-checkbox.disabled::after {
        background-color: #a8a8a8;
    }

    .ui-checkbox.checked::after {
        opacity: 1;
        transform: scale(1);
    }

    .ui-checkbox input[type=checkbox] {
        opacity: 0;
        margin: 0;
    }
</style>
