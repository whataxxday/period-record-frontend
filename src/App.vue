<template>
    <div id="app">
        <Main/>
    </div>
</template>

<script>
    import Main from './components/Main.vue'

    export default {
        name: 'App',
        components: {
            Main
        }
    }
</script>

<style>
    html {
        height: 100%;
    }

    body {
        margin:auto;
        height: 100%;
    }

    #app {
        height: 100%;
        font-family: Avenir, Helvetica, Arial, sans-serif;
        -webkit-font-smoothing: antialiased;
        -moz-osx-font-smoothing: grayscale;
        text-align: center;
        color: #2c3e50;
    }
</style>
