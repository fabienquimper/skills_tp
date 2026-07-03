---
name: apports-nutritionnels
description: Calcule les apports nutritionnels d'une recette (slug ou texte collé) — énergie en kcal, macros (protéines, glucides dont sucres, lipides dont saturés, fibres), vitamines, minéraux et oligo-éléments, en total et par portion, avec % des AJR indicatifs. Utiliser quand l'utilisateur veut les calories, valeurs nutritionnelles, apports ou table nutritionnelle d'un plat.
user-invocable: true
allowed-tools:
  - Read
  - Glob
  - Grep
---

# /apports-nutritionnels — Apports nutritionnels d'une recette

> 📁 **Emplacement.** `recettes/`, `profils/`, `data/`, `commandes/` sont à la **racine du harness**
> (ton répertoire de travail, contenant `recettes/` et `data/`) — **pas** dans `.claude/skills/`.
> Tous les chemins ci-dessous sont relatifs à cette racine (cf. `CLAUDE.md`).

Estime les apports d'une recette à partir de la table de référence locale.

Arguments : `$ARGUMENTS`
Ex. : `apports-nutritionnels curry-pois-chiches` ou coller une recette.

## 1. Récupérer les ingrédients en grammes

- Charger la recette (slug → `recettes/<slug>.md`, ou texte collé) et son nombre de `portions`.
- Convertir **chaque ingrédient en grammes** via `data/conversions.md` (même logique que le skill
  `ingredients-recette` ; tu peux t'appuyer dessus).

## 2. Calculer depuis la table

- Lire `data/nutrition-100g.csv` (valeurs **pour 100 g/ml**, colonnes documentées en en-tête).
- Pour chaque ingrédient : `apport = (grammes / 100) × valeur_100g`, pour **chaque** colonne
  (kcal, proteines_g, glucides_g, sucres_g, lipides_g, satures_g, fibres_g, sodium_mg, calcium_mg,
  fer_mg, magnesium_mg, potassium_mg, zinc_mg, vit_c_mg, vit_d_ug, vit_b9_ug, vit_a_ug).
- **Sommer** sur tous les ingrédients = total recette. Diviser par `portions` = par portion.
- **Ingrédient absent de la table** : faire une estimation explicite et la **marquer `(estimé)`** ;
  lister ces ingrédients à part. Ne pas masquer l'incertitude.

## 3. Restituer

En-tête : nom de la recette, portions.

**Tableau « par portion » (principal) + colonne total** :
- Énergie (kcal)
- Protéines / Glucides (dont sucres) / Lipides (dont saturés) / Fibres — en g
- Minéraux & oligo-éléments : sodium, calcium, fer, magnésium, potassium, zinc
- Vitamines : C, D, B9, A

Ajouter une colonne **% AJR** indicative par portion (repères adulte : 2000 kcal ; protéines 50 g ;
glucides 260 g ; lipides 70 g ; fibres 30 g ; sodium 2000 mg ; calcium 800 mg ; fer 14 mg ;
magnésium 375 mg ; potassium 2000 mg ; zinc 10 mg ; vit C 80 mg ; vit D 5 µg ; vit B9 200 µg ;
vit A 800 µg). Préciser « repères moyens adulte, variables selon profil ».

## Encart obligatoire (toujours afficher)
> ℹ️ Valeurs **indicatives** calculées à partir d'une table moyenne ; elles varient selon les
> produits, la cuisson et les pertes. Ce calcul **n'a pas de valeur médicale** et ne remplace pas
> l'avis d'un professionnel de santé.

## Si profil médical
Pour une recette tagguée `diabetique`, mettre en avant **glucides dont sucres + fibres/portion** ;
pour `anorexie`, mettre en avant **kcal/portion** et rappeler le cadrage médical de `profils/`.
