<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue'

type Point = { x: number, y: number }
type Event = { point: Point, time: number, duration: number }

const WIDTH = 500
const HEIGHT = 500

const panel = ref<HTMLDListElement | null>(null)
const canvas = ref<HTMLCanvasElement | null>(null)
const context = ref<CanvasRenderingContext2D | null>(null)
const mouse: Point = { x: 0, y: 0 }

let time = 0


onMounted(() => {
  if (!canvas.value) return
  context.value = canvas.value!.getContext('2d')
  if (!context.value) return
  if (!panel.value) return
  canvas.value.addEventListener('mousemove', (e) => {
    mouse.x = e.offsetX
    mouse.y = e.offsetY
  })
  context.value.font = '10px monospace'
  requestAnimationFrame(render)
})

onUnmounted(() => {
  context.value = null
})

function easeOutSine(x: number): number {
  return Math.sin((x * Math.PI) / 2);
}

const events: Event[] = []

function render(timestamp: number) {
  if (!context.value) return
  const ctx = context.value
  time = timestamp
  ctx.clearRect(0, 0, WIDTH, HEIGHT)

  events.forEach((event, index) => {
    const progress = (time - event.time) / event.duration
    if (progress > 1) {
      events.splice(index, 1)
      return
    }
    const circle = new Path2D()
    const radius = easeOutSine(progress) * 50
    circle.arc(event.point.x, event.point.y, radius, 0, 2 * Math.PI, true)
    ctx.strokeStyle = 'blue'
    ctx.stroke(circle)
  })
  requestAnimationFrame(render)
}

function create() {
  events.push({ point: { x: mouse.x, y: mouse.y }, time, duration: 2000 })
}

</script>

<template>
  <div ref="panel"></div>
  <canvas :width="WIDTH" :height="HEIGHT" ref="canvas" @click="create" ></canvas>
</template>

<style scoped>
canvas {
  cursor: pointer;
}
</style>