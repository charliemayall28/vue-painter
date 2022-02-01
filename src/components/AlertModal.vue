<template>
  <div id="modal" class="modalContainer">
    <div class="modalText">{{ message }}</div>
    <div v-if="button == true">
      <ControlButton
        :color="buttonColor"
        :style="{ height: '50px', width: '70px' }"
        >Dismiss</ControlButton
      >
    </div>
  </div>
</template>

<style scoped>
.modalContainer {
  display: block;
  overflow: hidden;
  position: absolute;
  min-width: 100px;
  max-width: 300px;
  min-height: 50px;
  width: 20%;
  height: 10%;
  top: 50%;
  left: 50%;
  border-radius: 20px;
  transition: 0.3s ease;
}
.modalText {
  position: relative;
  top: 10%;
}
</style>

<script>
import ControlButton from "./ControlButton.vue";
export default {
  name: "AlertModal",
  components: {
    ControlButton,
  },

  props: {
    message: {
      type: String,
      required: true,
    },
    bodyColor: {
      type: String,
      default: "green",
    },
    buttonColor: {
      type: String,
      default: "lightgreen",
    },
    button: {
      type: Boolean,
      default: false,
    },
  },
  mounted() {
    let modal = document.getElementById("modal");
    modal.style.backgroundColor = this.bodyColor;
  },
};
</script>
