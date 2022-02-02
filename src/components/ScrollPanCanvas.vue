<template>
  <canvas id="canvas"> </canvas>
</template>

<script>
export default {
  name: "ScrollPanCanvas",
  props: {
    parentDisable: {
      type: Boolean,
      default: false,
    },
  },
  data() {
    return {
      canvas: null,
      cameraOffset: null,
      cameraZoom: null,
      MAX_ZOOM: 5,
      MIN_ZOOM: 1,
      SCROLL_SENSITIVITY: 0.0005,
      isDragging: false,
      initialPinchDistance: null,
      lastZoom: null,
      dragStart: { x: 0, y: 0 },
      initialPinchDistance: null,
      lastZoom: null,
      firstDraw: true,
    };
  },
  methods: {
    init() {
      let canvas = document.getElementById("canvas");
      this.ctx = canvas.getContext("2d");

      this.cameraOffset = {
        x: window.innerWidth / 2,
        y: window.innerHeight / 2,
      };
      this.cameraZoom = 1;
      this.MAX_ZOOM = 5;
      this.MIN_ZOOM = 0.1;
      this.SCROLL_SENSITIVITY = 0.0005;
      canvas.addEventListener("mousedown", this.onPointerDown);
      canvas.addEventListener("touchstart", (e) =>
        this.handleTouch(e, this.onPointerDown)
      );
      canvas.addEventListener("mouseup", this.onPointerUp);
      canvas.addEventListener("touchend", (e) =>
        this.handleTouch(e, this.onPointerUp)
      );
      canvas.addEventListener("mousemove", this.onPointerMove);
      canvas.addEventListener("touchmove", (e) =>
        this.handleTouch(e, this.onPointerMove)
      );
      canvas.addEventListener("wheel", (e) =>
        this.adjustZoom(e.deltaY * this.SCROLL_SENSITIVITY)
      );
      this.canvas = canvas;
    },
    draw() {
      let canvas = document.getElementById("canvas");
      // console.log(this.canvas);
      // console.log(canvas);
      if (this.canvas.value == "disabled" && !this.firstDraw) {
        return;
      }
      this.canvas.width = window.innerWidth;
      this.canvas.height = window.innerHeight;

      // Translate to the canvas centre before zooming - so you'll always zoom on what you're looking directly at
      this.ctx.translate(window.innerWidth / 2, window.innerHeight / 2);
      this.ctx.scale(this.cameraZoom, this.cameraZoom);
      this.ctx.translate(
        -window.innerWidth / 2 + this.cameraOffset.x,
        -window.innerHeight / 2 + this.cameraOffset.y
      );
      this.ctx.clearRect(0, 0, window.innerWidth, window.innerHeight);
      this.ctx.fillStyle = "#991111";
      this.drawRect(-50, -50, 100, 100);

      this.ctx.fillStyle = "#eecc77";
      this.drawRect(-35, -35, 20, 20);
      this.drawRect(15, -35, 20, 20);
      this.drawRect(-35, 15, 70, 20);

      this.ctx.fillStyle = "#fff";
      this.drawText("Simple Pan and Zoom Canvas", -255, -100, 32, "courier");

      this.ctx.rotate((-31 * Math.PI) / 180);
      this.ctx.fillStyle = `#${(Math.round(Date.now() / 40) % 4096).toString(
        16
      )}`;
      this.drawText("Now with touch!", -110, 100, 32, "courier");

      this.ctx.fillStyle = "#fff";
      this.ctx.rotate((31 * Math.PI) / 180);

      this.drawText("Wow, you found me!", -260, -2000, 48, "courier");

      requestAnimationFrame(this.draw);
      this.firstDraw = false;
    },
    getEventLocation(e) {
      if (e.touches && e.touches.length == 1) {
        return { x: e.touches[0].clientX, y: e.touches[0].clientY };
      } else if (e.clientX && e.clientY) {
        return { x: e.clientX, y: e.clientY };
      }
    },
    drawRect(x, y, width, height) {
      this.ctx.fillRect(x, y, width, height);
    },

    drawText(text, x, y, size, font) {
      this.ctx.font = `${size}px ${font}`;
      this.ctx.fillText(text, x, y);
    },

    onPointerDown(e) {
      this.isDragging = true;
      this.dragStart.x =
        this.getEventLocation(e).x / this.cameraZoom - this.cameraOffset.x;
      this.dragStart.y =
        this.getEventLocation(e).y / this.cameraZoom - this.cameraOffset.y;
    },

    onPointerUp(e) {
      this.isDragging = false;
      this.initialPinchDistance = null;
      this.lastZoom = this.cameraZoom;
    },

    onPointerMove(e) {
      if (this.isDragging) {
        this.cameraOffset.x =
          this.getEventLocation(e).x / this.cameraZoom - this.dragStart.x;
        this.cameraOffset.y =
          this.getEventLocation(e).y / this.cameraZoom - this.dragStart.y;
      }
    },

    handleTouch(e, singleTouchHandler) {
      if (e.touches.length == 1) {
        this.singleTouchHandler(e);
      } else if (e.type == "touchmove" && e.touches.length == 2) {
        this.isDragging = false;
        this.handlePinch(e);
      }
    },
    handlePinch(e) {
      e.preventDefault();

      let touch1 = { x: e.touches[0].clientX, y: e.touches[0].clientY };
      let touch2 = { x: e.touches[1].clientX, y: e.touches[1].clientY };

      // This is distance squared, but no need for an expensive sqrt as it's only used in ratio
      let currentDistance =
        (touch1.x - touch2.x) ** 2 + (touch1.y - touch2.y) ** 2;

      if (initialPinchDistance == null) {
        this.initialPinchDistance = currentDistance;
      } else {
        adjustZoom(null, currentDistance / this.initialPinchDistance);
      }
    },

    adjustZoom(zoomAmount, zoomFactor) {
      if (!this.isDragging) {
        if (zoomAmount) {
          this.cameraZoom += zoomAmount;
        } else if (zoomFactor) {
          console.log(zoomFactor);
          this.cameraZoom = zoomFactor * this.lastZoom;
        }

        this.cameraZoom = Math.min(this.cameraZoom, this.MAX_ZOOM);
        this.cameraZoom = Math.max(this.cameraZoom, this.MIN_ZOOM);

        console.log(zoomAmount);
      }
    },
    getEventLocation(e) {
      if (e.touches && e.touches.length == 1) {
        return { x: e.touches[0].clientX, y: e.touches[0].clientY };
      } else if (e.clientX && e.clientY) {
        return { x: e.clientX, y: e.clientY };
      }
    },
  },
  mounted() {
    setTimeout(() => {
      this.init();
      this.draw();
    }, 1000);

    // this.draw();
  },
};
</script>

<style>
html,
body {
  height: 100%;
  margin: 0;
  padding: 0px;
  overflow: hidden;
}

#canvas {
  width: 100%;
  height: 100%;
  background: #111;
}
</style>


