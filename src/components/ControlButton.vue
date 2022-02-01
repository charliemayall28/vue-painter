<template>
  <button :id="this.id" @click="emitClick($event)" class="ctrlButton neo">
    <slot></slot>
  </button>
</template>

<style scoped>
.ctrlButton {
  width: 80px;
  height: 50px;
  background-color: lightblue;
  border-radius: 10px;
  border: none;
  transition: all 0.2s ease;
}
.ctrlButton:hover {
  transform: scale(1.05);
}
.neo {
  border-radius: 55px;
  background: #5a5858;
  box-shadow: 7px 7px 14px #444343, -7px -7px 19px #706d6d;
}
</style>

<script>
export default {
  name: "ControlButton",
  props: {
    id: {
      type: String,
      required: true,
    },

    class: {
      type: String,
      default: "ctrlButton",
    },
  },
  methods: {
    emitClick(event) {
      this.$emit("clickButton", event);
    },
  },
};
</script>
