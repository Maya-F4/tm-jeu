<script lang="ts" setup>
import { computed, ref } from 'vue';

const plateau = ref(
    [
        1, 0, 1,
        0, 0, 1,
        0, 0, 0
    ]
)

function couleur(jeton : number) {
    if (jeton === 1) {
        return "bg-gray-900"
    } else {
        return "bg-stone-200 border border-gray-300"
    }
}

function retournerDiagonale(index : number) {
    if (index===0 || index===8) {
        plateau.value[0] = 1 - plateau.value[0]
        plateau.value[4] = 1 - plateau.value[4]
        plateau.value[8] = 1 - plateau.value[8]
    }
     if (index===2 || index===6) {
        plateau.value[2] = 1 - plateau.value[2]
        plateau.value[4] = 1 - plateau.value[4]
        plateau.value[6] = 1 - plateau.value[6]
    }
    compter()
}

function retournerLigne(index: number) {
    const ligne = Math.floor(index / 3)
    const debut = ligne * 3

    plateau.value[debut] = 1 - plateau.value[debut]
    plateau.value[debut + 1] = 1 - plateau.value[debut + 1]
    plateau.value[debut + 2] = 1 - plateau.value[debut + 2]
    compter()
}

function retournerColonne(index : number) {
    const colonne = index % 3
    plateau.value[colonne] = 1 - plateau.value[colonne]
    plateau.value[colonne + 3] = 1 - plateau.value[colonne + 3]
    plateau.value[colonne + 6] = 1 - plateau.value[colonne + 6]
    compter()
}

const mode = ref<'ligne' | 'colonne' | 'diagonale'>("ligne")

function jouer(index) {
    if (mode.value === "ligne") {
        retournerLigne(index)
    }
    else if (mode.value === "diagonale") {
        retournerDiagonale(index)
    }
    else if (mode.value === "colonne") {
        retournerColonne(index)
    }
}

const compteur = ref(0)

function compter() {
    compteur.value = compteur.value + 1
}

function reset() {
    if (confirm("Voulez-vous recommencer ?")) {
        plateau.value = [
            1, 0, 1,
            0, 0, 1,
            0, 0, 0
        ]
        compteur.value = 0
    }
}

const cases_blanches = computed<number>(() => {
    return plateau.value.filter(jeton => jeton === 0).length
})

const victoire = computed<boolean>(() => {
    return cases_blanches.value === 0
})

function plateauAleatoire() {
    plateau.value = Array.from({ length: 9 }, () => Math.round(Math.random()))
    compteur.value = 0
}

</script>

<template>
    <div class="flex flex-col items-center justify-center min-h-screen bg-stone-50">
        <h1 class="text-2xl text-center font-bold mb-4">Plateau de jeu</h1>
        
        <div class="bg-white p-6 rounded-2xl border border-stone-200 flex flex-col items-center gap-4 w-fit mx-auto">
        
            <div class="grid grid-cols-3 gap-2 w-fit">
                <div 
                    v-for="(jeton, index) in plateau" 
                    :key="index" 
                    @click="jouer(index)"
                    :class="['w-16', 'h-16', 'flex', 'items-center', 'justify-center', 'rounded-xl', couleur(jeton)]">
                </div>
            </div>

        <div class="flex gap-2">
            <button 
            class="rounded-2xl p-4 text-stone-600 mt-4" 
            :class="mode === 'ligne' ? ' bg-blue-200 , border border-blue-300' : 'bg-stone-100 , border border-gray-200'"
            @click="mode = 'ligne' " 
            >Ligne</button>

            <button 
            class=" rounded-2xl p-4 text-stone-600 mt-4 " 
            :class="mode === 'colonne' ? 'bg-blue-200 , border border-blue-300' : 'bg-stone-100 , border border-gray-200'"
            @click="mode = 'colonne'" >Colonne</button>

            <button 
            class=" rounded-2xl p-4 text-stone-600 mt-4"
            :class="mode === 'diagonale' ? 'bg-blue-200 , border border-blue-300' : 'bg-stone-100 , border border-gray-200'" 
            @click="mode = 'diagonale'" >Diagonale</button>
        </div>

        <div class="flex gap-4">
            <div class = "bg-stone-100 text-center rounded-xl px-6 py-3 border border-stone-200 ">
                <p class = " text-center text-stone-800 font-bold text-xl" >{{ compteur }}</p>
                <p class = "text-center text-stone-600 font-normal text-xs" >coups</p>
            </div>
            <div class = "bg-stone-100 text-center rounded-xl px-6 py-3 border border-stone-200 ">
                <p class = " text-center text-stone-800 font-bold text-xl" >{{ cases_blanches }}</p>
                <p class = "text-center text-stone-600 font-normal text-xs" >cases blanches</p>
            </div>
        </div>

        <div class="flex gap-2">
            <button 
            class="w-34 h-16 rounded-2xl p-4 text-stone-600 mt-4 text-center bg-orange-100 border border-orange-200" 
            @click="reset()" >Plateau initial</button>

            <button 
            class="w-34 h-16 rounded-2xl p-4 text-stone-600 mt-4 text-center bg-green-100 border border-green-200" 
            @click="plateauAleatoire()" >Plateau aléatoire</button>
        </div>

            
            <div v-if="victoire" class="col-span-3 text-center mt-2">
            <p class="text-2xl font-bold text-green-700">🎉 Bravo !</p>
            <p class="text-green-600 mt-1">Résolu en {{ compteur }} coups</p>
            </div>
        </div>
    </div>
</template>


