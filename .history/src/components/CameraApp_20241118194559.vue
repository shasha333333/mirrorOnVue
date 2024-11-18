<template>
  <div id="camera-app">
    <h1>摄像头调用示例</h1>
    <video
      ref="video"
      :class="{ mirrored: isMirrored }"
      autoplay
      width="60%"
      height="auto"
    ></video>
    <div class="controls">
      <button @click="startCamera">打开摄像头</button>
      <button @click="toggleMirror">镜像</button>
    </div>
  </div>
</template>

<script>
export default {
  name: "CameraApp",
  data() {
    return {
      stream: null,
      isMirrored: false, // 控制是否开启镜像
    };
  },
  methods: {
    async startCamera() {
      try {
        this.stream = await navigator.mediaDevices.getUserMedia({ video: true });
        this.$refs.video.srcObject = this.stream;
      } catch (error) {
        console.error("无法访问摄像头:", error);
        alert("请检查摄像头权限或设备状态。");
      }
    },

    toggleMirror() {
      this.isMirrored = !this.isMirrored; // 切换镜像状态
    },
  },
  beforeUnmount() {
    this.stopCamera();
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
.mirrored {
  transform: scaleX(-1); /* 实现镜像效果 */
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
