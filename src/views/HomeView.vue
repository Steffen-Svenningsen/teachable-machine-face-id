<script setup lang="ts">
import WebCam from '@/components/WebCam.vue';

</script>

<template>
  <main>
    <h1>Person Recognition</h1>
    <p>By pressing start You'll be able to turn on your webcam/camera to detect weither there's a person in the picture through <a target="_blank" href="https://teachablemachine.withgoogle.com/">Machine Learning Models</a>.</p>
    <WebCam />
  </main>
</template>

<style lang="sass" scoped>
h1, p
  max-width: 420px
  text-align: center
  line-height: 1.2
  margin: 1rem 0 0.5rem 0

  a
    color: #487E01
    text-decoration: none
    filter: saturate(150%)
    transition: 0.2s

    &:hover
      filter: saturate(400%)
</style>