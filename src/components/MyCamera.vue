<template>
    <div class="container">
        <video ref="video" autoplay playsinline class="media" style="display: none;"></video>
        <canvas v-show="!photoCaptured" ref="videoCanvas" class="media"
            :class="{ mirror: !isUserFacingCamera }"></canvas>
        <canvas v-show="photoCaptured" ref="photoCanvas" class="media"
            :class="{ mirror: !isUserFacingCamera }"></canvas>
        <div class="buttons">
            <div v-if="!photoCaptured" class="photo-buttons">
                <button @click="capturePhoto" class="captureBtn">📸</button>
                <button @click="changeCamera" class="captureBtn">🔄</button>
            </div>
            <div v-if="photoCaptured && !photoApproved" class="action-buttons">
                <button @click="downloadPhoto" class="actionBtn">⬇️</button>
                <button @click="retakePhoto" class="actionBtn">🔄</button>
            </div>
        </div>
    </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted, computed } from 'vue';

const photoCaptured = ref(false);
const videoCanvas = ref(null);
const photoCanvas = ref(null);
const video = ref(null);
const photoApproved = ref(false);
const currentFacingMode = ref('user');
let mediaStream = null;

const drawVideo = () => {
    if (!video.value) return;
    videoCanvas.value.width = video.value.videoWidth;
    videoCanvas.value.height = video.value.videoHeight;
    const ctx = videoCanvas.value.getContext('2d');
    ctx.drawImage(video.value, 0, 0, videoCanvas.value.width, videoCanvas.value.height);
    requestAnimationFrame(drawVideo);
};

const startCamera = async () => {
    try {
        mediaStream = await navigator.mediaDevices.getUserMedia({ video: { facingMode: currentFacingMode.value }, audio: false });
        video.value.srcObject = mediaStream;
        video.value.play();
        drawVideo();
    } catch (err) {
        console.error('Ошибка при доступе к камере:', err);
    }
};

const capturePhoto = () => {
    photoCaptured.value = true;
    photoCanvas.value.width = video.value.videoWidth;
    photoCanvas.value.height = video.value.videoHeight;
    const ctx = photoCanvas.value.getContext('2d');
    ctx.drawImage(video.value, 0, 0, photoCanvas.value.width, photoCanvas.value.height);
};

const stopCamera = () => {
    if (mediaStream) mediaStream.getTracks().forEach(track => track.stop());
};

const downloadPhoto = () => {
    const link = document.createElement('a');
    link.href = photoCanvas.value.toDataURL('image/png');
    link.download = 'photo.png';
    link.click();
};

const retakePhoto = () => {
    photoCaptured.value = false;
    photoApproved.value = false;
    startCamera();
};

const changeCamera = () => {
    currentFacingMode.value = currentFacingMode.value === 'user' ? 'environment' : 'user';
    stopCamera();
    startCamera();
};

const isUserFacingCamera = computed(() => currentFacingMode.value === 'user');

onMounted(() => startCamera());
onUnmounted(() => stopCamera());
</script>

<style scoped>
.container {
    width: 100vw;
    height: 100vh;
    overflow: hidden;
    position: relative;
    display: flex;
    justify-content: center;
    align-items: center;
    background-color: black;
}

.media {
    position: absolute;
    top: env(safe-area-inset-top, 0);
    left: env(safe-area-inset-left, 0);
    width: 100vw;
    height: 100vh;
    object-fit: cover;
}

.mirror {
    transform: scaleX(-1);
    /* Эффект зеркала */
}

.buttons {
    position: absolute;
    bottom: calc(20px + env(safe-area-inset-bottom, 0));
    width: 100%;
    display: flex;
    justify-content: center;
    align-items: center;
}

.captureBtn {
    width: 80px;
    height: 80px;
    background-color: rgb(255, 255, 255);
    border: 4px solid rgba(0, 0, 0, 0.3);
    border-radius: 50%;
    cursor: pointer;
    box-shadow: 0 0 10px rgba(0, 0, 0, 0.5);
    font-size: 28px;
}

.action-buttons {
    display: flex;
    gap: 20px;
    position: absolute;
    bottom: calc(50px + env(safe-area-inset-bottom, 0));
    
}

.photo-buttons {
    display: flex;
    gap: 20px;
    position: absolute;
    bottom: calc(50px + env(safe-area-inset-bottom, 0));
}

.actionBtn {
    padding: 10px 20px;
    font-size: 16px;
    background-color: rgba(255, 255, 255, 0.8);
    border: none;
    border-radius: 12px;
    cursor: pointer;
    box-shadow: 0 2px 6px rgba(0, 0, 0, 0.3);
}

@media (orientation: landscape) {
    .photo-buttons {
        bottom: calc(50px + env(safe-area-inset-bottom, 0));
    }

    .action-buttons {
        bottom: calc(50px + env(safe-area-inset-bottom, 0));
    }
}
</style>
