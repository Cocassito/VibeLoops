<script setup lang="ts">
import { ref } from 'vue'
import * as Tone from 'tone'

// Variables pour contrôler les états des éléments
let playKick = ref(false)
let playSnare = ref(false)
let playHiHat = ref(false)
let playMelody = ref(false)
let isPlaying = ref(false) // Pour éviter de lancer plusieurs boucles
let currentLoop: Tone.Loop | null = null; // Référence à la boucle en cours
let hiHatMode = ref<'4' | '2'>('4') // Mode Hi-Hat : '4' = tous les 4 temps, '2' = tous les 2 temps

const transport = Tone.getTransport();
transport.bpm.value = 83; 

const kickPlayer = new Tone.Player({ url: '/src/assets/sound/kick/WAAWF_Kick3_Rev.mp3' }).toDestination();
const bassPlayer = new Tone.Player({ url: '/src/assets/bass.wav' }).toDestination();
const hiHatClosed = new Tone.Player({ url: '/src/assets/hithat2.wav' }).toDestination();
const melody1 = new Tone.Player({ url: '/src/assets/melody83.wav', loop: true }).toDestination();

function play() {
    if (transport.state === 'started') {
        console.log("La boucle est déjà en cours !");
        return;
    }

    isPlaying.value = true;
    let i = 0; 

    currentLoop = new Tone.Loop((time) => {
        if (playKick.value && i % 4 === 0) kickPlayer.start(time);
        if (playSnare.value && i % 4 === 0) bassPlayer.start(time);

        // Hi-Hat logique conditionnelle basée sur hiHatMode
        if (playHiHat.value) {
            if (hiHatMode.value === '4' && i % 4 === 0) {
                // Mode : tous les 4 temps
                hiHatClosed.start(time);
            } else if (hiHatMode.value === '2' && i % 2 === 0) {
                // Mode : tous les 2 temps
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

// Fonction pour arrêter la musique
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
