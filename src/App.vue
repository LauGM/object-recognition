<template>
  <div class="container-fluid">
    <div>
      <h1 class="text-info">Detección de objetos</h1>
      <div v-if="!isStreaming">
        <button class="btn btn-sm btn-dark" @click="openCamera">Abrir Camara</button>
      </div>
       <div v-else>
              <button
                class="btn btn-sm btn-dark"
                @click="stopStreaming"
              >
                Detener streaming
              </button>
              <button
                class="btn btn-sm btn-dark"
                @click="snapshot"
              >
                Tomar foto
              </button>
       </div>
       <video v-show="isStreaming" ref="videoRef" autoplay class="img-fluid" id="video"/>
      <div>
        <img class="img-fluid" ref="imgRef" id="imagen" src="https://images.hola.com/imagenes/mascotas/20221020219416/razas-perros-toy/1-154-394/01-que-son-perros-toy-a.jpg"
         alt="imagen a predecir" crossorigin="anonymous">
         <div v-if="result.length > 0">
              <h3>{{ result.toUpperCase() }}</h3>
          </div>
      </div>
      <div v-if="imgRef" class="input-group input-group-sm mb-3">
        <div class="input-group-prepend">
          <span class="input-group-text" id="inputGroup-sizing-sm">Link</span>
        </div>
        <input v-model="imgRef.src" type="text" class="form-control" aria-label="Small" aria-describedby="inputGroup-sizing-sm">
      </div>
        <button
          class="btn btn-sm btn-dark"
          @click="detect"
        >
              <span v-if="isLoading">Intentando predecir ... </span>
              <span v-else>Detectar Objecto</span>
        </button>
    </div>
  </div>
</template>

<script lang="ts">
import { ref } from "vue";
import { defineComponent } from 'vue';
require('@tensorflow/tfjs-backend-cpu')
require('@tensorflow/tfjs-backend-webgl')
const cocoSsd = require('@tensorflow-models/coco-ssd')
const video = document.getElementById("video") as HTMLVideoElement;
const img = document.getElementById("imagen") as HTMLImageElement;
export default defineComponent({
  name: 'App',
  setup(){
    const imgRef = ref<HTMLImageElement>(img);
    const isLoading = ref(false);
    const videoRef = ref<HTMLVideoElement>(video);
    const isStreaming = ref(false);
    const result = ref("");

    async function detect() {
      isLoading.value = true;
      const img = imgRef.value;
      console.log(img);
      const model = await cocoSsd.load();
      const predictions = await model.detect(img);
      console.log("Prediccion:" + predictions[0]?.class || "vacio");
      result.value = predictions[0]?.class || "vacio";
      console.log(predictions, img);
      isLoading.value = false;
    }  
    async function openCamera() {
      if (navigator.mediaDevices.getUserMedia) {
        navigator.mediaDevices.getUserMedia({video: true}).then((stream)=>{
           isStreaming.value = true;
           
          videoRef.value.srcObject = stream;
        })
      }
    }
    function stopStreaming() {
      const stream = videoRef.value.srcObject as MediaStream;
      const tracks = stream.getTracks();
      tracks.map((track) => track.stop());
      isStreaming.value = false;
    }
    function snapshot() {
      const canvas = document.createElement("canvas");
      const ctx = canvas.getContext("2d") as CanvasRenderingContext2D;
      ctx.drawImage(videoRef.value, 0, 0, 200, 200);
      const data = canvas.toDataURL("image/png");
      imgRef.value.setAttribute("src", data);
    }
  
    return {
      imgRef,
      detect,
      result,
      isStreaming,
      videoRef,
      openCamera,
      stopStreaming,
      snapshot,
      isLoading
    }
  }
});
</script>

<style>
</style>
