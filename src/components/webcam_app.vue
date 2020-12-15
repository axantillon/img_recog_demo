<!-- 

Credits to Dita Rahma Puspitasari for base app and webcam functionality
https://codepen.io/ditarahma08/pen/GRRxZLW

-->

<template>
  <div class="web-camera-container" >
    <div class="camera-button">
      <button type="button" class="button is-rounded" :class="{ 'is-primary' : !isCameraOpen, 'is-danger' : isCameraOpen}" @click="toggleCamera">
        <span v-if="!isCameraOpen">Open Camera</span>
        <span v-else>Close Camera</span>
      </button>
    </div>
    
    <div v-show="isCameraOpen && isLoading" class="camera-loading">
      <ul class="loader-circle">
        <li></li>
        <li></li>
        <li></li>
      </ul>
    </div>
    
    <div v-if="isCameraOpen" v-show="!isLoading" class="camera-box" :class="{ 'flash' : isShotPhoto }">
      
      <div class="camera-shutter" :class="{'flash' : isShotPhoto}"></div>
        
      <video v-show="!isPhotoTaken" ref="camera" width="80%" height="80%" autoplay></video>
      
      <canvas v-show="isPhotoTaken" id="photoTaken" width="300px" height="80%" ref="canvas"></canvas>
    </div>
    
    <div v-if="isCameraOpen && !isLoading" class="camera-shoot">
      <button type="button" class="button" @click="takePhoto">
        <img src="https://img.icons8.com/material-outlined/50/000000/camera--v2.png">
      </button>
    </div>

    <div v-if="isPhotoTaken" style="display:flex; justify-content:center; flex-direction:column">
      <b>
        Class:
      </b>
        {{this.prediction.label}}
      <b>
        Confidence:
      </b>
        {{this.prediction.confidence}}
    </div>

  </div>
</template>

<script>
import ml5 from "ml5"

export default {

  data: () => ({
    isCameraOpen: false,
    isLoading: false,
    isPhotoTaken: false,
    isShotPhoto: false,
    //Use Ml.5 for the builtin MobileNet Image Classifier
    classifier: ml5.imageClassifier('MobileNet', () => {console.log("Model Loaded! ")}),
    prediction: {},
  }),

  mounted() {
  },
  
  methods: {
    toggleCamera() {
      if(this.isCameraOpen) {
          this.isCameraOpen = false;
          this.isPhotoTaken = false;
          this.isShotPhoto = false;
          this.stopCameraStream();
      } else {
          this.isCameraOpen = true;
          this.createCameraElement();
      }
    },

    takePhoto() {
      if(!this.isPhotoTaken) {
        this.isShotPhoto = true;

        //Create a Flass effect when taking a Photo
        const FLASH_TIMEOUT = 50;

        setTimeout(() => {
          this.isShotPhoto = false;
        }, FLASH_TIMEOUT);
      }
      
      this.isPhotoTaken = !this.isPhotoTaken;
      
      //Draw the Snapshot on top of the camera feed
      const context = this.$refs.canvas.getContext('2d');
      const camera_style = window.getComputedStyle(this.$refs.camera)
      const canvas_width = parseInt(camera_style.getPropertyValue('width'), 10)
      const canvas_height = parseInt(camera_style.getPropertyValue('height'), 10)
      this.$refs.canvas.width = canvas_width
      this.$refs.canvas.height = canvas_height
      context.drawImage(this.$refs.camera, 0, 0, canvas_width, canvas_height);

      //Use Ml.5 to classify the image and store the results
      this.classifier.classify(this.$refs.camera, (err, results) => {
        console.log(results[0]);
        this.prediction = results[0];
      });
    },
    
    createCameraElement() {
      this.isLoading = true;
    
      //Establish what the browser should record
      const constraints = (window.constraints = {
        audio: false,
        //The 'environment' value ensures that on a smartphone the outer camera is utilized
        //This is ignored if utilizing a laptop
        video: {facingMode: 'environment'}
      });

      //Establish connection to webcam
      navigator.mediaDevices
        .getUserMedia(constraints)
        .then(stream => {
            this.isLoading = false;
            this.$refs.camera.srcObject = stream;
        })
        .catch(error => {
            this.isLoading = false;
            console.log(error)
            alert("May the browser didn't support or there is some errors.");
        });
    },
    
    stopCameraStream() {
    let tracks = this.$refs.camera.srcObject.getTracks();

            tracks.forEach(track => {
                track.stop();
            });
    },
  }
}
</script>

<style lang="scss">

body {
  display: flex;
  justify-content: center;
}

.web-camera-container {
  margin-top: 2rem;
  margin-bottom: 2rem;
  padding: 2rem;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  border: 1px solid #ccc;
  border-radius: 4px;

  
  .camera-button {
    margin-bottom: 2rem;
  }
  
  .camera-box {    
    .camera-shutter {
      opacity: 0;
      background-color: #fff;
      position: absolute;
      
      &.flash {
        opacity: 1;
      }
    }
  }
  
  .camera-shoot {
    margin: 1rem 0;
    
    button {
      height: 60px;
      width: 60px;
      display: flex;
      align-items: center;
      justify-content: center;
      border-radius: 100%;
      
      img {
        height: 35px;
        object-fit: cover;
      }
    }
  }
  
  .camera-loading {
    overflow: hidden;
    height: 100%;
    position: absolute;
    width: 100%;
    min-height: 150px;
    margin: 3rem 0 0 -1.2rem;
    
    ul {
      height: 100%;
      position: absolute;
      width: 100%;
      z-index: 999999;
      margin: 0;
    }
    
    .loader-circle {
      display: block;
      height: 14px;
      margin: 0 auto;
      top: 50%;
      left: 100%;
      transform: translateY(-50%);
      transform: translateX(-50%);
      position: absolute;
      width: 100%;
      padding: 0;
      
      li {
        display: block;
        float: left;
        width: 10px;
        height: 10px;
        line-height: 10px;
        padding: 0;
        position: relative;
        margin: 0 0 0 4px;
        background: #999;
        animation: preload 1s infinite;
        top: -50%;
        border-radius: 100%;
        
        &:nth-child(2) {
          animation-delay: .2s;
        }
        
        &:nth-child(3) {
          animation-delay: .4s;
        }
      }
    }
  }

  @keyframes preload {
    0% {
      opacity: 1
    }
    50% {
      opacity: .4
    }
    100% {
      opacity: 1
    }
  }
}
</style>