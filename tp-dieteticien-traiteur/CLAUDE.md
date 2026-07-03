# Harness Diététicien & Traiteur — repères pour l'agent

Projet de **skills** pour un diététicien-traiteur. Lis ceci avant d'exécuter un skill : ça évite
de chercher les fichiers au mauvais endroit.

## ⚠️ Hiérarchie des fichiers (à connaître absolument)

La **racine du harness** = ce répertoire (celui qui contient `recettes/` et `data/`). C'est
normalement ton **répertoire de travail courant**.

```
<racine>/                     ← ressources partagées ICI
├── CLAUDE.md                 ← ce fichier
├── README.md
├── recettes/                 ← base de recettes (.md) + _format.md + README (index)
├── profils/                  ← profils diététiques (saine, diabetique, surpoids…)
├── data/                     ← nutrition-100g.csv, allergenes.csv, conversions.(md|csv), rayons.(md|csv)
├── commandes/                ← sorties générées (plans de semaine .md, classeurs .xlsx)
└── .claude/skills/<nom>/SKILL.md   ← les skills (NE contiennent PAS les données)
```

**Règle :** quand un `SKILL.md` dit de lire `recettes/...`, `profils/...`, `data/...` ou
`commandes/...`, ces chemins sont **relatifs à la racine du harness ci-dessus**, PAS au dossier
du skill. Les skills ne stockent aucune donnée chez eux (seul `fiche-commande-excel` contient en
plus son script `generer.py`).

**Ne cherche pas** `recettes/` ou `data/` sous `.claude/skills/…`. Si tu doutes de l'emplacement,
la racine est le dossier contenant à la fois `recettes/` et `data/` (parent de `.claude/`). Tu
peux le confirmer en une commande : `ls recettes data profils` depuis la racine.

## Principe d'architecture

Skills fins, **données partagées**. Un skill ne ré-encode pas les profils/valeurs : il lit
`profils/` et `data/`. Enrichir une donnée profite à tous les skills.

## Cohérence des données (si tu modifies `data/`)

- `conversions.md` ↔ `conversions.csv` et `rayons.md` ↔ `rayons.csv` sont des paires
  (doc humaine ↔ miroir machine lu par `generer.py`) : **mets à jour les deux**.
- Tout ingrédient utilisé dans une recette doit exister dans `nutrition-100g.csv`,
  `allergenes.csv` et `rayons.csv` (sinon il est marqué « estimé » / rangé en « Divers »).
- Vocabulaire allergènes = **14 slugs INCO** (voir `data/allergenes-inco.md`).

## Cadre

Outil d'**aide** : repères nutritionnels/allergènes **indicatifs**, pas un avis médical ni un
étiquetage certifié. Conserver les encarts d'avertissement des profils médicaux.
