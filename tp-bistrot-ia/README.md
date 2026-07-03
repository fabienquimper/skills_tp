# TP — Le Bistrot Numérique 🍽️
### Skills et agents Claude : construire son équipe de restaurant virtuelle

**Durée :** 2h30 — 3h
**Prérequis :** Claude Code installé, savoir utiliser un terminal, notions de Markdown.
**Aucune ligne de code à écrire** : tout se fait en langage naturel dans des fichiers Markdown.

---

## 🎯 Objectifs pédagogiques

À la fin de ce TP, vous saurez :

1. Créer une **skill** (compétence réutilisable, invocable par `/nom`)
2. Créer un **agent** (spécialiste autonome avec son propre rôle et contexte)
3. Faire **collaborer plusieurs agents** entre eux (le cœur des systèmes multi-agents)
4. Comprendre **quand utiliser une skill vs un agent** — la question clé en entreprise

## 📖 Le scénario

Vous venez de reprendre un petit restaurant. Vous n'avez pas encore d'équipe...
alors vous allez la construire en IA :

| Rôle | Type | Mission |
|------|------|---------|
| 🍳 Le Chef | Agent | Invente des plats à partir des ingrédients disponibles |
| 🧐 Le Critique | Agent | Évalue les plats sans pitié et exige des corrections |
| 📋 Le menu du jour | Skill | Génère le menu affiché en vitrine |
| ⭐ L'analyse d'avis | Skill | Dépouille les avis clients et en tire des actions |
| 👔 Le Patron (vous) | Orchestrateur | Fait travailler tout ce monde ensemble |

---

## Partie 0 — Mise en place (10 min)

1. Créez un dossier de travail et copiez-y le dossier `donnees/` fourni :

```bash
mkdir mon-bistrot && cd mon-bistrot
cp -r ../donnees .
mkdir -p .claude/skills .claude/agents
claude
```

2. Vérifiez que Claude Code démarre. Tapez `bonjour` pour tester.

> 💡 **À retenir :** tout ce que vous mettez dans `.claude/` du projet est chargé
> automatiquement au démarrage de Claude Code dans ce dossier.

---

## Partie 1 — Votre première skill : le menu du jour (30 min)

Une **skill** est un fichier `SKILL.md` qui décrit une procédure. Elle s'invoque
avec `/nom-de-la-skill` et Claude suit alors les instructions du fichier.

**1.1** Créez le fichier `.claude/skills/menu-du-jour/SKILL.md` :

```markdown
---
name: menu-du-jour
description: Génère le menu du jour du bistrot à partir des ingrédients disponibles
---

Tu es responsable du menu du Bistrot Numérique.

1. Lis le fichier `donnees/ingredients.txt` pour connaître les ingrédients du jour.
2. Compose un menu avec : 1 entrée, 1 plat, 1 dessert.
3. N'utilise QUE les ingrédients disponibles (le sel, poivre, huile sont supposés en stock).
4. Donne un nom appétissant à chaque plat et une description d'une ligne.
5. Affiche le menu dans un joli cadre en Markdown, avec un prix cohérent par plat.
```

**1.2** Redémarrez Claude Code (`Ctrl+C` puis `claude`), puis tapez :

```
/menu-du-jour
```

**1.3** ✍️ **Questions (à noter dans votre compte-rendu) :**
- Que se passe-t-il si vous relancez `/menu-du-jour` deux fois ? Le menu est-il identique ? Pourquoi ?
- Modifiez `donnees/ingredients.txt` (retirez le saumon). Relancez. Que constatez-vous ?
- À quoi sert la ligne `description:` dans l'en-tête ? *(Indice : demandez à Claude "quelles skills as-tu ?")*

---

## Partie 2 — Une skill avec paramètre : l'analyse d'avis (25 min)

Les skills peuvent recevoir des **arguments** : `/analyse-avis negatifs` par exemple.

**2.1** Créez `.claude/skills/analyse-avis/SKILL.md` :

```markdown
---
name: analyse-avis
description: Analyse les avis clients du bistrot et propose des actions concrètes
---

Tu es l'assistant qualité du Bistrot Numérique.

1. Lis le fichier `donnees/avis-clients.txt`.
2. Si un argument est fourni (ex : "negatifs", "service", "cuisine"),
   concentre-toi uniquement sur ce filtre. Sinon, analyse tout.
3. Produis un tableau : Thème | Sentiment | Nombre d'avis concernés.
4. Termine par les 3 actions prioritaires que le patron devrait mener,
   classées par impact.
```

**2.2** Testez les trois appels suivants et comparez les résultats :

```
/analyse-avis
/analyse-avis negatifs
/analyse-avis service
```

**2.3** ✍️ **Question :** en entreprise, citez deux exemples de skills de ce type qui
feraient gagner du temps dans VOTRE domaine (compta, RH, support, dev...).

---

## Partie 3 — Les agents : le Chef et le Critique (40 min)

Un **agent** est différent d'une skill : c'est un **spécialiste autonome** qui
travaille dans son propre contexte, avec sa propre personnalité et ses propres
outils, puis rend son rapport. On le définit dans `.claude/agents/`.

**3.1** Créez `.claude/agents/chef.md` :

```markdown
---
name: chef
description: Chef cuisinier créatif du bistrot. À utiliser pour inventer ou corriger des plats.
tools: Read
---

Tu es Auguste, chef cuisinier passionné du Bistrot Numérique.
Tu es créatif, un peu théâtral, et fier de ta cuisine.

Quand on te demande un plat :
- Lis `donnees/ingredients.txt` et n'utilise que ces ingrédients.
- Propose le plat avec : nom, ingrédients, technique de cuisson, dressage.
- Si on te transmet une critique, corrige ton plat SANS te vexer (ou presque)
  et explique ce que tu as changé.
```

**3.2** Créez `.claude/agents/critique.md` :

```markdown
---
name: critique
description: Critique gastronomique exigeant. À utiliser pour évaluer un plat proposé.
tools: Read
---

Tu es Anton, critique gastronomique redouté. Tu es exigeant mais juste.

Quand on te présente un plat :
- Note-le sur 10.
- Donne 2 points forts et 2 points faibles PRÉCIS (équilibre, saisonnalité,
  cohérence des ingrédients, originalité).
- Si la note est < 7, exige explicitement une correction en indiquant quoi changer.
- Si la note est ≥ 7, valide le plat pour le menu.
```

**3.3** Redémarrez Claude Code, puis tapez en langage naturel :

```
Demande au chef de proposer un plat du jour, puis fais-le évaluer par le
critique. Si la note est inférieure à 7, renvoie la critique au chef pour
qu'il corrige, et refais évaluer. Montre-moi chaque échange.
```

**3.4** Observez bien ce qui se passe : Claude (le "patron") **délègue** aux deux
agents et fait circuler l'information entre eux. C'est le pattern
**orchestrateur / travailleurs**, à la base de la plupart des systèmes
multi-agents en production.

**3.5** ✍️ **Questions :**
- Combien d'allers-retours ont été nécessaires avant validation ?
- Pourquoi avoir donné à chaque agent uniquement l'outil `Read` ? Que risquerait-on avec un accès complet ?
- Quelle différence fondamentale voyez-vous entre appeler `/menu-du-jour` (skill) et demander au `chef` (agent) ?

---

## Partie 4 — L'orchestration : le service complet (30 min)

On assemble tout : une skill qui pilote les agents. C'est votre restaurant qui tourne tout seul.

**4.1** Créez `.claude/skills/service-complet/SKILL.md` :

```markdown
---
name: service-complet
description: Lance un service complet - création du menu par le chef, validation par le critique, publication
---

Tu es le patron du Bistrot Numérique. Orchestre un service complet :

1. Demande à l'agent `chef` de proposer une entrée, un plat et un dessert
   à partir de `donnees/ingredients.txt`.
2. Fais évaluer CHAQUE plat par l'agent `critique`.
3. Pour tout plat noté < 7 : renvoie la critique au chef, obtiens une
   correction, refais évaluer (2 tours maximum par plat).
4. Une fois les 3 plats validés, écris le menu final dans `menu-final.md`
   avec les notes du critique en garantie de qualité.
5. Affiche-moi un résumé : plats retenus, notes, nombre de corrections.
```

**4.2** Lancez :

```
/service-complet
```

**4.3** Ouvrez `menu-final.md` — c'est le livrable produit par votre équipe d'agents.

---

## 🏆 Bonus (si le temps le permet)

Choisissez UN défi :

- **Le Comptable** : créez un agent `comptable` qui vérifie que chaque plat coûte
  moins de 8€ en matières premières (inventez un fichier `donnees/prix-ingredients.txt`).
  Intégrez-le dans `/service-complet`.
- **Le client mystère** : créez un agent `client` avec une allergie (gluten) qui
  vérifie le menu final et rédige un avis... qui alimente `/analyse-avis` !
- **Le duel de chefs** : créez un second chef (`chef-vegan.md`) et faites départager
  leurs plats par le critique.

---

## 📝 Livrables attendus

1. Votre dossier `mon-bistrot/` complet (skills + agents + `menu-final.md`)
2. Un compte-rendu avec les réponses aux questions ✍️
3. **La question de synthèse** (5-10 lignes) :
   > *"Skill ou agent : pour chacun des cas suivants, que choisiriez-vous et pourquoi ?
   > (a) générer le rapport hebdomadaire des ventes, (b) négocier avec un fournisseur
   > en plusieurs échanges, (c) traduire les menus en anglais."*
