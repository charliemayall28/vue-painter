<template>
  <div class="wrapper">
    <link
      rel="stylesheet"
      href="https://stackpath.bootstrapcdn.com/bootstrap/3.4.1/css/bootstrap.min.css"
      integrity="sha384-HSMxcRTRxnN+Bdg0JdbxYKrThecOKuH5zCYotlSAcp1+c8xmyTe9GYg1l9a69psu"
      crossorigin="anonymous"
    />
    <div class="col-sm-12">
      <!-- make 2 columns-->

      <div class="col-sm-6">
        <div
          class="col-sm-12"
          style="display: flex; justify-content: space-evenly"
        >
          <ControlButton
            id="'clear'"
            @click="clear()"
            :style="{
              height: '50px',
              width: '95px',
              marginLeft: '10px',
              marginRight: '10px',
              backgroundColor: '#e74c3c',
            }"
            >Clear</ControlButton
          >
          <ControlButton
            id="'undo'"
            @click="undo()"
            :style="{
              height: '50px',
              width: '95px',
              marginLeft: '10px',
              marginRight: '10px',
              backgroundColor: '#fa6c1e',
            }"
          >
            Undo
          </ControlButton>
          <ControlButton
            id="'redo'"
            @click="redo()"
            :style="{
              height: '50px',
              width: '95px',
              marginLeft: '10px',
              marginRight: '10px',
              backgroundColor: '#fa6c1e',
            }"
            >Redo</ControlButton
          >
          <ControlButton
            id="'Save'"
            @click="saveImage()"
            :style="{
              height: '50px',
              width: '95px',
              marginLeft: '10px',
              marginRight: '10px',
              backgroundColor: '#774fa3',
            }"
            >Save</ControlButton
          >
          <ControlButton
            id="'Submit'"
            @click="submitImage()"
            :style="{
              height: '50px',
              width: '95px',
              marginLeft: '10px',
              marginRight: '10px',
              backgroundColor: '#bcb4d0',
            }"
            >Submit</ControlButton
          >
        </div>
      </div>
      <div class="col-sm-6">
        <div class="col-sm-12 left" style="margin-bottom: 10px">
          <ControlButton
            v-for="(brush, index) in brushes"
            :key="index"
            :id="brush"
            @clickButton="setBrush(brush)"
            :color="brush"
            :style="this.brushBtnStyle"
          >
            {{ brush }}
          </ControlButton>
        </div>
        <div class="col-sm-12">
          <Slider
            id="hi"
            lineColor="purple"
            @sliderChange="sliderChange($event)"
            :vertical="false"
            :style="{ height: '100%', width: '100%' }"
          />
        </div>
      </div>
    </div>
    <div>
      <div class="col-sm-12 left">
        <canvas
          id="canvas"
          style="border: 1px solid #d3d3d3; background-color: white"
        >
          >
        </canvas>
        <ColorToggle
          :colors="this.colorOptions"
          @clickColor="setColor($event)"
        />
      </div>
    </div>
  </div>
</template>

<script>
const Atrament = require("atrament");
import ColorToggle from "./colorToggle.vue";
import Slider from "./slider.vue";
import ControlButton from "./ControlButton.vue";
import axios from "axios";
export default {
  name: "SketchPad",
  components: {
    ControlButton,
    Slider,
    ColorToggle,
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
      brushBtnStyle: {
        height: "50px",
        width: "25%",
        color: "white",
        marginLeft: "10px",
        marginRight: "10px",
        zIndex: "999",
        backgroundColor: "#5a5858",
      },
      colorOptions: [
        {
          name: "green",
          color: "#62bb47",
        },
        {
          name: "red",
          color: "#e03a3c",
        },
        {
          name: "blue",
          color: "#009ddc",
        },
        {
          name: "yellow",
          color: "#fcb827",
        },
        {
          name: "orange",
          color: "#f6821f",
        },
        {
          name: "purple",
          color: "#963d97",
        },
        {
          name: "grey",
          color: "#999999",
        },
        {
          name: "black",
          color: "#000000",
        },
        {
          name: "white",
          color: "#ffffff",
        },
      ],
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
          currentBrush.style.backgroundColor =
            this.brushBtnStyle.backgroundColor;
          currentBrush.style.color = this.brushBtnStyle.color;
        }
      }
      let btn = document.getElementById(brush);
      btn.style.backgroundColor = "lightgreen";
      btn.style.color = "black";
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

    saveImage() {
      let dataURL = this.sketchpad.toImage();
      window.open(dataURL);
      var a = document.createElement("a");
      a.href = dataURL;
      a.download = "my-image.png";
      a.click();
    },
    async submitImage() {
      async function _makeGcode(self) {
        let res = await axios.post("http://127.0.0.1:5000/submit", {
          strokes: self.strokes,
          brush: self.brush,
        });
        let gcode = res.data.commands;
        return gcode;
      }
      let gcode = await _makeGcode(this);
      await navigator.clipboard.writeText(gcode);
      window.open("http://ncviewer.com/");
    },
  },
};
</script>

<style>
.wrapper {
  background-color: #5a5858;
  height: 100vh;
  width: 100vw;
  position: absolute;
  top: 0;
  left: 0;
  box-sizing: border-box;
  padding: 0;
  margin: 0;
  padding-top: 1vh;
}
.row {
  display: flex;
  padding: 0;
  margin: 0;
  width: auto;
  flex-direction: column;
  /* vertically center children */
  align-items: center;
  justify-content: space-evenly;

  /* justify-content: center;
  align-items: center; */
}
.right {
  display: flex;
  /* align to the right */
  justify-content: flex-end;
}
.left {
  display: flex;
  justify-content: flex-start;
}

.even {
  display: flex;
  justify-content: space-evenly;
}
</style>