<script lang="ts" setup>
import { computed, ref } from 'vue';


const compteur = ref(0)

const trop = computed(()=>{
    return compteur.value > 10
})
/*
const cliquer = ()=>{
    compteur.value = compteur.value + 1
}
*/

function cliquer(){
    compteur.value = compteur.value + 1
}

</script>
<template>
    <div>
        C'est un exemple avec un bouton

        <button 
        class="rounded-2xl shadow-lg p-4 text-white font-bold mt-4 bg-rose-300 hover:bg-rose-800 transition-colors duration-300"
        @click="cliquer">joli bouton</button>

        On a cliqué {{ compteur }}

        <div>
            <p v-if="trop">C'est trop !</p>
        </div>
    </div>
</template>