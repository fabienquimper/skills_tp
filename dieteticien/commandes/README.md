# Commandes & plans — sorties générées

Ce dossier contient les **sorties** du harness : plans de semaine et classeurs Excel récapitulatifs.

## Qu'est-ce qu'une « commande » ?

Une commande = **un ou plusieurs plats** avec, pour chacun, un **nombre de couverts**. Ça peut être
un buffet ponctuel comme l'ensemble des repas d'une semaine.

Deux formats d'entrée acceptés par `/fiche-commande-excel` :

1. **En ligne** : `curry-pois-chiches:20, veloute-potimarron:20, compote-pomme-sans-sucre:20`
2. **Fichier** (`--commande`) : un `.md` (typiquement `semaine-AAAA-MM-JJ.md` produit par
   `/organiser-semaine-dieteticien`) contenant des lignes `- <slug>: <couverts>`. Le générateur
   récupère tous les `slug: nombre` correspondant à une recette existante et **cumule** les doublons.

## Contenu généré

- `semaine-AAAA-MM-JJ.md` — plan de repas hebdomadaire + section `## Commande` réutilisable.
- `<nom>.xlsx` — classeur 4 onglets : **Récap commande**, **Liste de courses**,
  **Apports par plat**, **Allergènes par plat**.

## Exemple de chaînage

```bash
# 1. Planifier la semaine → écrit commandes/semaine-2026-07-06.md
/organiser-semaine-dieteticien surpoids, 2 personnes, sans fruits-a-coque

# 2. Générer le classeur Excel de cette semaine
/fiche-commande-excel --commande commandes/semaine-2026-07-06.md --nom semaine-06-07
```

> Les fichiers de ce dossier sont des artefacts régénérables. Valeurs nutritionnelles et allergènes
> **indicatives** — voir les avertissements des skills correspondants.
