<template>
  <div class="container">
    <input
      type="range"
      :id="this.id"
      :min="this.min"
      :max="this.max"
      class="slider"
      v-model="sliderValue"
      @input="emitChange($event)"
      :style="{
        width: '100%',
        height: '100%',
        background: this.lineColor,
        color: this.buttonColor,
      }"
    />
    <div class="slider-value" :style="{ backgroundColor: this.lineColor }">
      {{ sliderValue }}
    </div>
  </div>
</template>
<style scoped>
.container {
  position: relative;
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}
.slider-value {
  position: absolute;
  top: 0%;
  bottom: 0%;
  right: 20px;
  max-width: 100%;
  height: 100%;
  text-align: right;
  font-size: 100;
  font-weight: bold;
  color: #fff;
  overflow: hidden;
}
.slider {
  --thumbColor: "blue";
  display: flex;
  -webkit-appearance: none;
  border-radius: 20px;
  width: 100%;
  height: 90%;
  outline: none;
  opacity: 0.7;
  -webkit-transition: 0.2s;
  transition: opacity 0.2s;
}
.slider::-webkit-slider-thumb {
  -webkit-appearance: none;
  background-color: var(--thumbColor);
  opacity: 0.5;
  border-radius: 50%;
  width: 20px;
  height: 20px;
}
.slider::-moz-range-thumb {
  -moz-appearance: none;
  background-color: var(--thumbColor);
  opacity: 0.5;
  border-radius: 50%;
  width: 20px;
  height: 20px;
}
.slider::-moz-focus-outer {
  border: 0;
}
</style>

<script>
export default {
  name: "Slider",
  data() {
    return {
      sliderValue: 1,
    };
  },
  props: {
    id: {
      type: String,
      required: true,
    },
    min: {
      type: Number,
      default: 0,
    },
    max: {
      type: Number,
      default: 100,
    },
    start: {
      type: Number,
      default: 0,
    },
    lineColor: {
      type: String,
      default: "#ff0000",
    },
    buttonColor: {
      type: String,
      default: "white",
    },
    vertical: {
      type: Boolean,
      default: false,
    },
  },
  methods: {
    emitChange(event) {
      this.$emit("sliderChange", [event, this.sliderValue]);
    },
  },
  mounted() {
    this.sliderValue = this.start;
    let slider = document.getElementById(this.id);
    slider.style.background = this.lineColor;
    slider.style.setProperty("--thumbColor", this.buttonColor);
    if (this.vertical == true) {
      let container = document.getElementsByClassName("container")[0];
      container.style.transform = "rotate(90deg)";
      let text = document.getElementsByClassName("sliderContainer");

      text[0].style.transform = "rotate(-90deg)";
    }
  },
};
</script>