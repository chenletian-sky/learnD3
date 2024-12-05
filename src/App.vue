<script setup lang="ts">
import HelloWorld from './components/HelloWorld.vue'
import d3Animation from './components/d3Animation/index.vue'
import d3Interaction from './components/d3Interaction/index.vue'

</script>

<template>
  <div id="d3-root" style="height: 100%"></div>
<!--  <HelloWorld/>-->
<!--  <d3Animation/>-->
  <d3Interaction/>
</template>

<style scoped>
.logo {
  height: 6em;
  padding: 1.5em;
  will-change: filter;
  transition: filter 300ms;
}
.logo:hover {
  filter: drop-shadow(0 0 2em #646cffaa);
}
.logo.vue:hover {
  filter: drop-shadow(0 0 2em #42b883aa);
}
#app{
  height: 100vh;
}
</style>
<style>
#app{
  height: 100vh;
}
</style>
