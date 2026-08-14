<script lang="ts" setup>
import { ref } from 'vue';

const grille = ref([
    [0, null, null, null, null, null],
    [null, 0, null, null, 1, 1],
    [null, null, null, null, null, null],
    [null, null, 1, 1, null, null],
    [null, null, null, null, 1, null],
    [null, 0, null, null, 0, 0],
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

const mode = ref<null | 0 | 1 | "effacer" >(null)

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
    if (mode.value === "effacer") {
        grille.value[L][C] = null
        return
    }
    if (mode.value === null) {
        return
    }
    grille.value[L][C] = mode.value
    sonClic.currentTime = 0
    sonClic.play()
    résoluParOrdinateur.value = false
    /*dernierCoup.value = {L ,C}*/
    if (partieGagnee()===true){
        sonWin.currentTime =0
        sonWin.play()
        arreterChrono()
    }
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
    résoluParOrdinateur.value = false
    démarrerChrono()
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
    résoluParOrdinateur.value = false
    démarrerChrono()
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
    while (casesCachées < nombreDeCaseACacher.value && tentatives < 200) {
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

let résoluParOrdinateur = ref(false)

function résoudreClic(){
    solveurLogique()
    résoluParOrdinateur.value = true
    arreterChrono()
}

let afficherRègles= ref(false)

const sonClic = new Audio ("/sons/clic.wav")

const sonWin = new Audio ("/sons/victoire.wav")

function indice(){
    const sauvegarde = JSON.parse(JSON.stringify(grille.value))
    solveurLogique()
    const candidats = []
    for (let L=0 ; L<6 ; L=L+1){
        for (let C=0 ; C<6 ; C=C+1){
            if (sauvegarde[L][C]===null && grille.value[L][C]!==null) {
                candidats.push({L, C})
            }
        }
    }
    const choixAléatoireCandidat = Math.floor(Math.random() * candidats.length)
    const caseChoisie =candidats[choixAléatoireCandidat]
    const valeurTrouvée = grille.value[caseChoisie.L][caseChoisie.C]
    grille.value = sauvegarde
    grille.value[caseChoisie.L][caseChoisie.C] = valeurTrouvée
}

const secondesEcoulées = ref(0)

let idChrono = null 

function démarrerChrono (){
    clearInterval(idChrono)
    secondesEcoulées.value = 0
    idChrono = setInterval(()=>{secondesEcoulées.value = secondesEcoulées.value+1},1000)
}

function arreterChrono(){
    clearInterval(idChrono)
}

function tempsFormaté () {
    let minute= Math.floor(secondesEcoulées.value / 60)
    let seconde= secondesEcoulées.value % 60
    let temps= minute + " : " + seconde 
    return temps
}

/* page d'accueil */ 

const écran=ref("accueil")

const afficherChoixDifficulté = ref(false)

const nombreDeCaseACacher = ref(20)

</script>

<template>
    <div class="bg-rose-50 flex items-center justify-center min-h-screen">
    <div class="flex flex-col items-center justify-center py-8  ">
        <button @click="afficherRègles= !afficherRègles"
        class="text-xs text-gray-500 gap-y-2 bg-gray-200 rounded-2xl border-1 border-gray-300 w-15 h-5 m-1">
            Règles
        </button>
            <p v-if="afficherRègles" class="text-xs text-gray-500 m-1 text-center">
            · pas plus de 2 symboles identiques à la suite <br>
            · autant de X que de O par ligne/colonne <br>
            · deux lignes ou deux colonnes ne peuvent pas être identiques.
        </p>
    <div v-if="écran==='jeu'" 
    class="bg-white w-110 rounded-3xl shadow-md p-4 flex flex-col items-center justify-center gap-4">
    <div>
        <h1 class="text-2xl font-bold mb-4">Binoxxo</h1>
    </div>
    <div class="grid grid-cols-6 gap-1 w-fit">
        <template v-for="(Ligne, L) in grille" :key="L" >
            <div v-for="(Case_, C) in Ligne" :key="C" 
                class="w-10 h-10 border border-gray-300 text-center rounded-lg"
                :class="{ 'bg-red-400': caseEnErreur(L, C), 'bg-amber-200': Ligne[C] === 0 && caseModifiable(L,C)===false && caseEnErreur(L,C)===false, 'bg-teal-200': Ligne[C] === 1 && caseModifiable(L,C)===false && caseEnErreur(L,C)===false,'bg-gray-100': caseModifiable(L,C)===true && caseEnErreur(L,C)===false}"
                @click= "jouerCase(L , C)">
                {{ changer10enXO(L,C)}}
            </div>
        </template>
    </div>
    <div>
        <p class="px-3 py-1 rounded-full text-sm mt-2"
        :class="{ 'bg-green-200 text-green-800': vérifierGrille() === true, 'bg-red-200 text-red-800': vérifierGrille() === false }"
        >
        Grille: {{ vérifierGrille() ? 'Valide' : 'Invalide' }}</p>
    </div>
    <div>
        <button 
        class="bg-rose-200 border-1 border-pink-300 rounded-2xl text-white p-2 mx-2 hover:bg-rose-300 "
        :class="{ 'bg-rose-300': mode === 0 }"
        @click="mode=0">Mode O</button>
        <button 
        class="bg-rose-200 border-1 border-pink-300 rounded-2xl text-white p-2 mx-2 hover:bg-rose-300 "
        :class="{ 'bg-rose-300': mode === 1 }"
        @click="mode=1">Mode X</button>
        <button 
        class="bg-rose-200 border-1 border-pink-300 rounded-2xl text-white p-2 mx-2 hover:bg-rose-300 "
        :class="{ 'bg-rose-300': mode === 'effacer'}"
        @click="mode='effacer'">Effacer</button>
    </div>
    


    <!-- <button 
        class="bg-green-300 hover:bg-green-400 rounded-2xl p-2 mx-2"
        @click="choisirGrille()">
        nouvelle grille
    </button> -->

    <div>
        <button
        class="bg-yellow-200 hover:bg-yellow-300 border-1 border-yellow-300 rounded-2xl p-2 mx-2"
        @click="resetGrille()">
        recommencer
        </button>

        <button
        class="bg-purple-200 hover:bg-purple-300 border-1 border-purple-300 rounded-2xl p-2 mx-2"
        @click="générerGrilleAléatoire()">
        grille aléatoire
        </button>

        <button
        class="bg-red-200 hover:bg-red-300 border-1 border-red-300 rounded-2xl p-2 mx-2"
        @click="résoudreClic()">
        résoudre
        </button>
    </div>
    <button
        class="bg-green-200 hover:bg-green-300 border-1 border-green-300 rounded-2xl p-2 mx-2 "
        @click="indice()" >
        indice
    </button>
    <div>
        <p>{{ tempsFormaté() }}</p>
    </div>

        <!-- <button
        class="bg-pink-300 hover:bg-pink-400 rounded-2xl p-2 mx-2"
        @click="testerSolveurLogique()">
        tester le solveur logique
        </button> -->

    <div>
        <p v-if="partieGagnee() && résoluParOrdinateur === false" class="text-green-700 text-bold text-xl border-2 border-green-300 bg-green-200 rounded-2xl p-4">Félicitations ! Vous avez gagné ! 🎉</p>
    </div>
    <div>
        <p v-if="résoluParOrdinateur === true" class="text-blue-700 text-bold text-xl border-2 border-blue-300 bg-blue-200 rounded-2xl p-4">La grille a été résolue par l'ordinateur ! 🤖</p>
    </div>
    </div>

    <div v-if="écran==='accueil'" class="bg-white rounded-3xl shadow-md p-4 flex flex-col items-center gap-4 w-110 min-h-96 ">
        <h1 class="text-5xl">Binoxxo</h1>
        <button @click="afficherChoixDifficulté=true" 
        class="bg-teal-200 border-1 border-teal-300 rounded-full hover:bg-teal-300 p-2 text-3xl">
            ▶︎
        </button>
        <div v-if="afficherChoixDifficulté">
            <button class="bg-green-200 border-1 border-green-300 rounded-2xl hover:bg-green-300 p-2 mx-2"
            @click="nombreDeCaseACacher=15; générerGrilleAléatoire() ; écran='jeu'">
                facile
            </button>
            <button class="bg-yellow-200 border-1 border-yellow-300 rounded-2xl hover:bg-yellow-300 p-2 mx-2"
            @click="nombreDeCaseACacher=20; générerGrilleAléatoire() ;écran='jeu'">
                moyen
            </button>
            <button class="bg-rose-200 border-1 border-rose-300 rounded-2xl hover:bg-rose-300 p-2 mx-2"
            @click="nombreDeCaseACacher=26; générerGrilleAléatoire() ;écran='jeu'">
                difficile
            </button>

        </div>
        
    </div>

    </div>
    </div>
</template> 

