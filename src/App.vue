<template>
  <div class="container">
    <video class="camera" ref="camera"></video>
    <canvas class="canvas" ref="canvas" />
  </div>
</template>

<script lang="ts" setup>
import * as tf from '@tensorflow/tfjs';
import * as handpose from '@tensorflow-models/handpose';
import { ref, onMounted } from 'vue';

const camera = ref<MediaDeviceInfo[]>([]);
const canvas = ref(null);

const loadCamera = () => {
  navigator.mediaDevices.getUserMedia({ video: true }).then((stream) => {
    camera.value.srcObject = stream;
    camera.value.play();
  });
};

const loadHandpose = async () => {
  const net = await handpose.load();
  setInterval(() => detect(net), 1);
};

const detect = async (net: handpose.HandPose) => {
  if (
    typeof camera.value !== 'undefined' &&
    camera.value !== null &&
    camera.value.readyState === 4
  ) {
    const videoWidth = camera.value.videoWidth;
    const videoHeight = camera.value.videoHeight;

    camera.value.width = videoWidth;
    camera.value.height = videoHeight;

    canvas.value.width = videoWidth;
    canvas.value.height = videoHeight;

    const hand = await net.estimateHands(camera.value);
    const ctx = canvas.value.getContext('2d');
    drawHand(hand, ctx);
  }
};

const fingerJoints = {
  thumb: [0, 1, 2, 3, 4],
  indexFinger: [0, 5, 6, 7, 8],
  middleFinger: [0, 9, 10, 11, 12],
  ringFinger: [0, 13, 14, 15, 16],
  pinky: [0, 17, 18, 19, 20],
};

const drawHand = (predictions: any, ctx: any) => {
  if (predictions.length > 0) {
    predictions.forEach((prediction: any) => {
      const landmarks = prediction.landmarks;

      for (let j = 0; j < Object.keys(fingerJoints).length; j++) {
        let finger = Object.keys(fingerJoints)[j];
        for (let k = 0; k < fingerJoints[finger].length - 1; k++) {
          const firstJointIndex = fingerJoints[finger][k];
          const secondJointIndex = fingerJoints[finger][k + 1];

          ctx.beginPath();
          ctx.moveTo(landmarks[firstJointIndex][0], landmarks[firstJointIndex][1]);
          ctx.lineTo(landmarks[secondJointIndex][0], landmarks[secondJointIndex][1]);
          ctx.strokeStyle = 'plum';
          ctx.lineWidth = 4;
          ctx.stroke();
        }
      }

      for (let i = 0; i < landmarks.length; ++i) {
        const x = landmarks[i][0];
        const y = landmarks[i][1];
        ctx.beginPath();
        ctx.arc(x, y, 5, 0, 3 * Math.PI);
        ctx.fillStyle = 'indigo';
        ctx.fill();
      }
    });
  }
};

onMounted(() => {
  loadCamera();
  loadHandpose();
});
</script>
<style lang="scss">
html {
  background-color: rgb(30, 34, 40);
}
body {
  margin: 0;
}
.container {
  height: 100vh;
  width: 100vw;
}
.canvas {
  text-align: center;
  position: absolute;
  z-index: 1;
  width: 640px;
  height: 480px;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}
.camera {
  text-align: center;
  position: absolute;
  z-index: 1;
  width: 640px;
  height: 480px;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}
</style>
