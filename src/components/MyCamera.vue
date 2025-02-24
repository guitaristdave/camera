<template>
    <div class="container">
        <h1>{{ title }}</h1>
        <video ref="video" autoplay playsinline style="display: none;"></video>
        <canvas v-show="!photoCaptured" ref="videoCanvas" style="transform: scaleX(-1);"></canvas>
        <canvas v-show="photoCaptured" ref="photoCanvas" style="transform: scaleX(-1);"></canvas>
        <div class="buttons">
            <button v-if="!photoCaptured" @click="capturePhoto">Capture</button>
            <button v-if="photoCaptured && !photoApproved" @click="approvePhoto">Approve</button>
            <button v-if="photoCaptured && !photoApproved" @click="retakePhoto">Retake</button>
        </div>
    </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue';

const props = defineProps({
    title: String
});

const photoCaptured = ref(false);
const videoCanvas = ref(null); 
const photoCanvas = ref(null); 
const video = ref(null);
const photoApproved = ref(false);
let mediaStream = null;


const drawVideo = () => {
    videoCanvas.value.width = video.value.videoWidth;
    videoCanvas.value.height = video.value.videoHeight;
    const ctx = videoCanvas.value.getContext('2d');
    ctx.drawImage(video.value, 0, 0, videoCanvas.value.width, videoCanvas.value.height);
    requestAnimationFrame(drawVideo);
}

const startCamera = async () => {
    mediaStream = await navigator.mediaDevices.getUserMedia({ video: {width: {ideal: 1280}, height: {ideal: 720}}, audio: false });
    video.value.srcObject = mediaStream;
    video.value.play();
    drawVideo();
}

const capturePhoto = () => {
    photoCaptured.value = true;
    photoCanvas.value.width = video.value.videoWidth;
    photoCanvas.value.height = video.value.videoHeight;
    const ctx = photoCanvas.value.getContext('2d');
    ctx.drawImage(video.value, 0, 0, photoCanvas.value.width, photoCanvas.value.height);
}

const stopCamera = () => {
    mediaStream.getTracks().forEach(track => track.stop());
}


const approvePhoto = () => {
    photoApproved.value = true;
}

const retakePhoto = () => {
    photoCaptured.value = false;
    photoApproved.value = false;
    startCamera();
}

onMounted(() => {
    startCamera();
});

onUnmounted(() => {
    stopCamera();
});
</script>

