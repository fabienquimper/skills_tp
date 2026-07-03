# Allergènes à déclaration obligatoire — référence INCO

Les **14 allergènes** dont la présence doit être déclarée dans l'Union européenne (règlement
**INCO n° 1169/2011**), y compris pour la vente de plats préparés / traiteur. Source de vérité
des libellés utilisés par le harness (`data/allergenes.csv`, skill `allergenes-recette`, onglet
« Allergènes par plat » du classeur Excel).

> ⚠️ **Avertissement réglementaire.** Cet outil est une **aide à l'information allergènes**, il ne
> constitue **ni un étiquetage certifié, ni une garantie d'absence**. Les recettes industrielles,
> additifs et risques de **contamination croisée** varient selon les produits et fournisseurs.
> Toujours **vérifier les étiquettes** des matières premières et, en cas de doute pour un convive
> allergique, ne pas servir. La responsabilité de la déclaration finale incombe au professionnel.

## Les 14 catégories (slug → libellé → exemples)

| Slug | Libellé officiel | Exemples courants |
|------|------------------|-------------------|
| `gluten` | Céréales contenant du gluten | blé, seigle, orge, avoine, épeautre + dérivés (farine, pain, pâtes, semoule) |
| `crustaces` | Crustacés | crevette, crabe, langoustine, homard |
| `oeufs` | Œufs | œuf et dérivés (mayonnaise, certaines pâtes) |
| `poissons` | Poissons | thon, saumon, cabillaud, anchois, sauces de poisson |
| `arachides` | Arachides | cacahuète, huile/beurre d'arachide |
| `soja` | Soja | tofu, sauce soja, lécithine de soja |
| `lait` | Lait | lait, beurre, crème, fromage, yaourt (inclut le lactose) |
| `fruits-a-coque` | Fruits à coque | amande, noisette, noix, noix de cajou, pistache, noix de pécan, macadamia |
| `celeri` | Céleri | céleri-branche/rave, sel de céleri, bouillons |
| `moutarde` | Moutarde | moutarde, graines, mélanges d'épices |
| `sesame` | Graines de sésame | graines, huile de sésame, tahini, pain aux graines |
| `sulfites` | Anhydride sulfureux et sulfites | > 10 mg/kg : fruits secs, vin, certaines conserves |
| `lupin` | Lupin | farine de lupin (boulangerie, substituts) |
| `mollusques` | Mollusques | moule, huître, calamar, escargot |

## Notes d'usage
- La **noix de coco** n'est **pas** classée parmi les fruits à coque INCO → le lait de coco ne
  porte pas l'allergène `fruits-a-coque`.
- Le **bouillon de légumes** du commerce contient très souvent du **céleri** → déclaré présent par défaut.
- Distinguer **présent** (dans l'ingrédient) et **traces possibles** (selon produit / contamination).
