---
name: ingredients-recette
description: À partir d'une recette (slug de la base ou texte collé), produit la liste d'ingrédients détaillée — quantités normalisées, équivalent en grammes, mise à l'échelle pour N portions, substitutions. Utiliser quand l'utilisateur veut extraire/détailler/recalculer les ingrédients d'une recette ou l'adapter à un nombre de convives.
user-invocable: true
allowed-tools:
  - Read
  - Glob
  - Grep
---

# /ingredients-recette — Liste d'ingrédients détaillée

> 📁 **Emplacement.** `recettes/`, `profils/`, `data/`, `commandes/` sont à la **racine du harness**
> (ton répertoire de travail, contenant `recettes/` et `data/`) — **pas** dans `.claude/skills/`.
> Tous les chemins ci-dessous sont relatifs à cette racine (cf. `CLAUDE.md`).

Transforme une recette en liste d'ingrédients précise et mise à l'échelle.

Arguments : `$ARGUMENTS`
Ex. : `ingredients-recette curry-pois-chiches pour 10 personnes`
ou bien coller une recette / un texte d'ingrédients directement.

## 1. Récupérer la recette

- Si un **slug** est donné (ou un nom proche) : lire `recettes/<slug>.md`. Si introuvable, lister
  les recettes proches via `recettes/README.md` et demander.
- Si du **texte** est collé : le parser directement.
- Noter le nombre de **portions de référence** (frontmatter `portions`, ou déduit du texte).

## 2. Mettre à l'échelle

- Cible = portions demandées par l'utilisateur (défaut : portions de référence).
- Facteur = portions_cible / portions_référence. Multiplier chaque quantité.

## 3. Normaliser et convertir en grammes

Pour chaque ligne `- <quantité> <unité> <ingrédient>` :
- Convertir l'unité ménagère en **grammes/ml** à l'aide de `data/conversions.md`
  (c. à soupe, c. à café, pièce, gousse, pincée, verre...).
- Appliquer les **règles d'arrondi** de `conversions.md`.
- Regrouper les éventuelles lignes du même ingrédient.

## 4. Restituer

Tableau :

| Ingrédient | Quantité (recette) | Équivalent (g/ml) | Note / substitution |
|------------|--------------------|-------------------|----------------------|

- Colonne note : substitutions utiles (issues de `## Notes` de la recette), allergènes, astuce.
- Indiquer en en-tête : nom de la recette, portions cible, facteur d'échelle appliqué.
- Si une conversion est incertaine (ingrédient/unité non couvert par `conversions.md`), le **signaler**
  explicitement plutôt que d'inventer une valeur précise.

## Enchaînements utiles
- Pour les apports → `apports-nutritionnels`.
- Pour acheter (1 ou plusieurs plats) → `liste-course-traiteur`.
