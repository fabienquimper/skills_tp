# Rayons de magasin — classement des ingrédients

> 🔧 Un **miroir machine** de ce classement existe dans [`rayons.csv`](rayons.csv) (utilisé par les
> scripts comme `generer.py`). En cas de modification, mettre à jour **les deux** fichiers.

Sert au skill `liste-course-traiteur` pour regrouper la liste de courses par rayon
(parcours de magasin logique). Si un ingrédient n'est pas listé, appliquer la **règle par
mots-clés** ci-dessous, sinon le classer en « Divers / à vérifier ».

## Ordre de parcours conseillé

1. Fruits & légumes
2. Boucherie / Volaille
3. Poissonnerie
4. Crémerie / Frais
5. Épicerie salée
6. Épicerie sucrée
7. Surgelés
8. Boissons
9. Divers / à vérifier

## Classement par ingrédient

### Fruits & légumes
oignon, échalote, ail, tomate, courgette, poivron rouge, carotte, pomme de terre, potimarron,
épinard, salade verte, champignon de Paris, banane, pomme, citron, petits pois (frais),
gingembre, chou, thym

### Boucherie / Volaille
blanc de poulet, cuisse de poulet, boeuf haché 5%, jambon blanc, lardons, saucisse fumée

### Poissonnerie
thon (frais), saumon, crevette

### Crémerie / Frais
oeuf, lait demi-écrémé, yaourt nature, fromage blanc, crème fraîche, feta, emmental, parmesan,
beurre

### Épicerie salée
pois chiches, lentilles vertes, haricots rouges, riz blanc cuit, riz basmati cru, pâtes,
farine de blé, semoule de blé, pain, flocons d'avoine, thon au naturel (conserve),
tomate concassée, concentré de tomate, lait de coco, huile d'olive, huile de tournesol, sel,
bouillon de légumes, curry en poudre, curcuma, amandes, noix, moutarde, protéine de soja texturée

### Épicerie sucrée
sucre, miel, cannelle

### Surgelés
petits pois (surgelés), épinard (surgelé), légumes mélangés surgelés

### Boissons
eau, jus de fruit

## Règle par mots-clés (ingrédient non listé)

- légume / fruit / herbe fraîche / salade → **Fruits & légumes**
- poulet / boeuf / porc / agneau / jambon / lardon / saucisse → **Boucherie / Volaille**
- poisson / saumon / cabillaud / crevette / moule frais → **Poissonnerie**
- lait / yaourt / fromage / crème / beurre / oeuf → **Crémerie / Frais**
- conserve / pâtes / riz / farine / huile / épice / légumineuse sèche → **Épicerie salée**
- sucre / miel / chocolat / confiture / biscuit → **Épicerie sucrée**
- mention « surgelé » → **Surgelés**
- boisson / eau / jus / soda → **Boissons**
- sinon → **Divers / à vérifier**
