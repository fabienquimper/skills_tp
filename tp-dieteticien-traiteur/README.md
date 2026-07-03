# Harness Diététicien & Traiteur

Boîte à outils Claude Code pour un **diététicien-traiteur** : concevoir des recettes adaptées à
différents publics, en détailler les ingrédients, calculer leurs apports nutritionnels, déclarer
les allergènes, planifier une semaine de repas et générer la liste de courses / un classeur Excel
récapitulatif — le tout adossé à une base de recettes réutilisable.

> ⚠️ **Cadre d'usage.** Cet outil aide à concevoir des repas et donne des repères nutritionnels et
> allergènes **indicatifs**. Il **ne remplace pas** un avis médical, un suivi diététique
> personnalisé, ni un étiquetage allergènes certifié. Les profils médicaux (diabète,
> anorexie/renutrition) relèvent d'un encadrement par des professionnels de santé — voir les
> encarts dans `profils/` et `data/allergenes-inco.md`.

## Architecture

```
dieteticien/
├── .claude/skills/        # les 8 skills (commandes /…)
├── recettes/              # base de recettes (Markdown) + _format.md + index README.md
├── profils/               # profils diététiques (réf. partagée)
├── data/                  # données partagées (nutrition, allergènes, conversions, rayons)
└── commandes/             # sorties générées : plans de semaine + classeurs .xlsx
```

**Principe : skills fins, données partagées.** Les skills ne ré-encodent pas les profils ni les
valeurs nutritionnelles/allergènes ; ils lisent `profils/` et `data/`. Enrichir un fichier de
données profite à tous les skills.

## Les 8 skills

| Commande | Rôle |
|----------|------|
| `/creer-recette` | Crée une recette adaptée à 1..n profils + contraintes (portions, allergènes, budget, ingrédients). |
| `/ingredients-recette` | Liste d'ingrédients détaillée : quantités, équivalent en grammes, mise à l'échelle pour N personnes. |
| `/apports-nutritionnels` | Apports d'une recette : kcal, macros, vitamines, minéraux/oligo-éléments, total + par portion, % AJR. |
| `/liste-course-traiteur` | Liste de courses agrégée (texte) pour 1..N plats : fusion des ingrédients, par rayon, coût. |
| `/base-recettes` | Parcourir/maintenir la base : lister, chercher, montrer, ajouter (avec validation), supprimer. |
| `/allergenes-recette` | Allergènes **14 INCO** + traces possibles d'un ingrédient, d'une recette ou d'un menu. |
| `/fiche-commande-excel` | Classeur **Excel 4 onglets** (récap, courses, apports, allergènes) à partir d'une commande. |
| `/organiser-semaine-dieteticien` | Plan de repas **7 j × 3 repas** par objectif + export en « commande » réutilisable. |

## Workflows types

**Prestation traiteur ponctuelle**
1. **Concevoir** — `/creer-recette gourmand + diabétique, 6 personnes, type plat`.
2. **Enregistrer** — `/base-recettes ajouter` → la recette rejoint la base, l'index est mis à jour.
3. **Vérifier** — `/apports-nutritionnels <slug>` et `/allergenes-recette <slug>`.
4. **Adapter les quantités** — `/ingredients-recette <slug> pour 30 personnes`.
5. **Récapituler** — `/fiche-commande-excel "platA:30, platB:30, dessert:30"` → un `.xlsx` complet
   (ou `/liste-course-traiteur` pour une liste de courses en texte).

**Semaine de repas**
1. **Planifier** — `/organiser-semaine-dieteticien surpoids, 2 personnes, sans fruits-a-coque`
   → plan 7×3 + écrit `commandes/semaine-AAAA-MM-JJ.md`.
2. **Générer le classeur** — `/fiche-commande-excel --commande commandes/semaine-AAAA-MM-JJ.md`.

## Profils diététiques disponibles

`saine` · `surpoids` (encart) · `diabetique` (encart médical) · `anorexie` (encart médical fort,
risque de SRI) · `gourmand` · `petit-budget`. Ils se **combinent** ; en cas de conflit, le profil
médical est prioritaire (voir `profils/`).

## Données de référence (`data/`)

- `nutrition-100g.csv` — valeurs nutritionnelles pour 100 g (~58 ingrédients, enrichissable).
- `allergenes.csv` (+ `allergenes-inco.md`) — allergènes 14 INCO par ingrédient + référentiel.
- `conversions.md` / `conversions.csv` — unités ménagères → grammes (le `.csv` = miroir machine).
- `rayons.md` / `rayons.csv` — classement des ingrédients par rayon (le `.csv` = miroir machine).

Les skills « pilotés par Claude » lisent les `.md` ; le générateur Excel (`generer.py`) lit les
`.csv`. **En cas de modification d'une donnée, mettre à jour le `.md` ET le `.csv` correspondants.**

## Étendre le harness

- **Nouvel ingrédient** : ajouter une ligne dans `nutrition-100g.csv`, `allergenes.csv`,
  `rayons.csv` (+ `rayons.md`) et, si pesée en pièce, `conversions.csv`. Tous les skills suivent.
- **Nouvelle recette** : passer par `/base-recettes ajouter` (valide le format + met à jour l'index).
- **Nouveau profil** : créer `profils/<nom>.md` sur le même modèle ; les skills le liront.

## Comprendre le projet (pédagogie / IA agentique)

📚 `docs/meta-prompt-pedagogique.md` — un document complet qui explique **tout le projet** comme un
cours d'IA agentique (agents, outils vs skills, autonomie, garde-fous, déterminisme, futur de l'IA)
et comment un·e professionnel·le non technique peut l'intégrer chez lui. Sert aussi de **méta-prompt**
pour briefer un agent qui reprendrait le harness.

## Pistes d'extension (hors périmètre actuel)
Gestion des stocks/inventaire, devis/facturation traiteur, étiquetage allergènes **réglementaire
certifié**, import automatisé d'une table CIQUAL complète.
