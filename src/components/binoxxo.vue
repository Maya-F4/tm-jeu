<script lang="ts" setup>
import { ref } from 'vue';

const grille = ref([
    [null, null, null, null, 1, null],
    [null, null, 1, null, null, 1],
    [null, null, null, 0, null, null],
    [0, null, 0, 0, null, null],
    [null, null, null, null, null, null],
    [1, null, 0, null, null, 1],
])

const copieGrilleInitiale = JSON.parse(JSON.stringify(grille.value))

function afficherLigne(numero: number) {
    console.log(grille.value[numero])
}

function afficherColonne(numero: number) {
    const colonne = []
    for (let L = 0; L < 6; L=L+1) {
        colonne.push(grille.value[L][numero])
    }
    console.log(colonne)
}

function getColonne(numero: number) {
    const colonne = []
    for (let L = 0; L < 6; L=L+1) {
        colonne.push(grille.value[L][numero])
    }
    return colonne
}

function pasdeTriplet(Ligne) { 
    for (let C = 0 ; C<4; C=C+1) {
        const a = Ligne[C]
        const b = Ligne[C+1]
        const c = Ligne[C+2]
        if ( a===b && b===c && a !== null) {
            return false
    }
    }
    return true
}

function vérifierGrille() {
    for (let L=0; L<6 ; L=L+1) {
        if (pasdeTriplet(grille.value[L]) === false) {
            return false
        }
    }
    for (let C=0; C<6 ; C=C+1) {
        if (pasdeTriplet(getColonne(C)) === false) {
            return false
        }
    }
    for (let L=0; L<6 ; L=L+1) {
        if (equilibre(grille.value[L]) === false) {
            return false
        }
    }
    for (let C=0; C<6 ; C=C+1) {
        if (equilibre(getColonne(C)) === false) {
            return false
        }
    }
    return true
}

function equilibre(Ligne) {
    let nb0 = 0
    let nb1 = 0
    for (let C=0; C<6 ; C=C+1) {
        if (Ligne[C] === 0) {
            nb0 = nb0 + 1
            if (nb0 > 3) {
                return false
            }
        }
        if (Ligne[C] === 1) {
            nb1 = nb1 + 1
            if (nb1 > 3) {
                return false
            }
        }
    }
    return true
}

const mode = ref<null | 0 | 1>(null)

function caseModifiable(L ,C) {
    if (copieGrilleInitiale[L][C] === null) {
        return true
    }
    return false
}

function jouerCase(L ,C) {
    if (caseModifiable(L ,C) === false) {
        return
    }
    if (mode.value === null) {
        return
    }
    grille.value[L][C] = mode.value
}

</script>
<template>
    <div>
        <h1 class="text-2xl font-bold mb-4">Binoxxo</h1>
    </div>
    <div class="grid grid-cols-6 gap-1 w-fit">
        <template v-for="(Ligne, L) in grille" :key="L" >
            <div v-for="(Case_, C) in Ligne" :key="C" 
                class="w-10 h-10 border border-gray-300 text-center"
                @click= "jouerCase(L , C)">
                {{ Case_ ?? '.' }}
            </div>
        </template>
    </div>
    <div>
        <button 
        class="bg-blue-400 border-blue-500 rounded-2xl text-white p-2 m-2 hover:bg-blue-500"
        @click="mode=0">Mode 0</button>
        <button 
        class="bg-blue-400 border-blue-500 rounded-2xl text-white p-2 m-2 hover:bg-blue-500"
        @click="mode=1">Mode 1</button>
    </div>
    <div>
        <p>Mode actuel: {{ mode ?? 'Aucun' }}</p>
    </div>
</template> 

