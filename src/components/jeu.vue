<script setup lang="ts" >
import { ref } from 'vue';

const score = ref(0)

</script>


<template>
  <div>  c'est le jeu du bouton, ton score est de {{score}} 

    <button 
        class=" bg-lime-400 border-lime-600 border-2 rounded-2xl shadow-lg shadow-lime-300 hover:bg-green-800 transition-colors duration-300 px-4 py-2 m-2 hover:scale-120"
        @click="score = score + 1"> BOUTON </button>
     

  </div>
</template>
