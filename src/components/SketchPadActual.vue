<template>
  <div>
    <AlertModal
      :message="this.am.Message"
      :bodyColor="this.am.BodyColor"
      :buttonColor="this.am.ButtonColor"
      :button="this.am.Button"
      v-if="this.am.Visible != false"
    />

    <div class="controlsWrapper">
      <div class="row">
        <div id="controls">
          <ControlButton
            id="'Save'"
            @click="saveImage()"
            color="pink"
            :style="{
              height: '50px',
              width: '60px',
              marginLeft: '10px',
              marginRight: '10px',
            }"
            >Save</ControlButton
          >
          <ControlButton
            id="'Save'"
            @click="submitImage()"
            color="pink"
            :style="{
              height: '50px',
              width: '60px',
              marginLeft: '10px',
              marginRight: '10px',
            }"
            >Submit</ControlButton
          >
          <ControlButton
            id="'clear'"
            @click="clear()"
            color="red"
            :style="{
              height: '50px',
              width: '60px',
              marginLeft: '10px',
              marginRight: '10px',
            }"
            >Clear</ControlButton
          >
          <ControlButton
            id="'undo'"
            @click="undo()"
            color="blue"
            :style="{
              height: '50px',
              width: '60px',
              marginLeft: '10px',
              marginRight: '10px',
            }"
          >
            Undo &lt;&lt;
          </ControlButton>
          <ControlButton
            id="'redo'"
            @click="redo()"
            color="green"
            :style="{
              height: '50px',
              width: '60px',
              marginLeft: '10px',
              marginRight: '10px',
            }"
            >Redo &gt;&gt;</ControlButton
          >
        </div>
        <div id="lineWeight">
          <Slider
            id="hi"
            lineColor="purple"
            @sliderChange="sliderChange($event)"
            :vertical="false"
            :style="{ height: '100%', width: '50%' }"
          />
        </div>

        <div id="brushes">
          <ControlButton
            v-for="(brush, index) in brushes"
            :key="index"
            :id="brush"
            @clickButton="setBrush(brush)"
            :color="brush"
            :style="{
              height: '50px',
              width: '60px',
              marginLeft: '10px',
              marginRight: '10px',
            }"
          >
            {{ brush }}
          </ControlButton>
        </div>
      </div>
    </div>

    <div class="row">
      <div id="canvasContainer">
        <canvas
          id="canvas"
          style="border: 1px solid #d3d3d3; background-color: white"
        >
          >
        </canvas>

        <ColorToggle
          @clickColor="setColor($event)"
          :style="{ right: '50%', width: '5%', float: 'right' }"
        />
      </div>
    </div>
  </div>
</template>

<style>
.controlsWrapper {
  height: 100px;
  width: 100%;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  box-sizing: content-box;
  padding-top: 10px;
}
#brushes {
  display: flex;
  flex-direction: row;
  justify-content: center;
  align-items: center;
  float: right;
}
#controls {
  display: flex;
  flex-direction: row;
  justify-content: right;
  align-items: flex-end;
  float: left;
}

#lineWeight {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  float: right;
  width: 40%;
}
#canvasContainer {
  display: flex;
  flex-direction: row;
  justify-content: center;
  align-items: center;
  max-height: 80vh;
  max-width: 90vw;

  padding-bottom: 10px;
  padding-left: 10px;
}
.spacer {
  display: block;
  background-color: black;
  width: 2px;
  height: 100%;
  margin-left: 5px;
  margin-right: 5px;
}
</style>

<script>
const Atrament = require("atrament");
import ColorToggle from "./colorToggle.vue";
import Slider from "./slider.vue";
import ControlButton from "./ControlButton.vue";
import AlertModal from "./AlertModal.vue";
import axios from "axios";
export default {
  name: "SketchPad",
  components: {
    ColorToggle,
    Slider,
    ControlButton,
    AlertModal,
  },
  data() {
    return {
      pixelArray: [],
      sketchpad: null,
      canvas: null,
      lineWeight: 1,
      prevStrokeLengths: [],
      strokes: [],
      removedStrokes: [],
      brushes: ["draw", "fill", "erase", "disabled"],
      brush: null,
      am: {
        Visible: false,
        Message: "",
        BodyColor: "",
        ButtonColor: "",
        Button: "",
      },
    };
  },
  mounted() {
    let vw = window.innerWidth;
    let vh = window.innerHeight;
    const canvas = document.getElementById("canvas");
    const sketchpad = new Atrament(canvas, {
      width: 0.9 * vw,
      height: 0.85 * vh,
      color: "black",
    });
    sketchpad.recordStrokes = true;
    sketchpad.adaptiveStroke = false;
    sketchpad.addEventListener("strokerecorded", ({ stroke }) => {
      this.recordStroke(stroke);
    });

    this.sketchpad = sketchpad;
    this.canvas = canvas;
    let colorBtn = document.getElementById("orange");
    colorBtn.click();
    this.setBrush("draw");
    // send a post request to /submit on port 5000 with the json '{"brush": this.brush}'
    // this.brush = "draw";
  },
  methods: {
    clear() {
      this.pixelArray = [];
      this.strokes = [];
      this.sketchpad.clear();

      this.showModal("Cleared", "red", "", false);
      // show alert modal
    },
    undo() {
      if (this.strokes.length == 0) {
        return;
      }
      let strokes = this.strokes.slice(0, this.strokes.length - 1);
      let removedStroke = this.strokes[this.strokes.length - 1];
      this.removedStrokes.push(removedStroke);
      this.strokes = [];
      this.pixelArray = this.pixelArray.slice(0, -removedStroke.points.length);
      this.sketchpad.clear();

      let stroke;

      for (let i = 0; i < strokes.length; i++) {
        stroke = strokes[i];
        this.drawStroke(stroke);
      }
    },
    drawStroke(stroke) {
      let point;
      let firstPoint;
      let prevPoint;
      let points = stroke.points;
      this.sketchpad.mode = stroke.mode;
      this.sketchpad.color = stroke.color;
      this.sketchpad.weight = stroke.weight;
      this.sketchpad.smoothing = stroke.smoothing;
      firstPoint = points[0].point;
      this.sketchpad.beginStroke(firstPoint.x, firstPoint.y);
      prevPoint = firstPoint;

      for (let j = 1; j < points.length; j++) {
        point = points[j].point;

        const { x, y } = this.sketchpad.draw(
          point.x,
          point.y,
          prevPoint.x,
          prevPoint.y
        );
        prevPoint = { x, y };
      }

      this.sketchpad.endStroke(prevPoint.x, prevPoint.y);
    },

    redo() {
      if (this.removedStrokes.length == 0) {
        return;
      }
      let stroke = this.removedStrokes.pop();
      this.drawStroke(stroke);
    },
    recordStroke(stroke) {
      this.pixelArray.push(...stroke.points);
      this.strokes.push(stroke);
    },
    setBrush(brush) {
      let currentBrush = this.brush;

      if (currentBrush != null) {
        if (currentBrush.id == brush) {
          this.showModal("This brush is already selected.", "red", "", false);
          return;
        } else {
          currentBrush.style.transform = "";
          currentBrush.style.backgroundColor = "lightblue";
        }
      }
      let btn = document.getElementById(brush);
      let canvas = document.getElementById("canvas");
      let rectCanvas = canvas.getBoundingClientRect();
      let rectBtn = btn.getBoundingClientRect();
      btn.style.transform = `translate(${
        rectCanvas.right - rectBtn.right - 0.05 * rectCanvas.width
      }px, ${rectCanvas.top - rectBtn.top + 10}px)`;
      btn.style.backgroundColor = "lightgreen";
      this.sketchpad.mode = brush;
      this.brush = btn;
    },
    setLineWeight() {
      let lineWidth = parseInt(this.lineWeight);
      this.sketchpad.weight = lineWidth;
    },
    setColor(color) {
      this.sketchpad.color = color;
    },
    sliderChange(event) {
      this.lineWeight = event[1];
      this.setLineWeight();
    },
    showModal(message, bodyColor, buttonColor, button) {
      this.am = {
        Visible: true,
        Message: message,
        BodyColor: bodyColor,
        ButtonColor: buttonColor,
        Button: button,
      };
      setTimeout(() => {
        this.am.Visible = false;
      }, 2000);
    },
    saveImage() {
      let dataURL = this.sketchpad.toImage();
      window.open(dataURL);
      var a = document.createElement("a");
      a.href = dataURL;
      a.download = "my-image.png";
      a.click();
    },
    async submitImage() {
      // post this.strokes to 127.0.0.1:5000/submit as a json object
      axios
        .post("http://127.0.0.1:5000/submit", {
          strokes: this.strokes,
          brush: this.brush,
        })
        .then(function (response) {
          let data = response.data;
          navigator.clipboard.writeText(data.commands);
          window.alert("Gcode copied to clipboard");
          window.open("https://www.ncviewer.com");
          // write the text with line breaks to the new window
        });
    },
    copyGcode(gcode) {
      // copy the gcode to the clipboard

      navigator.clipboard.writeText(gcode);
    },
  },
};
</script>

<style>
</style>