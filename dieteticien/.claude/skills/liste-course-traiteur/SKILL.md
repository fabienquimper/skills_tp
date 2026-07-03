---
name: liste-course-traiteur
description: À partir d'un ou plusieurs plats (slugs de la base et nombre de couverts par plat), génère une liste de courses complète et agrégée — mise à l'échelle, fusion des ingrédients communs, regroupement par rayon de magasin, et estimation du coût total. Utiliser pour préparer les achats d'un service traiteur, d'un menu ou d'un repas à plusieurs plats.
user-invocable: true
allowed-tools:
  - Read
  - Glob
  - Grep
---

# /liste-course-traiteur — Liste de courses agrégée

> 📁 **Emplacement.** `recettes/`, `profils/`, `data/`, `commandes/` sont à la **racine du harness**
> (ton répertoire de travail, contenant `recettes/` et `data/`) — **pas** dans `.claude/skills/`.
> Tous les chemins ci-dessous sont relatifs à cette racine (cf. `CLAUDE.md`).

Produit la liste de courses complète pour une prestation (1..N plats, quantités variables).

Arguments : `$ARGUMENTS`
Ex. : `liste-course-traiteur curry-pois-chiches x20, veloute-potimarron x20, compote-pomme-sans-sucre x20`

## 1. Lire la commande

Pour chaque plat : un **slug** (ou nom proche) + un **nombre de couverts/portions** (`x20`,
`pour 20`, ...). Si un slug est introuvable, proposer les recettes proches via `recettes/README.md`.
Si un nombre de couverts manque, demander.

## 2. Mettre à l'échelle chaque plat

- Charger `recettes/<slug>.md`, lire `portions` de référence.
- Facteur = couverts_demandés / portions_référence. Multiplier toutes les quantités d'ingrédients.

## 3. Convertir et fusionner

- Convertir en grammes/ml via `data/conversions.md`.
- **Fusionner** les ingrédients identiques across plats : sommer les quantités (même clé d'ingrédient).
- Reconvertir vers une **unité d'achat réaliste** (ex. 6 gousses d'ail, 2,4 kg de riz, 1,2 L de lait
  de coco) — voir conseils d'arrondi de `conversions.md`.

## 4. Regrouper par rayon

- Classer chaque ingrédient via `data/rayons.md` (mapping + règle par mots-clés).
- Ordonner les rayons selon l'ordre de parcours conseillé du fichier.

## 5. Estimer le coût

- Coût total ≈ Σ sur les plats de `cout-estime-eur × (couverts / portions_référence)`.
- Présenter le total et un coût/couvert moyen. Préciser « estimation indicative ».

## 6. Restituer

```
🧾 Liste de courses — <récap : plats × couverts>

## Fruits & légumes
- [ ] Oignon — 2,2 kg (≈ 20 pièces)
- [ ] ...
## Boucherie / Volaille
- [ ] ...
...
💶 Coût estimé total : ~XX € (~X,XX €/couvert)
```

- Liste **cochable** (`- [ ]`), groupée par rayon, quantités en unités d'achat.
- Terminer par un **récap** : plats commandés, couverts, total ingrédients fusionnés, coût.
- Signaler les ingrédients dont l'unité d'achat est incertaine plutôt que d'inventer.
