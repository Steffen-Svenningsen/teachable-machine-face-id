<script setup lang="ts">
import * as tmImage from '@teachablemachine/image';
import { ref } from "vue";

const videoActive = ref(false);

// the link to your model provided by Teachable Machine export panel
const URL = '/src/static/model/';

let model, webcam, labelContainer, maxPredictions;

let isIos = false; 
// fix when running demo in ios, video will be frozen;
if (window.navigator.userAgent.indexOf('iPhone') > -1 || window.navigator.userAgent.indexOf('iPad') > -1) {
    isIos = true;
}
// Load the image model and setup the webcam
async function init() {
    const modelURL = URL + 'model.json';
    const metadataURL = URL + 'metadata.json';

    // load the model and metadata
    // Refer to tmImage.loadFromFiles() in the API to support files from a file picker
    // or files from your local hard drive
    model = await tmImage.load(modelURL, metadataURL);
    maxPredictions = model.getTotalClasses();

    // Convenience function to setup a webcam
    const flip = true; // whether to flip the webcam
    const width = 200;
    const height = 200;
    webcam = new tmImage.Webcam(width, height, flip);
    await webcam.setup(); // request access to the webcam

    if (isIos) {
        document.getElementById('webcam-container').appendChild(webcam.webcam); // webcam object needs to be added in any case to make this work on iOS
        // grab video-object in any way you want and set the attributes
        const webCamVideo = document.getElementsByTagName('video')[0];
        webCamVideo.setAttribute("playsinline", true); // written with "setAttribute" bc. iOS buggs otherwise
        webCamVideo.muted = "true";
        webCamVideo.style.width = width + 'px';
        webCamVideo.style.height = height + 'px';
        webCamVideo.style.borderRadius = '6px';
    } else {
        document.getElementById("webcam-container").appendChild(webcam.canvas);
        const canvas = document.getElementById("webcam-container").querySelector("canvas");
        canvas.style.borderRadius = "6px";
    }
    // append elements to the DOM
    labelContainer = document.getElementById('label-container');
    for (let i = 0; i < maxPredictions; i++) { // and class labels
        labelContainer.appendChild(document.createElement('div'));
    }
    webcam.play();
    videoActive.value = true;
    window.requestAnimationFrame(loop);
}

async function loop() {
    webcam.update(); // update the webcam frame
    await predict();
    window.requestAnimationFrame(loop);
}

// run the webcam image through the image model
async function predict() {
    // predict can take in an image, video or canvas html element
    let prediction;
    if (isIos) {
        prediction = await model.predict(webcam.webcam);
    } else {
        prediction = await model.predict(webcam.canvas);
    }
    for (let i = 0; i < maxPredictions; i++) {
        const classPrediction =
            prediction[i].className + ': ' + prediction[i].probability.toFixed(2);
        labelContainer.childNodes[i].innerHTML = classPrediction;
    }
}

const stopRecording = () => {
    webcam.stop();
    videoActive.value = false;
    const webcamContainer = document.getElementById('webcam-container');
    webcamContainer.innerHTML = '';
    const labelContainer = document.getElementById('label-container');
    labelContainer.innerHTML = '';
}
</script>

<template>
    <div id='webcam-container'></div>
    <button v-if="videoActive" type="button" @click="stopRecording">Stop</button>
    <button v-else type='button' @click="init">Start</button>
    <div id='label-container'></div>
</template>

<style lang="sass" scoped>
button
    padding: 0.5rem 1rem
    cursor: pointer
</style>