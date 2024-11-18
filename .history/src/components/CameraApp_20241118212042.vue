<template>
  <div class="camera-app">
    <div class="camera-container">
      <video
        ref="video"
        autoplay
        width="60%"
        height="auto"
        @play="onVideoPlay"
      ></video>
      <!-- Canvas 用于绘制标准线和姿态关键点 -->
      <canvas ref="canvas" class="overlay"></canvas>
    </div>
    <div class="controls">
      <button @click="toggleMirror">镜像</button>
      <button @click="toggleLines">切换标准线</button>
    </div>
  </div>
</template>

<script>
import * as posenet from '@tensorflow-models/posenet';
import '@tensorflow/tfjs';

export default {
  data() {
    return {
      showLines: false, // 控制标准线的显示与隐藏
      isMirrored: false, // 控制是否开启镜像
      poseNetModel: null, // PoseNet 模型实例
      videoWidth: 0,
      videoHeight: 0,
    };
  },
  methods: {
    // 获取摄像头流并开始播放
    async startCamera() {
      try {
        const stream = await navigator.mediaDevices.getUserMedia({ video: true });
        this.$refs.video.srcObject = stream;
        this.videoWidth = this.$refs.video.videoWidth;
        this.videoHeight = this.$refs.video.videoHeight;
        // 初始化 PoseNet 模型
        this.poseNetModel = await posenet.load();
      } catch (error) {
        console.error("无法访问摄像头:", error);
      }
    },

    // 切换标准线的显示与隐藏
    toggleLines() {
      this.showLines = !this.showLines;
    },

    // 切换镜像效果
    toggleMirror() {
      this.isMirrored = !this.isMirrored;
    },

    // 姿态检测与关键点绘制
    async detectPose() {
      const video = this.$refs.video;
      const canvas = this.$refs.canvas;
      const context = canvas.getContext('2d');

      // 确保 canvas 大小与 video 一致
      canvas.width = video.videoWidth;
      canvas.height = video.videoHeight;

      // 使用 PoseNet 模型进行姿态检测
      const pose = await this.poseNetModel.estimateSinglePose(video, {
        flipHorizontal: false, // 根据镜像状态决定是否翻转
      });

      // 清空画布
      context.clearRect(0, 0, canvas.width, canvas.height);

      // 绘制标准线
      if (this.showLines) {
        this.drawLines(context, canvas.width, canvas.height);
      }

      // 绘制姿态关键点
      this.drawPose(pose, context);

      // 在每一帧中进行姿态检测
      requestAnimationFrame(this.detectPose);
    },

    // 绘制标准线
    drawLines(context, canvasWidth, canvasHeight) {
      context.strokeStyle = "rgba(255, 0, 0, 0.7)"; // 红色标准线
      context.lineWidth = 2;

      // 画竖直标准线（屏幕中心）
      context.beginPath();
      context.moveTo(canvasWidth / 2, 0);
      context.lineTo(canvasWidth / 2, canvasHeight);
      context.stroke();

      // 画水平标准线（屏幕中心）
      context.beginPath();
      context.moveTo(0, canvasHeight / 2);
      context.lineTo(canvasWidth, canvasHeight / 2);
      context.stroke();
    },

    // 绘制姿态关键点
    drawPose(pose, context) {
      const keypoints = pose.keypoints;

      keypoints.forEach(point => {
        if (point.score > 0.5) { // 只绘制置信度高于0.5的关键点
          const { x, y } = point.position;
          context.beginPath();
          context.arc(x, y, 5, 0, 2 * Math.PI);
          context.fillStyle = 'aqua';
          context.fill();
        }
      });

      // 连接关键点（如果存在相应的连接）
      const adjacentKeyPoints = posenet.getAdjacentKeyPoints(keypoints, 0.5);
      adjacentKeyPoints.forEach(([point1, point2]) => {
        context.beginPath();
        context.moveTo(point1.position.x, point1.position.y);
        context.lineTo(point2.position.x, point2.position.y);
        context.strokeStyle = 'aqua';
        context.lineWidth = 2;
        context.stroke();
      });
    },

    // 当视频播放时触发
    onVideoPlay() {
      this.detectPose(); // 开始进行姿态检测
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
