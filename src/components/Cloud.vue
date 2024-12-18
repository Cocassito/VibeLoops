<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue'
import * as Tone from 'tone'

const WIDTH = 1000
const HEIGHT = 1000

const panel = ref<HTMLDListElement | null>(null)
const canvas = ref<HTMLCanvasElement | null>(null)
const context = ref<CanvasRenderingContext2D | null>(null)

const circle = [] as { x: number, y: number, radius: number, velocity: number, color: string }[]
const colorKick = 'red'
const colorDefault = 'blue'

const numCircles = 5
const centerX = 500
const centerY = 500

for (let i = 0; i < numCircles; i++) {
  const radius = i * 10 + 50
  const velocity = i + 5 / 2.2
  const color = colorDefault
  circle.push({ x: centerX, y: centerY, radius, velocity, color })
}

// Configuration Tone.js
let kickPlayer: Tone.Player
let isKicking = false
let hasUserInteracted = false // Flag pour suivre l'interaction utilisateur

onMounted(async () => {
  if (!canvas.value) return
  context.value = canvas.value!.getContext('2d')
  if (!context.value) return
  if (!panel.value) return
  context.value.font = '10px monospace'
  requestAnimationFrame(render)

  // Charger le kick sound avec Tone.js (autostart désactivé)
  kickPlayer = new Tone.Player({
    url: '/src/assets/sound/kick/WAAWF_Kick3_Rev.mp3',
    onload: () => console.log('Kick loaded!'),
  }).toDestination()

  kickPlayer.autostart = false
})

onUnmounted(() => {
  context.value = null
  if (kickPlayer) kickPlayer.dispose()
})

type Point = { x: number; y: number }
function getPointAtAngle(center: Point, angle: number, distance: number): Point {
  return {
    x: center.x + Math.cos(angle) * distance,
    y: center.y + Math.sin(angle) * distance,
  }
}

function kick() {
  if (!hasUserInteracted) {
    // Nécessaire pour démarrer le contexte audio Tone.js après une interaction
    Tone.start()
    hasUserInteracted = true
  }

  const kickIndex = 1
  if (!isKicking) {
    playKick(kickIndex)
  } else {
    stopKick(kickIndex)
  }
}

function playKick(index: number) {
  circle[index].color = colorKick
  Tone.loaded().then(() => {
    kickPlayer.start()
    isKicking = true
  })

  // Définir un comportement pour la fin du son
  kickPlayer.onstop = () => {
    stopKick(index)
    setupAutoRestart()
  }
}

function stopKick(index: number) {
  circle[index].color = colorDefault
  kickPlayer.stop()
  isKicking = false
}

function setupAutoRestart() {
  setTimeout(() => {
    if (!isKicking && hasUserInteracted) {
      playKick(1)
    }
  }, 4000) 
}

function render(timestamp: number) {
  if (!context.value) return

  const ctx = context.value
  ctx.clearRect(0, 0, WIDTH, HEIGHT)

  circle.forEach(c => {
    const angle1 = getPointAtAngle({ x: 500, y: 500 }, timestamp / 100000 * c.velocity, 100).x % (Math.PI * 2)
    ctx.beginPath()
    ctx.arc(c.x, c.y, c.radius, angle1 - Math.PI, angle1)
    ctx.strokeStyle = c.color
    ctx.lineWidth = 2
    ctx.stroke()
  })

  requestAnimationFrame(render)
}
</script>

<template>
  <div ref="panel"></div>
  <button @click="kick">Kick</button>
  <canvas :width="WIDTH" :height="HEIGHT" ref="canvas"></canvas>
</template>

<style scoped></style>
