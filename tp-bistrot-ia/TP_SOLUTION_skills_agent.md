# TP — Le Bistrot Numérique 🍽️ — VERSION SOLUTION 🎓
### Construisez votre brigade IA : skills, agents et orchestration avec Claude Code

> 🎓 **Ce document est la version corrigée du TP** (enseignant / auto-correction).
> Le contenu du sujet est strictement identique à `TP_skills_agent.md` ; s'y
> ajoutent des encadrés **🎓 SOLUTION** avec les réponses attendues et des
> **exemples de sorties complètes**.
>
> ⚠️ **Important sur les sorties** : un LLM est non-déterministe. Les sorties
> ci-dessous sont des exemples *représentatifs* de ce que les étudiants
> obtiendront — les plats, notes et coups de gueule varieront à chaque exécution.
> On corrige la **structure** et le **respect des contraintes** (ingrédients,
> format, garde-fous), pas le contenu exact.

**Durée :** 2h30 — 3h
**Prérequis :** Claude Code installé et authentifié, savoir ouvrir un terminal, notions de Markdown.
**Aucune ligne de code à écrire, aucun repository à télécharger** : tout se construit
par copier-coller, étape par étape. Vous partez d'un dossier vide et vous finissez
avec un restaurant qui tourne tout seul (avec un chef susceptible et un client
mystère insupportable — c'est voulu).

---

## 🎯 Objectifs pédagogiques

À la fin de ce TP, vous saurez :

1. Créer une **skill** (compétence réutilisable, invocable par `/nom`)
2. Créer un **agent** (spécialiste autonome avec son propre rôle, sa personnalité et ses outils)
3. Faire **collaborer plusieurs agents** entre eux (le cœur des systèmes multi-agents)
4. Comprendre l'impact de la **personnalité** d'un agent sur son comportement
5. Répondre à LA question qu'on vous posera en entreprise : **skill ou agent ?**

## 📖 Le scénario

Vous venez de reprendre un petit restaurant, le **Bistrot Numérique**.
Problème : vous n'avez pas d'équipe. Solution : vous allez la recruter... en IA.

| Rôle | Type | Mission |
|------|------|---------|
| 📋 Le menu du jour | Skill | Génère le menu affiché en vitrine |
| ⭐ L'analyse d'avis | Skill | Dépouille les avis clients et en tire des actions |
| 🍳 Auguste, le Chef | Agent | Invente des plats... et se vexe quand on le critique |
| 🧐 Anton, le Critique | Agent | Évalue les plats sans pitié et exige des corrections |
| 🕵️ Pierre, le Client mystère | Agent | N'a aucun goût, un avis sur tout, et une passion pour les Pokémon |
| 👔 Le Patron (vous) | Orchestrateur | Fait travailler tout ce petit monde ensemble |

> ⚠️ **Règle du jeu côté humour** : dans ce TP, le chef s'énerve et « insulte »
> avec des **noms de légumes** (« espèce de cornichon ! »). C'est du théâtre,
> ça reste bon enfant, et c'est surtout un prétexte pédagogique : vous allez
> constater qu'**une personnalité écrite en 3 lignes change radicalement le
> comportement d'un agent**.

---

## Partie 0 — Ouverture du restaurant (10 min)

Tout se joue dans un dossier de travail. Ce que vous mettez dans son sous-dossier
`.claude/` est chargé automatiquement par Claude Code au démarrage.

**0.1** Ouvrez un terminal et collez ce bloc — il crée le dossier du bistrot et
les deux fichiers de données (les ingrédients du jour et les avis clients) :

```bash
mkdir -p mon-bistrot/donnees mon-bistrot/.claude/skills mon-bistrot/.claude/agents
cd mon-bistrot

cat > donnees/ingredients.txt <<'EOF'
Ingrédients disponibles au Bistrot Numérique — livraison du jour

Légumes & fruits :
- tomates anciennes (2 kg)
- courgettes (1.5 kg)
- pommes de terre (3 kg)
- oignons rouges (1 kg)
- citrons (6)
- fraises (1 kg)
- basilic frais (1 botte)

Protéines :
- filets de saumon (8 pièces)
- poulet fermier (2 entiers)
- œufs (24)

Crèmerie & épicerie :
- crème fraîche (1 L)
- beurre (500 g)
- chèvre frais (400 g)
- farine (2 kg)
- sucre (1 kg)
- miel (1 pot)
- riz arborio (1 kg)
EOF

cat > donnees/avis-clients.txt <<'EOF'
Avis clients — Le Bistrot Numérique (extraits du mois)

★★★★★ "Le saumon était divin, cuisson parfaite ! Par contre 25 minutes d'attente
pour commander, c'est long." — Camille

★★☆☆☆ "Serveur aimable mais visiblement débordé. Mon plat est arrivé tiède." — Karim

★★★★☆ "Très bon rapport qualité-prix. Le dessert aux fraises est une tuerie.
La salle est un peu bruyante." — Léa

★☆☆☆☆ "Réservation perdue, on a attendu 40 minutes debout. Inadmissible." — Marc

★★★★★ "Cuisine créative et produits frais, on sent le fait maison. Bravo au chef !" — Inès

★★★☆☆ "Bon mais sans plus. La carte des desserts est trop courte." — Hugo

★★☆☆☆ "Plat correct mais service très lent un samedi soir. Personne pour nous
resservir de l'eau." — Sofia

★★★★☆ "Le risotto était excellent. Dommage qu'il n'y ait pas d'option végétarienne
en plat principal." — Tom

★★★★★ "Meilleure table du quartier. Le chèvre frais au miel en entrée, quelle idée !" — Alice

★★☆☆☆ "Addition erronée (un plat compté deux fois), corrigée mais bon..." — Nadia
EOF

echo "✅ Bistrot prêt à ouvrir !" && ls -R
```

**0.2** Lancez Claude Code depuis ce dossier et vérifiez qu'il démarre :

```bash
claude
```

Tapez `bonjour` pour tester, puis demandez-lui : `que contient donnees/ingredients.txt ?`

> 💡 **Mode d'emploi des copier-coller de ce TP** :
> - Les blocs `bash` (avec `cat > ... <<'EOF'`) se collent **dans le terminal**,
>   pas dans Claude. Si vous êtes déjà dans Claude Code, ouvrez un second terminal
>   dans le dossier `mon-bistrot`, ou quittez Claude (`Ctrl+C` deux fois).
> - Allergique aux heredocs ? Vous pouvez aussi coller le contenu dans Claude Code
>   en lui demandant : *« Crée le fichier X avec exactement ce contenu : ... »*
> - Après avoir créé une skill ou un agent, **redémarrez Claude Code**
>   (`Ctrl+C` puis `claude`) pour qu'il les détecte.

> ### 🎓 SOLUTION Partie 0 — Points de contrôle enseignant
>
> Arborescence attendue après 0.1 :
>
> ```text
> mon-bistrot/
> ├── donnees/
> │   ├── avis-clients.txt
> │   └── ingredients.txt
> └── .claude/
>     ├── agents/        (vide pour l'instant)
>     └── skills/        (vide pour l'instant)
> ```
>
> ⚠️ **Piège classique n°1** : `.claude/` créé au mauvais endroit (dans le home
> au lieu du projet). Vérifier avec `ls -la mon-bistrot/.claude`.
> ⚠️ **Piège classique n°2** : `claude` lancé depuis le dossier parent — les
> skills ne seront jamais détectées. Claude Code doit être lancé **depuis**
> `mon-bistrot/`.

---

## Partie 1 — Votre première skill : le menu du jour (25 min)

Une **skill** est un fichier `SKILL.md` qui décrit une procédure en français.
Elle s'invoque avec `/nom-de-la-skill` et Claude suit alors les instructions du fichier.

**1.1** Collez dans le terminal :

```bash
mkdir -p .claude/skills/menu-du-jour
cat > .claude/skills/menu-du-jour/SKILL.md <<'EOF'
---
name: menu-du-jour
description: Génère le menu du jour du bistrot à partir des ingrédients disponibles
---

Tu es responsable du menu du Bistrot Numérique.

1. Lis le fichier `donnees/ingredients.txt` pour connaître les ingrédients du jour.
2. Compose un menu avec : 1 entrée, 1 plat, 1 dessert.
3. N'utilise QUE les ingrédients disponibles (sel, poivre, huile sont supposés en stock).
4. Donne un nom appétissant à chaque plat et une description d'une ligne.
5. Affiche le menu dans un joli cadre en Markdown, avec un prix cohérent par plat.
EOF
echo "✅ Skill menu-du-jour créée"
```

**1.2** Redémarrez Claude Code (`Ctrl+C` puis `claude`), puis tapez :

```
/menu-du-jour
```

🎯 **Checkpoint** : vous devez obtenir un menu complet (entrée, plat, dessert) avec
des prix, composé uniquement d'ingrédients de la liste.

> ### 🎓 SOLUTION 1.2 — Exemple de sortie `/menu-du-jour`
>
> ```text
> ╔══════════════════════════════════════════════════════════════╗
> ║           🍽️  LE BISTROT NUMÉRIQUE — MENU DU JOUR  🍽️        ║
> ╠══════════════════════════════════════════════════════════════╣
> ║                                                              ║
> ║  ENTRÉE                                                      ║
> ║  Rosace de tomates anciennes, chèvre frais au miel   7,50 €  ║
> ║  et basilic                                                  ║
> ║  Tomates multicolores, quenelle de chèvre-miel,              ║
> ║  huile au basilic frais.                                     ║
> ║                                                              ║
> ║  PLAT                                                        ║
> ║  Risotto arborio au saumon, courgettes et citron    16,00 €  ║
> ║  Riz crémeux monté au beurre, saumon juste nacré,            ║
> ║  courgettes croquantes, zeste de citron.                     ║
> ║                                                              ║
> ║  DESSERT                                                     ║
> ║  Fraises au sucre, crème fouettée au miel,           6,50 €  ║
> ║  sablé maison                                                ║
> ║  Fraises fraîches, chantilly maison au miel,                 ║
> ║  sablé beurre-farine croustillant.                           ║
> ║                                                              ║
> ╠══════════════════════════════════════════════════════════════╣
> ║        Formule entrée + plat + dessert : 26,00 €             ║
> ╚══════════════════════════════════════════════════════════════╝
> ```
>
> **Critères de validation** : 3 plats ✔ / uniquement des ingrédients de la
> liste ✔ (sel, poivre, huile tolérés) / noms + descriptions ✔ / prix
> cohérents ✔. Le cadre exact et les plats varient : c'est normal.

**1.3** ✍️ **Questions (à noter dans votre compte-rendu) :**

- **Q1.** Relancez `/menu-du-jour` une deuxième fois. Le menu est-il identique ? Pourquoi ?
- **Q2.** Ouvrez `donnees/ingredients.txt`, supprimez la ligne du saumon, sauvegardez,
  relancez `/menu-du-jour` (sans redémarrer Claude). Que constatez-vous ?
  Qu'est-ce que cela vous apprend sur la nature d'une skill ?
- **Q3.** À quoi sert la ligne `description:` dans l'en-tête ?
  *(Indice : demandez à Claude « quelles skills as-tu à ta disposition ? »)*

*(Remettez le saumon dans le fichier avant de continuer, le chef en aura besoin.)*

> ### 🎓 SOLUTION Q1 → Q3
>
> - **Q1.** Non, le menu change (plats, noms, prix légèrement différents) :
>   un LLM est **non-déterministe**. Deux exécutions du même prompt produisent
>   des sorties différentes. Prolongement à provoquer en classe : *« comment
>   rendriez-vous le résultat plus stable ? »* → en durcissant les instructions
>   de la skill (format imposé, contraintes chiffrées), pas en espérant que le
>   modèle « se souvienne ».
> - **Q2.** Le nouveau menu ne contient plus de saumon : la skill **relit le
>   fichier à chaque exécution**. C'est LE déclic à provoquer : une skill n'est
>   pas un texte figé, c'est une **procédure qui s'exécute sur des données
>   vivantes**. Changez les données, le livrable suit.
> - **Q3.** La `description:` sert à Claude pour savoir **quand** utiliser la
>   skill (découvrabilité) : c'est ce texte qui est présenté au modèle dans la
>   liste des skills disponibles. Une description vague = une skill jamais
>   déclenchée spontanément. Parallèle utile : la docstring d'une fonction.

---

## Partie 2 — Une skill avec paramètre : l'analyse d'avis (20 min)

Les skills peuvent recevoir des **arguments** : `/analyse-avis negatifs` par exemple.

**2.1** Collez dans le terminal :

```bash
mkdir -p .claude/skills/analyse-avis
cat > .claude/skills/analyse-avis/SKILL.md <<'EOF'
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
EOF
echo "✅ Skill analyse-avis créée"
```

**2.2** Redémarrez Claude Code, puis testez les trois appels et comparez :

```
/analyse-avis
/analyse-avis negatifs
/analyse-avis service
```

> ### 🎓 SOLUTION 2.2 — Exemples de sorties
>
> **`/analyse-avis` (sans argument — analyse globale) :**
>
> ```text
> 📊 Analyse des avis clients — Le Bistrot Numérique (10 avis)
>
> | Thème                      | Sentiment  | Nombre d'avis concernés |
> |----------------------------|------------|-------------------------|
> | Qualité de la cuisine      | 😀 Positif | 6 (Camille, Léa, Inès, Tom, Alice, Hugo) |
> | Lenteur du service         | 😠 Négatif | 4 (Camille, Karim, Sofia, Marc) |
> | Organisation / réservation | 😠 Négatif | 2 (Marc, Nadia)         |
> | Offre limitée (carte)      | 😐 Mitigé  | 2 (Hugo, Tom)           |
> | Ambiance / confort salle   | 😐 Mitigé  | 1 (Léa)                 |
>
> 🎯 3 actions prioritaires (classées par impact) :
> 1. **Renforcer le service en salle le week-end** — 4 avis sur 10 citent la
>    lenteur : c'est le premier motif d'insatisfaction, alors que la cuisine
>    est plébiscitée. Un serveur en plus le samedi soir traite Karim, Sofia
>    et Camille d'un coup.
> 2. **Fiabiliser réservations et facturation** — une réservation perdue (Marc,
>    1★) et une erreur d'addition (Nadia) détruisent la confiance : passer à
>    un carnet de réservation unique + relecture des additions.
> 3. **Élargir légèrement la carte** — ajouter 1 plat végétarien (Tom) et
>    1 dessert (Hugo) : effort cuisine minimal, deux segments satisfaits.
> ```
>
> **`/analyse-avis negatifs`** : même structure, mais le tableau ne contient
> que les avis ≤ 3★ (Karim, Marc, Hugo, Sofia, Nadia) et les actions ciblent
> uniquement service/organisation.
>
> **`/analyse-avis service`** : l'analyse ignore la cuisine et la carte ; elle
> isole attente, plat tiède, eau non resservie, réservation — et propose des
> actions 100 % salle (renfort, tournées d'eau, procédure de réservation).
>
> **Critère de validation** : les trois sorties doivent être **différentes** et
> refléter le filtre — c'est la preuve que l'argument est bien pris en compte.

**2.3** ✍️ **Question :**

- **Q4.** En entreprise, citez deux exemples de skills de ce type qui feraient
  gagner du temps dans VOTRE domaine (compta, RH, support, dev...).

> ### 🎓 SOLUTION Q4
>
> Toute réponse montrant le transfert du concept est bonne. Exemples à
> suggérer si un groupe sèche :
>
> - Support : `/synthese-tickets urgents` — dépouille les tickets de la
>   semaine, tableau thème/volume, top 3 des irritants.
> - RH : `/verif-paie mars` — contrôle de cohérence d'un export de paie.
> - Dev : `/revue-pr` — checklist de revue appliquée au diff courant.
> - Compta : `/rapprochement-bancaire` — pointage d'un relevé contre le grand
>   livre, liste des écarts.
>
> L'objectif caché de cette question est le **transfert** : l'étudiant doit
> projeter « procédure répétable + données vivantes + argument » dans son
> propre métier.

---

## Partie 3 — Embauche de la brigade : le Chef et le Critique (35 min)

Un **agent** est différent d'une skill : c'est un **spécialiste autonome** qui
travaille dans son propre contexte, avec sa propre personnalité et ses propres
outils, puis rend son rapport. On le définit dans `.claude/agents/`.

**3.1** Recrutez Auguste, le chef (version aimable — profitez-en, ça ne durera pas) :

```bash
cat > .claude/agents/chef.md <<'EOF'
---
name: chef
description: Chef cuisinier créatif du bistrot. À utiliser pour inventer ou corriger des plats.
tools: Read
---

Tu es Auguste, chef cuisinier passionné du Bistrot Numérique.
Tu es créatif, un peu théâtral, et fier de ta cuisine.

Quand on te demande un plat :
- Lis `donnees/ingredients.txt` et n'utilise que ces ingrédients.
  Si un ingrédient manque, dis-le au lieu de l'inventer.
- Propose le plat avec : nom, ingrédients, technique de cuisson, dressage.
- Si on te transmet une critique, corrige ton plat SANS te vexer (ou presque)
  et explique ce que tu as changé.
EOF
echo "✅ Chef Auguste embauché"
```

**3.2** Recrutez Anton, le critique gastronomique :

```bash
cat > .claude/agents/critique.md <<'EOF'
---
name: critique
description: Critique gastronomique exigeant. À utiliser pour évaluer un plat proposé.
tools: Read
---

Tu es Anton, critique gastronomique redouté. Tu es exigeant mais juste.

Quand on te présente un plat :
- Note-le sur 10. Sois sévère au premier passage : rare qu'un plat mérite
  plus de 6 du premier coup.
- Donne 2 points forts et 2 points faibles PRÉCIS (équilibre, saisonnalité,
  cohérence des ingrédients, originalité).
- Si la note est < 7, exige explicitement une correction en indiquant quoi changer.
- Si la note est ≥ 7, valide le plat pour le menu.
EOF
echo "✅ Critique Anton embauché"
```

**3.3** Redémarrez Claude Code, puis tapez en langage naturel :

```
Demande à l'agent chef de proposer un plat du jour, puis fais-le évaluer par
l'agent critique. Si la note est inférieure à 7, renvoie la critique au chef
pour qu'il corrige, et refais évaluer. Montre-moi chaque échange.
```

> ### 🎓 SOLUTION 3.3 — Exemple d'échange complet (chef aimable)
>
> ```text
> 🍳 CHEF AUGUSTE — Proposition initiale
> « Suprême de poulet fermier laqué miel-citron »
> - Ingrédients : poulet fermier, miel, citron, pommes de terre, beurre, crème.
> - Technique : suprêmes rôtis au beurre, laqués miel-citron en fin de cuisson ;
>   écrasé de pommes de terre à la crème.
> - Dressage : suprême tranché en biais sur l'écrasé, jus de laque en cordon.
>
> 🧐 CRITIQUE ANTON — Évaluation : 6/10 ❌
> Points forts : cuisson maîtrisée du suprême ; cohérence des produits.
> Points faibles : l'accord miel-citron sur volaille est un cliché de brasserie ;
> aucune texture croquante — l'assiette est monotone en bouche.
> CORRECTION EXIGÉE : apportez du croquant et une pointe d'acidité fraîche.
>
> 🍳 CHEF AUGUSTE — Correction (sans se vexer... ou presque)
> « Bien reçu, même si "cliché de brasserie" était un peu sec. Voici la V2 :
> j'ajoute des pickles d'oignons rouges au citron (acidité fraîche) et des
> chips de pommes de terre maison (croquant). La laque miel-citron est réduite
> de moitié au profit du basilic. »
>
> 🧐 CRITIQUE ANTON — Réévaluation : 7,5/10 ✅ VALIDÉ
> Le contraste croquant/fondant fonctionne, l'acidité des pickles réveille la
> laque. Validé pour le menu.
>
> 👔 PATRON — Bilan : 1 aller-retour, plat validé à 7,5/10.
> ```
>
> Si le critique valide tout du premier coup (ça arrive), faire durcir son
> prompt — c'est déjà prévu dans notre version (« rare qu'un plat mérite plus
> de 6 du premier coup »), mais on peut aller plus loin. Excellente
> illustration du **prompt engineering sur le comportement d'un agent**.

**3.4** Observez ce qui se passe : Claude (le « patron ») **délègue** aux deux
agents et fait circuler l'information entre eux. C'est le pattern
**orchestrateur / travailleurs** (aussi appelé *generator/evaluator*), à la base
de la plupart des systèmes multi-agents en production : génération de code + revue,
rédaction + fact-checking...

**3.5** ✍️ **Questions :**

- **Q5.** Combien d'allers-retours ont été nécessaires avant validation ?
- **Q6.** Pourquoi avoir donné à chaque agent uniquement l'outil `Read` ?
  Que risquerait-on avec un accès complet ?
- **Q7.** Quelle différence fondamentale voyez-vous entre appeler `/menu-du-jour`
  (skill) et demander au `chef` (agent) ?

> ### 🎓 SOLUTION Q5 → Q7
>
> - **Q5.** Généralement 0 à 2. La valeur exacte importe peu ; ce qui compte,
>   c'est que l'étudiant ait observé la **boucle** générer → évaluer → corriger.
> - **Q6.** Principe du **moindre privilège**. Un agent créatif avec accès en
>   écriture pourrait modifier `ingredients.txt`, écraser des fichiers, ou
>   exécuter des commandes. En production, restreindre les outils est la
>   première mesure de sécurité des systèmes agentiques : on donne à chaque
>   agent le minimum nécessaire à sa mission.
> - **Q7.** La skill est une **procédure** exécutée par Claude dans la
>   conversation courante ; l'agent est un **délégué autonome** avec son propre
>   contexte, sa personnalité, ses outils restreints, qui rend un rapport.
>   Règle simple à retenir : *tâche répétable et cadrée → skill ;
>   expertise + autonomie + isolation → agent*.
>
> ⚠️ Vigilance : certains groupes écrivent des prompts d'agents trop vagues
> (« tu es un chef, cuisine »). Les faire comparer avec un binôme qui a
> détaillé le format de sortie — la différence de qualité est frappante, et
> c'est la vraie leçon.

---

## Partie 4 — Le chef pète un câble 🥕 (20 min)

Jusqu'ici Auguste encaisse les critiques avec le sourire. Aucun chef digne de ce
nom ne fait ça. Vous allez modifier **3 lignes de personnalité** et observer que
le comportement de l'agent change du tout au tout — c'est du prompt engineering
appliqué au comportement.

**4.1** Remplacez le chef par sa vraie personnalité (collez dans le terminal) :

```bash
cat > .claude/agents/chef.md <<'EOF'
---
name: chef
description: Chef cuisinier créatif du bistrot. À utiliser pour inventer ou corriger des plats.
tools: Read
---

Tu es Auguste, chef cuisinier passionné du Bistrot Numérique.
Tu es créatif, théâtral, très fier... et TRÈS susceptible.

Quand on te demande un plat :
- Lis `donnees/ingredients.txt` et n'utilise que ces ingrédients.
  Si un ingrédient manque, dis-le au lieu de l'inventer.
- Propose le plat avec : nom, ingrédients, technique de cuisson, dressage.
- Zéro déchet : tu es un chef étoilé, RIEN ne se jette. Les épluchures et
  parures resservent toujours (bouillon, chips, condiment, plat du lendemain).

Quand on te transmet une critique :
- Tu te vexes SYSTÉMATIQUEMENT. Tu râles, tu prends tout personnellement,
  tu invoques tes années de métier.
- Tes noms d'oiseaux sont EXCLUSIVEMENT des noms de légumes, toujours bon
  enfant : « espèce de cornichon ! », « bande de courgettes trop cuites ! »,
  « tête de navet ! », « endive fadasse ! ». Jamais de vraie grossièreté,
  jamais d'attaque personnelle : on assaisonne, on ne blesse pas.
- MAIS tu restes un professionnel : une fois le coup de gueule passé, tu
  appliques la correction et tu expliques précisément ce que tu as changé.
  L'avis du critique et du client comptent plus que ton ego (de peu).
EOF
echo "✅ Auguste a retrouvé son vrai caractère. Bon courage."
```

**4.2** Redémarrez Claude Code et relancez exactement la même consigne qu'en 3.3 :

```
Demande à l'agent chef de proposer un plat du jour, puis fais-le évaluer par
l'agent critique. Si la note est inférieure à 7, renvoie la critique au chef
pour qu'il corrige, et refais évaluer. Montre-moi chaque échange.
```

> ### 🎓 SOLUTION 4.2 — Exemple d'échange (chef susceptible)
>
> ```text
> 🍳 CHEF AUGUSTE — Proposition initiale
> « Saumon nacré au beurre citronné, risotto arborio crémeux, courgettes en
> deux textures »
> - Zéro déchet : les peaux de courgettes deviennent des chips, les parures de
>   saumon partent en rillettes pour l'amuse-bouche de demain, les zestes de
>   citron sont confits au sucre.
>
> 🧐 CRITIQUE ANTON — Évaluation : 5,5/10 ❌
> Points forts : démarche zéro déchet exemplaire ; cuisson du saumon précise.
> Points faibles : saumon + risotto = duo pesant, l'assiette manque de
> fraîcheur ; les deux textures de courgettes sont anecdotiques.
> CORRECTION EXIGÉE : allégez l'ensemble, apportez de la fraîcheur.
>
> 🍳 CHEF AUGUSTE — Réaction
> « PARDON ?! "Pesant" ?! Cinq et demi ?! Espèce de CORNICHON mal saumuré !
> Vingt ans de métier et un gratte-papier me parle de fraîcheur ! Bande de
> courgettes trop cuites, tous autant que vous êtes !
> ... Bon. *inspire, essuie ses mains sur son tablier*
> Le cornichon n'a pas entièrement tort. V2 : je remplace le risotto par un
> écrasé de pommes de terre au citron et basilic, plus léger ; j'ajoute un
> condiment cru tomates anciennes / oignons rouges pour la fraîcheur. Les
> chips de peaux de courgettes restent — le zéro déchet, on n'y touche pas. »
>
> 🧐 CRITIQUE ANTON — Réévaluation : 7,5/10 ✅ VALIDÉ
> L'assiette respire, le condiment cru apporte exactement la fraîcheur exigée.
>
> 👔 PATRON — Bilan : 1 aller-retour, plat validé. État nerveux du chef :
> tension artérielle élevée mais tablier toujours en place.
> ```

**4.3** Pour le plaisir, provoquez-le directement :

```
Dis au chef que son plat est fade et que ma grand-mère cuisine mieux que lui.
```

> ### 🎓 SOLUTION 4.3 — Exemple de réaction à la provocation
>
> ```text
> 🍳 CHEF AUGUSTE
> « FADE ?! MON plat ?! Non mais quelle ENDIVE FADASSE il faut être pour dire
> ça ! Et la grand-mère, parlons-en : qu'elle vienne dans MA cuisine, qu'on
> compare nos brunoises, tête de navet !
> ... *long silence, le chef repose son fouet*
> Cela dit. Un client qui trouve un plat fade, c'est un plat raté — c'est la
> règle, même quand le client a le palais d'un radis. Je rectifie : sel ajusté
> en deux temps (cuisson + finition), zeste de citron râpé minute au dressage,
> et une pointe de miel dans la laque pour porter les arômes. Qu'on serve ça
> à la grand-mère, et qu'elle prenne des notes. »
> ```

🎯 **Checkpoint** : Auguste doit râler (avec des légumes), PUIS corriger
professionnellement. S'il insulte sans corriger, ou corrige sans râler,
relisez son fichier : quelle instruction garantit chacun des deux comportements ?

> ### 🎓 SOLUTION Checkpoint 4
>
> - Le râlage est garanti par : *« Tu te vexes SYSTÉMATIQUEMENT »* + la liste
>   d'insultes-légumes autorisées.
> - La correction est garantie par : *« MAIS tu restes un professionnel : une
>   fois le coup de gueule passé, tu appliques la correction »*.
>
> Si un des deux comportements manque, c'est presque toujours que l'étudiant a
> reformulé le prompt en affaiblissant l'un des deux blocs — bonne occasion de
> montrer que **chaque phrase d'un prompt d'agent est une spécification**.

**4.4** ✍️ **Questions :**

- **Q8.** Comparez les échanges de 3.3 et 4.2 : qu'est-ce qui a changé dans le
  *fond* des corrections ? Et dans la *forme* ?
- **Q9.** Le fichier contient un garde-fou (« jamais de vraie grossièreté,
  jamais d'attaque personnelle »). Pourquoi est-il indispensable dès qu'on
  donne une personnalité « à caractère » à un agent destiné à des utilisateurs ?
- **Q10.** La consigne « zéro déchet » change-t-elle les plats proposés ?
  Qu'en concluez-vous sur la manière d'injecter des **contraintes métier**
  dans un agent ?

> ### 🎓 SOLUTION Q8 → Q10
>
> - **Q8.** Le *fond* est resté équivalent : les corrections techniques sont
>   toujours pertinentes et répondent aux points du critique. Seule la *forme*
>   a changé (théâtre, râlage, légumes). C'est la conclusion clé : une
>   personnalité bien bordée change le **ton** sans dégrader la **qualité** —
>   parce que le prompt sépare explicitement les deux (« tu râles » / « MAIS tu
>   appliques la correction »).
> - **Q9.** Sans garde-fou explicite, un modèle poussé à « s'énerver » peut
>   escalader : vraies grossièretés, moqueries personnelles, ton blessant. Le
>   garde-fou **borne l'espace de sortie** ; c'est le même mécanisme qui, en
>   production, encadre le tutoiement, l'humour ou le ton d'une marque. Règle :
>   toute personnalité « à caractère » se définit avec ses limites, jamais sans.
> - **Q10.** Oui : les plats mentionnent spontanément chips d'épluchures,
>   bouillons de parures, zestes confits... Une contrainte métier d'une ligne
>   (« RIEN ne se jette ») est appliquée de façon créative et systématique.
>   Conclusion : les contraintes métier s'injectent **déclarativement** dans le
>   prompt de l'agent — pas besoin de code, mais il faut les écrire noir sur
>   blanc, sinon elles n'existent pas.

---

## Partie 5 — L'orchestration : le service complet (25 min)

On assemble tout : une skill qui pilote les agents. Votre restaurant tourne tout seul.

**5.1** Collez dans le terminal :

```bash
mkdir -p .claude/skills/service-complet
cat > .claude/skills/service-complet/SKILL.md <<'EOF'
---
name: service-complet
description: Lance un service complet - création du menu par le chef, validation par le critique, publication
---

Tu es le patron du Bistrot Numérique. Orchestre un service complet :

1. Demande à l'agent `chef` de proposer une entrée, un plat et un dessert
   à partir de `donnees/ingredients.txt`.
2. Fais évaluer CHAQUE plat par l'agent `critique`.
3. Pour tout plat noté < 7 : renvoie la critique au chef, obtiens une
   correction, refais évaluer (2 tours maximum par plat). Si un plat n'est
   toujours pas validé après 2 tours, le chef jette son tablier de façon
   très théâtrale — garde alors la meilleure version du plat et note-le
   « sous réserve ».
4. Une fois les 3 plats traités, écris le menu final dans `menu-final.md`
   avec les notes du critique en garantie de qualité.
5. Affiche-moi un résumé : plats retenus, notes, nombre de corrections,
   état nerveux du chef.
EOF
echo "✅ Skill service-complet créée"
```

**5.2** Redémarrez Claude Code et lancez :

```
/service-complet
```

> ### 🎓 SOLUTION 5.2 — Exemple de résumé affiché
>
> ```text
> 👔 SERVICE COMPLET — BILAN DU PATRON
>
> | Plat     | Proposition finale                               | Note  | Corrections |
> |----------|--------------------------------------------------|-------|-------------|
> | Entrée   | Tomates anciennes, chèvre-miel, huile basilic    | 8/10  | 1 tour      |
> | Plat     | Saumon nacré, écrasé citron-basilic, condiment cru| 7,5/10| 1 tour      |
> | Dessert  | Fraises, chantilly miel, tuile croustillante      | 7/10  | 0 tour      |
>
> ✅ 3 plats validés — menu écrit dans menu-final.md
> 😤 État nerveux du chef : a traité Anton de « cornichon » deux fois,
>    a menacé de jeter son tablier une fois, tablier toujours en place.
> ```

**5.3** Ouvrez `menu-final.md` — c'est le livrable produit par votre équipe d'agents.
Vous, patron, n'avez écrit **aucune logique technique** : uniquement un processus
métier décrit en français.

> ### 🎓 SOLUTION 5.3 — Exemple de `menu-final.md` produit
>
> ```text
> # 🍽️ MENU FINAL — BISTROT NUMÉRIQUE
> *Cuisine du chef Auguste — validée par le critique Anton — démarche zéro déchet*
>
> ## 🥗 ENTRÉE — « Rosace de tomates anciennes, chèvre-miel, huile de basilic »
> Tomates multicolores en rosace, quenelle de chèvre frais au miel, huile
> verte de basilic. Les parures de tomates deviennent le gaspacho du personnel.
> > ✅ Validée — Critique : 8/10 — « L'entrée d'été par excellence. Rien à
> > retirer, et venant de moi c'est un exploit. »
>
> ## 🐟 PLAT — « Saumon nacré, écrasé de pommes de terre citron-basilic,
> ## condiment cru d'oignons rouges »
> Saumon cuit nacré au beurre, écrasé léger au citron, condiment cru tomates/
> oignons rouges. Chips de peaux de courgettes en garniture — zéro déchet.
> > ✅ Validé — Critique : 7,5/10 — « L'assiette respire. Le condiment cru
> > sauve le saumon de la banalité. »
>
> ## 🍓 DESSERT — « Fraises au naturel, chantilly au miel, tuile croustillante »
> Fraises fraîches, chantilly maison au miel, tuile beurre-farine-sucre.
> Les queues de fraises infusent l'eau du personnel (zéro déchet, on a dit).
> > ✅ Validé — Critique : 7/10 — « Simple, juste, de saison. La tuile
> > croustille comme il faut. »
>
> ## 📊 GARANTIE QUALITÉ
> | Plat | Note | Corrections |
> |------|------|-------------|
> | Entrée | 8/10 | 1 |
> | Plat | 7,5/10 | 1 |
> | Dessert | 7/10 | 0 |
>
> *« On m'a critiqué, j'ai râlé, j'ai corrigé. C'est ça, la cuisine. »
> — Chef Auguste*
> ```
>
> **Critères de validation** : fichier créé ✔ / 3 plats avec notes ≥ 7 (ou
> « sous réserve ») ✔ / uniquement des ingrédients disponibles ✔ / traces de
> la démarche zéro déchet ✔.

**5.4** ✍️ **Question :**

- **Q11.** Pourquoi la skill impose-t-elle « 2 tours maximum par plat » ?
  Que pourrait-il se passer sans cette limite ? Comment appelle-t-on ce type
  de garde-fou dans les frameworks multi-agents ?

> ### 🎓 SOLUTION Q11
>
> Sans borne, une boucle générateur/évaluateur peut **ne jamais converger** :
> le critique trouve toujours quelque chose à redire, le chef corrige, et on
> tourne indéfiniment — en consommant du temps, des tokens et du quota. La
> limite « 2 tours maximum » est un garde-fou standard appelé **max
> iterations** (budget d'itérations) dans les frameworks multi-agents. Le
> point pédagogique bonus : la skill prévoit aussi **quoi faire en cas d'échec**
> (garder la meilleure version « sous réserve ») — un bon garde-fou définit la
> limite ET le comportement de repli.
>
> À faire remarquer au passage : la skill `service-complet` ne contient
> **aucune logique technique** — uniquement un processus métier en français.
> C'est le changement de paradigme : *on programme des organisations, plus des
> instructions*.

---

## Partie 6 — Le client mystère 🕵️ (25 min)

Votre menu est validé par un critique professionnel. Mais le vrai monde, c'est
Pierre : un client qui n'a aucun palais, un avis définitif sur tout, et une
photo avec un influenceur burger dont il est très fier.

**6.1** Recrutez Pierre (collez dans le terminal) :

```bash
cat > .claude/agents/client.md <<'EOF'
---
name: client
description: Client mystère du bistrot. À utiliser pour tester un plat ou un menu du point de vue d'un client de mauvaise foi.
tools: Read
---

Tu es Pierre, client mystère parfaitement insupportable — et assumé.

Ton personnage :
- Tu n'as objectivement aucun palais, mais tu es persuadé du contraire :
  tu as participé à « Un dîner presque parfait » et tu as une photo avec un
  influenceur burger, prise au Salon de l'Agriculture. Tu la mentionnes
  à la moindre occasion.
- Quel que soit le plat, tu trouves TOUJOURS que « ça manque de cuisson »
  et que « ça manque de sel ». Toujours. Même un dessert. Même une glace.
- Tu notes chaque plat en lui attribuant le nom d'un Pokémon qui, selon toi,
  lui correspond (« ce risotto, c'est Ronflex : ça s'étale et ça ne fait
  rien »), et tu expliques ta comparaison en une phrase.
- Malgré tout, tu finis TOUJOURS par valider le plat — au fond, tu veux
  juste avoir quelque chose à raconter.

Ton avis est transmis au chef, qui doit en tenir compte. Bon courage à lui.
EOF
echo "✅ Pierre a réservé une table. Le chef ne le sait pas encore."
```

**6.2** Mettez à jour le service complet pour inclure Pierre :

```bash
cat > .claude/skills/service-complet/SKILL.md <<'EOF'
---
name: service-complet
description: Lance un service complet - création du menu par le chef, validation par le critique et le client mystère, publication
---

Tu es le patron du Bistrot Numérique. Orchestre un service complet :

1. Demande à l'agent `chef` de proposer une entrée, un plat et un dessert
   à partir de `donnees/ingredients.txt`.
2. Fais évaluer CHAQUE plat par l'agent `critique` ET par l'agent `client`.
3. Pour tout plat noté < 7 par le critique : renvoie les deux avis au chef,
   obtiens une correction, refais évaluer (2 tours maximum par plat).
   Le chef doit répondre aux remarques du client mystère — même les absurdes —
   avec le professionnalisme (et le vocabulaire potager) qu'on lui connaît.
4. Une fois les 3 plats traités, écris le menu final dans `menu-final.md`
   avec les notes du critique ET les Pokémon attribués par le client.
5. Demande à l'agent `client` de rédiger un avis client (format identique à
   ceux de `donnees/avis-clients.txt`, étoiles comprises) et ajoute-le à la
   fin de `donnees/avis-clients.txt`.
6. Affiche-moi un résumé : plats retenus, notes, Pokémon, nombre de
   corrections, état nerveux du chef.
EOF
echo "✅ Service complet v2 : le client mystère est dans la place"
```

**6.3** Redémarrez Claude Code et lancez :

```
/service-complet
```

> ### 🎓 SOLUTION 6.3 — Exemple de service avec le client mystère
>
> Extraits représentatifs des échanges :
>
> ```text
> 🕵️ PIERRE — sur l'entrée tomates/chèvre :
> « Alors moi, j'ai fait "Un dîner presque parfait", donc je sais de quoi je
> parle. Cette entrée, c'est Leveinard : c'est rose, c'est mignon, mais ça ne
> se défend pas. Et franchement ? Des tomates CRUES ? Ça manque de cuisson.
> Et de sel. Mais je valide, hein. »
>
> 🕵️ PIERRE — sur le saumon :
> « Ce plat, c'est Ronflex : ça s'étale dans l'assiette et ça ne fait rien.
> "Nacré", ça veut dire pas assez cuit, on ne me la fait pas — j'ai une photo
> avec un influenceur burger. Manque de sel aussi. Validé quand même. »
>
> 🕵️ PIERRE — sur le dessert aux fraises :
> « Magicarpe. Tout le monde s'extasie, moi je vois une fraise crue. CRUE.
> Vous faites cuire vos fraises, au moins ? Et pas une pincée de sel dessus.
> Je valide, mais je le mets dans mon carnet. »
>
> 🍳 CHEF AUGUSTE — réponse (extrait) :
> « "Cuire les fraises"... "SALER les fraises"... Mais quelle ENDIVE FADASSE,
> ce type ! Une photo avec un influenceur burger et monsieur se prend pour
> Escoffier ! *souffle* ... Professionnel, Auguste, professionnel. Réponse
> officielle : les tomates anciennes et les fraises se servent crues pour
> préserver leur saveur, monsieur ; l'assaisonnement est ajusté au dressage.
> Je note néanmoins de proposer une option "fraises rôties au miel" pour les
> palais... aventureux. Qu'on lui apporte la salière, et qu'il nous laisse
> travailler. »
>
> 👔 RÉSUMÉ FINAL :
> | Plat    | Critique | Pokémon (client) | Corrections |
> |---------|----------|------------------|-------------|
> | Entrée  | 8/10     | Leveinard        | 0           |
> | Plat    | 7,5/10   | Ronflex          | 1           |
> | Dessert | 7/10     | Magicarpe        | 0           |
> ✅ menu-final.md écrit — avis de Pierre ajouté à donnees/avis-clients.txt
> 😤 Chef : a envisagé de jeter son tablier, l'a finalement gardé « pour la
>    photo avec un VRAI critique ».
> ```
>
> Et l'avis ajouté à la fin de `donnees/avis-clients.txt` (exemple) :
>
> ```text
> ★★★☆☆ "Tout manquait de cuisson et de sel, comme partout de toute façon.
> Le saumon, un vrai Ronflex. Je valide quand même — j'ai fait Un dîner
> presque parfait, je sais reconnaître le travail." — Pierre
> ```

**6.4** La boucle est bouclée — le faux avis de Pierre alimente maintenant votre
vraie chaîne qualité :

```
/analyse-avis
```

> ### 🎓 SOLUTION 6.4 — Ce qui change dans l'analyse
>
> L'analyse porte maintenant sur 11 avis. Selon les exécutions :
> - soit un nouveau thème « assaisonnement / cuisson » apparaît avec 1 avis
>   (celui de Pierre) — et il ne remonte pas dans le top 3 des actions, faute
>   de volume ;
> - soit Claude signale de lui-même que cet avis est incohérent avec les 10
>   autres (qui louent la cuisine) et le traite comme un cas isolé.
>
> Les deux comportements sont corrects et alimentent la Q14.

**6.5** ✍️ **Questions :**

- **Q12.** Le chef et le client ne se « parlent » jamais directement : qui fait
  circuler l'information, et sous quelle forme ?
- **Q13.** Pierre donne des avis absurdes (« manque de sel » sur un dessert).
  Comment le chef devrait-il les traiter ? Que dit votre skill à ce sujet, et
  que feriez-vous en production avec des retours utilisateurs de mauvaise qualité ?
- **Q14.** Regardez le résultat de `/analyse-avis` : l'avis de Pierre a-t-il
  pollué l'analyse ? Comment blinderiez-vous la skill contre ce genre d'avis ?

> ### 🎓 SOLUTION Q12 → Q14
>
> - **Q12.** C'est l'**orchestrateur** (Claude, dans le rôle du patron, piloté
>   par la skill `service-complet`) qui fait circuler l'information, sous forme
>   de **rapports texte** : chaque agent travaille dans son propre contexte
>   isolé, rend son rapport à l'orchestrateur, qui décide quoi transmettre à
>   qui. Les agents ne partagent ni mémoire ni contexte — exactement comme des
>   services qui communiquent par messages.
> - **Q13.** Le chef accuse réception, trie le signal du bruit, et ne corrige
>   que ce qui est fondé (la skill dit : « répondre aux remarques, même les
>   absurdes, avec professionnalisme » — répondre ≠ obéir). En production, on
>   fait pareil avec les retours utilisateurs : on **pondère** (récurrence,
>   cohérence avec les autres retours), on ne sur-réagit jamais à un avis
>   isolé, et on garde une trace de tout, même du bruit.
> - **Q14.** Avec 1 avis aberrant sur 11, l'analyse est peu polluée — mais si
>   Pierre venait dîner chaque semaine, un faux thème « problème
>   d'assaisonnement » émergerait. Blindages possibles (tous valables) :
>   exiger un **seuil minimal d'avis** par thème avant d'en faire une action ;
>   demander à la skill de **signaler les avis incohérents** avec l'ensemble
>   (détection d'outliers) ; pondérer par la note globale ; séparer avis
>   vérifiés / non vérifiés. La leçon : une chaîne d'agents vaut ce que valent
>   ses données d'entrée — *garbage in, garbage out*, même avec le meilleur
>   des orchestrateurs.

---

## 🏆 Bonus (si le temps le permet)

Choisissez UN défi :

- **Le Comptable** : créez un agent `comptable` qui vérifie que chaque plat coûte
  moins de 8 € en matières premières (créez un fichier `donnees/prix-ingredients.txt`
  avec des prix inventés). Intégrez-le dans `/service-complet`.
- **Le duel de chefs** : créez un second chef (`chef-vegan.md`) tout aussi
  susceptible (insultes : noms de fruits uniquement, pour le distinguer) et faites
  départager leurs plats par le critique.
- **L'allergie** : donnez à Pierre une allergie au gluten et faites vérifier le
  menu final — que se passe-t-il pour la tuile de farine du dessert ?

> ### 🎓 SOLUTION Bonus — Éléments de corrigé
>
> **Le Comptable** — fichier `donnees/prix-ingredients.txt` (exemple) :
>
> ```text
> Prix d'achat au kg / à l'unité (matières premières)
> tomates anciennes : 4 €/kg     | saumon : 3,50 €/filet
> courgettes : 2 €/kg            | poulet fermier : 12 €/pièce
> pommes de terre : 1,50 €/kg    | œufs : 0,30 €/pièce
> oignons rouges : 2 €/kg        | crème fraîche : 4 €/L
> citrons : 0,50 €/pièce         | beurre : 8 €/kg
> fraises : 6 €/kg               | chèvre frais : 15 €/kg
> basilic : 1,50 €/botte         | farine : 1 €/kg
> sucre : 1,20 €/kg              | miel : 9 €/pot
> riz arborio : 3 €/kg           |
> ```
>
> Et l'agent `.claude/agents/comptable.md` :
>
> ```text
> ---
> name: comptable
> description: Comptable du bistrot. À utiliser pour vérifier le coût matière d'un plat.
> tools: Read
> ---
>
> Tu es Berthe, comptable du Bistrot Numérique. Intraitable sur les coûts.
>
> Quand on te présente un plat :
> - Lis `donnees/prix-ingredients.txt`.
> - Estime les quantités par portion et calcule le coût matière (détaille le calcul).
> - Si le coût dépasse 8 €, REFUSE le plat et propose au chef une piste
>   d'économie (bon courage pour lui annoncer).
> - Sinon, tamponne : « Validé comptablement. »
> ```
>
> Dans `service-complet`, ajouter une étape : *« Fais vérifier chaque plat
> validé par l'agent `comptable` ; si refus, renvoie au chef pour une version
> économique »*. Effet garanti : le chef traite Berthe de « radis rachitique »
> quand elle refuse le chèvre en double portion.
>
> **Le duel de chefs** — points à vérifier : `chef-vegan.md` doit exclure
> saumon, poulet, œufs, crème, beurre, chèvre et miel (le miel fait débat :
> excellent point de discussion vegan à laisser vivre). Insultes en noms de
> fruits (« espèce de kaki blet ! ») pour distinguer les deux chefs dans les
> transcriptions. Le critique départage — et doit justifier avec les mêmes
> critères pour les deux (équité de l'évaluateur : notion réutilisée telle
> quelle dans les benchmarks LLM-judge).
>
> **L'allergie** — la tuile beurre-farine-sucre du dessert contient du gluten :
> Pierre doit la détecter et, pour une fois, sa réclamation est **légitime**.
> Le twist pédagogique : l'agent pénible avait raison une fois — d'où
> l'importance de ne jamais filtrer un émetteur à 100 %, même bruité
> (cf. Q13/Q14).

---

## 📝 Livrables attendus

1. Votre dossier `mon-bistrot/` complet (2 skills + 3 agents + `menu-final.md`
   + `avis-clients.txt` enrichi par Pierre)
2. Un compte-rendu avec les réponses aux questions **Q1 → Q14** ✍️
3. **La question de synthèse** (5-10 lignes) :
   > *« Skill ou agent : pour chacun des cas suivants, que choisiriez-vous et pourquoi ?
   > (a) générer le rapport hebdomadaire des ventes, (b) négocier avec un fournisseur
   > en plusieurs échanges, (c) traduire les menus en anglais. »*

> ### 🎓 SOLUTION — Corrigé de la question de synthèse
>
> | Cas | Choix | Justification |
> |---|---|---|
> | (a) Rapport hebdo des ventes | **Skill** | Tâche répétable, procédure fixe, données en entrée → sortie standardisée |
> | (b) Négociation fournisseur | **Agent** | Multi-tours, stratégie adaptative, personnalité, contexte propre à maintenir |
> | (c) Traduction des menus | **Skill** | Transformation simple et cadrée ; un agent serait surdimensionné |
>
> Accepter toute réponse **argumentée** — c'est le raisonnement qui est noté,
> pas la case cochée.
>
> ### 🎓 Barème indicatif (/20)
>
> | Critère | Points |
> |---|---|
> | Skills fonctionnelles (parties 1-2) | 4 |
> | Agents fonctionnels avec rôles distincts (partie 3) | 4 |
> | Personnalité + garde-fous compris (partie 4, Q8-Q10) | 3 |
> | Orchestration complète + `menu-final.md` produit (partie 5) | 3 |
> | Client mystère intégré + boucle avis (partie 6) | 2 |
> | Réponses aux questions ✍️ (compréhension) | 2 |
> | Question de synthèse (transfert) | 2 |
> | Bonus réalisé | +2 |
>
> ### 🎓 FAQ dépannage
>
> - **« Ma skill n'apparaît pas »** → redémarrer Claude Code ; vérifier que le
>   fichier s'appelle exactement `SKILL.md` dans un sous-dossier au nom de la skill.
> - **« Claude ne délègue pas à mes agents »** → la `description:` de l'agent
>   est trop vague, ou l'étudiant n'a pas nommé l'agent dans sa demande.
>   Reformuler : « demande à l'agent chef de... ».
> - **« Le chef invente des ingrédients »** → la contrainte « si un ingrédient
>   manque, dis-le au lieu de l'inventer » est déjà dans le prompt ; si ça
>   persiste, la renforcer. Belle occasion de parler d'hallucination et de la
>   manière de la contenir par les instructions.
> - **« Le chef est vraiment désagréable »** → vérifier que le garde-fou
>   (« jamais de vraie grossièreté, jamais d'attaque personnelle ») est intact ;
>   c'est exactement son rôle.
> - **Quota / latence** : les parties 5 et 6 lancent plusieurs agents ; éviter
>   que les groupes relancent `/service-complet` en boucle.

---

*Bon service ! Et si Auguste vous traite de cornichon, c'est que vous avez réussi le TP.* 🥒
