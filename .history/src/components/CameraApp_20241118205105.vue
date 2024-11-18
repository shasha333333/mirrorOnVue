<template>
  <div class="camera-app">
    <div class="controls">
      <button @click="toggleLines">切换标准线</button>
    </div>
    <div class="camera-container">
      <!-- 摄像头视频 -->
      <video ref="video" autoplay></video>
      <!-- Canvas 用于绘制标准线 -->
      <canvas ref="canvas" class="overlay"></canvas>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      showLines: false, // 控制标准线的显示与隐藏
    };
  },
  methods: {
    // 获取摄像头流并开始播放
    async startCamera() {
      try {
        const stream = await navigator.mediaDevices.getUserMedia({ video: true });
        this.$refs.video.srcObject = stream;
      } catch (error) {
        console.error("无法访问摄像头:", error);
      }
    },
    
    // 切换标准线的显示与隐藏
    toggleLines() {
      this.showLines = !this.showLines;
      this.drawLines(); // 在切换时绘制/清除标准线
    },

    // 绘制标准线
    drawLines() {
      const canvas = this.$refs.canvas;
      const context = canvas.getContext("2d");
      const video = this.$refs.video;
      
      // 确保 canvas 大小与 video 一致
      canvas.width = video.videoWidth;
      canvas.height = video.videoHeight;

      // 清空画布
      context.clearRect(0, 0, canvas.width, canvas.height);

      if (this.showLines) {
        // 设置绘制标准线的样式
        context.strokeStyle = "rgba(255, 0, 0, 0.7)";
        context.lineWidth = 2;

        // 画竖直标准线（屏幕中心）
        context.beginPath();
        context.moveTo(canvas.width / 2, 0);
        context.lineTo(canvas.width / 2, canvas.height);
        context.stroke();

        // 画水平标准线（屏幕中心）
        context.beginPath();
        context.moveTo(0, canvas.height / 2);
        context.lineTo(canvas.width, canvas.height / 2);
        context.stroke();

        // 你可以添加更多的标准线，如斜线或其他样式
      }
    }
  },
  mounted() {
    this.startCamera(); // 组件挂载时启动摄像头
  },
  watch: {
    showLines() {
      this.drawLines(); // 在切换时绘制标准线
    }
  }
};
</script>

<style scoped>
.camera-app {
  position: relative;
  width: 100%;
  height: 100vh;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}

.controls {
  margin-bottom: 10px;
}

.camera-container {
  position: relative;
}

video {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.overlay {
  position: absolute;
  top: 0;
  left: 0;
  pointer-events: none; /* 禁止 Canvas 和视频的交互 */
}
</style>
