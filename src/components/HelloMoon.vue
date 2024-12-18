<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue'

const WIDTH = 1000
const HEIGHT = 1000
const EARTH_DIAMETER = 17.7
const MOON_DIAMETER = 3.5
const SUN_DIAMETER = 80
const EARTH_MOON_DISTANCE = 384.4
const FACTOR = 2
let angleEarth: number

const panel = ref<HTMLDListElement | null>(null)
const canvas = ref<HTMLCanvasElement | null>(null)
const context = ref<CanvasRenderingContext2D | null>(null)


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

function getAngle(a: Point, b: Point): number {
    return Math.atan2(b.y - a.y, b.x - a.x)
}
function getMouse(ev: MouseEvent) {
    angleEarth = getAngle({ x: 500, y: 500 }, { x: ev.offsetX, y: ev.offsetY })
}


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

    const sun = new Path2D()
    sun.arc(400, 400, SUN_DIAMETER * FACTOR, 0, 2 * Math.PI, true)

    let xEarth = getPointAtAngle({ x: 400, y: 400 }, angleEarth, 500).x
    let yEarth = getPointAtAngle({ x: 400, y: 400 }, angleEarth, 500).y



    const earth = new Path2D()
    earth.arc(xEarth, yEarth, EARTH_DIAMETER * FACTOR, 0, 2 * Math.PI, true)

    let x = getPointAtAngle({ x: xEarth, y: yEarth }, timestamp / 100, 100).x
    let y = getPointAtAngle({ x: xEarth, y: yEarth }, timestamp / 100, 100).y

    const moon = new Path2D()
    moon.arc(x, y, MOON_DIAMETER * FACTOR, 0, 2 * Math.PI, true)


    ctx.fillStyle = 'blue'
    ctx.fill(earth)
    ctx.fillStyle = 'white'
    ctx.fill(moon)

    ctx.fillStyle = 'yellow'
    ctx.fill(sun)

    requestAnimationFrame(render)
}


</script>

<template>
    <div ref="panel"></div>
    <canvas :width="WIDTH" :height="HEIGHT" ref="canvas" @mousemove="getMouse"></canvas>
</template>

<style scoped></style>