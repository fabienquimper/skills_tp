---
name: fiche-commande-excel
description: Génère un classeur Excel (.xlsx) récapitulatif à partir d'une commande traiteur (1 à plusieurs plats avec leurs couverts, ou un plan de semaine). Le classeur contient 4 onglets — récap des commandes/quantités par plat, liste de courses agrégée, apports nutritionnels par plat, et allergènes par plat. Utiliser quand l'utilisateur veut un fichier Excel/tableur récapitulatif d'une commande ou d'une semaine.
user-invocable: true
allowed-tools:
  - Read
  - Glob
  - Bash(python3 *)
  - Bash(python *)
---

# /fiche-commande-excel — Classeur Excel d'une commande

> 📁 **Emplacement.** Lance les commandes depuis la **racine du harness** (ton répertoire de
> travail, contenant `recettes/` et `data/`). Le script `generer.py` auto-détecte la racine, mais
> les chemins d'exemple (`commandes/...`, `.claude/skills/...`) sont relatifs à elle (cf. `CLAUDE.md`).

Produit un fichier `.xlsx` à **4 onglets** à partir d'une commande. Le calcul est **déterministe**
(script Python + `openpyxl`), pas une estimation à la volée.

Arguments : `$ARGUMENTS`

## Onglets produits
1. **Récap commande** — plat · type · couverts · portions de réf. · facteur · coût (€) + total.
2. **Liste de courses** — ingrédients **fusionnés** entre plats, quantité d'achat, par rayon.
3. **Apports par plat** — kcal/portion + macros (protéines, glucides dont sucres, lipides dont
   saturés, fibres) + ligne **total commande servi**.
4. **Allergènes par plat** — matrice plat × 14 allergènes INCO (`X` présent / `trace` / `—`).

## Comment l'exécuter

Le script est `generer.py` (dans ce dossier de skill). Deux entrées possibles :

**A. Commande en ligne** (`slug:couverts`) :
```bash
python3 .claude/skills/fiche-commande-excel/generer.py \
  "curry-pois-chiches:20, veloute-potimarron:20, compote-pomme-sans-sucre:20" \
  --nom "buffet-dupont"
```

**B. À partir d'un plan / d'une semaine** (fichier contenant des `slug: couverts`) :
```bash
python3 .claude/skills/fiche-commande-excel/generer.py \
  --commande commandes/semaine-2026-07-06.md --nom "semaine-06-07"
```
En mode `--commande`, le script récupère tous les `slug: nombre` du fichier correspondant à une
recette existante et **cumule** les occurrences d'un même plat (utile pour une semaine).

Options : `--sortie chemin.xlsx` (défaut : `commandes/<nom>.xlsx`), `--racine` (auto-détectée).

## Déroulé du skill
1. Construire la commande depuis `$ARGUMENTS` : repérer les couples `plat : couverts`. Si un slug
   est inconnu, proposer les recettes proches via `recettes/README.md` et demander.
2. Lancer `generer.py` avec la bonne entrée (A ou B) et un `--nom` parlant.
3. Lire la sortie console : confirmer le **chemin du .xlsx**, le nb de plats/couverts, le coût, et
   relayer l'éventuel avertissement « ingrédients hors référentiel ».
4. Rappeler que les valeurs nutritionnelles et allergènes sont **indicatives** (voir disclaimers
   des skills `apports-nutritionnels` et `allergenes-recette`).

## Notes
- Prérequis : `openpyxl` (présent). Le script lit `recettes/*.md` + `data/*.csv`
  (`nutrition-100g.csv`, `allergenes.csv`, `conversions.csv`, `rayons.csv`).
- Données nutrition/allergènes/rayons = **mêmes référentiels** que les autres skills → cohérence garantie.
- S'enchaîne avec `organiser-semaine-dieteticien`, qui écrit justement un fichier `commandes/semaine-*.md`.
