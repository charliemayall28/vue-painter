<template>
  <div class="home">
    <!-- Latest compiled and minified CSS -->
    <link
      rel="stylesheet"
      href="https://stackpath.bootstrapcdn.com/bootstrap/3.4.1/css/bootstrap.min.css"
      integrity="sha384-HSMxcRTRxnN+Bdg0JdbxYKrThecOKuH5zCYotlSAcp1+c8xmyTe9GYg1l9a69psu"
      crossorigin="anonymous"
    />
    <SketchPad />
    <!-- <ScrollPanCanvas /> -->
  </div>
</template>

<style scoped>
.home {
  background-color: #5a5858;
  height: 100vh;
  width: 100vw;
  position: absolute;
  top: 0;
  left: 0;
  box-sizing: border-box;
  padding: 0;
  margin: 0;
}
</style>
<script>
// @ is an alias to /src
import SketchPad from "@/components/SketchPad.vue";
import ScrollPanCanvas from "@/components/ScrollPanCanvas.vue";
export default {
  name: "Home",
  components: {
    SketchPad,
    ScrollPanCanvas,
  },
};
</script>
