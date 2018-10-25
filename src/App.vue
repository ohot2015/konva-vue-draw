<template>
  <div id="app">
    <konva2 />
  </div>
</template>

<script>
import konva2 from './components/konva2.vue'
import Vue from 'vue';
import VueKonva from 'vue-konva'

Vue.use(VueKonva)
export default {
  name: 'app',
  components: {
    konva2
  }
}
</script>

<style>

</style>
