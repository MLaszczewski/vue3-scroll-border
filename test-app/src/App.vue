<template>
  <InfiniteContent />
  <div class="scroll-area">

  </div>
</template>

<script setup>

import UnstableContentExperiment from "./components/UnstableContentExperiment.vue"
import InfiniteContentExperiment from "./components/InfiniteContentExperiment.vue"
import InfiniteContent from "./components/InfiniteContent.vue"



</script>

<style lang="scss">

html, body {
  position: absolute;
  left: 0;
  top: 0;
  width: 100%;
  height: 100%;
  padding: 0;
  margin: 0;
}

#app {
  position: absolute;
  left: 0;
  top: 0;
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: row;
}

.stableContent {
  background: white;
}

.scroll-area {
  height: 100%;
  flex: 1;
  overflow-y: auto;
  box-sizing: border-box;
  border: 1px solid black;
  display: flex;
  flex-direction: column-reverse;
}

.scroll-border-placement-bottom {
  .drop-sensor {
    left: 10px;
    width: 10px;
    background: red;
    z-index: 100;
    visibility: visible !important;
  }
  .load-sensor {
    left: 30px;
    width: 10px;
    background: green;
    z-index: 100;
    visibility: visible !important;
  }
}

.scroll-border-placement-top {
  .drop-sensor {
    right: 10px;
    width: 10px;
    background: red;
    z-index: 100;
    visibility: visible !important;
  }
  .load-sensor {
    right: 30px;
    width: 10px;
    background: green;
    z-index: 100;
    visibility: visible !important;
  }
}

</style>
