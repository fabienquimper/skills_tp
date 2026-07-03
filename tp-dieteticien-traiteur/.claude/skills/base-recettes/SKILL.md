---
name: base-recettes
description: Gère la base de recettes du harness — lister/filtrer (par profil, type de repas, allergène), chercher, afficher une recette, en ajouter une (avec validation du format canonique et mise à jour de l'index) ou en supprimer. Utiliser pour parcourir, rechercher ou maintenir la collection de recettes.
user-invocable: true
allowed-tools:
  - Read
  - Glob
  - Grep
  - Write
  - Edit
---

# /base-recettes — Gérer la base de recettes

> 📁 **Emplacement.** `recettes/`, `profils/`, `data/`, `commandes/` sont à la **racine du harness**
> (ton répertoire de travail, contenant `recettes/` et `data/`) — **pas** dans `.claude/skills/`.
> Tous les chemins ci-dessous sont relatifs à cette racine (cf. `CLAUDE.md`).

Point d'entrée pour parcourir et maintenir `recettes/`. **Seul skill autorisé à écrire** dans la base.

Arguments : `$ARGUMENTS` (première sous-commande + paramètres). Si vide → `lister`.

## `lister [--profil X] [--type Y] [--allergene Z]`
1. Lire `recettes/README.md` (index) ; au besoin compléter via `Glob recettes/*.md` + frontmatter.
2. Appliquer les filtres demandés (profil, type-repas, exclure un allergène).
3. Afficher un tableau : slug, nom, type, profils, portions, coût/portion.

## `chercher <mots-clés>`
1. `Grep` les mots-clés dans `recettes/*.md` (nom, ingrédients, notes).
2. Restituer les recettes correspondantes avec un court extrait.

## `montrer <slug>`
1. Lire `recettes/<slug>.md` et l'afficher. Si absent, proposer les slugs proches.

## `ajouter <slug>` (ou recette fournie / issue de creer-recette)
1. **Valider le format** contre `recettes/_format.md` : frontmatter complet (`nom`, `slug`,
   `portions`, `profils`, `type-repas`, temps, `cout-estime-eur`, `allergenes`, `regime`) ;
   sections `## Ingrédients` (lignes parsables `quantité unité ingrédient`), `## Préparation`,
   `## Notes`. Refuser/corriger si non conforme.
2. Vérifier que `slug` est unique (pas de fichier existant). `slug` = kebab-case sans accent.
3. Pour un profil médical (`diabetique`, `anorexie`), vérifier la présence de l'**encart
   d'avertissement** dans `## Notes` ; sinon l'ajouter depuis `profils/`.
4. Écrire `recettes/<slug>.md`.
5. **Mettre à jour l'index** `recettes/README.md` (ligne du tableau + filtres rapides) via Edit.
6. Confirmer.

## `supprimer <slug>`
1. Confirmer l'intention avec l'utilisateur (action destructive).
2. Supprimer la ligne correspondante de `recettes/README.md`.
3. Indiquer le fichier `recettes/<slug>.md` à retirer (la suppression de fichier se fait via Bash
   si l'utilisateur l'autorise ; sinon, vider/annoncer). Ne jamais supprimer sans confirmation.

## Cohérence
- Après toute mutation, l'index `recettes/README.md` doit refléter l'état réel de `recettes/`.
- Encourager des noms d'ingrédients alignés sur `data/nutrition-100g.csv` pour que
  `apports-nutritionnels` et `liste-course-traiteur` fonctionnent sans estimation.
