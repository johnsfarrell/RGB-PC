<template>
    <div class="card mt-3 mb-3">
      <h5 class="card-header">
        <div
          class="collapsed d-block"
          data-bs-toggle="collapse"
          data-bs-target="#collapse-wavelight-collapsed"
          id="heading-collapsed"
        >
          <i class="fa fa-chevron-down pull-right"></i>
          Wave Light
        </div>
      </h5>
      <div
        id="collapse-wavelight-collapsed"
        class="collapse show p-3"
        aria-labelledby="heading-collapsed"
      >
        <button @click="cycle" class="float-end btn btn-primary me-2">
          Start Cycle
        </button>
        <button @click="stopCycleVar" class="float-end btn btn-primary me-2">
          Stop
        </button>
  
        <div class="form-group">
          <label for="mph">Speed (mph):</label>
          <input type="number" v-model="mph" class="form-control" id="mph" />
        </div>
  
        <div class="form-group">
          <label for="color">Choose Color:</label>
          <input type="input" class="color-picker" value="#FF0000" name="color" />
        </div>
      </div>
    </div>
  </template>
  
  <script>
  import { createPicker } from "../models/Picker.js";
  export default {
    props: {
      strip: {
        type: Object,
        required: true,
      },
    },
    data: () => {
      return {
        stopCycle: false,
        picker: null,
        mph: 0,
        color: "#FF0000",
      };
    },
    methods: {
      async stopCycleVar() {
        this.stopCycle = true;
      },
      async cycle() {
        if (this.mph === 0) return;
        
        const sleep = (ms) => new Promise(resolve => setTimeout(resolve, ms));
        
        const numSegments = 15;
        const lengthFeet = 50;
        const segmentLengthFeet = lengthFeet / numSegments;
        const feetPerSecond = this.mph * (5280 / 3600);
        if (feetPerSecond === 0) return;
        
        const delayMs = (segmentLengthFeet / feetPerSecond) * 1000;
        const secsPerMile = (5280 / segmentLengthFeet) * (delayMs / 1000);
        const secsPerMileFormatted = new Date(secsPerMile * 1000).toISOString().substr(11, 8);
        
        console.log("==============================");
        console.log("Delay (secs):", delayMs / 1000);
        console.log("1 mile in", secsPerMile, "seconds.");
        console.log("Mile pace", secsPerMileFormatted);
        console.log("==============================");
        
        await this.strip.setColor("#000000");
        await sleep(500);
        
        let distance = 5200;
        let i = 1;
        let dir = -1;
        
        let isEdge = (i) => i === numSegments || i === 1;
        
        while (distance >= 0) {
          if (this.stopCycle) break;
          
          if (this.mph < 12 || i % 2) await this.strip.setSegment(i, this.color);
          await sleep(isEdge(i) ? delayMs * 2 : delayMs);
          if (this.mph < 12 || i % 2) await this.strip.setSegment(i, "#000000");
          
          distance -= segmentLengthFeet;
          
          if (isEdge(i)) dir *= -1;
          i += dir;
        }
        
        this.stopCycle = !this.stopCycle;
        await this.strip.setColor("#000000");
      },
      async turnOn() {
        this.isOn = true;
        await this.strip.turnOn();
      },
      async turnOff() {
        this.isOn = false;
        await this.strip.turnOff();
      },
      onColorChange(color) {
        let hexColor = color.toHEXA().toString();
        this.color = hexColor;
        this.picker.setColor(hexColor);
      },
    },
    mounted() {
      this.picker = createPicker(this.onColorChange);
    },
  };
  </script>
  
  <style scoped>
  #heading-collapsed:hover {
    color: #0d6efd;
  }
  video {
    max-width: 200px;
  }
  </style>
  
  