<template>
  <div id="app">
    <h1>Train:</h1>
    <div class="train-poses">
      <img class="pose" @click="learn(0)" src="./assets/rock.png" />
      <img class="pose" @click="learn(1)" src="./assets/paper.png" />
      <img class="pose" @click="learn(2)" src="./assets/scissors.png" />      
    </div>
    <video autoplay playsinline muted ref="webcam" width="224" height="224"></video>
    <div><img v-if="prediction" class="prediction-pose" @click="learn(2)" :src="getPredictionImage()" /></div>
    <div class='prediction' >{{prediction}}</div>
    <div><button class='play-btn'>Play</button></div>
    
  </div>
</template>

<script>
import HelloWorld from "./components/HelloWorld.vue";
import * as tf from '@tensorflow/tfjs';
import * as mobilenet from '@tensorflow-models/mobilenet';
import * as knnClassifier from '@tensorflow-models/knn-classifier';
import { setInterval } from 'timers';

let model
const classifier = knnClassifier.create();

export default {
  name: "app",
  data() {
    return {
      trained: [],
      prediction : '',
      probability: ''
    }
  },
  async mounted() {        
    model = await mobilenet.load({
      version: 1,
      alpha: 1.0,
      modelUrl:
        "http://localhost:8081/savedmodel_mobilenet_v1_1.0_224_model.json"
    });

    await this.setupWebcam();
    setInterval(async () => {
      if (classifier.getNumClasses() > 0) {               
        const activation = model.infer(this.$refs.webcam, 'conv_preds');        
        const result = await classifier.predictClass(activation);
        console.log(result);
        const classes = ['rock', 'paper', 'scissors'];
        this.prediction = classes[result.classIndex];  
               
      } 
    },250);    
  },
  methods: {
    async setupWebcam() {
      return new Promise((resolve, reject) => {
        const navigatorAny = navigator;
        navigator.getUserMedia =
          navigator.getUserMedia ||
          navigatorAny.webkitGetUserMedia ||
          navigatorAny.mozGetUserMedia ||
          navigatorAny.msGetUserMedia;
        if (navigator.getUserMedia) {
          navigator.getUserMedia(
            { video: true },
            stream => {
              this.$refs.webcam.srcObject = stream;
              this.$refs.webcam.addEventListener(
                "loadeddata",
                () => resolve(),
                false
              );
            },
            error => reject()
          );
        } else {
          reject();
        }
      });
    },
    learn(classID) {          
      console.log(classID);
      const activation = model.infer(this.$refs.webcam, 'conv_preds');
      classifier.addExample(activation, classID);       
    },
    getPredictionImage(){      
      return require(`./assets/${this.prediction}.png`);
    }
  },
  components: {
    //HelloWorld
  }
};
</script>

<style lang="scss">
html,
body {
  padding: 0;
  margin: 0;
  background-color: rgb(254, 232, 52);
}

#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

.train-poses {
  display: flex;
  justify-content: center;
}

.play-btn{
  
  width: 180px;
  height: 50px;
  background-color: rgb(16, 104, 206);
  color: white;
  cursor: pointer;
  border: none;
  border-radius: 5px;
  font-size: 20px;
  
}

.prediction-pose{
  width:50px;
}

.pose {
  width: 200px;
  cursor: pointer;
  align-items: flex-start;
  opacity: 0.8;
  &:hover {
    opacity: 1;
    transform: translateY(5px);
  }
  &:active {
    opacity: 1;
    transform: translateY(8px);
  }
}
</style>
