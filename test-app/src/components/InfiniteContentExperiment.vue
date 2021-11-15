<template>
  <div class="scroll-area">
    <div class="bottomWrapper">
      <div class="bottomSensorWrapper">
        <div class="bottomLoad" :ref="el => sensors.bottomLoad.element.value = el"></div>
        <div class="bottomDrop" :ref="el => sensors.bottomDrop.element.value = el"></div>
      </div>
      <div class="bottomEmpty" :style="{ 'min-height': bottomEmptyHeight + 'px' }"
           :ref="el => sensors.bottomEmpty.element.value = el"></div>
    </div>
    <div v-for="block in blocks" :key="block.id" :ref="el => block.element = el"
         class="block" :style="{ background: `hsl(${block.hue}, 100%, 50%)`, 'min-height': block.height + 'px' }">
      {{ block.id }}
    </div>
    <div class="topWrapper">
      <div class="topEmpty" :style="{ 'min-height': topEmptyHeight + 'px' }"
           :ref="el => sensors.topEmpty.element.value = el"></div>
      <div class="topSensorWrapper">
        <div class="topLoad" :ref="el => sensors.topLoad.element.value = el"></div>
        <div class="topDrop" :ref="el => sensors.topDrop.element.value = el"></div>
      </div>
    </div>

  </div>
</template>

<script setup>

import { useIntersectionObserver } from '@vueuse/core'
import { reactive, ref, nextTick, computed, watch, onMounted } from "vue"

let lastId = 0
let blocks = reactive(new Array(10).fill(1).map(x => ({
  id: ++lastId,
  height: Math.random() * 50 + 25,
  hue: Math.random() * 360
})))
const blocksCount = computed(() => blocks.length)

let topEmptyPlaces = reactive([])
let bottomEmptyPlaces = reactive([])
const topEmptyHeight = computed(() => topEmptyPlaces.reduce((a, b) => a + b, 0))
const bottomEmptyHeight = computed(() => bottomEmptyPlaces.reduce((a, b) => a + b, 0))

const sensorNames = ['topLoad', 'bottomLoad', 'topDrop', 'bottomDrop', 'topEmpty', 'bottomEmpty']
const sensors = {}
for(const sensorName of sensorNames) {
  const sensor = {
    element: ref(),
    visible: ref(true)
  }
  useIntersectionObserver(sensor.element, ([{ isIntersecting }], observerElement) =>
      sensor.visible.value = isIntersecting)
  sensors[sensorName] = sensor
}

const shouldDropTop = () => blocks.length > 0 && !sensors.topDrop.visible.value && !sensors.topEmpty.visible.value
const shouldDropBottom = () => blocks.length > 0 && !sensors.bottomDrop.visible.value && !sensors.bottomEmpty.visible.value
const shouldLoadTop = () => sensors.topLoad.visible.value || sensors.topEmpty.visible.value
const shouldLoadBottom = () => sensors.bottomLoad.visible.value || sensors.bottomEmpty.visible.value

let topTimeout = null
let bottomTimeout = null

function loadTop() {
  if(topTimeout) clearTimeout(topTimeout)
  if(!shouldLoadTop()) return
  blocks.push({
    id: ++lastId,
    height: Math.random() * 50 + 25,
    hue: Math.random() * 360
  })
  topEmptyPlaces.pop()
  topTimeout = setTimeout(() => loadTop(), 0)
}

function loadBottom() {
  if(bottomTimeout) clearTimeout(bottomTimeout)
  if(!shouldLoadBottom()) return
  blocks.unshift({
    id: ++lastId,
    height: Math.random() * 50 + 25,
    hue: Math.random() * 360
  })
  bottomEmptyPlaces.pop()
  bottomTimeout = setTimeout(() => loadBottom(), 1000)
}

function dropTop() {
  if(topTimeout) clearTimeout(topTimeout)
  if(!shouldDropTop()) return
  const dropped = blocks.pop()
  topEmptyPlaces.push(dropped.element?.offsetHeight || 0)
  topTimeout = setTimeout(() => dropTop(), 100)
}

function dropBottom() {
  if(bottomTimeout) clearTimeout(bottomTimeout)
  if(!shouldDropBottom()) return
  const dropped = blocks.shift()
  bottomEmptyPlaces.push(dropped.element?.offsetHeight || 0)
  bottomTimeout = setTimeout(() => dropBottom(), 100)
}

onMounted(() => {
  watch(() => shouldLoadTop(), (should) => should && loadTop())
  watch(() => shouldLoadBottom(), (should) => should && loadBottom())
  watch(() => shouldDropTop(), (should) => should && dropTop())
  watch(() => shouldDropBottom(), (should) => should && dropBottom())

  loadTop()
  loadBottom()
})

</script>

<style scoped>
  .topWrapper, .bottomWrapper {
    
  }
  
  .topSensorWrapper, .bottomSensorWrapper {
    position: relative;
    height: 0;
  }
  .topLoad {
    position: absolute;
    top: 0;
    height: 1000px;
    visibility: hidden;
  }
  .topDrop {
    position: absolute;
    top: 0;
    height: 1500px;
    visibility: hidden;
  }
  .bottomLoad {
    position: absolute;
    bottom: 0;
    height: 1000px;
    visibility: hidden;
  }
  .bottomDrop {
    position: absolute;
    bottom: 0;
    height: 1500px;
    visibility: hidden;
  }
</style>
