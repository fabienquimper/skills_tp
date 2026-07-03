---
name: creer-recette
description: Crée une recette de cuisine adaptée à un ou plusieurs profils diététiques (saine, diabétique, anorexie/renutrition, gourmand, petit budget) et à des contraintes (portions, allergènes, budget, ingrédients imposés/évités). Utiliser quand l'utilisateur veut concevoir/inventer une recette, un plat ou un menu pour un public donné.
user-invocable: true
allowed-tools:
  - Read
  - Glob
  - Grep
  - Write
---

# /creer-recette — Concevoir une recette adaptée

> 📁 **Emplacement.** `recettes/`, `profils/`, `data/`, `commandes/` sont à la **racine du harness**
> (ton répertoire de travail, contenant `recettes/` et `data/`) — **pas** dans `.claude/skills/`.
> Tous les chemins ci-dessous sont relatifs à cette racine (cf. `CLAUDE.md`).

Génère une recette au **format canonique** du harness, respectant un ou plusieurs profils
diététiques et des contraintes. Conçu pour un diététicien-traiteur.

Arguments libres : `$ARGUMENTS`
Ex. : `creer-recette diabétique + petit-budget, 4 personnes, sans porc, type plat`

## 1. Comprendre la demande

Extraire des arguments (demander seulement si une info bloquante manque) :
- **Profils** visés (1..n) : `saine`, `diabetique`, `anorexie`, `gourmand`, `petit-budget`.
  Défaut si non précisé : `saine`.
- **Portions** (défaut 4), **type de repas** (entrée/plat/dessert/petit-déjeuner/collation),
  **allergènes/régimes** à respecter (vegan, sans-gluten, halal, sans-porc...),
  **ingrédients imposés ou à éviter**, **budget** éventuel.

## 2. Charger les contraintes

1. Lire `recettes/_format.md` → **format de sortie exact** à respecter.
2. Pour **chaque** profil demandé, lire `profils/<profil>.md` → objectifs, à privilégier/limiter,
   réglages, et **encart d'avertissement** éventuel.
3. Préférer des ingrédients présents dans `data/nutrition-100g.csv` (clés exactes) pour que les
   apports et la liste de courses fonctionnent ensuite. Tu peux en introduire d'autres si besoin.

## 3. Gérer les combinaisons de profils

- Si plusieurs profils, **croiser** leurs contraintes.
- **Conflit** (ex. `gourmand` vs `diabetique`, ou `gourmand` vs `petit-budget`) : le profil
  **médical est prioritaire** (diabetique, anorexie). Garder l'esprit de l'autre profil via un
  **compromis explicite** (ex. « version gourmande mais sucres maîtrisés »). Le signaler dans `## Notes`.

## 4. Produire la recette

Écrire la recette **strictement** au format de `recettes/_format.md` :
frontmatter complet (`nom`, `slug` en kebab-case sans accent, `portions`, `profils`, `type-repas`,
temps, `cout-estime-eur`, `allergenes`, `regime`) puis sections `## Ingrédients`, `## Préparation`,
`## Notes`.

Exigences par profil :
- **diabetique** : afficher une estimation des **glucides par portion** dans `## Notes`, viser un IG
  bas, remplacer le sucre par fruits/épices. **Recopier l'encart d'avertissement** de `profils/diabetique.md`.
- **anorexie** : proposer densité énergétique douce / collation, textures rassurantes. **Recopier
  l'encart fort** de `profils/anorexie.md` (renutrition = supervision médicale, risque de SRI).
- **petit-budget** : renseigner `cout-estime-eur` réaliste et le coût/portion dans `## Notes`.
- **saine / gourmand** : suivre les réglages du profil.

## 5. Proposer l'enregistrement

Après affichage, **proposer** d'ajouter la recette à la base via `base-recettes` (`ajouter`).
**Ne pas écrire** dans `recettes/` sans accord explicite de l'utilisateur. S'il accepte, écrire
`recettes/<slug>.md` et rappeler de mettre à jour `recettes/README.md` (ou déléguer à `base-recettes`).

## Garde-fous
- Pour tout profil médical, ne jamais présenter la recette comme un soin : ce sont des repères, pas
  une prescription. Conserver les encarts d'avertissement intégralement.
