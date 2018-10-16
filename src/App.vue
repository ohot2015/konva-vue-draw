<template>
  <div id="app">
    <konva />
  </div>
</template>

<script>
import konva from './components/konva.vue'
import Vue from 'vue';
import VueKonva from 'vue-konva'

Vue.use(VueKonva)
export default {
  name: 'app',
  components: {
    konva
  }
}
</script>

<style>

</style>
