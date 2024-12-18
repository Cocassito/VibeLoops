<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue'

type Point = { x: number, y: number }
type Event = { point: Point, start: number, duration: number }

const events: Event[] = []

const canvas = ref<HTMLCanvasElement | null>(null)
const context = ref<CanvasRenderingContext2D | null>(null)

let diametre: number = 0
let time: number = 0
let startTime: number | null = null
let moonPos: Point = { x: 200, y: 200 }

onMounted(() => {
  context.value = canvas.value!.getContext('2d')
  requestAnimationFrame(draw)
})

onUnmounted(() => {
  context.value = null
})

function clic(ev: PointerEvent) {
  startTime = time;
  diametre = 0;
  moonPos = { x: ev.offsetX, y: ev.offsetY }
}

function draw(timestamp: number) {
  if (!context.value) return

  if (time < startTime! + 1000) {
    diametre += 1
  } else {
    diametre = 0
  }

  context.value.clearRect(0, 0, 600, 600)

  context.value.fillStyle = 'white'
  const moon = new Path2D;
  moon.arc(moonPos.x, moonPos.y, diametre, 0, 2 * Math.PI, true)
  context.value.stroke(moon)

  time = timestamp
  requestAnimationFrame(draw)
}

</script>

<template>
  <button>bondoir</button>
  <canvas width="600" height="600" ref="canvas" @click="clic"></canvas>
</template>
