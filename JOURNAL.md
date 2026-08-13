# Journal de bord

## 2026-07-20 — Début du Binoxxo

### Ce qui a été codé (commit `0dce8b3` — "Début binoxxo 2ère règles")
- Remplacement du composant affiché dans `App.vue` : `jetonretourner` → `binoxxo`.
- Nouveau composant `src/components/binoxxo.vue` :
  - Grille 6×6 codée en dur (`grille`), cases contenant `0`, `1` ou `null` (vide).
  - Affichage de la grille en HTML/Tailwind (`grid-cols-6`).
  - `afficherLigne` / `afficherColonne` : fonctions de debug qui affichent une ligne/colonne dans la console.
  - `getColonne` : récupère une colonne sous forme de tableau.
  - `pasdeTriplet` : vérifie qu'une ligne/colonne n'a pas 3 valeurs identiques consécutives.
  - `equilibre` : vérifie qu'une ligne/colonne ne contient pas plus de 3 zéros ou plus de 3 uns.
  - `vérifierGrille` : combine les deux règles ci-dessus sur toutes les lignes et colonnes.

### Ce qui a été discuté (sessions "Maya TM_Binoxxo", dossier `TM-Binoxxo`)
- Contexte posé : ce projet est le travail de maturité, l'objectif est d'apprendre à coder avec l'IA — donc l'IA explique et guide, mais **n'écrit ni ne modifie le code** à la place de Maya.
- Relecture pédagogique de `equilibre` : remarque que la fonction ne compte que les `0` (`nb0`), question posée sur si c'est suffisant et sur le traitement des cases vides (`null`).
- Conseil : lancer `npm run dev` régulièrement pour voir l'état réel de l'app pendant le développement.
- Prochaine étape suggérée : terminer soi-même la fonction `equilibre(ligne)`, avec des questions pour guider la réflexion (pas de code fourni).

## 2026-07-26 — Grille interactive et jouable

### Décisions prises
- La 3ème règle du Binoxxo (unicité des lignes/colonnes) est mise de côté pour l'instant, priorité donnée à l'interactivité de la grille.
- Choix de conception pour jouer un coup : **option A**, un mode global (bouton "0" / bouton "1") plutôt qu'un bouton par case — on sélectionne d'abord une valeur, puis on cliquera sur une case pour l'y placer.

### Ce qui a été codé dans `src/components/binoxxo.vue`
- Nouvelle variable réactive `mode` : `const mode = ref<null | 0 | 1>(null)` — retient quelle valeur (0, 1, ou aucune) est sélectionnée pour le prochain clic sur une case.
- Deux boutons "Mode 0" / "Mode 1" (avec mise en forme Tailwind) qui mettent `mode` à `0` ou `1` au clic (`@click="mode=0"` / `@click="mode=1"`).
- Affichage temporaire de vérification : `<p>Mode actuel: {{ mode ?? 'Aucun' }}</p>` pour confirmer visuellement que les boutons changent bien `mode`. Testé avec `npm run dev` — fonctionne.

### Notions expliquées pendant la session
- Rôle de `ref()` et pourquoi/quand ajouter un type générique (`ref<Type>(valeur)`) plutôt que laisser TypeScript deviner le type tout seul.
- Syntaxe des types union avec `|` (ex. `null | 0 | 1` = "soit l'un, soit l'autre, rien d'autre").
- Différence entre une chaîne de caractères `"null"`/`'0'`/`'1'` et les vraies valeurs `null`/`0`/`1` — piège repéré et corrigé dans la déclaration de `mode`.
- Fonctionnement de `@click` en Vue (exécute une instruction ou appelle une fonction au clic), et règle du `.value` : nécessaire dans le `<script>`, mais Vue l'enlève automatiquement dans le `<template>`.

### Suite de la session : cases fixes vs modifiables, et clic sur une case
- Nouvelle constante `copieGrilleInitiale = JSON.parse(JSON.stringify(grille.value))`, créée juste après `grille` : une copie indépendante de l'état de départ, pour toujours pouvoir savoir quelles cases étaient vides au départ (donc jouables) et lesquelles faisaient partie de l'énoncé (donc fixes).
  - Première tentative avec `structuredClone(grille.value)`, qui plantait l'app entièrement (page blanche) avec l'erreur `DataCloneError`. Cause : `grille.value` n'est pas un tableau "normal" mais un proxy réactif de Vue, que `structuredClone` ne sait pas cloner. Remplacé par `JSON.parse(JSON.stringify(...))`, qui ne garde que les données brutes (nombres/`null`) sans le mécanisme réactif — ça a résolu le plantage.
- Fonction `caseModifiable(L, C)` : retourne `true`/`false` selon que la case était `null` dans `copieGrilleInitiale`.
- Fonction `jouerCase(L, C)` : ne fait rien (`return` seul) si la case n'est pas modifiable, ni si aucun mode n'est sélectionné (`mode.value === null`) ; sinon, met à jour `grille.value[L][C] = mode.value`.
- `@click="jouerCase(L, C)"` ajouté sur la `div` de chaque case dans le `<template>`, au même niveau que `class` et `:key` (pas comme contenu affiché).
- **Résultat testé et fonctionnel** : la grille est maintenant jouable — on choisit un mode (0 ou 1), on clique sur une case vide pour la remplir, et les cases de départ restent bien protégées contre les clics.

### Notions expliquées (suite)
- Structure d'une balise HTML : attributs (`class`, `:key`, `@click`) dans la balise ouvrante, contenu affiché entre balise ouvrante et fermante.
- Les deux boucles `v-for` imbriquées (lignes puis colonnes) pour parcourir la grille dans le template.
- `return` seul (sans valeur) pour arrêter l'exécution d'une fonction ("ne rien faire") à une condition donnée.
- Pourquoi `structuredClone` échoue sur les données réactives de Vue, et pourquoi `JSON.parse(JSON.stringify(...))` fonctionne pour une grille de nombres/`null`.
- Méthode de débogage : lire les erreurs dans le terminal (erreurs de compilation) vs dans la console du navigateur (erreurs d'exécution), et l'utilité de vider la console + recharger la page après un plantage.

### Prochaine étape (à faire la prochaine fois)
- Relier `vérifierGrille()` à l'affichage, pour montrer si la grille est valide/invalide en direct pendant que Maya joue.
- Implémenter la 3ème règle du Binoxxo (unicité des lignes/colonnes), mise de côté jusqu'ici.

## 2026-07-28 — Affichage de la validité + case en erreur en rouge

### Ce qui a été codé dans `src/components/binoxxo.vue`
- Affichage en direct de la validité de la grille : `<p>Etat de la grille: {{ vérifierGrille() ? 'Valide' : 'Invalide' }}</p>`. Comme `vérifierGrille()` lit `grille.value`, Vue réévalue automatiquement ce texte à chaque coup joué.
- Feedback visuel en rouge sur la dernière case jouée si elle casse une règle :
  - `dernierCoup` : `const dernierCoup = ref<null | {L: number, C: number}>(null)`, mis à jour dans `jouerCase` (`dernierCoup.value = {L, C}`) à chaque coup valide joué.
  - `dernierCoupErreur(L, C)` : retourne `true` seulement si `dernierCoup.value` correspond exactement à la case `(L, C)` **et** que `vérifierGrille()` est `false` — simplification décidée pendant la session : pas besoin de revérifier la ligne/colonne isolément, puisqu'un coup ne modifie qu'une case, donc si la grille devient invalide juste après ce coup, le problème vient forcément de sa ligne ou de sa colonne.
  - `:class="{ 'bg-red-200': dernierCoupErreur(L, C) }"` ajouté sur la `div` de chaque case, en plus de la `class` statique existante (Vue fusionne les deux automatiquement).
- **Résultat testé et fonctionnel** : le texte "Valide"/"Invalide" se met à jour en direct, et la dernière case jouée s'affiche en rouge quand elle casse une règle.

### Notions expliquées pendant la session
- L'opérateur ternaire `condition ? valeurSiVrai : valeurSiFaux` (raccourci d'un `if/else` qui produit une valeur), comparé à l'opérateur `??` déjà connu.
- Le style conditionnel en Vue avec `:class="{ 'classe': condition }"`, et le fait que `class` (statique) et `:class` (dynamique) peuvent coexister sur une même balise.
- Accès aux propriétés d'un objet stocké dans un `ref` (ex. `dernierCoup.value.L`), et pourquoi comparer à la fois la ligne **et** la colonne est nécessaire pour identifier une case unique parmi les 36 (comparer une seule coordonnée désignerait toute une ligne ou toute une colonne, pas une case précise).
- Pourquoi le code d'une fonction (comme `jouerCase`) peut utiliser une variable déclarée plus loin dans le fichier (comme `dernierCoup`) sans planter : le corps de la fonction ne s'exécute qu'au moment de l'appel (au clic), pas à la déclaration — bonne pratique de lisibilité malgré tout de déclarer les variables avant.

### Prochaine étape (à faire la prochaine fois)
- Implémenter la 3ème règle du Binoxxo (unicité des lignes/colonnes), toujours en attente.
- Réfléchir à l'objectif à plus long terme de génération aléatoire de grilles (évoqué précédemment), une fois les règles de validation complètes.

## 2026-07-29 — 3ème règle : unicité des lignes/colonnes

### Décision prise
- La règle ne s'applique qu'aux lignes/colonnes **complètes** (sans `null`) : deux lignes encore partiellement vides ne sont pas comparées, pour éviter de signaler une erreur qui n'en est pas encore une.
- Le peaufinage visuel de la case en rouge (`dernierCoupErreur`) est mis en pause pour l'instant — le code correspondant a été commenté dans `binoxxo.vue` (fonction + `:class` retirée du template), à reprendre plus tard.

### Ce qui a été codé dans `src/components/binoxxo.vue`
- `ligneComplete(Ligne)` : retourne `false` dès qu'un `null` est trouvé dans les 6 cases, `true` sinon. Fonctionne aussi bien pour une ligne que pour une colonne (peu importe l'origine du tableau passé en paramètre).
- `ligneIdentique(Ligne1, Ligne2)` : compare deux tableaux de 6 valeurs case par case, retourne `false` dès qu'une différence est trouvée, `true` si tout est identique.
- `unicitéLignes()` : double boucle (`Lx` de 0 à 5, `Ly` de `Lx+1` à 5, pour ne comparer chaque paire qu'une seule fois et ne jamais comparer une ligne à elle-même) qui retourne `false` si deux lignes complètes et identiques sont trouvées.
- `unicitéColonnes()` : même principe, en réutilisant `getColonne(Cx)`/`getColonne(Cy)` à la place des lignes.
- Les deux fonctions intégrées dans `vérifierGrille()`, avec le même pattern que les vérifications existantes (`if (xxx() === false) { return false }`).
- **Résultat testé et fonctionnel** : les 3 règles du Binoxxo sont maintenant toutes prises en compte dans `vérifierGrille()` (pas de triplet, équilibre des 0/1, unicité des lignes/colonnes complètes).

### Notions expliquées pendant la session
- En JavaScript, `===` sur des tableaux compare la référence (le même tableau en mémoire), pas le contenu — deux tableaux avec les mêmes valeurs mais séparés sont considérés différents. D'où la nécessité de comparer case par case pour une vraie égalité de contenu.
- Technique de la double boucle avec la deuxième boucle démarrant après l'index de la première (`Ly = Lx+1`) pour comparer chaque paire d'éléments une seule fois, sans comparer un élément à lui-même.
- Une fonction qui vérifie "une seule ligne/paire" (comme `ligneComplete`, `ligneIdentique`) est une brique séparée de la fonction qui boucle sur toute la grille (comme `unicitéLignes`) — même logique déjà présente entre `pasdeTriplet`/`equilibre` et `vérifierGrille`.

### Prochaine étape (à faire la prochaine fois)
- Reprendre et peaufiner l'affichage visuel des erreurs (la case en rouge, actuellement en commentaire dans le code).
- Réfléchir à la génération aléatoire de grilles, maintenant que les 3 règles de validation sont complètes.

## 2026-07-31 — Plusieurs grilles + tirage aléatoire, nouveau système d'erreur, reset

### Première partie : choisir une grille au hasard parmi plusieurs
- Nouveau tableau `grillesDisponibles` : contient 3 grilles complètes (la grille d'origine + 2 nouvelles écrites par Maya), structure "tableau de grilles" (un niveau d'imbrication de plus que `grille` seule).
- `choisirGrille()` : tire un index aléatoire avec `Math.floor(Math.random() * grillesDisponibles.length)`, puis remplace `grille.value` par une copie indépendante (`JSON.parse(JSON.stringify(...))`) de la grille choisie.
- `copieGrilleInitiale` changée de `const` à `let` (pour pouvoir être réassignée), et mise à jour à l'intérieur de `choisirGrille()` avec une copie indépendante de la nouvelle grille — sinon `caseModifiable` continuerait à se baser sur l'ancienne grille de départ après un changement.
- Bug corrigé en cours de route : `grille` avait été commentée par erreur en créant `grillesDisponibles`, provoquant un crash (`grille is not defined`) — remise en place.
- Bug de syntaxe corrigé : `<bouton>` (français) n'est pas une vraie balise HTML, remplacé par `<button>`.
- Discussion sur les limites de cette approche : ce n'est pas une vraie génération aléatoire (juste un tirage parmi un choix limité et fixe de grilles pré-écrites), la vraie génération algorithmique (par backtracking/récursion) reste un chantier à part, plus complexe, remis à plus tard.

### Deuxième partie : refonte du système de case en erreur (rouge)
- Constat de Maya en testant l'ancienne version (`dernierCoupErreur`, basée sur la dernière case jouée) : elle n'était pas persistante (s'éteignait dès qu'on jouait ailleurs) et pouvait désigner la mauvaise case (ex. le 4ème `1` d'une ligne s'allumait seul, alors que les 4 cases sont collectivement responsables du problème).
- Grande discussion sur deux approches possibles pour la suite :
  - **Approche A (comparaison à une solution connue)** : plus précise, mais suppose d'avoir toujours la solution complète de la grille jouée en mémoire, et soulève un vrai problème d'**unicité de solution** (surtout pour les futures grilles générées aléatoirement) — nécessiterait un second algorithme (un "vérificateur d'unicité", lui-même basé sur du backtracking) pour être fiable, chantier non négligeable.
  - **Approche B (basée sur les règles)** : reprend `pasdeTriplet`/`equilibre` appliqués à la ligne/colonne d'une case précise, indépendamment de qui a joué en dernier. Moins précise (désigne un groupe de cases responsables, pas forcément LA case fautive), mais robuste peu importe le nombre de solutions possibles, et réutilise le travail déjà fait.
  - Décision : partir sur l'approche B pour avoir quelque chose de solide maintenant, l'approche A restant une piste d'amélioration future si l'algorithme de génération avec garantie d'unicité est fait un jour.
- `caseEnErreur(L, C)` : retourne `true` si la ligne `L` ou la colonne `C` de cette case casse `pasdeTriplet` ou `equilibre` (combinaison de 4 conditions avec `||`).
- `:class="{ 'bg-red-400': caseEnErreur(L, C) }"` reliée dans le template (remplace l'ancien `dernierCoupErreur`, resté commenté avec `dernierCoup`).
- Discussion sur le fonctionnement réel : `caseEnErreur` est appelée indépendamment 36 fois (une fois par case, via les `v-for`) ; une ligne entière s'allume parce que chaque case de cette ligne pose la même question sur la même ligne et obtient la même réponse — pas parce qu'il existe un mécanisme qui "peint" la ligne entière d'un coup.

### Troisième partie : bouton de réinitialisation
- `resetGrille()` : remet `grille.value` à une copie indépendante de `copieGrilleInitiale` (encore une fois via `JSON.parse(JSON.stringify(...))`, en évitant le piège du partage de référence).
- Bouton "recommencer" ajouté dans le template, relié à `resetGrille()`.
- **Résultat testé et fonctionnel** : tirage aléatoire de grille, coloration des lignes/colonnes en erreur, et réinitialisation de la grille en cours, tout fonctionne.

### Notions expliquées pendant la session
- Différence entre une variable non déclarée (erreur en mode module JS/TS) et une déclaration `const`/`let`.
- Pourquoi appeler `JSON.parse(JSON.stringify(...))` deux fois de suite est redondant (une seule fois suffit pour une copie indépendante).
- Le principe d'évaluation indépendante d'une expression répétée dans une boucle (`v-for`) : la même condition, évaluée plusieurs fois avec des données différentes, peut donner la même réponse sans qu'il y ait de logique centralisée.

### Prochaine étape (à faire la prochaine fois)
- Vrai algorithme de génération aléatoire de grilles complètes (backtracking/récursion), à la place du simple tirage parmi des grilles pré-écrites.
- Si envie d'aller plus loin un jour : vérificateur d'unicité de solution, pour permettre l'approche A (comparaison à la solution) de façon fiable.

## 2026-08-07 — Début de l'algorithme de génération (backtracking)

### Contexte et plan
- Début du chantier "vrai algorithme de génération aléatoire" (remplacer le simple tirage parmi 3 grilles pré-écrites par une vraie génération case par case).
- Rappel/confirmation du concept de récursion (déjà vu en Python à l'école) : une fonction qui s'appelle elle-même, avec toujours une condition d'arrêt pour éviter une boucle infinie.
- Plan détaillé posé pour la suite (découpage en petites étapes, explication avant écriture, Maya écrit tout le code, tests fréquents) :
  1. Calculer la "case suivante" à partir d'une case donnée.
  2. Cas de base (fin de grille atteinte).
  3. Fonction récursive principale : essayer une valeur, vérifier avec `vérifierGrille()`, avancer à la case suivante si valide.
  4. Essayer les deux valeurs (0 et 1) avant d'abandonner une case.
  5. Vrai retour en arrière (undo + signal d'échec) si aucune valeur ne fonctionne.
  6. Bouton pour déclencher la génération.
  7. Tests et débogage.
  8. (Plus tard) Cacher des cases dans la grille complète générée pour créer un vrai puzzle.
- Bonne nouvelle réalisée pendant la session : `pasdeTriplet`, `equilibre`, `unicitéLignes`/`unicitéColonnes` géraient déjà bien les grilles partiellement remplies (les `null`) — donc `vérifierGrille()` peut être réutilisée telle quelle comme "test de cohérence" à chaque étape du remplissage, sans rien réécrire.

### Ce qui a été codé dans `src/components/binoxxo.vue`
- `caseSuivante(L, C)` : calcule les coordonnées de la case suivante en remplissant ligne par ligne (colonne suivante, ou ligne suivante si on est en bout de ligne), et retourne `null` quand on dépasse la dernière case de la grille (signal "tout est rempli", qui servira de cas de base pour la récursion).

### Prochaine étape (à faire la prochaine fois)
- Écrire `remplirGrille(L, C)`, la fonction récursive principale : essayer `0`, vérifier avec `vérifierGrille()`, avancer avec `caseSuivante` si valide (en s'arrêtant juste après le calcul de la case suivante pour cette première session de code) ; puis compléter avec l'essai du `1`, et le vrai retour en arrière (undo + `return false`) si aucune valeur ne convient.

## 2026-08-08 — Algorithme de génération terminé et fonctionnel 🎉

### Ce qui a été codé dans `src/components/binoxxo.vue`
- `remplirGrille(L, C)` terminée : fonction récursive de backtracking qui remplit la grille case par case.
  - Tirage aléatoire de l'ordre d'essai des valeurs à chaque case (`choix1ou0 = Math.random()`, puis `premierEssai`/`deuxiemeEssai` déterminés par un ternaire) — ajouté en cours de route suite à une remarque de Maya : sans ça, l'algorithme aurait toujours essayé 0 avant 1 dans le même ordre de cases, donc produit systématiquement la même grille (pas vraiment aléatoire).
  - Pour chaque valeur essayée : pose la valeur, vérifie avec `vérifierGrille()` (réutilisée telle quelle, elle gère déjà bien les grilles partiellement remplies), avance sur la case suivante via `caseSuivante` si valide, et propage `true` en cas de succès jusqu'au bout de la grille (`caseSuivante` renvoyant `null`).
  - Si aucune des deux valeurs ne fonctionne : remet la case à `null` et renvoie `false` — le vrai "retour en arrière" (backtracking).
- `grilleVide()` : fonction qui retourne une grille 6×6 neuve remplie uniquement de `null`, pour repartir de zéro avant chaque génération.
- `générerGrilleAléatoire()` : combine les deux — réinitialise `grille.value` avec `grilleVide()`, puis lance `remplirGrille(0, 0)`.
- Bouton "générer une grille aléatoire" ajouté dans le template.
- **Résultat testé et fonctionnel** : clic sur le bouton → grille complète, valide, générée aléatoirement (grilles différentes à chaque clic). Premier vrai algorithme de génération du projet, plus besoin de piocher parmi des grilles pré-écrites.

### Bugs corrigés en cours de route
- `caseSuivante` (écrite la session précédente) n'avait pas été sauvegardée sur le disque — juste un oubli de `Cmd+S`, pas un vrai bug.
- Première tentative de `remplirGrille` : essayer `0` et `1` dans des blocs `if`/sinon séparés faisait qu'une des deux valeurs n'était **jamais** testée dans la moitié des cas (bug de logique) — corrigé en généralisant avec `premierEssai`/`deuxiemeEssai`, deux blocs symétriques peu importe l'ordre tiré.
- Condition finale `if (vérifierGrille() === false)` avant le retour en arrière : inutile et bugguée (ne se déclenchait pas dans certains cas d'échec légitimes) — simplifiée en un reset + `return false` inconditionnel, puisqu'arriver à ces lignes signifie déjà que rien n'a fonctionné.
- `grilleVide()` : oubli du mot-clé `return` (le tableau était construit mais jamais renvoyé), puis confusion entre `return grilleVide` (qui renvoie la fonction elle-même) et `return` suivi directement du tableau.

### Notions expliquées pendant la session
- Confirmation/rappel de la récursion (déjà vue en Python à l'école) : une fonction qui s'appelle elle-même, avec une condition d'arrêt obligatoire.
- Pourquoi `return true` doit remonter explicitement à chaque étage de la récursion pour qu'un succès en profondeur soit reconnu par tous les appels précédents (sinon ils continueraient à chercher inutilement).
- Le mécanisme concret du retour en arrière : chaque appel de `remplirGrille` reste "en attente" pendant son appel récursif ; quand celui-ci renvoie `false`, l'exécution reprend simplement à la ligne suivante dans la même fonction (essayer l'autre valeur) — pas de mécanisme magique séparé, juste la suite normale du code après un appel qui n'a pas satisfait la condition.
- Différence entre déclarer une constante (créée une fois) et une fonction (recrée sa valeur à chaque appel) pour éviter le piège du partage de référence, appliqué ici à `grilleVide()`.

### Prochaine étape (à faire la prochaine fois)
- Cacher certaines cases de la grille complète générée pour en faire un vrai puzzle jouable (au lieu d'une grille toujours entièrement remplie).
- Réfléchir, si envie d'aller plus loin, à la question de l'unicité de la solution une fois des cases cachées.

## 2026-08-09 — Cases cachées, affichage X/O, et début d'un vrai solveur logique

### Cacher des cases pour créer un vrai puzzle
- `cacherUneCase()` : tire une case au hasard (avec une boucle `while`, nouveau type de boucle appris — répète tant qu'une condition reste vraie, contrairement à `for` qui répète un nombre de fois fixé à l'avance) jusqu'à tomber sur une case pas encore cachée, puis la vide.
  - Bug corrigé : les coordonnées de départ étaient initialisées à `(0, 0)` au lieu d'un premier tirage aléatoire, ce qui faisait que le tout premier appel cachait toujours la même case au lieu d'une case aléatoire.
- `cacherPlusieursCases()` : répète `cacherUneCase()` un nombre de fois aléatoire (`nombreACacher`, recalculé à chaque appel — attention, une variable déclarée en dehors d'une fonction ne se "rafraîchit" pas toute seule), puis met à jour `copieGrilleInitiale` avec la nouvelle grille (cases cachées comprises), sinon `caseModifiable` resterait basée sur l'ancien état.
- Intégré dans `générerGrilleAléatoire()` : grille vide → remplissage complet par backtracking → puis cases cachées. Premier vrai puzzle généré de bout en bout.

### Affichage X/O au lieu de 0/1
- Discussion sur deux options : changer les valeurs stockées partout (risqué, casserait toutes les fonctions de règles qui comparent à `0`/`1`) vs changer uniquement l'affichage (choisi).
- `changer10enXO(L, C)` : retourne `'O'` pour `0`, `'X'` pour `1`, `''` pour une case vide — utilisée dans le template à la place de `{{ Case_ ?? '.' }}`.

### Investigation d'un faux "Invalide"
- Maya a résolu une grille écrite à la main (tirée d'un journal papier) correctement (vérifiée avec le corrigé), mais le jeu affichait quand même "Invalide", sans case rouge.
- Recherche des règles officielles du Binoxxo/Takuzu : confirmé que la règle d'unicité s'applique **aussi bien aux colonnes qu'aux lignes** (`unicitéColonnes`, qui n'a pas d'équivalent visuel rouge puisque `caseEnErreur` ne vérifie que `pasdeTriplet`/`equilibre`, pas l'unicité).
- Cause trouvée : deux colonnes de la grille papier étaient identiques — le journal source utilisait des règles simplifiées (uniquement l'unicité des lignes, pas des colonnes). Conclusion : pas de bug dans le code, juste une différence de règles entre la source papier et l'implémentation (plus complète/standard).

### Grande réflexion : "résolvable" vs "solution unique" vs "résolvable par pure logique"
- En essayant de résoudre une grille générée avec peu de cases visibles, Maya s'est retrouvée bloquée sans pouvoir déduire la suite — a soulevé une distinction importante :
  - **Résolvable** (au moins une solution existe) : déjà garanti automatiquement, puisqu'on cache des cases à partir d'une grille déjà complète et valide.
  - **Solution unique** (vérificateur d'unicité, évoqué avant) : garantit qu'il n'existe qu'UNE grille valide possible, mais pas que cette solution soit trouvable par un raisonnement logique simple — elle pourrait exiger de deviner puis vérifier.
  - **Résolvable par pure déduction logique** (ce que Maya veut vraiment, comme les vrais puzzles de journaux) : à chaque étape, on doit pouvoir déduire la valeur d'au moins une case avec certitude, sans jamais avoir à deviner.
- Recherche des techniques de résolution logique standards du Binairo/Takuzu (confirmé : *"Binairo puzzles can be solved using logic and deduction, and do not require any guessing or trial-and-error"*), quatre techniques identifiées :
  1. Paire adjacente (`X X _` ou `_ X X`) → force la case d'à côté à la valeur opposée.
  2. "Sandwich" (`X _ X`) → force la case du milieu à la valeur opposée.
  3. Comptage → si une ligne/colonne a déjà 3 fois un symbole, le reste doit être l'autre symbole.
  4. Déduction par unicité (technique avancée, mise de côté pour l'instant).
- Nouveau plan : construire un "solveur logique" (différent du générateur par backtracking) qui applique ces techniques en boucle jusqu'à blocage ou grille complète, pour ensuite s'en servir pendant la génération : ne cacher une case que si le solveur logique arrive encore à tout déduire sans elle.

### Ce qui a été codé dans `src/components/binoxxo.vue` (début du solveur logique)
- `déduireParComptage(Ligne)` : compte les `0`/`1` d'une ligne, puis (dans une deuxième boucle séparée, après le comptage complet) remplit les cases vides restantes avec le symbole manquant si l'autre a déjà atteint 3.
- `valeurOpposée(valeur)` : retourne l'inverse de `0`/`1`, utilisée par les fonctions suivantes.
- `déduireSandwich(Ligne)` : détecte le motif `X _ X` et déduit la case du milieu. Bug corrigé en cours de route : sans vérifier que les voisins ne sont pas `null`, `null === null` se serait évalué à vrai par erreur.
- `déduireAprèsPaire(Ligne)` : détecte `X X _` (paire avant la case vide). Bug de borne corrigé (la boucle commençait à `C=3` au lieu de `C=2`, ratant un cas valide).
- `déduireAvantPaire(Ligne)` : détecte `_ X X` (paire après la case vide) — écrite en autonomie, correcte du premier coup.

### Prochaine étape (à faire la prochaine fois)
- Assembler les 4 fonctions de déduction en un vrai solveur qui les applique à **toutes** les lignes et colonnes, en boucle, jusqu'à grille complète ou blocage.
- Utiliser ce solveur pendant `cacherPlusieursCases()` : ne cacher une case que si le solveur logique peut encore tout déduire sans elle, sinon la remettre.
- Petit oubli remarqué (pas encore signalé à Maya) : le type de `mode` a été changé en `ref<null | 'O' | 'X'>(null)`, mais les boutons lui assignent toujours `0`/`1` (`@click="mode=0"`) — incohérence de typage à nettoyer, sans impact fonctionnel pour l'instant.

## 2026-08-10 — Solveur logique terminé, intégré à la génération, et nouvelle technique perso

### Assemblage final du solveur logique
- `appliquerColonne(numéro, colonne)` : l'inverse de `getColonne` — réécrit un tableau de colonne (modifié par les déductions) dans la vraie `grille.value`, colonne par colonne. Nécessaire car `getColonne` ne renvoie qu'une copie, pas un lien vers la grille.
- `UnePasseDeDéduction()` : applique les 4 (puis 5, voir plus bas) techniques de déduction à chacune des 6 lignes, puis à chacune des 6 colonnes (récupérées avec `getColonne`, puis réécrites avec `appliquerColonne`).
- `compterCasesVides()` : compte le nombre total de `null` dans toute la grille.
- `solveurLogique()` : boucle sur `UnePasseDeDéduction()` jusqu'à ce que la grille soit complète (`true`) ou qu'une passe entière ne fasse plus progresser le compte de cases vides (`false`, bloqué). Plusieurs bugs de "shadowing" corrigés en cours de route (des `let` redéclarés par erreur à l'intérieur de boucles, qui créaient des variables séparées au lieu de réutiliser les bonnes) — même piège rencontré et corrigé plusieurs fois dans la session.
- Testé avec un bouton temporaire `testerSolveurLogique()` (généré une grille, lancé le solveur, comparé avant/après dans la console) — confirmé que le solveur fait de vraies déductions (progrès visible), réussit parfois, échoue parfois selon la répartition aléatoire des cases cachées à ce stade.

### Grande intégration : cacher les cases avec vérification du solveur
- Discussion importante : Maya a fait remarquer qu'un vérificateur d'unicité classique ne suffit pas — un puzzle peut avoir une solution mathématiquement unique sans être **résolvable par déduction logique pure** (sans jamais deviner). C'est justement ce que le solveur logique permet de vérifier.
- `essayerCacherCase(L, C)` : sauvegarde une copie de la grille actuelle, cache la case candidate, lance `solveurLogique()`, restaure la sauvegarde (annulant tout ce que le solveur a fait), et ne recache la case que si le solveur avait réussi. Astuce pour éviter de réécrire tout le solveur pour qu'il travaille sur une "copie de test" séparée.
- `cacherPlusieursCases()` réécrite : au lieu de cacher des cases au hasard sans contrôle, elle tire une case candidate pas encore cachée, appelle `essayerCacherCase`, et compte les vraies réussites, avec une limite de tentatives (200) pour éviter une boucle infinie si plus aucune case n'est cachable.
- **Résultat testé et confirmé** : Maya a généré un puzzle et l'a résolu elle-même à la main, avec ses propres déductions, sans jamais deviner — la preuve que l'objectif initial (des puzzles "comme dans les vrais journaux") est atteint.

### Nettoyage
- Ancien système à 3 grilles pré-écrites (`grillesDisponibles`, `choisirGrille`, bouton "nouvelle grille") mis en commentaire — plus utile maintenant que la vraie génération existe, mais gardé en trace du cheminement pour le TM.
- `testerSolveurLogique()` et son bouton mis en commentaire aussi (il ne testait de toute façon pas la grille affichée, mais en créait une nouvelle à chaque clic — devenu trompeur et redondant maintenant que la vérification est intégrée directement dans `cacherPlusieursCases`).
- Bug récurrent rencontré deux fois dans le projet : mettre `grille` en commentaire par erreur en même temps qu'un autre bloc — corrigé à chaque fois.

### Nouvelle technique de déduction perso : `déduireParExclusion`
- Maya a proposé sa propre technique de résolution (utilisée quand elle joue elle-même) : si une valeur n'a plus qu'une seule occurrence à placer dans une ligne, et qu'un bloc de 3 cases vides est encerclé par cette valeur des deux côtés (ex. `X O _ _ _ X`), alors la case juste après l'un des bords ne peut pas recevoir cette dernière valeur — sinon les cases restantes formeraient un triplet de l'autre symbole. Bien comprise et confirmée avec un exemple tracé à la main avant de coder.
- Implémentée pour les deux orientations possibles du motif (bloc collé à `Ligne[0]`/`Ligne[5]` d'un côté ou de l'autre), et intégrée dans `UnePasseDeDéduction()` pour les lignes et les colonnes.
- Remarque notée : cette version ne reconnaît que le cas précis où le bloc de 3 cases vides est au centre exact de la ligne (positions 2, 3, 4) — pas une position décalée. Piste de généralisation possible plus tard si besoin.

### Prochaine étape (à faire la prochaine fois)
- Éventuellement généraliser `déduireParExclusion` à d'autres positions du bloc de cases vides dans la ligne.
- Depuis la liste de idées plus large évoquée : message "Bravo, gagné !", vrai bouton "résoudre" (sur la grille en cours, pas en générer une nouvelle), bouton "indice", chronomètre, 4ème technique de déduction avancée (unicité), et les gros chantiers (tailles 4×4/8×8, son, page d'accueil) toujours en attente.
- Nettoyage mineur en attente : incohérence de typage sur `mode` (`'O'|'X'` déclaré, `0`/`1` assignés).

## 2026-08-11 — Généralisation de la déduction par exclusion, technique d'unicité, victoire et bouton résoudre

### Généralisation de `déduireParExclusion`
- Discussion sur les positions possibles d'un bloc de 3 cases vides dans une ligne de 6 (4 positions de départ possibles). Maya a confirmé, à partir de sa propre expérience de joueuse, que seuls 3 cas sur 4 sont réellement utiles à coder (le "milieu décalé" ne se présente pas ou est déjà couvert autrement) — décision prise en lui faisant confiance sur ce point.
- Deux nouveaux blocs `if` ajoutés à `déduireParExclusion` : le bloc de 3 cases collé au **début** de la ligne (ex. `___OXX`) et celui collé à la **fin** (ex. `XXO___`), en miroir des deux premiers déjà écrits.
- Un 5ème cas ajouté par Maya elle-même, plus subtil : un bloc de **4** cases vides entourées par la même valeur des deux côtés (ex. `X____X`) — dans ce cas, les deux cases aux extrémités du bloc sont déductibles (empêchent un triplet des deux côtés), mais les deux du milieu restent indéterminées. Vérifié et confirmé correct.
- Bug récurrent rencontré et corrigé : accolade de fermeture de fonction oubliée après l'ajout d'un nouveau `if`, cassant la compilation de tout le fichier (même piège que plusieurs fois précédemment).

### Nouvelle technique de déduction : l'unicité (celle mise de côté au tout début du chantier du solveur)
- Maya a formulé elle-même le principe avec sa propre façon de jouer : si une ligne n'a plus que 2 cases vides, et qu'une autre ligne complète lui est identique sur toutes les cases déjà remplies, alors les 2 cases manquantes doivent être l'**inverse** de cette ligne complète (sinon les deux lignes seraient identiques, ce qui casse la règle d'unicité).
- `ligneCompatible(LigneComplète, LigneIncomplète)` : nouvelle brique qui compare une ligne complète à une ligne incomplète, en ignorant les cases vides de l'incomplète.
- `déduireComparaison()` : contrairement aux techniques précédentes, ne prend pas de `Ligne` en paramètre — elle boucle elle-même sur toute la grille (comme `unicitéLignes()`), cherche les lignes à 2 cases vides, les compare à toutes les lignes complètes, et déduit si une correspondance est trouvée.
- `déduireComparaisonColonnes()` : même principe pour les colonnes, avec `getColonne`/`appliquerColonne`.
- Plusieurs bugs corrigés en construisant ces deux fonctions : boucle `L2`/`Co2` oubliée après une réécriture, ordre des paramètres de `ligneCompatible` inversé (cassait silencieusement la comparaison), variable oubliée avant modification de `getColonne` (même piège que la première fois qu'`appliquerColonne` avait été créée).
- Intégrées dans `UnePasseDeDéduction()` : appelées une seule fois chacune (pas dans les boucles ligne/colonne, puisqu'elles gèrent déjà toute la grille elles-mêmes).
- Le solveur logique a maintenant **6 techniques de déduction** au total, dont 2 identifiées et formulées par Maya elle-même à partir de sa pratique du jeu.

### Message de victoire et bouton "résoudre"
- `partieGagnee()` : combine `compterCasesVides() === 0` et `vérifierGrille() === true`.
- `<p v-if="partieGagnee()">Félicitations ! Vous avez gagné !</p>` — première utilisation de `v-if` (affiche l'élément seulement si la condition est vraie, contrairement à `{{ }}` qui affiche toujours quelque chose). Bug initial corrigé : `v-if="partieGagnee"` sans parenthèses (référence à la fonction elle-même, toujours "vraie") au lieu de `v-if="partieGagnee()"` (appel réel de la fonction).
- Bouton "résoudre la grille" relié directement à `solveurLogique()` — complète la grille **actuellement affichée** en utilisant les 6 techniques de déduction, sans en générer une nouvelle.

### Prochaine étape (à faire la prochaine fois)
- Bouton "indice" (révèle une seule case) et chronomètre, toujours dans la liste des idées moyennes.
- Nettoyage mineur en attente : incohérence de typage sur `mode`.
- Gros chantiers toujours en attente : grilles 4×4/8×8, son, page d'accueil.

## 2026-08-12 — Mise en page et direction artistique

### Choix de conception
- Comparaison visuelle de 4 ambiances possibles (épuré, ludique, carnet papier, sombre néon) — **ludique** choisi : couleurs pastel douces, formes arrondies.
- Comparaison de deux mises en page (deux colonnes grille+panneau vs tout centré avec boutons groupés en rangées) — **tout centré, boutons groupés** choisi, en pensant aussi aux futurs ajouts (niveau, son, chrono).

### Mise en place de la DA
- Cases de la grille : couleur douce selon la valeur pour les cases **données** uniquement (`bg-amber-200` pour `O`, `bg-teal-200` pour `X`, seulement si `caseModifiable(L,C) === false`), et une couleur neutre grise (`bg-gray-100`) pour les cases **jouables** (données ou non), pour bien distinguer les indices de départ de ce que la joueuse remplit elle-même.
- Boutons harmonisés dans une palette rose/pastel cohérente, en forme de pilule.
- Noms de boutons raccourcis ("Mode O"/"Mode X" restent complets par choix, "générer une grille aléatoire" → "grille aléatoire", "résoudre la grille" → "résoudre").
- Boutons d'action regroupés dans une même `<div>` (côte à côte) au lieu d'un par ligne.
- Une carte blanche arrondie (`bg-white rounded-3xl shadow-md`) enveloppe tout le contenu, elle-même posée sur un fond de page coloré (`bg-rose-50`) qui occupe tout l'écran.
- Badge "Grille: Valide/Invalide" (pastille arrondie, verte ou rouge selon la validité) à la place du texte brut.
- Message de victoire stylé (encadré vert, texte en gras et en couleur, plus festif).

### Bugs de mise en page rencontrés et corrigés (bonne session d'apprentissage CSS/Vue)
- **`grille.value[L][C]` utilisé directement dans le `<template>`** : rappel important que Vue "déballe" automatiquement les `ref` dans le template — `grille` seule y équivaut déjà à `grille.value` du `<script>`. Écrire `.value` en plus dans le template casse tout. Corrigé en utilisant directement `Ligne[C]` (déjà disponible via le `v-for`).
- **`items-center`/`justify-center` sans `flex`** : ces classes ne font rien sans `display: flex` sur le même élément — piège rencontré deux fois (conteneur de page, puis carte interne) avant que le centrage fonctionne partout.
- **`min-h-screen` manquant** : sans elle, le fond coloré ne couvrait pas toute la hauteur de l'écran.
- **`</div>` en trop / mal placée** : une fermeture prématurée de la carte blanche coupait le reste du contenu (boutons, message de victoire) hors de la carte — corrigée en repérant, avec l'aide du clic sur les balises dans VS Code, quelle accolade fermait quoi.
- **Conflit de guillemets** : `mode === "effacer"` à l'intérieur d'un attribut `:class="..."` déjà délimité par des guillemets doubles cassait le HTML — corrigé avec des guillemets simples à l'intérieur.
- **`effacer` sans guillemets du tout** (traité comme une variable inexistante au lieu du texte `'effacer'`) — corrigé.
- **Marges qui s'additionnent** : deux groupes de boutons avec `m-2` chacun créaient un espace double entre eux (16px) par rapport aux autres écarts (8px) — piste de correction en cours : passer à un `gap` sur le conteneur flex plutôt que des marges individuelles sur chaque bouton, encore à finaliser (dernier réglage d'espacement pas encore satisfaisant).

### Nouvelle fonctionnalité : mode "Effacer"
- Troisième valeur possible pour `mode` (`'effacer'`, en plus de `'O'`/`'X'`), avec un bouton dédié.
- `jouerCase` : si `mode.value === 'effacer'`, remet la case à `null` au lieu d'y mettre `mode.value` — permet d'annuler un coup précis sans réinitialiser toute la grille.

### Distinguer une victoire du joueur d'une résolution automatique
- Constat de Maya : cliquer sur "résoudre" affichait quand même "Félicitations, vous avez gagné !", alors que c'est le solveur qui a joué, pas elle.
- `résoluParOrdinateur` (nouvelle variable réactive) : mise à `true` uniquement par `résoudreClic()` (la nouvelle fonction reliée au bouton "résoudre", qui appelle `solveurLogique()` puis pose ce indicateur), et remise à `false` dans `jouerCase` (dès qu'un coup est joué), `resetGrille`, et `générerGrilleAléatoire`.
- Deux messages désormais distincts dans le template : "Félicitations !" seulement si `partieGagnee() && !résoluParOrdinateur`, et "La grille a été résolue par l'ordinateur !" si `résoluParOrdinateur`.

### Prochaine étape (à faire la prochaine fois)
- Finaliser l'espacement entre les groupes de boutons (passer proprement à un `gap` sur le conteneur plutôt que des marges individuelles, dernier réglage pas encore terminé).
- Reprendre ensuite la liste plus large : bouton "indice", chronomètre, nettoyage du typage de `mode`, et les gros chantiers (tailles 4×4/8×8, son, page d'accueil).

## 2026-08-13 — Bugs d'affichage, règles, son, indice et chronomètre

### Finalisation de l'espacement
- `gap-y-8` déplacé sur le bon élément (la carte blanche elle-même, celle qui a plusieurs enfants directs), au lieu du conteneur du milieu qui n'en avait qu'un seul — corrige le souci d'espacement laissé en suspens hier.

### Bug de superposition des couleurs de case en erreur
- Constat de Maya : quand une règle est cassée, toutes les cases de la ligne devraient devenir rouges, mais les `X` donnés de base restaient bleus (teal) alors que les `O` donnés devenaient bien rouges.
- Cause : dans Vue, plusieurs classes `:class` peuvent être vraies en même temps ; celle qui l'emporte visuellement dépend de l'ordre de génération de Tailwind (pas de l'ordre écrit dans le code), ce qui expliquait l'incohérence entre X et O.
- Corrigé en rendant les conditions de couleur mutuellement exclusives (`&& caseEnErreur(L,C)===false` ajouté aux trois autres conditions de couleur), pour que le rouge d'erreur soit toujours seul à s'appliquer quand il est vrai.

### Affichage des règles du jeu
- Décision (après réflexion, en évitant d'ajouter "un bouton partout") : un bouton "Règles" qui affiche/cache un texte, plutôt qu'un texte permanent.
- `afficherRègles` (ref booléen), bouton qui l'inverse au clic (`afficherRègles = !afficherRègles` — nouvelle notion, l'opérateur `!` qui donne le contraire d'un booléen), et un `<p v-if="afficherRègles">` avec le texte des 3 règles.

### Ajout du son
- Deux fichiers trouvés sur freesound.org (clic + victoire), copiés dans `public/sons/`.
- `new Audio('/sons/...')` pour créer les objets son, `.play()` pour les jouer.
- Bug du double-clic rapide : `.play()` ne relance pas un son déjà en cours — corrigé avec `sonClic.currentTime = 0` juste avant `.play()`, pour forcer un redémarrage du début.
- Bug du son de victoire qui se rejouait à chaque clic (même "Règles") après avoir gagné : causé par `sonWin.play()` placé à l'intérieur de `partieGagnee()`, une fonction rappelée par Vue à chaque rafraîchissement du template, pas seulement au moment du vrai gain. Corrigé en déplaçant l'appel dans `jouerCase`, juste après qu'un coup soit joué, avec vérification de `partieGagnee()` à cet instant précis — bon rappel sur la différence entre fonction "pure" (juste répondre à une question) et effet de bord (jouer un son, écrit une seule fois au bon moment).

### Nettoyage : type de `mode`
- `mode` était déclaré `ref<null | 'O' | 'X' | 'effacer'>`, mais les boutons assignaient en réalité des nombres (`0`/`1`), pas les textes `'O'`/`'X'`. Ça fonctionnait quand même car TypeScript ne vérifie les types qu'au moment du développement (pas à l'exécution dans le navigateur), et parce que `0`/`1` sont en fait cohérents avec le reste de la grille. Corrigé pour que le type dise la vérité : `ref<null | 0 | 1 | 'effacer'>`.

### Bouton "Indice"
- Réutilise le même principe que `essayerCacherCase` : sauvegarder la grille, lancer `solveurLogique()`, comparer avant/après pour repérer les cases nouvellement déduites, restaurer la grille, puis ne réappliquer qu'**une seule** case.
- Amélioration en cours de route : au lieu de ne garder que la dernière case trouvée (ce qui donnait un indice toujours prévisible, en bas à droite en premier), toutes les cases candidates sont collectées dans un tableau (`candidats.push({L, C})`), puis une est tirée au hasard avec `Math.floor(Math.random() * candidats.length)` — pour un vrai indice aléatoire.

### Chronomètre
- `secondesEcoulées` (ref), `idChrono` (identifiant du minuteur, pas réactif).
- `démarrerChrono()` : remet le compteur à 0 et lance `setInterval(...)` (nouvelle notion : une fonction qui répète une action toutes les X millisecondes), en gardant l'identifiant retourné.
- `arreterChrono()` : `clearInterval(idChrono)` pour stopper proprement.
- Démarré dans `générerGrilleAléatoire`/`resetGrille`, arrêté dans `jouerCase` en même temps que le son de victoire.
- `tempsFormaté()` : convertit les secondes en minutes:secondes (`Math.floor(.../60)` pour les minutes, `% 60` — nouvel opérateur, le reste d'une division — pour les secondes restantes).
- Limite identifiée et volontairement non corrigée : le chrono ne démarre pas sur la toute première grille au chargement de la page (seulement via les boutons "grille aléatoire"/"recommencer"). Décision de Maya : laisser tel quel, ce sera naturellement réglé par la future page d'accueil (avec un bouton "commencer" qui déclenchera le chrono).

### Prochaine étape (à faire la prochaine fois)
- Les gros chantiers restent : grilles 4×4/8×8, page d'accueil (qui réglera aussi le petit souci du chrono au premier chargement).
- Continuer à ajouter des techniques de déduction personnelles si Maya en trouve en jouant.
