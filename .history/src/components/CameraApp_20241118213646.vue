<template>
  <div class="camera-app">
    <div class="camera-container">
      <video
        ref="video"
        :class="{ mirrored: isMirrored }"
        autoplay
        width="60%"
        height="auto"
      ></video>
      <!-- Canvas 用于绘制边缘检测结果 -->
      <canvas ref="canvas" class="overlay"></canvas>
    </div>
    <div class="controls">
      <button @click="toggleMirror">镜像</button>
      <button @click="toggleEdgeDetection">切换边缘检测</button>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      isMirrored: false, // 控制是否开启镜像
      edgeDetectionEnabled: false, // 控制边缘检测是否启用
    };
  },
  methods: {
    // 获取摄像头流并开始播放
    async startCamera() {
      try {
        const stream = await navigator.mediaDevices.getUserMedia({ video: true });
        this.$refs.video.srcObject = stream;
        // 开始绘制视频帧
        this.drawFrame();
      } catch (error) {
        console.error("无法访问摄像头:", error);
      }
    },

    // 切换镜像状态
    toggleMirror() {
      this.isMirrored = !this.isMirrored;
    },

    // 切换边缘检测状态
    toggleEdgeDetection() {
      this.edgeDetectionEnabled = !this.edgeDetectionEnabled;
    },

    // 绘制视频帧并进行边缘检测
    drawFrame() {
      const video = this.$refs.video;
      const canvas = this.$refs.canvas;
      const context = canvas.getContext("2d");

      // 确保 canvas 大小与 video 一致
      canvas.width = video.videoWidth;
      canvas.height = video.videoHeight;

      // 清空画布
      context.clearRect(0, 0, canvas.width, canvas.height);

      // 在 canvas 上绘制当前视频帧
      context.drawImage(video, 0, 0, canvas.width, canvas.height);

      if (this.edgeDetectionEnabled) {
        this.applyEdgeDetection(canvas, context);
      }

      // 使用 requestAnimationFrame 循环更新
      requestAnimationFrame(this.drawFrame);
    },

    // 应用边缘检测（使用 Sobel 算子）
    applyEdgeDetection(canvas, context) {
      const imageData = context.getImageData(0, 0, canvas.width, canvas.height);
      const data = imageData.data;
      const width = canvas.width;
      const height = canvas.height;

      // Sobel 算子
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

      const output = new Uint8ClampedArray(data.length);

      // 遍历每个像素，应用 Sobel 算子
      for (let y = 1; y < height - 1; y++) {
        for (let x = 1; x < width - 1; x++) {
          let pixelX = 0;
          let pixelY = 0;

          for (let ky = -1; ky <= 1; ky++) {
            for (let kx = -1; kx <= 1; kx++) {
              const pixel = this.getPixel(data, width, x + kx, y + ky);
              pixelX += pixel.r * sobelX[ky + 1][kx + 1];
              pixelY += pixel.r * sobelY[ky + 1][kx + 1];
            }
          }

          const magnitude = Math.sqrt(pixelX * pixelX + pixelY * pixelY);

          const index = (y * width + x) * 4;
          const color = magnitude > 255 ? 255 : magnitude;

          output[index] = color; // R
          output[index + 1] = color; // G
          output[index + 2] = color; // B
          output[index + 3] = 255; // A
        }
      }

      // 将结果放回 canvas
      imageData.data.set(output);
      context.putImageData(imageData, 0, 0);
    },

    // 获取指定像素的 RGB 值
    getPixel(data, width, x, y) {
      const index = (y * width + x) * 4;
      return {
        r: data[index],
        g: data[index + 1],
        b: data[index + 2]
      };
    },
  },
  mounted() {
    this.startCamera(); // 组件挂载时启动摄像头
  },
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
