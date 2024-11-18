<template>
  <div class="camera-app" id="camera-app">
    <div class="camera-container">
      <video
        ref="video"
        :class="{ mirrored: isMirrored }"
        autoplay
        width="100%" 
        height="auto"
      ></video>
      <!-- Canvas 用于绘制标准线 -->
      <canvas ref="canvas" class="overlay"></canvas>
    </div>
    <div class="controls">
      <button @click="toggleMirror">镜像</button>
      <button @click="toggleLines">切换标准线</button>
      
      <!-- 滚动条调整 -->
      <div>
        <label for="offsetX">水平偏移:</label>
        <input
          type="range"
          id="offsetX"
          min="0"
          max="100"
          v-model="offsetXPercent"
          @input="adjustOffsetX"
        />
        <span>{{ offsetXPercent }}%</span>
      </div>
      
      <div>
        <label for="offsetY">垂直偏移:</label>
        <input
          type="range"
          id="offsetY"
          min="0"
          max="100"
          v-model="offsetYPercent"
          @input="adjustOffsetY"
        />
        <span>{{ offsetYPercent }}%</span>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      showLines: false, // 控制标准线的显示与隐藏
      isMirrored: false, // 控制是否开启镜像
      offsetXPercent: 50, // 水平偏移百分比
      offsetYPercent: 50, // 垂直偏移百分比
    };
  },
  methods: {
    // 获取摄像头流并开始播放
    async startCamera() {
      try {
        const stream = await navigator.mediaDevices.getUserMedia({ video: true });
        this.$refs.video.srcObject = stream;
        
        // 等待视频加载后设置 canvas 尺寸
        this.$refs.video.onloadedmetadata = () => {
          this.drawLines();
        };
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
        context.strokeStyle = "#007bff"; // 蓝色，50%透明度
        context.lineWidth = 2;

        // 计算基于视频尺寸的偏移量
        const offsetX = (this.offsetXPercent / 100) * canvas.width; // 水平偏移量
        const offsetY = (this.offsetYPercent / 100) * canvas.height; // 垂直偏移量

        // 画竖直标准线
        context.beginPath();
        context.moveTo(offsetX, 0);
        context.lineTo(offsetX, canvas.height);
        context.stroke();

        // 画水平标准线
        context.beginPath();
        context.moveTo(0, offsetY);
        context.lineTo(canvas.width, offsetY);
        context.stroke();
      }
    },

    // 切换镜像效果
    toggleMirror() {
      this.isMirrored = !this.isMirrored; // 切换镜像状态
    },

    // 调整水平偏移
    adjustOffsetX() {
      this.drawLines();
    },

    // 调整垂直偏移
    adjustOffsetY() {
      this.drawLines();
    },
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
.overlay {
  top: 0;
  left: 0;
  pointer-events: none; /* 禁止 Canvas 和视频的交互 */
}
#camera-app {
  text-align: center;
  font-family: Arial, sans-serif;
  padding: 20px;
}

.camera-container {
  position: relative;
  display: inline-block;
}

video {
  border: 2px solid #ddd;
  border-radius: 8px;
  margin: 20px 0;
  object-fit: cover;
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

input[type="range"] {
  width: 80%;
  margin: 10px 0;
}
</style>
