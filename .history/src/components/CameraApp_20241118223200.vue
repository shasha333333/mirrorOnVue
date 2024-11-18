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
      <!-- Canvas 用于绘制标准线和边缘检测 -->
      <canvas ref="canvas" class="overlay"></canvas>
    </div>
    <div class="controls">
      <button @click="toggleMirror">镜像</button>
      <button @click="toggleLines">切换标准线</button>
      <button @click="toggleEdgeDetection">切换边缘检测</button> <!-- 新增边缘检测按钮 -->
      
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
      edgeDetectionEnabled: false, // 是否启用边缘检测
    };
  },
  methods: {
    // 获取摄像头流并开始播放
    startCamera() {
      try {
        const stream = await navigator.mediaDevices.getUserMedia({ video: true });
        this.$refs.video.srcObject = stream;

        // 等待视频加载后设置 canvas 尺寸并开始实时处理
        this.$refs.video.onloadedmetadata = () => {
          this.drawVideo();
        };
      } catch (error) {
        console.error("无法访问摄像头:", error);
      }
    },

    // 每一帧处理视频流并应用边缘检测
    drawVideo() {
      const video = this.$refs.video;
      const canvas = this.$refs.canvas;
      const context = canvas.getContext("2d");

      // 确保 canvas 大小与 video 一致
      canvas.width = video.videoWidth;
      canvas.height = video.videoHeight;

      // 每一帧进行边缘检测
      const drawFrame = () => {
        context.drawImage(video, 0, 0, canvas.width, canvas.height); // 绘制视频帧到 canvas
        if (this.edgeDetectionEnabled) {
          this.applyEdgeDetection(video, context, canvas); // 应用边缘检测
        }
        requestAnimationFrame(drawFrame); // 循环调用下一帧
      };

      drawFrame(); // 开始渲染
    },
    
    // 切换标准线的显示与隐藏
    toggleLines() {
      this.showLines = !this.showLines;
      this.drawLines(); // 在切换时绘制/清除标准线
    },

    // 切换边缘检测的启用状态
    toggleEdgeDetection() {
      this.edgeDetectionEnabled = !this.edgeDetectionEnabled;
      this.drawLines(); // 在切换时重新绘制图像（包括边缘检测）
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

      // 获取视频帧
      context.drawImage(video, 0, 0, canvas.width, canvas.height);



      if (this.edgeDetectionEnabled) {
        this.applyEdgeDetection(context, canvas);
      }
    },

    // 绘制标准线
    drawStandardLines(context, canvas) {
      context.strokeStyle = "#007bff"; // 蓝色，50%透明度
      context.lineWidth = 2;

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
    },

    // 应用边缘检测
    applyEdgeDetection(video, context, canvas) {
      const width = video.videoWidth;
      const height = video.videoHeight;
      canvas.width = width;
      canvas.height = height;

      const imageData = context.getImageData(0, 0, width, height);
      const data = imageData.data;

      const sobelX = [
        [-1, 0, 1],
        [-2, 0, 2],
        [-1, 0, 1]
      ];
      const sobelY = [
        [-1, -2, -1],
        [0, 0, 0],
        [1, 2, 1]
      ];

      const resultData = new Uint8ClampedArray(data.length);

      for (let y = 1; y < height - 1; y++) {
        for (let x = 1; x < width - 1; x++) {
          let pixelX = 0;
          let pixelY = 0;

          // 执行 Sobel 核计算
          for (let ky = -1; ky <= 1; ky++) {
            for (let kx = -1; kx <= 1; kx++) {
              const pixel = (y + ky) * width + (x + kx);
              const weightX = sobelX[ky + 1][kx + 1];
              const weightY = sobelY[ky + 1][kx + 1];

              pixelX += weightX * data[pixel * 4];
              pixelY += weightY * data[pixel * 4];
            }
          }

          // 计算边缘强度
          const magnitude = Math.sqrt(pixelX * pixelX + pixelY * pixelY);
          const color = Math.min(255, magnitude);

          const pixelIdx = (y * width + x) * 4;
          resultData[pixelIdx] = color;
          resultData[pixelIdx + 1] = color;
          resultData[pixelIdx + 2] = color;
          resultData[pixelIdx + 3] = 255; // 保持 alpha 值为 255
        }
      }

      // 将结果数据写回画布
      const newImageData = new ImageData(resultData, width, height);
      context.putImageData(newImageData, 0, 0);
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
    },
    edgeDetectionEnabled() {
      this.drawLines(); // 在切换时应用边缘检测
    }
  }
};
</script>

<style scoped>
.overlay {
  position: absolute;
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
  width: 80%;         /* 可根据需要调整宽度 */
  max-width: 300px;   /* 限制最大宽度为 300px */
  margin: 10px 0;
}
</style>
