<script setup lang="ts">
import { onMounted, onUnmounted, ref } from 'vue'
import * as Tone from 'tone'

const WIDTH = 1000
const HEIGHT = 1000

const panel = ref<HTMLDListElement | null>(null)
const canvas = ref<HTMLCanvasElement | null>(null)
const context = ref<CanvasRenderingContext2D | null>(null)
const circle = [] as { x: number, y: number, radius: number, velocity: number, color: string }[]
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

onMounted(async () => {
    if (!canvas.value) return
    context.value = canvas.value!.getContext('2d')
    if (!context.value) return
    if (!panel.value) return
    context.value.font = '10px monospace'
    requestAnimationFrame(render)
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


let playKick = ref(false)
let playSnare = ref(false)
let playHiHat = ref(false)
let playMelody = ref(false)
let isPlaying = ref(false)
let currentLoop: Tone.Loop | null = null;
let hiHatMode = ref<'4' | '2'>('4')

const transport = Tone.getTransport();
const bassGain = new Tone.Gain(1.1).toDestination();

transport.bpm.value = 83;

const kickPlayer = new Tone.Player({ url: '/src/assets/sound/kick/kick.wav' }).toDestination();
const bassPlayer = new Tone.Player({ url: '/src/assets/sound/bass/bass.wav' }).connect(bassGain);
const hiHatClosed = new Tone.Player({ url: '/src/assets/sound/hihat/hihat2.wav' }).toDestination();
const melody1 = new Tone.Player({ url: '/src/assets/sound/melody/melody83.wav', loop: true }).toDestination();

function play() {
    if (transport.state === 'started') {
        console.log("La boucle est déjà en cours !");
        return;
    }

    isPlaying.value = true;
    let i = 0;

    currentLoop = new Tone.Loop((time) => {
        if (playKick.value && i % 8 === 0) kickPlayer.start(time);
        if (playSnare.value && i % 4 === 0) bassPlayer.start(time);

        if (playHiHat.value) {
            if (hiHatMode.value === '4' && i % 4 === 0) {
                hiHatClosed.start(time);
            } else if (hiHatMode.value === '2' && i % 2 === 0) {
                hiHatClosed.start(time);
            }
        }
        i++;
    }, "4n");

    melody1.start();
    currentLoop.start(0);
    transport.start();
    console.log("Le beat démarre !");
}

function stop() {
    if (transport.state === 'started') {
        transport.stop();
        currentLoop?.dispose();
        currentLoop = null;
        isPlaying.value = false;
    }
    if (melody1.state === 'started') melody1.stop();
}

function toggleHiHatMode() {
    hiHatMode.value = hiHatMode.value === '4' ? '2' : '4';
    console.log(`Hi-Hat Mode: ${hiHatMode.value === '4' ? 'Tous les 4 temps' : 'Tous les 2 temps'}`);
}

function toggleKick() { playKick.value = !playKick.value; }
function toggleSnare() { playSnare.value = !playSnare.value; }
function toggleHiHat() { playHiHat.value = !playHiHat.value; }
function toggleMelody() { playMelody.value = !playMelody.value; }


</script>


<template>
    <div>
        <div ref="panel"></div>

        <canvas :width="WIDTH" :height="HEIGHT" ref="canvas"></canvas>

        <button @click="toggleKick">{{ playKick ? 'Kick: On' : 'Kick: Off' }}</button>
        <button @click="toggleSnare">{{ playSnare ? 'Snare: On' : 'Snare: Off' }}</button>
        <button @click="toggleHiHat">{{ playHiHat ? 'Hi-Hat: On' : 'Hi-Hat: Off' }}</button>
        <button @click="toggleMelody">{{ playMelody ? 'Melody: On' : 'Melody: Off' }}</button>
        <button @click="toggleHiHatMode">
            {{ hiHatMode === '4' ? 'Hi-Hat: Mode 4 temps' : 'Hi-Hat: Mode 2 temps' }}
        </button>

        <button @click="play" v-if="!isPlaying">Start Beat</button>
        <button @click="stop" v-if="isPlaying">Stop</button>
    </div>
</template>
