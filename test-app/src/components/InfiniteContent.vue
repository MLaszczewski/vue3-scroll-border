<template>
  <div :class="['scroll-area', reverse ? 'scroll-reverse' : 'scroll-normal']">
    <scroll-border :load="loadBottom" :drop="dropTop" :loadDelay="1000" :placement="reverse ? 'bottom' : 'top'" />
    <div v-for="block in blocks" :key="block.id" :ref="el => block.element = el"
         class="block" :style="{ background: `hsl(${block.hue}, 100%, 50%)`, 'min-height': block.height + 'px' }">
      {{ block.id }}
    </div>
    <scroll-border :load="loadTop" :drop="dropTop" :placement="reverse ? 'top' : 'bottom'" />
  </div>
</template>

<script setup>
import ScrollBorder from "../../../lib/ScrollBorder.vue"
import { reactive, computed } from "vue"

const { reverse } = defineProps({
  reverse: {
    type: Boolean,
    required: true
  }
})

let lastId = 0
let blocks = reactive(new Array(10).fill(1).map(x => ({
  id: ++lastId,
  height: Math.random() * 50 + 25,
  hue: Math.random() * 360
})))
const blocksCount = computed(() => blocks.length)

function loadBottom() {
  blocks.unshift({
    id: ++lastId,
    height: Math.random() * 50 + 25,
    hue: Math.random() * 360
  })
}
function loadTop() {
  blocks.push({
    id: ++lastId,
    height: Math.random() * 50 + 25,
    hue: Math.random() * 360
  })
}
function dropBottom() {
  const dropped = blocks.pop()
  return dropped.element?.offsetHeight
}
function dropTop() {
  const dropped = blocks.shift()
  return dropped.element?.offsetHeight
}

</script>

<style scoped>
</style>
