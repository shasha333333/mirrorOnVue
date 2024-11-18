<template>
  <div id="camera-app">
    <h1>摄像头调用示例</h1>
    <video ref="video" autoplay width="100%" height="auto"></video>
    <div class="controls">
      <button @click="startCamera">打开摄像头</button>
      <button @click="stopCamera">关闭摄像头</button>
    </div>
  </div>
</template>

<script>
export default {
  name: "CameraApp",
  data() {
    return {
      stream: null,
    };
  },
  methods: {
    async startCamera() {
      try {
        // 请求摄像头权限并绑定到视频元素
        this.stream = await navigator.mediaDevices.getUserMedia({ video: true });
        this.$refs.video.srcObject = this.stream;
      } catch (error) {
        console.error("无法访问摄像头:", error);
        alert("请检查摄像头权限或设备状态。");
      }
    },
    stopCamera() {
      if (this.stream) {
        this.stream.getTracks().forEach((track) => track.stop());
        this.stream = null;
      }
    },
  },
  beforeDestroy() {
    this.stopCamera(); // 组件销毁时关闭摄像头
  },
};
</script>

<style>
#camera-app {
  text-align: center;
  font-family: Arial, sans-serif;
  padding: 20px;
}
video {
  border: 2px solid #ddd;
  border-radius: 8px;
  margin: 20px 0;
}
button {
  margin: 10px;
  padding: 10px 20px;
  border: none;
  background-color: #007bff;
  color: white;
  font-size: 16px;
  border-radius: 5px;
  cursor: pointer;
}
button:hover {
  background-color: #0056b3;
}
</style>
