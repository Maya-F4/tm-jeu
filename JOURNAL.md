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

## 2026-07-26 — Début de l'interactivité (mode 0/1 + boutons)

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

### Prochaine étape (à faire la prochaine fois)
- Distinguer les cases "données" (valeurs de départ, non modifiables) des cases "à jouer" (vides au départ) — probablement via une copie de la grille de départ à comparer avec `grille`.
- Écrire la fonction de clic sur une case (ex. `jouerCase(L, C)`) qui n'agit que sur les cases modifiables et applique la valeur de `mode`.
- Une fois la grille jouable, relier `vérifierGrille()` pour afficher si la grille est valide.
