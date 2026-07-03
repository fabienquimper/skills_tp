# Guide enseignant — TP « Le Bistrot Numérique »

## Vue d'ensemble

| | |
|---|---|
| **Public** | Étudiants découvrant les agents IA (aucun prérequis en programmation) |
| **Durée** | 2h30 – 3h (le bonus absorbe les groupes rapides) |
| **Matériel** | 1 poste par binôme, Claude Code installé et authentifié |
| **Concepts** | Skills (procédures réutilisables), agents (spécialistes autonomes), orchestration multi-agents, principe du moindre privilège |

## Pourquoi ce scénario fonctionne

- **L'analogie est parfaite** : un restaurant EST un système multi-agents (des rôles spécialisés, une hiérarchie, des livrables). Les étudiants n'ont pas à apprendre un domaine métier en plus des concepts IA.
- **La boucle chef ↔ critique** est le pattern *generator/evaluator* (ou *actor/critic*) qu'on retrouve dans les vrais systèmes de production (génération de code + revue, rédaction + fact-checking). Les étudiants le vivent avant de le nommer.
- **Chaque partie produit un résultat visible immédiatement** (un menu, un tableau, un fichier) — pas d'effet tunnel.

## Déroulé conseillé et points de vigilance

### Partie 0 (10 min) — Setup
⚠️ Piège classique : les étudiants créent `.claude/` au mauvais endroit (dans leur home au lieu du projet). Vérifier avec `ls -la mon-bistrot/.claude`.

### Partie 1 (30 min) — Première skill
**Réponses attendues aux questions :**
- *Menu différent à chaque appel* : le LLM est non-déterministe. Bon moment pour introduire la notion de température / variabilité, et demander « comment rendriez-vous le résultat plus stable ? » (instructions plus contraignantes dans la skill).
- *Retrait du saumon* : la skill relit le fichier à chaque exécution → une skill n'est pas un texte figé mais une **procédure qui s'exécute sur des données vivantes**. C'est LE déclic à provoquer.
- *La `description:`* sert à Claude pour savoir **quand** utiliser la skill (découvrabilité). Une mauvaise description = une skill jamais déclenchée. Parallèle : la docstring d'une fonction.

### Partie 2 (25 min) — Skill avec paramètre
L'objectif caché est le transfert : la question 2.3 force les étudiants à projeter le concept dans leur futur métier. Exemples à suggérer si un groupe sèche : `/synthese-tickets urgents` (support), `/verif-paie mars` (RH), `/revue-pr` (dev).

### Partie 3 (40 min) — Les agents ⭐ cœur du TP
**Réponses attendues :**
- *Allers-retours* : généralement 0 à 2. Si le critique valide tout du premier coup, faire durcir son prompt (« sois plus sévère, note rarement au-dessus de 6 au premier passage ») — excellente illustration du **prompt engineering sur le comportement d'un agent**.
- *Pourquoi seulement `Read`* : principe du **moindre privilège**. Un agent créatif avec accès en écriture pourrait modifier les données ou écraser des fichiers. En production, c'est la première mesure de sécurité des systèmes agentiques.
- *Skill vs agent* : la skill est une **procédure** exécutée par Claude dans la conversation courante ; l'agent est un **délégué autonome** avec son propre contexte, sa personnalité, ses outils restreints, qui rend un rapport. Règle simple à donner : *tâche répétable et cadrée → skill ; expertise + autonomie + isolation → agent*.

⚠️ Vigilance : certains groupes écrivent des prompts d'agents trop vagues (« tu es un chef, cuisine »). Les faire comparer avec un binôme qui a détaillé le format de sortie — la différence de qualité est frappante et c'est la vraie leçon.

### Partie 4 (30 min) — Orchestration
Le moment « waouh » : les étudiants voient les agents s'échanger des informations sans intervention. Faire remarquer que la skill `service-complet` ne contient **aucune logique technique** — uniquement une description du processus métier en français. C'est le changement de paradigme : *on programme des organisations, plus des instructions*.

⚠️ Limiter à « 2 tours maximum par plat » est volontaire : sans borne, une boucle générateur/évaluateur peut ne jamais converger. Mentionner que c'est un garde-fou standard (max iterations) des frameworks multi-agents.

## Corrigé de la question de synthèse

| Cas | Choix | Justification |
|---|---|---|
| (a) Rapport hebdo des ventes | **Skill** | Tâche répétable, procédure fixe, données en entrée → sortie standardisée |
| (b) Négociation fournisseur | **Agent** | Multi-tours, stratégie adaptative, personnalité, contexte propre à maintenir |
| (c) Traduction des menus | **Skill** | Transformation simple et cadrée ; un agent serait surdimensionné |

Accepter toute réponse argumentée — c'est le raisonnement qui est noté, pas la case cochée.

## Barème indicatif (/20)

| Critère | Points |
|---|---|
| Skills fonctionnelles (parties 1-2) | 5 |
| Agents fonctionnels avec rôles distincts (partie 3) | 5 |
| Orchestration complète + `menu-final.md` produit (partie 4) | 4 |
| Réponses aux questions ✍️ (compréhension) | 4 |
| Question de synthèse (transfert) | 2 |
| Bonus réalisé | +2 |

## FAQ dépannage

- **« Ma skill n'apparaît pas »** → redémarrer Claude Code ; vérifier que le fichier s'appelle exactement `SKILL.md` dans un sous-dossier au nom de la skill.
- **« Claude ne délègue pas à mes agents »** → la `description:` de l'agent est trop vague, ou l'étudiant n'a pas nommé l'agent dans sa demande. Reformuler : « demande à l'agent chef de... ».
- **« L'agent chef invente des ingrédients »** → renforcer la contrainte dans son prompt (« Si un ingrédient manque, dis-le au lieu de l'inventer »). Belle occasion de parler d'hallucination et de la manière de la contenir par les instructions.
- **Quota/latence** : la partie 4 lance plusieurs agents ; prévoir que les groupes ne relancent pas `/service-complet` en boucle.

## Prolongements possibles (TP suivant)

1. Ajouter un serveur MCP (ex : météo) → le chef adapte le menu à la saison réelle.
2. Brancher `/analyse-avis` sur un vrai export d'avis Google Maps.
3. Introduire les hooks pour valider automatiquement le format de `menu-final.md`.
