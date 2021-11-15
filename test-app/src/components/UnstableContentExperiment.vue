<template>
  <div class="scroll-area">
    <div v-for="block in blocks" :key="block.id"
         class="block" :style="{ background: `hsl(${block.hue}, 100%, 50%)`, 'min-height': block.height + 'px' }">
      {{ block.id }}
    </div>
    <div class="stableContent">
      <p v-for="row in (new Array(23)).fill(1).map((v,i) => i)">=== {{ row }} ===</p>
    </div>
    <div v-for="block in blocks" :key="block.id"
         class="block" :style="{ background: `hsl(${block.hue}, 100%, 50%)`, 'min-height': block.height + 'px' }">
      {{ block.id }}
    </div>
  </div>
</template>

<script setup>
import { useIntervalFn } from '@vueuse/core'
import { reactive } from "vue"

let lastId = 0
let blocks = reactive(new Array(100).fill(1).map(x => ({
  id: ++lastId,
  height: Math.random() * 50 + 25,
  hue: Math.random() * 360
})))


const { pause, resume, isActive } = useIntervalFn(() => {
  if(blocks.length > 100 && Math.random() < 0.01) {
    blocks.splice((Math.random() * blocks.length)|0, 1)
  }
  if(blocks.length > 100 && Math.random() < 0.01) {
    blocks.shift()
  }
  if(blocks.length > 100 && Math.random() < 0.01) {
    blocks.pop()
  }
  for(let i = 0; i < 3; i++) {
    blocks[(Math.random() * blocks.length)|0].height = Math.random() * 50 + 25
  }
  if(blocks.length < 150 && Math.random() < 0.015) {
    blocks.push({
      id: ++lastId,
      height: Math.random() * 50 + 25,
      hue: Math.random() * 360
    })
  }
  if(blocks.length < 150 && Math.random() < 0.015) {
    blocks.unshift({
      id: ++lastId,
      height: Math.random() * 50 + 25,
      hue: Math.random() * 360
    })
  }
}, 100)

</script>

<style scoped>
</style>
