<template>
  <CameraApp />
</template>

<script>
import CameraApp from './components/CameraApp.vue';

export default {
  name: "App",
  components: {
    CameraApp,
  },
};
</script>

<style>
/* 全局样式可以根据需要调整 */
body {
  margin: 0;
  font-family: Arial, sans-serif;
  background-color: #f5f5f5;
}
</style>
