<template>
  <div>
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
          @change="readURL"
        />
        <label class="custom-file-label" for="inputGroupFile01"
          >Choose file</label
        >
      </div>
    </div>
    <img
      id="uploadedImage"
      alt="img"
      src="../assets/MASK detector.png"
      class="myImage"
    />
    <br />
    <br />
    <br />
    <div class="alert alert-primary" role="alert">Prediction</div>
    <p id="prediction">This is prediction</p>

    <button class="btn btn-primary" :disabled="notReady" @click="predict">
      Predict
    </button>
  </div>
</template>

<script>
import * as tf from '@tensorflow/tfjs'
import 'vuejs-noty/dist/vuejs-noty.css'

export default {
  name: 'Tensorflow',
  data() {
    return {
      modelReady: false,
      notReady: true
    }
  },
  mounted() {
    let that = this

    async function loadModel() {
      that.model = await tf.loadLayersModel('/Model/model.json')

      that.modelReady = true
      alert('Ready to predict, model is fully loaded !')
      that.model.summary()
    }
    loadModel()
  },
  methods: {
    readURL(e) {
      if (e.target.files && e.target.files[0]) {
        if (e.target.files[0].type.split('/')[0] !== 'image') {
          this.$noty.warning('Please insert a proper image', {
            timeout: 2000
          })
        } else {
          var reader = new FileReader()
          reader.onload = (e) => {
            document
              .querySelector('#uploadedImage')
              .setAttribute('src', e.target.result)
          }
          reader.readAsDataURL(e.target.files[0])
          this.notReady = false
        }
      }
    },
    resizeImage(imgElement) {
      const IMAGE_SIZE = 100
      const canvas = document.createElement('canvas')
      canvas.height = IMAGE_SIZE
      canvas.width = IMAGE_SIZE

      const ctx = canvas.getContext('2d')
      ctx.clearRect(0, 0, canvas.width, canvas.height) // clear canvas
      ctx.drawImage(imgElement, 0, 0, canvas.width, canvas.height)

      const img = document.createElement('img')
      img.src = canvas.toDataURL('image/jpeg')
      img.width = IMAGE_SIZE
      img.height = IMAGE_SIZE

      return img
    },
    async predict() {
      const modelUrl =
        'https://storage.googleapis.com/tfjsaldo/keras_model/model.json'
      const img = document.querySelector('#uploadedImage')
      const rezImg = this.resizeImage(img)
      const model = await tf.loadLayersModel(modelUrl)
      const startTime1 = performance.now()

      let startTime2
      const logits = tf.tidy(() => {
        // tf.browser.fromPixels() returns a Tensor from an image element.
        // const img = tf.browser.fromPixels(imgElement, 1);

        // Normalize the image from [0, 225] to [INPUT_MIN, INPUT_MAX]
        const normalizationConstant = 1.0 / 255.0
        // const normalized = img.toFloat().mul(normalizationConstant);

        let tensor = tf.browser
          .fromPixels(rezImg, 3)
          .resizeBilinear([64, 64], false)
          .expandDims(0)
          .toFloat()
          .mul(normalizationConstant)

        startTime2 = performance.now()

        // Make a prediction through model.
        return model.predict(tensor)
      })
      const classes = await logits.data()

      const labelName = classes[0] < 0.5 ? 'MASK' : 'NO_MASK'
      const probability = classes[0]

      const totalTime1 = performance.now() - startTime1
      const totalTime2 = performance.now() - startTime2
      this.$noty.success(
        `Done in ${Math.floor(totalTime1)} ms ` +
          `\n(not including preprocessing: ${Math.floor(totalTime2)} ms)`,
        {
          timeout: 2000
        }
      )

      // Return the best probability label
      document.querySelector(
        '#prediction'
      ).innerText = `${labelName} (${Math.floor(probability * 100)}%)`
    }
  }
}
</script>

<style scoped>
.myImage {
  width: 50%;
  border: 2px solid lightblue;
}
</style>
