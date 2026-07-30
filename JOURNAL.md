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
