<script setup lang="ts">
import { onMounted, onUnmounted, ref } from 'vue'
import * as Tone from 'tone'

const WIDTH = 500
const HEIGHT = 500

const panel = ref<HTMLDListElement | null>(null)
const canvas = ref<HTMLCanvasElement | null>(null)
const context = ref<CanvasRenderingContext2D | null>(null)
const showStartScreen = ref(true);

function startFromScreen() {
    showStartScreen.value = false;
    if (context.value && canvas.value) {
        context.value = canvas.value.getContext('2d');
    }
    requestAnimationFrame(render);
    play();
}
onMounted(async () => {
    await Tone.loaded(); 

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



const colorDefault = '#000000';
const defaultCircles = [] as { radius: number, color: string }[];

const numCircles = 5;
const centerX = WIDTH / 2;
const centerY = HEIGHT / 2;

const circle = [] as {
    x: number;
    y: number;
    radius: number;
    velocity: number;
    color: string;
    shake: number;
    lineWidth: number;
}[];

for (let i = 0; i < numCircles; i++) {
    const radius = i * 40 + 50;
    const velocity = i + 5 / 2.2;
    const color = colorDefault;

    circle.push({
        x: centerX,
        y: centerY,
        radius,
        velocity,
        color,
        shake: 0,
        lineWidth: 2,
    });

    defaultCircles.push({ radius, color });
}

type Point = { x: number; y: number };
function getPointAtAngle(center: Point, angle: number, distance: number): Point {
    return {
        x: center.x + Math.cos(angle) * distance,
        y: center.y + Math.sin(angle) * distance,
    };
}

function updateVoiceAllCircle() {
    circle.forEach(element => {
        element.radius += 4;
    });
}

function resetVoiceAllCircle() {
    circle.forEach((element, index) => {
        element.radius = defaultCircles[index].radius;
    });
}

function resetMelodyState() {
    const melodyCircle = circle[0]
    melodyCircle.color = defaultCircles[1].color;
    melodyCircle.lineWidth = 2;


}
function updateCircleMelody() {
    const melodyCircle = circle[0]
    melodyCircle.color = '#8E7DEF';
    melodyCircle.lineWidth = 8;

}

function resetKickCircleState() {
    const kickCircle = circle[2];
    kickCircle.radius = defaultCircles[2].radius;
    kickCircle.color = defaultCircles[2].color;
    kickCircle.lineWidth = 2;
}


function updateCircleOnKick() {
    const kickCircle = circle[2];
    kickCircle.radius += 15;
    kickCircle.color = '#ff32ff';
    kickCircle.lineWidth = 8;
}

function resetCircleHihat() {
    const hihatCircle = circle[1];
    hihatCircle.color = defaultCircles[1].color;
    hihatCircle.lineWidth = 2;
}


function updateCircleHihat() {
    const hihatCircle = circle[1];
    hihatCircle.color = '#f00020';
    hihatCircle.lineWidth = 8;
}


let bassShakeTime = 0;
let bassShakeStart = 0;
let foleyShakeTime = 0;
let foleyShakeStart = 0;

function updateBassShake(timestamp: number, bassCircle: { x: number, y: number, shake: number }) {
    if (bassShakeTime > 0) {
        const elapsed = timestamp - bassShakeStart;

        if (elapsed < bassShakeTime) {
            bassCircle.x = centerX + Math.sin(elapsed / 40) * 10;
            bassCircle.y = centerY + Math.cos(elapsed / 20) * 10;
        } else {
            resetBassCircleState();
            bassShakeTime = 0;
        }
    }
}

function updateFoleyShake(timestamp: number, foleyCircle: { x: number, y: number, shake: number }) {
    if (foleyShakeTime > 0) {
        const elapsed = timestamp - foleyShakeStart;

        if (elapsed < foleyShakeTime) {
            foleyCircle.x = centerX + Math.sin(elapsed / 40) * 10;
            foleyCircle.y = centerY + Math.cos(elapsed / 20) * 10;
        } else {
            resetCircleFoley();
            foleyShakeTime = 0;
        }
    }
}

function resetBassCircleState() {
    const bassCircle = circle[4];
    bassCircle.x = centerX;
    bassCircle.y = centerY;
    bassCircle.shake = 0;
    bassCircle.lineWidth = 2;
    bassCircle.color = defaultCircles[4].color;
}

function updateCircleOnBass() {
    const bassCircle = circle[4];
    bassCircle.color = '#2ceaff';
    bassCircle.lineWidth = 8;
    bassShakeTime = 800;
    bassShakeStart = performance.now();
}

function resetCircleFoley() {
    const foleyCircle = circle[3];
    foleyCircle.color = defaultCircles[3].color;
    foleyCircle.x = centerX;
    foleyCircle.y = centerY;
    foleyCircle.shake = 0;
    foleyCircle.lineWidth = 2;
}

function updateCircleFoley() {
    const foleyCircle = circle[3];
    foleyCircle.color = '#00f048';
    foleyCircle.lineWidth = 8;
    foleyShakeTime = 500;
    foleyShakeStart = performance.now();
}

let playKick = ref(false)
let playBass = ref(false)
let playHiHat = ref(false)
let playMelody = ref(false)
let playVoice = ref(false)
let playFoley = ref(false)
let isPlaying = ref(false)
let currentLoop: Tone.Loop | null = null;
let isHiHatHovered = ref(false);
let hiHatMode = ref<'4' | '2'>('4')

const transport = Tone.getTransport();
const bassGain = new Tone.Gain(1.2).toDestination();

transport.bpm.value = 83;

const melody1 = new Tone.Player({ url: '/assets/sound/melody/melody83.wav', loop: true }).toDestination();
const kickPlayer = new Tone.Player({ url: '/assets/sound/kick/kick.wav' }).toDestination();
const bassPlayer = new Tone.Player({ url: '/assets/sound/bass/bass2.wav' }).connect(bassGain);
const hiHatClosed = new Tone.Player({ url: '/assets/sound/hihat/hihat2.wav' }).toDestination();
const voice = new Tone.Player({ url: '/assets/sound/vocal/BringItOn.wav' }).toDestination();
const foleyPlayer = new Tone.Player({ url: '/assets/sound/foley/foley.wav' }).toDestination();


//FONCTION POUR TOUT STOP
function stopAllSounds() {
    playKick.value = false;
    playBass.value = false;
    playFoley.value = false;
    playHiHat.value = false;
    playMelody.value = false;


    resetBassCircleState();
    resetCircleHihat();
    resetKickCircleState();
    resetMelodyState();
    resetVoiceAllCircle();
    resetCircleFoley();

    if (kickPlayer.state === 'started') kickPlayer.stop();
    if (bassPlayer.state === 'started') bassPlayer.stop();
    if (hiHatClosed.state === 'started') hiHatClosed.stop();
    if (melody1.state === 'started') melody1.stop();
    if (foleyPlayer.state === 'started') foleyPlayer.stop();

    console.log("Tous les sons ont été arrêtés !");
}

function isAnySoundActive() {
    return playKick.value || playBass.value || playHiHat.value || playMelody.value || playFoley.value;
}

//FONCTION POUR LES BOUTONS
function onHiHatHover() {
    isHiHatHovered.value = true;
}
function onHiHatLeave() {
    isHiHatHovered.value = false;

} function toggleHiHatMode(mode: '4' | '2') {
    hiHatMode.value = mode;
    playHiHat.value = true;
    console.log(`Hi-Hat Mode: ${hiHatMode.value === '4' ? 'Tous les 4 temps' : 'Tous les 2 temps'}`);
}

function toggleHiHat() { playHiHat.value = !playHiHat.value; }
function toggleKick() { playKick.value = !playKick.value; }
function toggleFoley() { playFoley.value = !playFoley.value; }
function toggleBass() { playBass.value = !playBass.value; }
function toggleMelody() {
    playMelody.value = !playMelody.value;
    
    if (playMelody.value) {
        if (melody1.buffer) { // Vérifie que le buffer est chargé avant de jouer
            melody1.start();
            updateCircleMelody();
        } else {
            console.error("Melody buffer is not loaded!");
        }
    } else {
        resetMelodyState();
        melody1.stop();
    }
}

function toggleVoice() {
    playVoice.value = !playVoice.value;

    if (playVoice.value) {
        voice.start();
        updateVoiceAllCircle();

        voice.onstop = () => {
            playVoice.value = false;
            resetVoiceAllCircle();
        };
    } else {
        voice.stop();
        resetVoiceAllCircle();
    }
}



function play() {
    if (transport.state === 'started') {
        console.log("La boucle est déjà en cours !");
        return;
    }
    isPlaying.value = true;
    let i = 0;
    currentLoop = new Tone.Loop((time) => {
        if (playKick.value && i % 2 === 0) {
            kickPlayer.start(time);
            updateCircleOnKick();
        } else {
            resetKickCircleState();
        }

        if (playBass.value && i % 4 === 0) {
            bassPlayer.start(time);
            updateCircleOnBass();
        } else {
            resetBassCircleState();
        }

        if (playFoley.value && i % 8 === 0) {
            foleyPlayer.start(time);
            updateCircleFoley();
        } else {
            resetCircleFoley();
        }

        if (playHiHat.value) {
            let triggered = false;

            if (hiHatMode.value === '4' && i % 4 === 0) {
                hiHatClosed.start(time);
                updateCircleHihat();
                triggered = true;
            } else if (hiHatMode.value === '2' && i % 2 !== 0) {
                hiHatClosed.start(time);
                updateCircleHihat();
                triggered = true;
            }

            if (!triggered) {
                resetCircleHihat();
            }
        }
        i++;
    }, "4n");
    currentLoop.start(0);
    transport.start();
    console.log("Le beat démarre !");
}

function render(timestamp: number) {
    if (!context.value) return;

    const ctx = context.value;
    ctx.clearRect(0, 0, WIDTH, HEIGHT);

    circle.forEach((c, index) => {
        let offsetX = 0;
        let offsetY = 0;

        if (index === 4) {
            updateBassShake(timestamp, c);
        }

        if (index === 3) {
            updateFoleyShake(timestamp, c);
        }

        const angle1 = getPointAtAngle({ x: 500, y: 500 }, timestamp / 100000 * c.velocity, 100).x % (Math.PI * 2);

        ctx.beginPath();
        ctx.arc(c.x + offsetX, c.y + offsetY, c.radius, angle1 - Math.PI, angle1);
        ctx.strokeStyle = c.color;
        ctx.lineWidth = c.lineWidth;
        ctx.stroke();
    });

    requestAnimationFrame(render);
}



</script>

<template>
    <div>
        <div v-show="showStartScreen" class="startScreen__container">
            <h1>Welcome to VibeLoops</h1>
            <button @click="startFromScreen">Commencer</button>
        </div>

        <div v-show="!showStartScreen" class="main__container">
            <div>
                <canvas :width="WIDTH" :height="HEIGHT" ref="canvas"></canvas>
            </div>
            <div class="button__container">
                <button @click="toggleMelody">{{ playMelody ? 'Melody: On' : 'Melody: Off' }}</button>
                <button @click="toggleKick">{{ playKick ? 'Kick: On' : 'Kick: Off' }}</button>
                <button @click="toggleBass">{{ playBass ? 'Bass: On' : 'Bass: Off' }}</button>

                <div class="hi-hat-container" @mouseover="onHiHatHover" @mouseleave="onHiHatLeave">
                    <button @click="toggleHiHat">{{ playHiHat ? 'Hi-Hat: On' : 'Hi-Hat: Off' }}</button>
                    <div class="hi-hat-modes" v-if="isHiHatHovered">
                        <button @click="toggleHiHatMode('4')">Mode 4 temps</button>
                        <button @click="toggleHiHatMode('2')">Mode 2 temps</button>
                    </div>
                </div>
                <button @click="toggleVoice">{{ playVoice ? 'Voice: On' : 'Voice: Off' }}</button>
                <button @click="toggleFoley">{{ playFoley ? 'Foley: On' : 'Foley: Off' }}</button>

                <button @click="stopAllSounds" :disabled="!isAnySoundActive()">Stop All Sounds</button>
            </div>
        </div>
    </div>
</template>
