# Conversions d'unités — vers grammes / millilitres

> 🔧 Un **miroir machine** de ces valeurs existe dans [`conversions.csv`](conversions.csv) (utilisé
> par les scripts comme `generer.py`). En cas de modification, mettre à jour **les deux** fichiers.

Table de référence pour fiabiliser l'extraction d'ingrédients et le calcul nutritionnel.
Toutes les valeurs sont **indicatives** (un « oignon moyen » varie de 90 à 130 g). En cas de
doute, privilégier une pesée. Les skills `ingredients-recette` et `apports-nutritionnels`
s'appuient sur cette table pour convertir les unités ménagères en grammes.

## Mesures de volume (ingrédients secs/épais)

| Unité            | Équivalent | Remarque                                  |
|------------------|-----------|-------------------------------------------|
| 1 c. à café (cc) | 5 ml      | ~5 g d'épice/sucre, ~4 g de farine, ~5 g d'huile |
| 1 c. à soupe (cs)| 15 ml     | ~15 g de liquide, ~12 g de farine, ~20 g de miel, ~13 g d'huile |
| 1 pincée         | ~0,5 g    | sel/épice                                 |
| 1 verre          | 200 ml    | verre à eau standard                      |
| 1 bol            | 350 ml    |                                           |
| 1 tasse (mug)    | 250 ml    |                                           |

## Liquides (1 ml ≈ 1 g sauf huile)

| Liquide          | Densité approx. | 1 c. à soupe | 1 verre (200 ml) |
|------------------|-----------------|--------------|------------------|
| eau / lait / bouillon | 1,00 g/ml  | 15 g         | 200 g            |
| huile            | 0,91 g/ml       | 13 g         | 182 g            |
| crème liquide    | 1,00 g/ml       | 15 g         | 200 g            |
| miel             | 1,40 g/ml       | 21 g         | —                |

## Poids unitaires d'aliments courants (1 pièce → g, partie comestible)

| Aliment              | Poids moyen | Aliment            | Poids moyen |
|----------------------|-------------|--------------------|-------------|
| oignon moyen         | 110 g       | gousse d'ail       | 5 g         |
| échalote             | 25 g        | tomate moyenne     | 120 g       |
| courgette moyenne    | 200 g       | poivron            | 150 g       |
| carotte moyenne      | 80 g        | pomme de terre moy.| 150 g       |
| potimarron entier    | 1 200 g     | citron             | 100 g       |
| pomme                | 150 g       | banane (épluchée)  | 120 g       |
| oeuf moyen           | 55 g        | blanc de poulet    | 150 g       |
| tranche de jambon    | 40 g        | tranche de pain    | 30 g        |
| saucisse fumée       | 75 g        | chou entier        | 1 000 g     |
| gousse de vanille    | 3 g         | feuille de laurier | 0,2 g       |

## Conseils d'arrondi

- Sous 10 g : arrondir au gramme. Entre 10 et 100 g : au 5 g. Au-delà : au 10 g.
- Pour une liste de courses, convertir vers l'**unité d'achat** réaliste (ex. 6 g d'ail →
  « 2 gousses » ; 1 050 g de riz → « 1 kg + 50 g »).
