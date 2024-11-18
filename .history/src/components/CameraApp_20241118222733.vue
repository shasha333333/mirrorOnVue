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
      <!-- Canvas 用于显示边缘检测效果 -->
      <canvas ref="canvas" class="overlay"></canvas>
    </div>
    <div class="controls">
      <button @click="toggleMirror">镜像</button>
      <button @click="toggleLines">切换标准线</button>
      <button @click="toggleEdgeDetection">边缘检测</button>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      showLines: false, // 控制标准线的显示与隐藏
      isMirrored: false, // 控制是否开启镜像
      edgeDetectionEnabled: false, // 控制边缘检测开关
    };
  },
  methods: {
    // 获取摄像头流并开始播放
    async startCamera() {
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

    // 切换标准线的显示与隐藏
    toggleLines() {
      this.showLines = !this.showLines;
      this.drawLines(); // 在切换时绘制/清除标准线
    },

    // 切换镜像效果
    toggleMirror() {
      this.isMirrored = !this.isMirrored; // 切换镜像状态
    },

    // 切换边缘检测效果
    toggleEdgeDetection() {
      this.edgeDetectionEnabled = !this.edgeDetectionEnabled; // 切换边缘检测开关
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
        if (this.showLines) {
          this.drawLines(); // 绘制标准线
        }
        requestAnimationFrame(drawFrame); // 循环调用下一帧
      };

      drawFrame(); // 开始渲染
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

        const fixX = 20; // 修正参数
        const offsetX = 0; // 水平偏移 100px
        const offsetY = 150; // 垂直偏移 50px

        // 画竖直标准线
        context.beginPath();
        context.moveTo(canvas.width / 2 + offsetX, fixX);
        context.lineTo(canvas.width / 2 + offsetX, canvas.height + fixX);
        context.stroke();

        // 画水平标准线
        context.beginPath();
        context.moveTo(0, canvas.height / 2 + offsetY);
        context.lineTo(canvas.width, canvas.height / 2 + offsetY);
        context.stroke();
      }
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
  font-size: 
