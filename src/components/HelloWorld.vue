<template>
  <div class="hello">
    <button class="test" @click="send()">TEST</button>
  </div>
</template>

<script>
export default {
  name: "HelloWorld",

  props: {
    msg: String,
  },

  methods: {
    send() {
      console.log("send");
      window.ipc.send("misc", "world");
      window.ipc.on("misc", (event, arg) => {
        console.log(arg);
      });
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
