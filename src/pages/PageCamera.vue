<template>
  <q-page class="constrain-more q-pa-md">
    <div class="camera-frame q-pa-md">
      <video 
        v-show="!imageCaptured"
        ref="video"
        class="full-width"
        autoplay
      />
      <canvas 
        v-show="imageCaptured"
        ref="canvas"
        class="full-width"
        height="240"
      />
    </div>
    <div class="text-center q-pa-md">
      <q-btn
        v-if="hasCanvasSupport"
        @click="captureImage"
        size="lg"
        color="grey-10"
        icon="eva-camera"
        round
      />
      <q-file
        v-else
        accept="image/*"
        @input="captureImageFallback"
        outlined
        label="Choose an image"
        v-model="imageUpload"
      >
        <template v-slot:prepend>
          <q-icon name="eva-attach-outline" />
        </template>
      </q-file>
      <div class="row justify-center q-ma-md">
        <q-input
          v-model="post.caption"
          class="col col-sm-6"
          label="Caption"
          dense
        />
      </div>
      <div class="row justify-center q-ma-md">
        <q-input
          v-model="post.location"
          class="col col-sm-6"
          label="Location"
          dense
        >
        <template v-slot:append>
          <q-btn
            @click="getLocation"
            round
            dense
            flat
            icon="eva-navigation-2-outline" 
          />
        </template>
        </q-input>
      </div>
      <div class="row justify-center q-ma-md q-mt-lg">
        <q-btn color="primary" rounded unelevated label="Post Image"/> 
      </div>
    </div>
  </q-page>
</template>

<script>
import {uid} from 'quasar'
require('md-gum-polyfill')

export default {
  name: 'PageCamera',
  data () {
    return {
      post: {
        id: uid(),
        caption: '',
        location: '',
        photo: null,
        date: Date.now()
      },
      imageCaptured: false,
      imageUpload: [],
      hasCanvasSupport: true
    }
  },
  methods: {
    initCamera(){
      navigator.mediaDevices.getUserMedia({
        video: true
      }).then(stream => {
        this.$refs.video.srcObject = stream
      }).catch(error => {
        this.hasCanvasSupport = false
      })
    },
    captureImage() {
      let video = this.$refs.video
      let canvas = this.$refs.canvas

      canvas.width = video.getBoundingClientRect().width
      canvas.height = video.getBoundingClientRect().height

      let context = canvas.getContext('2d')

      context.drawImage(video, 0, 0, canvas.width, canvas.height)

      this.imageCaptured = true
      this.post.photo = this.dataURItoBlob(canvas.toDataURL())
      this.disableCamera()
    },
    captureImageFallback(file){
      this.post.photo = file

      let canvas = this.$refs.canvas
      let context = canvas.getContext('2d')

      const reader = new FileReader();
      reader.onload = event => {
        const img = new Image();
        img.onload = () => {
          canvas.width = img.width;
          canvas.height = img.height;
          context.drawImage(img,0,0);
          this.imageCaptured = true
        }
        img.src = event.target.result;
      }
      reader.readAsDataURL(file);

    },
    disableCamera() {
      this.$refs.video.srcObject.getVideoTracks().forEach(track => {
        track.stop()
      })
    },
    dataURItoBlob(dataURI) {
      const byteString = atob(dataURI.split(',')[1]);

      const mimeString = dataURI.split(',')[0].split(':')[1].split(';')[0]

      const ab = new ArrayBuffer(byteString.length);

      const ia = new Uint8Array(ab);

      for (let i = 0; i < byteString.length; i++) {
        ia[i] = byteString.charCodeAt(i);
      }

      const blob = new Blob([ab], {type: mimeString});
      return blob;
    },
    getLocation() {
      navigator.geolocation.getCurrentPosition(position => {
        this.getCityAndCountry(position)
      }, err => {
        console.log(error)
      }, { timeout: 7000 })
    },
    getCityAndCountry(position) {
      let apiUrl = `http://geocode.xyz/${position.coords.latitude},${position.coords.longitude}?json=1`
      this.$axios.get(apiUrl).then((resp) => {
        this.locationSuccess(resp)
      }).catch((error)=> {
        console.log(error)
      })
    },
    locationSuccess(val) {
      this.post.location = val.data.city

      if(val.data.country) {
        this.post.location += `, ${resul.data.city}`
      }
    }
  },
  mounted() {
    this.initCamera()
  },
  beforeDestroy() {
    if(this.hasCanvasSupport) {
      this.disableCamera()
    }
  }
}
</script>

<style lang="sass">
  .camera-frame
    border: 2px solid $grey-10
    border-radius: 10px
</style>