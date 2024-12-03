<template>
  <div id="d3-animation" style="height: 100%"></div>
</template>
<script setup lang="ts">
import * as d3 from 'd3'
import {onMounted} from "vue";
import {MarginType} from "../interface";

viewof index = Scrubber(d3.range(data.keys.length), {
  delay: 2500,
  loop: false,
  format: i => data.keys[i]
})

const d3AnimationTreeMap = () => {
  const width:number = 500;
  const height:number = 500;
  const margin:MarginType = {top: 20, right: 30, bottom: 30, left: 40};
}



onMounted(() => {

})
</script>

<style scoped>

</style>