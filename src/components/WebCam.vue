<script setup>
import * as tmImage from '@teachablemachine/image';
import { ref } from "vue";

const videoActive = ref(false);

const URL = "https://teachablemachine.withgoogle.com/models/WUjKsLUR-/";

let model, webcam, labelContainer, maxPredictions;

// Load the image model and setup the webcam
async function init() {
    const modelURL = URL + "model.json";
    const metadataURL = URL + "metadata.json";

    // load the model and metadata
    // Refer to tmImage.loadFromFiles() in the API to support files from a file picker
    // or files from your local hard drive
    // Note: the pose library adds "tmImage" object to your window (window.tmImage)
    model = await tmImage.load(modelURL, metadataURL);
    maxPredictions = model.getTotalClasses();

    // Convenience function to setup a webcam
    const flip = true; // whether to flip the webcam
    webcam = new tmImage.Webcam(300, 300, flip); // width, height, flip
    await webcam.setup(); // request access to the webcam
    await webcam.play();
    videoActive.value = true;
    window.requestAnimationFrame(loop);

    // append elements to the DOM
    const webcamCanvas = document.getElementById("webcam-container").appendChild(webcam.canvas);
    webcamCanvas.style.borderRadius = "6px";
    labelContainer = document.getElementById("label-container");
    for (let i = 0; i < maxPredictions; i++) { // and class labels
        const labelChild = labelContainer.appendChild(document.createElement("div"));
        labelChild.style.borderRadius = "6px";
        labelChild.style.border = "1px solid #acacac";
        labelChild.style.padding = "1rem";
    }
}

async function loop() {
    webcam.update(); // update the webcam frame
    await predict();
    window.requestAnimationFrame(loop);
}

// run the webcam image through the image model
async function predict() {
    // predict can take in an image, video or canvas html element
    const prediction = await model.predict(webcam.canvas);
    for (let i = 0; i < maxPredictions; i++) {
        const classPrediction =
            prediction[i].className + ": " + (prediction[i].probability * 100).toFixed(0) + "%";
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
    <div id='label-container'></div>
    <button class="stop-btn" v-if="videoActive" type="button" @click="stopRecording">Stop</button>
    <button class="start-btn" v-else type='button' @click="init">Start</button>
</template>

<style lang="sass" scoped>
button
    padding: 0.5rem 1rem
    cursor: pointer
    position: relative
    margin: 1rem

    &::after
        position: absolute
        font-size: 20px
        width: 100%
        height: 100%
        top: 0
        left: 0
        transform: translate(50%, -50%)

.start-btn
    &::after
        content: '📸'
        transform: translate(50%, -50%) rotate(-20deg)

.stop-btn
    &::after
        content: '❌'

#label-container
    width: 300px
    display: flex
    flex-direction: column
    gap: 8px
</style>