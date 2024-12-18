<script setup lang="ts">
import { onMounted, onUnmounted, ref } from 'vue'
import * as Tone from 'tone'

const WIDTH = 500
const HEIGHT = 500

const panel = ref<HTMLDListElement | null>(null)
const canvas = ref<HTMLCanvasElement | null>(null)
const context = ref<CanvasRenderingContext2D | null>(null)
const showStartScreen = ref(true); // État de l'écran intermédiaire

function startFromScreen() {
    showStartScreen.value = false;
    if (context.value && canvas.value) {
        context.value = canvas.value.getContext('2d'); // Récupère le contexte
    }
    requestAnimationFrame(render);
    play();
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

const colorDefault = 'blue';
const defaultCircles = [] as { radius: number, color: string }[];

const numCircles = 5;
const centerX = WIDTH / 2;
const centerY = HEIGHT / 2;

const circle = [] as { x: number, y: number, radius: number, velocity: number, color: string, shake: number }[];

for (let i = 0; i < numCircles; i++) {
    const radius = i * 10 + 50;
    const velocity = i + 5 / 2.2;
    const color = colorDefault;

    // Ajouter chaque cercle à la liste de cercles
    circle.push({ x: centerX, y: centerY, radius, velocity, color, shake: 0 });

    // Sauvegarder l'état par défaut dans `defaultCircles`
    defaultCircles.push({ radius, color });
}

// Fonction pour obtenir un point sur un cercle à un certain angle
type Point = { x: number; y: number };
function getPointAtAngle(center: Point, angle: number, distance: number): Point {
    return {
        x: center.x + Math.cos(angle) * distance,
        y: center.y + Math.sin(angle) * distance,
    };
}

// Réinitialiser un cercle à son état par défaut
function resetCircleState(circleIndex: number) {
    const currentCircle = circle[circleIndex];
    currentCircle.radius = defaultCircles[circleIndex].radius;
    currentCircle.color = defaultCircles[circleIndex].color;
}

// Mettre à jour un cercle lorsqu'un kick est joué
function updateCircleOnKick(circleIndex: number) {
    const currentCircle = circle[circleIndex];
    currentCircle.radius += 5;  // Augmenter le rayon
    currentCircle.color = 'red';  // Changer la couleur en rouge
}

let bassShakeTime = 0;


function resetBassCircleState() {
    const bassCircle = circle[1];  // Utiliser le cercle à l'index 1
    bassCircle.x = centerX;
    bassCircle.y = centerY;
    bassCircle.shake = 0;
}

function updateCircleOnBass() {
    const bassCircle = circle[1];  // Utiliser le cercle à l'index 1
    bassCircle.shake = 5;  // Créer un tremblement (un effet de vibration)
    // Déplacer légèrement sur les axes X et Y pour simuler le tremblement
    bassCircle.x += (Math.random() - 0.5) * 10;  // Déplacer légèrement sur l'axe X
    bassCircle.y += (Math.random() - 0.5) * 10;  // Déplacer légèrement sur l'axe Y
}


let playKick = ref(false)
let playBass = ref(false)
let playHiHat = ref(false)
let playMelody = ref(false)
let isPlaying = ref(false)
let currentLoop: Tone.Loop | null = null;
let isHiHatHovered = ref(false);  // Suivi du survol du bouton Hi-Hat
let hiHatMode = ref<'4' | '2'>('4')

const transport = Tone.getTransport();
const bassGain = new Tone.Gain(1.2).toDestination();

transport.bpm.value = 83;

const kickPlayer = new Tone.Player({ url: '/src/assets/sound/kick/kick.wav' }).toDestination();
const bassPlayer = new Tone.Player({ url: '/src/assets/sound/bass/bass.wav' }).connect(bassGain);
const hiHatClosed = new Tone.Player({ url: '/src/assets/sound/hihat/hihat2.wav' }).toDestination();
const melody1 = new Tone.Player({ url: '/src/assets/sound/melody/melody83.wav', loop: true }).toDestination();



function stopAllSounds() {
    playKick.value = false;
    playBass.value = false;
    playHiHat.value = false;
    playMelody.value = false;

    if (kickPlayer.state === 'started') kickPlayer.stop();
    if (bassPlayer.state === 'started') bassPlayer.stop();
    if (hiHatClosed.state === 'started') hiHatClosed.stop();
    if (melody1.state === 'started') melody1.stop();

    console.log("Tous les sons ont été arrêtés !");
}

function isAnySoundActive() {
    return playKick.value || playBass.value || playHiHat.value || playMelody.value;
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
function toggleBass() { playBass.value = !playBass.value; }
function toggleMelody() {
    playMelody.value = !playMelody.value;
    if (playMelody.value) {
        melody1.start();
    } else {
        melody1.stop();
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
    if (playKick.value && i % 8 === 0) {
        kickPlayer.start(time);
        updateCircleOnKick(0); // Modifier le cercle du kick
    } else {
        resetCircleState(0); // Réinitialiser le cercle du kick
    }

    if (playBass.value && i % 4 === 0) {
        bassPlayer.start(time);
        updateCircleOnBass(); // Appliquer le tremblement pour le cercle de basse
    } else {
        resetBassCircleState(); // Réinitialiser le cercle de basse
    }

    if (playHiHat.value) {
        if (hiHatMode.value === '4' && i % 4 === 0) {
            hiHatClosed.start(time);
        } else if (hiHatMode.value === '2' && i % 2 === 0) {
            hiHatClosed.start(time);
        }
    }
    i++;
}, "4n");


    if (playMelody.value) {
        melody1.start();
    }

    currentLoop.start(0);
    transport.start();
    console.log("Le beat démarre !");
}

// Fonction de rendu pour afficher les cercles sur le canvas
function render(timestamp: number) {
    if (!context.value) return;

    const ctx = context.value;
    ctx.clearRect(0, 0, WIDTH, HEIGHT);

    // Dessiner les cercles normalement
    circle.forEach(c => {
        const angle1 = getPointAtAngle({ x: 500, y: 500 }, timestamp / 100000 * c.velocity, 100).x % (Math.PI * 2);
        ctx.beginPath();
        ctx.arc(c.x, c.y, c.radius, angle1 - Math.PI, angle1);
        ctx.strokeStyle = c.color;
        ctx.lineWidth = 2;
        ctx.stroke();
    });

    // Appliquer l'effet de tremblement sur le cercle de basse
    if (circle[1].shake > 0) {
        circle[1].shake -= (Date.now() - bassShakeTime) / 1000;
    }

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
                <button @click="toggleKick">{{ playKick ? 'Kick: On' : 'Kick: Off' }}</button>
                <button @click="toggleBass">{{ playBass ? 'Bass: On' : 'Bass: Off' }}</button>

                <!-- Bouton Hi-Hat avec gestion de survol pour afficher les modes -->
                <div class="hi-hat-container" @mouseover="onHiHatHover" @mouseleave="onHiHatLeave">
                    <button @click="toggleHiHat">{{ playHiHat ? 'Hi-Hat: On' : 'Hi-Hat: Off' }}</button>
                    <div class="hi-hat-modes" v-if="isHiHatHovered">
                        <button @click="toggleHiHatMode('4')">Mode 4 temps</button>
                        <button @click="toggleHiHatMode('2')">Mode 2 temps</button>
                    </div>
                </div>

                <button @click="toggleMelody">{{ playMelody ? 'Melody: On' : 'Melody: Off' }}</button>
                <button @click="stopAllSounds" :disabled="!isAnySoundActive()">Stop All Sounds</button>
            </div>
        </div>
    </div>
</template>
