<template>
  <div :class="['scroll-border', `scroll-border-placement-${placement}`]">
    <div class="scroll-sensors">
      <div class="load-sensor" :ref="el => sensors.load.element.value = el"
           :style="sensorStyle(placement, loadSensorSize)"></div>
      <div class="drop-sensor" :ref="el => sensors.drop.element.value = el"
           :style="sensorStyle(placement, dropSensorSize)"></div>
    </div>
    <div class="empty-space" :style="{ 'min-height': emptyHeight + 'px' }"
         :ref="el => sensors.empty.element.value = el"></div>
  </div>
</template>

<script setup>

const { load, drop, canLoad, canDrop, loadDelay, dropDelay, placement, loadSensorSize, dropSensorSize } = defineProps({
  load: {
    type: Function,
    default: null
  },
  drop: {
    type: Function,
    default: null
  },
  canLoad: {
    type: Function,
    default: () => true
  },
  canDrop: {
    type: Function,
    default: () => true
  },
  loadDelay: {
    type: Number,
    default: 0
  },
  dropDelay: {
    type: Number,
    default: 0
  },
  placement: {
    type: String,
    required: true,
    validator: (value) => ['top', 'bottom', 'left', 'right'].includes(value)
  },
  loadSensorSize: {
    type: String,
    default: '500px'
  },
  dropSensorSize: {
    type: String,
    default: '1000px'
  }
})

import { useIntersectionObserver } from '@vueuse/core'
import { reactive, ref, nextTick, computed, watch, onMounted, onUnmounted, unref } from "vue"

const emptyPlaces = reactive([])
const emptyHeight = computed(() => emptyPlaces.reduce((a, b) => a + b, 0))

const sensorNames = ['load', 'drop', 'empty']

const sensors = {}
for(const sensorName of sensorNames) {
  const sensor = {
    element: ref(),
    visible: ref(true)
  }
  const { stop } = useIntersectionObserver(sensor.element, ([{ isIntersecting }], observerElement) =>
    sensor.visible.value = isIntersecting)
  sensor.stop = stop
  sensors[sensorName] = sensor
}

const shouldDrop = () => !sensors.drop.visible.value && !sensors.empty.visible.value
const shouldLoad = () => sensors.load.visible.value || sensors.empty.visible.value

const lld = computed(() => canLoad() && shouldLoad())
const sl = computed(() => shouldLoad())
const cl = computed(() => canLoad())

let timeout = null
let loading = false

async function loadLoop() {
  const loadable = canLoad() && load
  if(!(shouldLoad() && (loadable || emptyPlaces.length) && !loading)) return
  if(timeout) clearTimeout(timeout)
  if(loadable) {
    loading = true
    await load()
    loading = false
  }
  emptyPlaces.pop()
  timeout = setTimeout(() => loadLoop(), loadDelay)
}

function dropLoop() {
  const dropable = canDrop() && drop
  if(!(shouldDrop() && dropable)) return
  if(timeout) clearTimeout(timeout)
  const height = drop()
  emptyPlaces.push(height || 0)
  timeout = setTimeout(() => dropLoop(), dropDelay)
}

onMounted(() => {
  if(load) {
    watch(() => (shouldLoad() && canLoad()), (should) => {
      //console.log("SHOULD LOAD", should)
      if(should) loadLoop()
    })
  }
  if(drop) {
    watch(() => shouldDrop(), (should) => {
      if (should) dropLoop()
    })
  }
  if(load) {
    loadLoop()
  }
})

onUnmounted(() => {
  for(const sensorName of sensorNames) {
    sensors[sensorName].stop()
  }
})

function sensorStyle(placement, size) {
  switch(placement) {
    case 'top': return { top: '0', height: size }
    case 'bottom': return { bottom: '0', height: size }
    case 'left': return { left: '0', width: size }
    case 'right': return { right: '0', width: size }
  }
  throw new Error("unknown scroll-border placement " + placement)
}

</script>

<style scoped>
.scroll-sensors {
  position: relative;
  height: 0;
}
.load-sensor, .drop-sensor {
  position: absolute;
  visibility: hidden;
}
.scroll-border {
  display: flex;
}
.scroll-border-placement-top {
  flex-direction: column-reverse;
}
.scroll-border-placement-bottom {
  flex-direction: column;
}
.scroll-border-placement-left {
  flex-direction: row-reverse;
}
.scroll-border-placement-right {
  flex-direction: row;
}

</style>
