<template>
  <div class="colorToggle">
    <div
      class="button"
      v-for="(color, i) in colors"
      :style="{ backgroundColor: color.color }"
      :value="color.name"
      :key="i"
      :id="color.name"
      @click="emitClick($event)"
    ></div>
  </div>
</template>

<style scoped>
.colorToggle {
  position: relative;
  bottom: 0px;
  left: 0px;
  width: 10%;
  height: 100%;
  background: none;
}

.button {
  position: relative;
  width: 50px;
  height: 50px;
  margin: 10px;
  cursor: pointer;
  border-radius: 50%;
  transition: all 0.2s ease-in-out;
  overflow: hidden;
}
</style>


<script>
export default {
  name: "ColorToggle",
  data() {
    return {
      shiftElement: null,
    };
  },
  props: {
    colors: {
      type: Array,
      default: () => [
        {
          name: "orange",
          color: "#ffa500",
        },
        {
          name: "amber",
          color: "#ffbf00",
        },
        {
          name: "lime",
          color: "#aaff00",
        },
        {
          name: "teal",
          color: "#00bfff",
        },
        {
          name: "blue",
          color: "#0000ff",
        },
        {
          name: "indigo",
          color: "#4b0082",
        },
        {
          name: "pink",
          color: "#ff00ff",
        },
      ],
    },
  },
  methods: {
    emitClick(e) {
      this.unshiftBtn();
      this.$emit("clickColor", e.target.style.backgroundColor);
      this.moveToCanvasParentCorner(e.target);
    },
    moveToCanvasParentCorner(element) {
      let canvas = document.getElementById("canvas");

      const parentRect = canvas.getBoundingClientRect();
      const elementRect = element.getBoundingClientRect();
      let xMove = parentRect.right - elementRect.right - 10;
      let yMove = parentRect.top - elementRect.top + 10;
      element.style.transform = `translate(${xMove}px, ${yMove}px)`;
      this.shiftElement = element;
    },

    unshiftBtn() {
      if (this.shiftElement) {
        this.shiftElement.style.transform = "";
      }
    },
  },
};
</script>

<style>
</style>