<script setup lang="ts">
import { ref } from 'vue'
import * as Tone from 'tone'

let playCrunch = ref(false)

const kickPlayer = new Tone.Player({
    url: '/src/assets/sound/kick/WAAWF_Kick3_Rev.mp3',
    onload: () => console.log('Kick loaded!'),
}).toDestination()

const crunch = new Tone.Player({
    url: '/src/assets/bip.mp3',
}).toDestination()

function play() {
    let i = 0;
    new Tone.Loop((time) => {
        if (i%2 === 0) kickPlayer.start()

        if (playCrunch.value) {
            crunch.start()
        } 
        i++
    }, "5n").start();
    Tone.Transport.start();

    // tweak the bpm ?
    // Tone.getTransport().bpm.rampTo(240,1)
}

function toggle() {
    console.log('prout')
    playCrunch.value = !playCrunch.value
}

</script>

<template>
    <button @click="play">bondoir </button>
    <button @click="toggle">crunch {{playCrunch ? 'yes':'no' }}</button>
</template>
