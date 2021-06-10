<template>
  <div>
    <LostHeart/>
  </div>
</template>

<script>
import LostHeart from './components/LostHeart.vue'

export default {
    name: 'App',
        components: {
            LostHeart
        }
    }
</script>

<style>
    body {
        background-color: black;
        background-image: url('assets/img/back.gif');
        background-repeat: no-repeat;
        background-size: cover;
        overflow: hidden;
    }
</style>
