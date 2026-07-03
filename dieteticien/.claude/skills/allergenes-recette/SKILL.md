---
name: allergenes-recette
description: Indique les allergènes à déclaration obligatoire (14 catégories INCO) présents et les traces possibles, à partir d'un ingrédient, d'une recette (slug ou texte collé) ou d'une commande. Utiliser pour la déclaration allergènes d'un plat traiteur, vérifier la compatibilité avec un convive allergique, ou lister les allergènes d'un menu.
user-invocable: true
allowed-tools:
  - Read
  - Glob
  - Grep
---

# /allergenes-recette — Déclaration des allergènes (INCO)

> 📁 **Emplacement.** `recettes/`, `profils/`, `data/`, `commandes/` sont à la **racine du harness**
> (ton répertoire de travail, contenant `recettes/` et `data/`) — **pas** dans `.claude/skills/`.
> Tous les chemins ci-dessous sont relatifs à cette racine (cf. `CLAUDE.md`).

Détermine les **14 allergènes à déclaration obligatoire** (règlement UE INCO) d'un ingrédient,
d'une recette ou d'un ensemble de plats.

Arguments : `$ARGUMENTS`
Ex. : `allergenes-recette gratin-courgettes` · `allergenes-recette feta` ·
`allergenes-recette curry-pois-chiches, salade-lentilles-feta`

## 1. Identifier la cible
- **Ingrédient seul** : chercher directement dans `data/allergenes.csv`.
- **Recette (slug)** : lire `recettes/<slug>.md`, extraire la section `## Ingrédients`.
- **Texte collé** : parser les lignes d'ingrédients.
- **Plusieurs plats** : traiter chacun, puis fournir une synthèse par plat + union globale.

## 2. Résoudre les allergènes
Pour chaque ingrédient :
1. Lire sa ligne dans `data/allergenes.csv` → colonne `allergenes` (**présents**) et
   `traces_possibles`.
2. Si l'ingrédient n'y figure pas, appliquer un **fallback par mots-clés** (voir
   `data/allergenes-inco.md` : ex. *farine/blé/pâtes/pain* → `gluten` ; *lait/fromage/crème/beurre*
   → `lait` ; *crevette/crabe* → `crustaces` ; *amande/noix/noisette* → `fruits-a-coque` ; etc.)
   et **signaler** que c'est déduit.
3. Unir les allergènes présents et, séparément, les traces possibles sur tous les ingrédients.

Libellés officiels et exemples : `data/allergenes-inco.md`.

## 3. Réconcilier avec le frontmatter (recette)
Comparer le résultat calculé au champ `allergenes` du frontmatter de la recette. En cas
d'**écart** (ex. frontmatter `aucun` mais un ingrédient apporte `lait`), **le signaler** et
proposer de corriger le frontmatter (vocabulaire INCO : `lait`, `oeufs`, `fruits-a-coque`...).

## 4. Restituer
```
🏷️ Allergènes — <cible>
Présents : lait, oeufs            (→ déclencheurs : feta [lait], œuf [oeufs])
Traces possibles : —
```
- Lister les **présents** (avec l'ingrédient déclencheur entre crochets), puis les **traces possibles**.
- Pour plusieurs plats : un bloc par plat + une **union** « allergènes du menu ».

## Encart obligatoire (toujours afficher)
> ⚠️ Aide à l'information allergènes — **ne vaut pas étiquetage certifié ni garantie d'absence**.
> Les recettes industrielles et la contamination croisée varient selon les produits/fournisseurs.
> Vérifier les étiquettes des matières premières. En cas de doute pour un convive allergique, ne pas servir.
