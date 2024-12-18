<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue'

const WIDTH = 1000
const HEIGHT = 1000

const panel = ref<HTMLDListElement | null>(null)
const canvas = ref<HTMLCanvasElement | null>(null)
const context = ref<CanvasRenderingContext2D | null>(null)

const circle = [] as { x: number, y: number, radius: number, velocity:number }[]

const numCircles = 20
const centerX = 500
const centerY = 500

for (let i = 0; i < numCircles; i++) {
    const radius = i * 10 + 50
    const velocity = i /2.1
    circle.push({ x: centerX, y: centerY, radius, velocity })
}

onMounted(() => {
    if (!canvas.value) return
    context.value = canvas.value!.getContext('2d')
    if (!context.value) return
    if (!panel.value) return
    context.value.font = '10px monospace'
    requestAnimationFrame(render)
})

onUnmounted(() => {
    context.value = null
})

type Point = { x: number; y: number }
function getPointAtAngle(
  center: Point,
  angle: number,
  distance: number
): Point {
  return {
    x: center.x + Math.cos(angle) * distance,
    y: center.y + Math.sin(angle) * distance,
  }
}


function render(timestamp: number) {
    if (!context.value) return

    const ctx = context.value
    ctx.clearRect(0, 0, WIDTH, HEIGHT)

    let angle2 = getPointAtAngle({ x: 500, y: 500 }, timestamp / -120000, 200).x + 4% (Math.PI * 2)
    // let y = getPointAtAngle({ x: 500, y: 500 }, timestamp / 100, 100).y
    
    circle.forEach(c => {
        let angle1 = getPointAtAngle({ x: 500, y: 500 }, timestamp / 100000 * c.velocity, 200).x % (Math.PI * 2)
        ctx.beginPath()
        ctx.arc(c.x, c.y, c.radius, angle1 -Math.PI , angle1  )
        ctx.strokeStyle = 'blue'
        ctx.lineWidth = 2
        ctx.stroke()
    })

    

    requestAnimationFrame(render)
}
</script>

<template>
    <div ref="panel"></div>
    <button>oui</button>
    <canvas :width="WIDTH" :height="HEIGHT" ref="canvas"></canvas>
</template>

<style scoped></style>
