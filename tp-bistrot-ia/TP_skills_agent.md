# TP — Le Bistrot Numérique 🍽️
### Construisez votre brigade IA : skills, agents et orchestration avec Claude Code

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

**1.3** ✍️ **Questions (à noter dans votre compte-rendu) :**

- **Q1.** Relancez `/menu-du-jour` une deuxième fois. Le menu est-il identique ? Pourquoi ?
- **Q2.** Ouvrez `donnees/ingredients.txt`, supprimez la ligne du saumon, sauvegardez,
  relancez `/menu-du-jour` (sans redémarrer Claude). Que constatez-vous ?
  Qu'est-ce que cela vous apprend sur la nature d'une skill ?
- **Q3.** À quoi sert la ligne `description:` dans l'en-tête ?
  *(Indice : demandez à Claude « quelles skills as-tu à ta disposition ? »)*

*(Remettez le saumon dans le fichier avant de continuer, le chef en aura besoin.)*

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

**2.3** ✍️ **Question :**

- **Q4.** En entreprise, citez deux exemples de skills de ce type qui feraient
  gagner du temps dans VOTRE domaine (compta, RH, support, dev...).

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

**4.3** Pour le plaisir, provoquez-le directement :

```
Dis au chef que son plat est fade et que ma grand-mère cuisine mieux que lui.
```

🎯 **Checkpoint** : Auguste doit râler (avec des légumes), PUIS corriger
professionnellement. S'il insulte sans corriger, ou corrige sans râler,
relisez son fichier : quelle instruction garantit chacun des deux comportements ?

**4.4** ✍️ **Questions :**

- **Q8.** Comparez les échanges de 3.3 et 4.2 : qu'est-ce qui a changé dans le
  *fond* des corrections ? Et dans la *forme* ?
- **Q9.** Le fichier contient un garde-fou (« jamais de vraie grossièreté,
  jamais d'attaque personnelle »). Pourquoi est-il indispensable dès qu'on
  donne une personnalité « à caractère » à un agent destiné à des utilisateurs ?
- **Q10.** La consigne « zéro déchet » change-t-elle les plats proposés ?
  Qu'en concluez-vous sur la manière d'injecter des **contraintes métier**
  dans un agent ?

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

**5.3** Ouvrez `menu-final.md` — c'est le livrable produit par votre équipe d'agents.
Vous, patron, n'avez écrit **aucune logique technique** : uniquement un processus
métier décrit en français.

**5.4** ✍️ **Question :**

- **Q11.** Pourquoi la skill impose-t-elle « 2 tours maximum par plat » ?
  Que pourrait-il se passer sans cette limite ? Comment appelle-t-on ce type
  de garde-fou dans les frameworks multi-agents ?

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

**6.4** La boucle est bouclée — le faux avis de Pierre alimente maintenant votre
vraie chaîne qualité :

```
/analyse-avis
```

**6.5** ✍️ **Questions :**

- **Q12.** Le chef et le client ne se « parlent » jamais directement : qui fait
  circuler l'information, et sous quelle forme ?
- **Q13.** Pierre donne des avis absurdes (« manque de sel » sur un dessert).
  Comment le chef devrait-il les traiter ? Que dit votre skill à ce sujet, et
  que feriez-vous en production avec des retours utilisateurs de mauvaise qualité ?
- **Q14.** Regardez le résultat de `/analyse-avis` : l'avis de Pierre a-t-il
  pollué l'analyse ? Comment blinderiez-vous la skill contre ce genre d'avis ?

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

---

## 📝 Livrables attendus

1. Votre dossier `mon-bistrot/` complet (2 skills + 3 agents + `menu-final.md`
   + `avis-clients.txt` enrichi par Pierre)
2. Un compte-rendu avec les réponses aux questions **Q1 → Q14** ✍️
3. **La question de synthèse** (5-10 lignes) :
   > *« Skill ou agent : pour chacun des cas suivants, que choisiriez-vous et pourquoi ?
   > (a) générer le rapport hebdomadaire des ventes, (b) négocier avec un fournisseur
   > en plusieurs échanges, (c) traduire les menus en anglais. »*

---

*Bon service ! Et si Auguste vous traite de cornichon, c'est que vous avez réussi le TP.* 🥒
