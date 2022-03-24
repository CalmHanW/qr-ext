<template>
  <div class="app">
    <Qr></Qr>
  </div>
</template>

<script>
import Qr from '@/components/Qr/index.vue'
export default {
  name: 'App',
  components: { Qr },
  mounted() {},
}
</script>

<style>
.app {
  width: 300px;
  height: 400px;
  padding: 16px;
}
</style>
