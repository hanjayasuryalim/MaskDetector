<template>
  <div>
    <div class="alert alert-primary" role="alert">Prediction</div>
    <p>This is prediction</p>
    <div class="alert alert-dark" role="alert">Input data</div>
    <div class="input-group mb-3">
      <div class="input-group-prepend">
        <span class="input-group-text" id="inputGroupFileAddon01">Upload</span>
      </div>
      <div class="custom-file">
        <input
          type="file"
          class="custom-file-input"
          id="inputGroupFile01"
          aria-describedby="inputGroupFileAddon01"
          @change="preview"
        />
        <label class="custom-file-label" for="inputGroupFile01">Choose file</label>
      </div>
    </div>
    <button class="btn btn-primary">Predict</button>
    <img :src="src" />
  </div>
</template>

<script>
import * as tf from '@tensorflow/tfjs'

export default {
  name: 'Tensorflow',
  data() {
    return {
      modelReady: false,
      src: ''
    }
  },
  mounted() {
    let that = this

    async function loadModel() {
      that.model = await tf.loadLayersModel('/Model/model.json')

      that.modelReady = true
      console.log('ready to predict, model fully loaded!')
      that.model.summary()
    }
    loadModel()
  },
  methods: {
    preview(event) {
      console.log(event)
    }
  }
}
</script>

<style>
</style>