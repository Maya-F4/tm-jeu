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

/*const grillesDisponibles = [[
    [null, null, null, null, 1, null],
    [null, null, 1, null, null, 1],
    [null, null, null, 0, null, null],
    [0, null, 0, 0, null, null],
    [null, null, null, null, null, null],
    [1, null, 0, null, null, 1],
], 
[
    [1, null, null, 1, null, null],
    [null, 0, null, 1, null, 1],
    [null, null, null, null, 0, null],
    [null, 1, null, null, null, null],
    [null, null, null, null, null, 0],
    [null, null, null, null, 1,null],
],
[
    [0, null, 0, null, null, 0],
    [null, null, null, 1, null, null],
    [null, null, 0, null, null, 1],
    [null, null, null, null, null, null],
    [null, null, 1, 1, null, 1],
    [null, null, null, null, null, null],
]]*/

let copieGrilleInitiale = JSON.parse(JSON.stringify(grille.value)) /*"quand la page s'ouvre, prends une photo de la grille de départ."*/

/*function choisirGrille(){
    const grilleChoisie = Math.floor(Math.random() * grillesDisponibles.length)
    grille.value = JSON.parse(JSON.stringify(grillesDisponibles[grilleChoisie]))
    copieGrilleInitiale = JSON.parse(JSON.stringify(grille.value))   "à chaque fois qu'on choisit une nouvelle grille au hasard, reprends une nouvelle photo, à jour."
}*/

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
    if(unicitéLignes() === false) {
        return false
    }
    if(unicitéColonnes() === false) {
        return false
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

const mode = ref<null | 'O' | 'X'>(null)

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
    /*dernierCoup.value = {L ,C}*/
}

/*const dernierCoup = ref<null | {L: number, C: number}>(null)*

/*function dernierCoupErreur(L ,C){
    if (dernierCoup.value === null) {
        return false
    }
    if ( dernierCoup.value.L === L && dernierCoup.value.C === C && vérifierGrille() === false) {
        return true
    }
    return false
}
    
:class="{ 'bg-red-200':dernierCoupErreur(L, C) }" 
dans le template pour colorer la case du dernier coup si la grille est invalide.
*/

function ligneComplete(Ligne){
    for (let C = 0 ; C < 6 ; C=C+1) {
        if (Ligne[C] === null) {
            return false
        }
    }
    return true
}

function ligneIdentique (Ligne1 , Ligne2) {
    for (let C=0 ; C < 6 ; C=C+1) {
        if (Ligne1[C]!== Ligne2[C]) {
            return false
        }
    }
    return true
}

function unicitéLignes() {
    for (let Lx=0 ; Lx < 6 ; Lx=Lx+1){
        for (let Ly=Lx+1 ; Ly < 6 ; Ly=Ly+1){
            if( ligneComplete(grille.value[Lx]) && ligneComplete(grille.value[Ly]) && ligneIdentique(grille.value[Lx],grille.value[Ly]) ) {
                return false
            }
        }
    }
    return true
}

function unicitéColonnes() {
    for (let Cx=0 ; Cx < 6 ; Cx=Cx+1){
        for (let Cy=Cx+1 ; Cy < 6 ; Cy=Cy+1){
            if( ligneComplete(getColonne(Cx)) && ligneComplete(getColonne(Cy)) && ligneIdentique(getColonne(Cx),getColonne(Cy)) ) {
                return false
            }
        }
    }
    return true
}

function caseEnErreur(L , C) {
    if(equilibre(grille.value[L]) === false || pasdeTriplet(grille.value[L]) === false || equilibre(getColonne(C)) === false || pasdeTriplet(getColonne(C)) === false) {
        return true
    }
    return false
}

function resetGrille(){
    grille.value = JSON.parse(JSON.stringify(copieGrilleInitiale))
}


/*création algorithme grille aléatoire*/

function caseSuivante(L ,C) {
    if (C<5){
        return {L:L ,C:C+1}
    }
    if (C===5 && L<5){
        return {L:L+1 ,C:0}
    }
    if (L===5 && C===5){
        return null
    }
}

function grilleVide() {
    const GrilleVide =
    [
    [null, null, null, null, null, null],
    [null, null, null, null, null, null],
    [null, null, null, null, null, null],
    [null, null, null, null, null, null],
    [null, null, null, null, null, null],
    [null, null, null, null, null, null],]
    return GrilleVide
}

function remplirGrille(L ,C) {
    const choix1ou0 = Math.random()
    const premierEssai = choix1ou0 < 0.5 ? 0 : 1
    const deuxiemeEssai = choix1ou0 < 0.5 ? 1 : 0
    grille.value[L][C] = premierEssai
    if (vérifierGrille()=== true){
        const nextCase = caseSuivante(L ,C)
        if (nextCase === null) {
            return true 
        }
        if (remplirGrille(nextCase.L ,nextCase.C) === true) {
            return true
        } 
    } 
    grille.value[L][C] = deuxiemeEssai
    if (vérifierGrille()=== true){
        const nextCase = caseSuivante(L ,C)
        if (nextCase === null) {
            return true 
        }
        if (remplirGrille(nextCase.L ,nextCase.C) === true) {
            return true
        }
    }
    grille.value[L][C] = null
    return false
}

function générerGrilleAléatoire(){
    grille.value = grilleVide()
    remplirGrille(0 ,0)
    cacherPlusieursCases()
}

/*création cases aléatoires à cacher*/


function cacherUneCase(){
    let L = Math.floor(Math.random()*6)
    let C = Math.floor(Math.random()*6)
    while (grille.value[L][C] === null) {
       L = Math.floor(Math.random()*6)
       C = Math.floor(Math.random()*6)
       } 
    grille.value[L][C] = null
}

function cacherPlusieursCases(){
    let tentatives = 0
    let casesCachées = 0
    while (casesCachées < 26 && tentatives < 200) {
        let L = Math.floor(Math.random()*6)
        let C = Math.floor(Math.random()*6)
        while (grille.value[L][C] === null) {
            L = Math.floor(Math.random()*6)
            C = Math.floor(Math.random()*6)
            }
        essayerCacherCase(L , C)
        tentatives = tentatives + 1
        if (grille.value[L][C] === null) {
            casesCachées = casesCachées + 1
    }
    }
     copieGrilleInitiale = JSON.parse(JSON.stringify(grille.value))   /*"à chaque fois qu'on choisit une nouvelle grille au hasard, reprends une nouvelle photo, à jour."*/
}

/*estétique*/

function changer10enXO(L,C){
    if (grille.value[L][C] === 0) {
        return 'O'
    }
    if (grille.value[L][C] === 1) {
        return 'X'
    }
    else {
        return ''
    }
}

/*solveur logique*/

function déduireParComptage(Ligne){
    let nb0=0
    let nb1=0
    for(let C=0 ; C<6 ; C=C+1){
        if(Ligne[C]===0){
            nb0=nb0+1
        }
        if(Ligne[C]===1){
            nb1=nb1+1
        }
    }
    if (nb0===3){
        for(let C=0 ; C<6 ; C=C+1){
            if(Ligne[C]===null){
                Ligne[C]=1
            }
        }
    }
    if (nb1===3){
        for(let C=0 ; C<6 ; C=C+1){
            if(Ligne[C]===null){
                Ligne[C]=0
            }
        }
    }
}

function valeurOpposée(valeur){
    if (valeur===0){
        return 1
    }
    if (valeur===1){
        return 0
    }
}

function déduireSandwich(Ligne){
    for(let C=1; C<5 ; C=C+1){
        if (Ligne[C]===null){
            if(Ligne[C-1]===Ligne[C+1] && Ligne[C-1]!==null){
                Ligne[C]=valeurOpposée(Ligne[C-1])
            }
        }
    }
}

function déduireAprèsPaire(Ligne){
    for(let C=2; C<6 ; C=C+1){
        if (Ligne[C]===null){
            if(Ligne[C-1]===Ligne[C-2] && Ligne[C-1]!==null){
                Ligne[C]=valeurOpposée(Ligne[C-1])
            }
        }
    }
}

function déduireAvantPaire(Ligne){
    for(let C=0; C<4 ; C=C+1){
        if (Ligne[C]===null){
            if(Ligne[C+1]===Ligne[C+2] && Ligne[C+1]!==null){
                Ligne[C]=valeurOpposée(Ligne[C+1])
            }
        }
    }
}

function ligneCompatible(LigneComplète , LigneIncomplète){
    for (let C=0 ; C<6 ; C=C+1){
        if (LigneIncomplète[C]!==null && LigneComplète[C]!==LigneIncomplète[C]){
            return false
        }
    }
    return true 
}  

function déduireComparaison(){
    for (let L=0 ; L<6 ; L=L+1){
        let nbnull=0
        for (let C=0 ; C<6 ; C=C+1){
            if(grille.value[L][C]===null){
                nbnull=nbnull+1
            }
        }
        if (nbnull===2){
            for (let L2=0 ; L2<6 ; L2=L2+1){
            if (ligneComplete(grille.value[L2])===true){
                if (ligneCompatible(grille.value[L2] , grille.value[L])===true){
                    for (let C=0 ; C<6 ; C=C+1){
                        if(grille.value[L][C]===null){
                            grille.value[L][C]=valeurOpposée(grille.value[L2][C])
                        }
                    }
                }
            }
            }
        }
    }
}

function déduireComparaisonColonnes(){
    for (let Co=0 ; Co<6 ; Co=Co+1){
        const colonne=getColonne(Co)
        let nbnull=0
        for (let C=0 ; C<6 ; C=C+1){
            if(colonne[C]===null){
                nbnull=nbnull+1
            }
        }
        if (nbnull===2){
            for (let Co2=0 ; Co2<6 ; Co2=Co2+1){
                const colonne2=getColonne(Co2)
            if (ligneComplete(colonne2)===true){
                if (ligneCompatible(colonne2 , colonne)===true){
                    for (let C=0 ; C<6 ; C=C+1){
                        if(colonne[C]===null){
                            colonne[C]=valeurOpposée(colonne2[C])
                        }
                    }
                }
            }
            }
            appliquerColonne(Co , colonne)
        }
    }
}






function déduireParExclusion(Ligne){
    if (Ligne[0]===Ligne[5] && Ligne[0]!==null && Ligne[1]!==null && Ligne[1]!==Ligne[0] && Ligne[2]===null && Ligne[3]===null && Ligne[4]===null){
        Ligne[4]=valeurOpposée(Ligne[0])
}
    if (Ligne[0]===Ligne[5] && Ligne[0]!==null && Ligne[4]!==null && Ligne[4]!==Ligne[0] && Ligne[2]===null && Ligne[3]===null && Ligne[1]===null){
    Ligne[1]=valeurOpposée(Ligne[5])
}
    if (Ligne[0]===Ligne[1] && Ligne[0]!==null && Ligne[2]!==null && Ligne[2]!==Ligne[0] && Ligne[3]===null && Ligne[5]===null && Ligne[4]===null){
    Ligne[5]=valeurOpposée(Ligne[0])
}
    if (Ligne[5]===Ligne[4] && Ligne[5]!==null && Ligne[3]!==null && Ligne[3]!==Ligne[5] && Ligne[0]===null && Ligne[1]===null && Ligne[2]===null){
    Ligne[0]=valeurOpposée(Ligne[5])
}
    if (Ligne[0]===Ligne[5] && Ligne[0]!==null && Ligne[1]===null && Ligne[2]===null && Ligne[3]===null && Ligne[4]===null){
    Ligne[1]=valeurOpposée(Ligne[0])
    Ligne[4]=valeurOpposée(Ligne[0])
}
}


function appliquerColonne(numéro , colonne){
    for (let L=0 ; L<6 ; L=L+1){
        grille.value[L][numéro]=colonne[L]
    }
}

function UnePasseDeDéduction(){
    for (let L=0 ; L<6 ; L=L+1){
        déduireParComptage(grille.value[L])
        déduireSandwich(grille.value[L])
        déduireAprèsPaire(grille.value[L])
        déduireAvantPaire(grille.value[L])
        déduireParExclusion(grille.value[L])
    }
    for (let C=0 ; C<6 ; C=C+1){
        const colonne = getColonne(C)
        déduireParComptage(colonne)
        déduireSandwich(colonne)
        déduireAprèsPaire(colonne)
        déduireAvantPaire(colonne)
        déduireParExclusion(colonne)
        appliquerColonne(C , colonne)
    }
    déduireComparaison()
    déduireComparaisonColonnes()
}

function compterCasesVides(){
    let count = 0
    for (let L=0 ; L<6 ; L=L+1){
        for (let C=0 ; C<6 ; C=C+1){
            if (grille.value[L][C] === null){
                count = count + 1
            }
        }
    }
    return count
}

function solveurLogique(){
    let casesVidesAvant=compterCasesVides()
    while (casesVidesAvant>0){
        UnePasseDeDéduction()
        let nouvellesCasesVides=compterCasesVides()
        if (nouvellesCasesVides===casesVidesAvant){
            return false
        }
        casesVidesAvant=nouvellesCasesVides
        }
    return true
}

/*function testerSolveurLogique(){
    générerGrilleAléatoire()
    const grilleAvant = JSON.parse(JSON.stringify(grille.value))
    const résultat = solveurLogique()
    if (résultat===true){
        console.log('la grille a été résolue par le solveur logique')
    }
    if (résultat===false){
        console.log('le solveur logique n\'a pas pu résoudre la grille')
    }
    console.log('grille avant le solveur logique :')
    console.log(grilleAvant)
    console.log('grille après le solveur logique :')
    console.log(grille.value)
}*/

function essayerCacherCase(L , C){
    const copieIndépendanteGrille = JSON.parse(JSON.stringify(grille.value))
    grille.value[L][C] = null
    const résultat = solveurLogique()
    grille.value = copieIndépendanteGrille
    if (résultat===true){
        grille.value[L][C] = null
    }
}

/* autres */

function partieGagnee(){
if (compterCasesVides()===0 && vérifierGrille()===true){
    return true
}
else {
    return false
}
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
                :class="{ 'bg-red-400': caseEnErreur(L, C) }"
                @click= "jouerCase(L , C)">
                {{ changer10enXO(L,C)}}
            </div>
        </template>
    </div>
    <div>
        <button 
        class="bg-blue-400 border-blue-500 rounded-2xl text-white p-2 m-2 hover:bg-blue-500"
        :class="{ 'bg-blue-500': mode === 0 }"
        @click="mode=0">Mode O</button>
        <button 
        class="bg-blue-400 border-blue-500 rounded-2xl text-white p-2 m-2 hover:bg-blue-500 "
        :class="{ 'bg-blue-500': mode === 1 }"
        @click="mode=1">Mode X</button>
    </div>
    <div>
        <p>Etat de la grille: {{ vérifierGrille() ? 'Valide' : 'Invalide' }}</p>
    </div>

    <!-- <button 
        class="bg-green-300 hover:bg-green-400 rounded-2xl p-2 m-2"
        @click="choisirGrille()">
        nouvelle grille
    </button> -->

    <div>
        <button
        class="bg-yellow-300 hover:bg-yellow-400 rounded-2xl p-2 m-2"
        @click="resetGrille()">
        recommencer
    </button>
    </div>
    <div>
        <button
        class="bg-purple-300 hover:bg-purple-400 rounded-2xl p-2 m-2"
        @click="générerGrilleAléatoire()">
        générer une grille aléatoire
        </button>
    </div>

        <!-- <button
        class="bg-pink-300 hover:bg-pink-400 rounded-2xl p-2 m-2"
        @click="testerSolveurLogique()">
        tester le solveur logique
        </button> -->

    <div>
        <p v-if="partieGagnee()">Félicitations ! Vous avez gagné !</p>
    </div>
    <div>
        <button
        class="bg-red-300 hover:bg-red-400 rounded-2xl p-2 m-2"
        @click="solveurLogique()">
        résoudre la grille
        </button>
    </div>
</template> 

