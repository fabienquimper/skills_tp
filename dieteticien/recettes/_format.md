# Format canonique d'une recette

**Source de vérité** du format. Tous les skills lisent et écrivent les recettes dans ce format.
Une recette = un fichier `recettes/<slug>.md`. Respecter les noms de sections **à l'identique**
(les skills les recherchent littéralement).

## En-tête (frontmatter YAML)

```yaml
---
nom: Curry de pois chiches            # libellé affiché
slug: curry-pois-chiches             # = nom du fichier (kebab-case, sans accent)
portions: 4                          # nombre de portions de référence de la recette
profils: [saine, petit-budget]       # 1..n parmi : saine, diabetique, anorexie, gourmand, petit-budget
type-repas: plat                     # entrée | plat | dessert | petit-déjeuner | collation
temps-preparation-min: 15
temps-cuisson-min: 25
cout-estime-eur: 6.50                # coût total indicatif pour `portions` portions
allergenes: [aucun]                  # slugs INCO : gluten, crustaces, oeufs, poissons, arachides, soja, lait, fruits-a-coque, celeri, moutarde, sesame, sulfites, lupin, mollusques ; ou [aucun] (voir data/allergenes-inco.md)
regime: [vegetarien, vegan]          # tags libres : vegetarien, vegan, sans-gluten, halal, sans-porc...
---
```

## Corps (sections obligatoires, dans cet ordre)

### `## Ingrédients`
Une ligne par ingrédient, format : `- <quantité> <unité> <ingrédient>`
- Unités normalisées : `g`, `ml`, `pièce`, `c. à soupe`, `c. à café`, `pincée`, `gousse`.
- Le `<ingrédient>` doit, autant que possible, **correspondre à une clé** de
  `data/nutrition-100g.csv` (sinon il sera marqué « estimé » par le calcul nutritionnel).
- Quantités pour le nombre de `portions` indiqué dans le frontmatter.

Exemple :
```
## Ingrédients
- 250 g pois chiches
- 1 pièce oignon
- 2 gousse ail
- 400 ml lait de coco
- 200 g tomate concassée
- 1 c. à soupe curry en poudre
- 1 c. à soupe huile d'olive
- 2 pincée sel
```

### `## Préparation`
Étapes numérotées (`1.`, `2.`, ...), claires et concises.

### `## Notes`
Substitutions, conseils de conservation, adaptations par profil, astuce anti-gaspillage.
Pour les recettes portant un profil **médical** (diabetique, anorexie), recopier ici l'encart
d'avertissement du profil concerné (voir `profils/`).

## Règles de cohérence
- `slug` == nom de fichier.
- Chaque ligne d'ingrédient parsable selon `quantité / unité / ingrédient`.
- Toute recette ajoutée doit être référencée dans `recettes/README.md` (index).
