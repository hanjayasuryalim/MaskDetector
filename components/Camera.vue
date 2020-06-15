<template>
  <div class="camera">
    <video autoplay class="feed"></video>
    <div class="button-container">
      <button class="btn btn-primary snap" v-on:click="$emit('takePicture')">
        Snap
      </button>
    </div>
  </div>
</template>

<script>
export default {
  name: 'Camera',
  methods: {
    init() {
      if (
        'mediaDevices' in navigator &&
        'getUserMedia' in navigator.mediaDevices
      ) {
        navigator.mediaDevices.getUserMedia({ video: true }).then((stream) => {
          const videoPlayer = document.querySelector('video')
          videoPlayer.srcObject = stream
          videoPlayer.play()
        })
      } else {
        alert(`Cannot get media access`)
      }
    }
  },
  beforeMount() {
    this.init()
  }
}
</script>

<style scoped>
.feed {
  background-color: #171717;
  width: 80%;
  margin: auto;
  float: left;
  box-shadow: 6px 6px 12px 0px rgba(0, 0, 0, 0.35);
}

.button-container {
  float: left;
  display: flex;
  flex-direction: column;
  align-content: center;
  justify-content: center;
  padding: 1rem;
}
</style>
