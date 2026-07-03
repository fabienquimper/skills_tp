---
name: organiser-semaine-dieteticien
description: Génère un plan de repas pour une semaine complète (lundi→dimanche, matin/midi/soir) adapté à un objectif (surpoids, diabète, alimentation saine, petit budget, gourmand) et à des contraintes (allergènes à éviter, nombre de personnes). Produit aussi une "commande" réutilisable pour générer la liste de courses / le classeur Excel. Utiliser pour planifier les menus de la semaine.
user-invocable: true
allowed-tools:
  - Read
  - Glob
  - Grep
  - Write
---

# /organiser-semaine-dieteticien — Plan de repas hebdomadaire

> 📁 **Emplacement.** `recettes/`, `profils/`, `data/`, `commandes/` sont à la **racine du harness**
> (ton répertoire de travail, contenant `recettes/` et `data/`) — **pas** dans `.claude/skills/`.
> Tous les chemins ci-dessous sont relatifs à cette racine (cf. `CLAUDE.md`).

Construit un **planning 7 jours × 3 repas** (matin / midi / soir) adapté à un objectif et à des
contraintes, puis l'exporte en **« commande »** réutilisable.

Arguments : `$ARGUMENTS`
Ex. : `organiser-semaine-dieteticien surpoids, 2 personnes, sans fruits-a-coque`
ou `organiser-semaine-dieteticien diabete + petit-budget, 4 personnes`

## 1. Lire la demande
- **Objectif / profil** (1..n) : `surpoids`, `diabete`, `saine`, `petit-budget`, `gourmand`
  (défaut : `saine`). Charger le(s) fichier(s) `profils/<profil>.md`.
- **Allergènes à éviter** (slugs INCO : `fruits-a-coque`, `lait`, `gluten`...).
- **Nombre de personnes** (défaut 2). Contraintes éventuelles (sans porc, végétarien...).

## 2. Sélectionner les repas
1. Lister la base via `recettes/README.md` ; filtrer par profil et par type de repas.
2. **Exclure** toute recette contenant un allergène à éviter — vérifier via `data/allergenes.csv`
   (mêmes règles que le skill `allergenes-recette`), pas seulement le frontmatter.
3. Remplir la grille 7×3 :
   - Utiliser les **recettes formelles** de la base quand elles collent au profil/type.
   - Là où la base est trop mince, proposer une **idée de repas légère** (non formalisée),
     cohérente avec le profil — ex. « Petit-déj : porridge avoine + fruit + yaourt nature ».
     Marquer ces lignes d'un astérisque `*` et suggérer `/creer-recette` pour les formaliser.
4. **Équilibrer** la semaine : varier les sources de protéines et les légumes, alterner, éviter de
   répéter le même plat plus de ~2 fois, respecter les repères du profil (ex. féculents maîtrisés
   en `surpoids`, sucres maîtrisés en `diabete`).

## 3. Restituer le plan
Tableau markdown :

| Jour | Matin | Midi | Soir |
|------|-------|------|------|
| Lundi | ... | ... | ... |
| ... | | | |

- Indiquer en tête : objectif(s), nb de personnes, allergènes exclus.
- `*` = idée à formaliser ; sinon utiliser le **slug** de la recette de la base.
- Si un profil médical est présent (`diabete`), rappeler son **encart** (voir `profils/`).

## 4. Exporter en « commande »
Proposer d'écrire `commandes/semaine-<AAAA-MM-JJ>.md` contenant :
- l'en-tête (objectif, personnes, allergènes exclus) ;
- le tableau du plan ;
- une **section `## Commande`** listant, pour chaque **recette formelle** retenue,
  une ligne `- <slug>: <couverts>` (couverts = nb de personnes × nb d'occurrences dans la semaine).
  Les idées `*` non formalisées n'y figurent pas (le rappeler).

Ce fichier est directement consommable :
```bash
/fiche-commande-excel --commande commandes/semaine-<AAAA-MM-JJ>.md
```
→ récap, liste de courses, apports et allergènes de toute la semaine dans un classeur Excel.

## Garde-fous
- Plan **indicatif** d'aide à l'organisation, pas une prescription. Pour `diabete`/`surpoids`,
  conserver les encarts des profils (suivi par un professionnel de santé).
- Toujours respecter strictement les allergènes à éviter ; en cas de doute sur un ingrédient, le signaler.
