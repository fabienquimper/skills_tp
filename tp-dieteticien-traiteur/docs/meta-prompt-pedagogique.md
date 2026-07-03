# Le harness « Diététicien & Traiteur » expliqué — un cours d'IA agentique

> **À qui s'adresse ce document ?** À des étudiant·e·s en intelligence artificielle qui veulent
> comprendre, à travers un projet *réel et complet*, ce qu'est un **agent**, comment on lui donne
> des **compétences (skills)**, comment on règle son **autonomie**, et ce que tout cela annonce pour
> le futur — y compris pour un·e professionnel·le « non technique » (ici un diététicien-traiteur)
> qui voudrait l'utiliser chez lui.
>
> **Double usage.** Ce fichier est *pédagogique* (il enseigne) **et** c'est un *méta-prompt* : on
> peut le donner tel quel à un agent IA pour qu'il comprenne l'esprit, l'architecture et les
> garde-fous du projet avant d'y travailler. Voir l'annexe finale.

---

## Sommaire

1. [Le projet en une page](#1-le-projet-en-une-page)
2. [Concept clé : qu'est-ce qu'un agent ?](#2-concept-clé--quest-ce-quun-agent-)
3. [Outils vs Skills : la distinction fondamentale](#3-outils-vs-skills--la-distinction-fondamentale)
4. [Anatomie d'un skill (lecture commentée)](#4-anatomie-dun-skill-lecture-commentée)
5. [Le principe d'architecture : « skills fins, données partagées »](#5-le-principe-darchitecture--skills-fins-données-partagées)
6. [Déterminisme vs stochastique : pourquoi un script Python ?](#6-déterminisme-vs-stochastique--pourquoi-un-script-python-)
7. [Le contexte et le « grounding » : l'histoire d'un bug instructif](#7-le-contexte-et-le-grounding--lhistoire-dun-bug-instructif)
8. [État, mémoire et idempotence : des fichiers comme cerveau](#8-état-mémoire-et-idempotence--des-fichiers-comme-cerveau)
9. [Composition : chaîner des skills en pipeline](#9-composition--chaîner-des-skills-en-pipeline)
10. [Autonomie et human-in-the-loop](#10-autonomie-et-human-in-the-loop)
11. [Garde-fous, sécurité, responsabilité](#11-garde-fous-sécurité-responsabilité)
12. [Étude de cas : dérouler une requête réelle](#12-étude-de-cas--dérouler-une-requête-réelle)
13. [Le futur : ce que ce petit projet laisse entrevoir](#13-le-futur--ce-que-ce-petit-projet-laisse-entrevoir)
14. [Connecter « ça » à un humain non technique](#14-connecter-ça-à-un-humain-non-technique)
15. [Travaux pratiques](#15-travaux-pratiques)
16. [Glossaire](#16-glossaire)
17. [Annexe : utiliser ce fichier comme méta-prompt](#17-annexe--utiliser-ce-fichier-comme-méta-prompt)

---

## 1. Le projet en une page

On a construit un **harness** (un « harnais » : l'ensemble outillé qui équipe un agent pour un
métier) destiné à un **diététicien-traiteur**. Concrètement, c'est un dossier de fichiers texte que
l'agent IA lit et exécute. Il sait :

- **concevoir une recette** adaptée à un public (saine, diabète, surpoids, anorexie/renutrition,
  gourmand, petit budget) ;
- **détailler les ingrédients** et les mettre à l'échelle pour N convives ;
- **calculer les apports nutritionnels** (kcal, macros, vitamines, minéraux) ;
- **déclarer les allergènes** (les 14 catégories réglementaires européennes, dites « INCO ») ;
- **agréger une liste de courses** pour une prestation traiteur ;
- **générer un classeur Excel** récapitulatif (4 onglets) ;
- **planifier une semaine** de repas (lundi→dimanche, matin/midi/soir).

Tout est en texte lisible :

```
dieteticien/
├── CLAUDE.md                 # « notice » chargée automatiquement par l'agent à chaque session
├── README.md
├── .claude/skills/<nom>/SKILL.md   # 8 compétences (les « modes d'emploi » de l'agent)
│   └── fiche-commande-excel/generer.py   # + 1 script déterministe
├── recettes/                 # 11 recettes (.md) + _format.md (le gabarit) + index
├── profils/                  # 6 profils diététiques (les « contraintes métier »)
├── data/                     # référentiels partagés (nutrition, allergènes, conversions, rayons)
└── commandes/                # sorties générées (plans de semaine, fichiers .xlsx)
```

**L'idée pédagogique centrale :** on n'a quasiment pas écrit de « programme ». On a écrit des
**instructions, des données et des garde-fous** en langage naturel structuré, et un modèle de
langage (Claude) joue le rôle de « processeur ». C'est tout le changement de paradigme de l'IA
agentique : *le logiciel devient en partie du langage*.

---

## 2. Concept clé : qu'est-ce qu'un agent ?

Un **modèle de langage** seul ne fait que prédire du texte. Un **agent**, c'est un modèle placé
dans une **boucle** où il peut **agir sur le monde** et **observer le résultat** :

```
        ┌────────────────────────────────────────────┐
        │                                            │
   [Perception] ──► [Raisonnement] ──► [Action via un outil]
   (contexte,          (décider               (lire un fichier,
    résultats             quoi faire)           lancer un script,
    d'outils)                                   écrire un .xlsx)
        ▲                                            │
        └──────────────── [Observation] ◄───────────┘
                          (sortie de l'outil)
```

Trois ingrédients font l'agent :
1. **Un modèle** capable de raisonner et de décider (ici Claude).
2. **Des outils** (tools) : des fonctions qu'il peut appeler (lire/écrire un fichier, exécuter du
   shell, chercher sur le web…).
3. **Une boucle** orchestrée par un *harness* logiciel (ici Claude Code) qui exécute les outils et
   renvoie les résultats au modèle, jusqu'à ce que la tâche soit finie.

> 🧠 **À retenir.** L'« intelligence » d'un agent ne vient pas que du modèle : elle vient surtout de
> **ce qu'on met dans son contexte** et **des outils qu'on lui donne**. Notre projet est presque
> entièrement une affaire de *context engineering* : bien écrire les instructions et les données.

---

## 3. Outils vs Skills : la distinction fondamentale

C'est la confusion la plus courante chez les débutants. Distinguons :

| | **Outil (tool)** | **Skill (compétence)** |
|---|---|---|
| Nature | Une *capacité primitive* (lire, écrire, exécuter) | Une *procédure de haut niveau*, un mode d'emploi |
| Qui l'écrit | Le fournisseur du harness | **Toi**, l'ingénieur du domaine |
| Exemple | `Read`, `Write`, `Bash` | « créer une recette diabétique », « générer le classeur Excel » |
| Analogie | Les muscles et les mains | Une *recette de cuisine* qui dit quoi faire avec ses mains |
| Format ici | Fourni par Claude Code | Un fichier `SKILL.md` (Markdown + en-tête) |

Un **skill** est donc un **playbook** : un texte qui dit à l'agent *quand* il est pertinent, *quels
fichiers lire*, *quelles étapes suivre*, *quels outils utiliser* et *quels garde-fous respecter*. Il
ne contient (presque) pas de code : il **oriente le raisonnement** du modèle.

C'est une bascule conceptuelle : **programmer devient « écrire des procédures en langage naturel »**,
à charge pour le modèle de les interpréter intelligemment selon le contexte.

---

## 4. Anatomie d'un skill (lecture commentée)

Prenons `creer-recette`. Un `SKILL.md` a deux parties : un **en-tête** (frontmatter) et un **corps**.

```yaml
---
name: creer-recette
description: Crée une recette adaptée à un ou plusieurs profils diététiques (saine, diabétique,
  anorexie, gourmand, petit budget) et à des contraintes… Utiliser quand l'utilisateur veut
  concevoir/inventer une recette…
user-invocable: true
allowed-tools: [Read, Glob, Grep, Write]
---
```

Décortiquons chaque champ — chacun illustre un concept d'agentique :

- **`name`** : l'identifiant. C'est la commande `/creer-recette`.
- **`description`** : 🔑 *le plus important*. Ce texte sert au **routage** : parmi des dizaines de
  skills, l'agent lit les descriptions pour décider *lequel* déclencher face à une demande. Une bonne
  description dit **ce que fait** le skill ET **quand l'utiliser**. C'est de la *discoverability* :
  un skill que l'agent ne sait pas reconnaître n'existe pas.
- **`allowed-tools`** : la liste des outils que ce skill a le droit d'utiliser. Ici, lecture +
  écriture de fichiers. C'est le **principe de moindre privilège** (least privilege) : un skill qui
  n'a besoin que de lire ne reçoit pas le droit d'écrire ou de lancer des commandes. Sécurité de base.
- **`user-invocable`** : l'humain peut le déclencher explicitement.

Le **corps** (en Markdown) est la procédure : étapes numérotées, fichiers à lire, règles de
décision (« si plusieurs profils, le profil médical est prioritaire »), format de sortie attendu, et
**encarts d'avertissement** à recopier. C'est, littéralement, *le programme — écrit en français*.

> 💡 **Exercice mental.** Relis la `description` d'un skill comme si tu étais l'agent face à 100
> skills. Saurais-tu, à la seule lecture, quand l'utiliser ? Si non, la description est à réécrire.
> *C'est 80 % du travail d'un bon skill.*

---

## 5. Le principe d'architecture : « skills fins, données partagées »

Regarde où vit l'information :

- Les **valeurs nutritionnelles** sont dans `data/nutrition-100g.csv` (une seule fois).
- Les **allergènes** dans `data/allergenes.csv`.
- Les **profils** (contraintes diététiques) dans `profils/*.md`.
- Les **skills** ne *recopient* pas ces données : ils disent « lis `data/nutrition-100g.csv` ».

C'est la séparation **politique / données** (ou *code / configuration*) :

```
   ┌──────────── SKILLS (la logique, le « comment ») ────────────┐
   │ creer-recette · apports · allergenes · liste-course · …     │
   └───────────────────────────┬────────────────────────────────┘
                               │ lisent
   ┌───────────────────────────▼────────────────────────────────┐
   │           DONNÉES PARTAGÉES (le « savoir », le quoi)         │
   │  nutrition-100g.csv · allergenes.csv · conversions · rayons │
   │  profils/*.md · recettes/*.md                                │
   └─────────────────────────────────────────────────────────────┘
```

**Pourquoi c'est puissant ?** Ajoute *un seul* ingrédient à `nutrition-100g.csv` (avec sa ligne dans
`allergenes.csv` et `rayons.csv`) et **les 8 skills en profitent instantanément** — calcul d'apports,
liste de courses, classeur Excel, tout suit. Aucune logique à modifier. C'est le principe DRY
(*Don't Repeat Yourself*) appliqué à un agent : *une donnée, un endroit, plusieurs usages*.

C'est aussi ce qui rend le système **maintenable par un non-développeur** : enrichir le « cerveau »
de l'agent = éditer un tableur, pas réécrire un programme.

---

## 6. Déterminisme vs stochastique : pourquoi un script Python ?

Un modèle de langage est **stochastique** : excellent pour raisonner et rédiger, mais il peut se
tromper dans une longue addition. Or, pour un traiteur, un **total de liste de courses** ou un
**coût** doivent être **exacts et reproductibles**.

Décision d'ingénierie clé du projet : le classeur Excel n'est **pas** calculé « de tête » par le
modèle. Le skill `fiche-commande-excel` lance un **script Python déterministe** (`generer.py`) qui
lit les CSV, fait les multiplications, fusionne les ingrédients et écrit le `.xlsx`.

```
  Tâche « floue » (comprendre la demande, choisir les plats)  ──►  confiée au MODÈLE (souple)
  Tâche « exacte » (additionner 1 100 g d'oignon, 538 kcal…)  ──►  confiée au CODE  (fiable)
```

C'est un patron récurrent en agentique : **laisser le LLM faire ce qu'il fait de mieux (juger,
interpréter, rédiger) et déléguer le calcul exact à du code**. On gagne en *fiabilité*, en
*reproductibilité* (mêmes entrées → mêmes sorties) et en *vérifiabilité* (on peut tester `generer.py`).

> 🧪 **Preuve par la vérification.** On a recoupé un nombre connu : le curry de pois chiches donne
> **538 kcal/portion**, identique à chaque exécution. Un calcul « à la main » par le modèle aurait pu
> varier d'une fois sur l'autre. *Un agent fiable sait quand ne pas faire confiance à son intuition.*

Note avancée : pour rester *grounded* (ancré dans des faits), le script lit des **miroirs CSV**
(`conversions.csv`, `rayons.csv`) des documents humains (`.md`). C'est une mini-illustration de
l'idée de **RAG** (*Retrieval-Augmented Generation*) : on ne demande pas au modèle de *savoir par
cœur* les valeurs, on les lui *fournit* depuis une source de vérité.

---

## 7. Le contexte et le « grounding » : l'histoire d'un bug instructif

Pendant le projet, un vrai incident a eu lieu (très formateur). Un skill disait « lis
`recettes/...` ». L'agent a cru que ce dossier était **à l'intérieur du dossier du skill**
(`.claude/skills/<nom>/`), ne l'a pas trouvé, et a perdu plusieurs tours à chercher.

**Diagnostic :** un problème de **grounding / context**. Le chemin était *relatif à la racine du
projet*, mais cette convention n'était écrite nulle part. L'agent a comblé le vide… par une
mauvaise hypothèse. (Les modèles *hallucinent* surtout quand le contexte est ambigu.)

**Correctif (deux couches) :**
1. Un fichier **`CLAUDE.md`** à la racine, **chargé automatiquement dans le contexte de chaque
   session**, qui décrit la hiérarchie et pose la règle : *« ces chemins sont relatifs à la racine,
   pas au dossier du skill »*.
2. Un **rappel court en tête de chaque skill**, au cas où le `CLAUDE.md` ne suffirait pas.

Leçons d'agentique, valables bien au-delà de ce projet :
- **Un agent ne « voit » que son contexte.** Ce qui n'y est pas, n'existe pas pour lui.
- **L'ambiguïté est l'ennemie.** Rendre explicites les conventions implicites évite les hypothèses
  hasardeuses.
- **La redondance utile est une vertu** : répéter l'information critique à plusieurs endroits
  (contexte global + local) la rend robuste.
- On *débogue un agent* surtout en **éditant son contexte**, pas en réécrivant du code.

---

## 8. État, mémoire et idempotence : des fichiers comme cerveau

Un agent, par défaut, n'a **pas de mémoire** entre deux sessions. Ici, la mémoire est **externalisée
dans des fichiers** :

- `recettes/` = la **mémoire à long terme** des savoir-faire (s'enrichit avec le temps).
- `recettes/README.md` = un **index** maintenu à jour à chaque ajout (pour retrouver vite).
- `commandes/` = la **trace** des décisions et sorties (les plans, les classeurs).

Le skill `base-recettes` est le **gardien de cet état** : c'est le *seul* autorisé à écrire dans
`recettes/`, et il **valide le format** avant d'écrire, puis **met à jour l'index**. On retrouve des
notions de génie logiciel :
- **Source unique de vérité** (l'index reflète toujours le contenu réel).
- **Invariants** (toute recette respecte le gabarit `_format.md`).
- **Idempotence / cohérence** : ré-exécuter ne casse pas l'état ; tout ingrédient cité doit exister
  dans les référentiels (sinon il serait marqué « estimé »).

> 🗃️ **Idée forte.** Donner à un agent un **espace de fichiers structuré**, c'est lui donner une
> mémoire *inspectable par un humain*. On peut *ouvrir le cerveau* de l'agent et le corriger à la main.

---

## 9. Composition : chaîner des skills en pipeline

Les skills ne sont pas des silos : ils se **composent**.

```
/organiser-semaine-dieteticien   ──►  écrit  commandes/semaine-2026-07-06.md
        (planifie 7 j × 3 repas)              (une « commande » réutilisable)
                                                       │
                                                       ▼
/fiche-commande-excel --commande …  ──►  classeur .xlsx : récap · courses · apports · allergènes
```

Le plan de semaine produit un artefact (un fichier `.md` listant `plat: couverts`) qui devient
l'**entrée** d'un autre skill. C'est exactement la philosophie Unix (*petits outils qui se
combinent*) transposée aux agents : des compétences **découplées mais chaînables**.

Cela préfigure les **systèmes multi-agents** : demain, un agent « planificateur » délègue à un agent
« acheteur » qui délègue à un agent « comptable ». Le contrat entre eux ? **Un format d'échange
clair** (ici, le fichier commande). *L'interopérabilité passe par des formats, pas par de la magie.*

---

## 10. Autonomie et human-in-the-loop

L'autonomie n'est pas binaire. Pense à une échelle (analogie avec la conduite autonome) :

| Niveau | Description | Exemple dans le projet |
|-------|-------------|------------------------|
| L0 | L'humain fait tout | (sans IA) |
| L1 | L'IA suggère, l'humain exécute | « voici une recette, à toi de l'enregistrer » |
| L2 | L'IA agit, **demande confirmation** aux étapes sensibles | `creer-recette` *propose* d'enregistrer, n'écrit pas sans accord |
| L3 | L'IA agit seule sur le **réversible**, confirme l'**irréversible** | lire/calculer librement ; **supprimer** une recette → confirmation |
| L4 | L'IA gère un workflow entier sous supervision | planifier la semaine + courses, l'humain valide à la fin |
| L5 | Autonomie complète | (volontairement *non* visé ici) |

Choix de conception assumés :
- **Actions réversibles** (lire, calculer, proposer) : l'agent les fait librement.
- **Actions à effet de bord** (écrire dans la base, supprimer) : **confirmation requise**. Le skill
  `base-recettes` ne supprime jamais sans accord explicite.
- **Le « plan mode »** : pour une tâche d'ampleur, l'agent **présente d'abord un plan** et attend
  l'approbation humaine avant d'agir (c'est ce qu'on a fait pour construire ce harness). *Réfléchir,
  faire valider, puis agir.*

> ⚖️ **Principe directeur.** Plus une action est *difficile à annuler* ou *tournée vers le monde
> extérieur*, plus on insère un humain dans la boucle. L'autonomie se **dose**, elle ne se subit pas.

---

## 11. Garde-fous, sécurité, responsabilité

Ce projet touche à la **santé** : terrain où l'erreur a un coût réel. D'où des garde-fous *intégrés
dans les données et les skills*, pas ajoutés après coup :

- **Encarts médicaux obligatoires.** Le profil `anorexie` impose un avertissement fort : la
  renutrition doit être **encadrée médicalement** (risque de *syndrome de renutrition inappropriée*).
  Le profil `diabetique` rappelle que les valeurs sont *indicatives*, pas une prescription.
- **Allergènes = aide, pas certification.** Le skill `allergenes-recette` affiche *toujours* un
  encart : « ne vaut pas étiquetage certifié ; vérifier les étiquettes fournisseurs ; en cas de
  doute, ne pas servir ». La **réglementation INCO** (les 14 allergènes) est ainsi *encodée dans le
  prompt* — un exemple concret d'**alignement sur une norme légale**.
- **Honnêteté sur l'incertitude.** Un ingrédient absent des tables est marqué `(estimé)` plutôt que
  présenté comme exact. Un agent digne de confiance **signale ce qu'il ne sait pas**.
- **Moindre privilège.** Chaque skill ne reçoit que les outils nécessaires (`allowed-tools`).

Ces points illustrent des notions centrales de l'IA responsable : **alignement** (faire ce qu'on
attend *et* ce qui est sûr), **transparence**, **traçabilité**, et **responsabilité humaine finale**.
Un agent ne « porte » pas la responsabilité juridique d'un plat servi : le professionnel, si.

---

## 12. Étude de cas : dérouler une requête réelle

Demande de l'utilisateur : *« Crée-moi un plan de repas équilibré et la liste de courses pour la
semaine prochaine. »* Voici la boucle agentique, étape par étape :

1. **Routage.** L'agent lit les `description` des skills → reconnaît
   `organiser-semaine-dieteticien` comme pertinent. *(Concept : discoverability.)*
2. **Chargement du contexte.** Il lit le profil `saine` et l'index `recettes/README.md`.
   *(Concept : grounding — il s'appuie sur des données réelles, pas sur des souvenirs.)*
3. **Décision sous contrainte.** La base a ~11 recettes ; une semaine = 21 repas. Il **réutilise** les
   recettes formelles et **complète** par des « idées légères » marquées `*`. *(Concept : raisonnement
   sous ressources limitées + transparence sur ce qui est formel vs improvisé.)*
4. **Production d'un artefact.** Il écrit `commandes/semaine-2026-07-06.md`, une commande réutilisable.
   *(Concept : état externalisé + format d'échange.)*
5. **Point de contrôle humain.** Il **demande** « combien de personnes ? je lance la suite ? »
   *(Concept : human-in-the-loop sur une étape à effet de bord.)*
6. **Délégation au code.** Sur accord, il lance `generer.py` → un `.xlsx` exact (courses fusionnées,
   apports, allergènes). *(Concept : déterminisme délégué.)*
7. **Restitution honnête.** Il rappelle que les idées `*` ne sont pas dans la liste de courses, et que
   les valeurs sont indicatives. *(Concept : transparence sur les limites.)*

Et l'**incident** : à l'étape 2, dans une session, l'agent a d'abord cherché les recettes au mauvais
endroit (cf. §7). Le correctif `CLAUDE.md` + rappels rend désormais l'étape 2 directe. *Un système
agentique s'améliore en affinant son contexte, pas seulement son modèle.*

---

## 13. Le futur : ce que ce petit projet laisse entrevoir

Ce harnais minuscule est une **maquette du futur logiciel**. Extrapolons :

**a) Le logiciel se « parle » et s'écrit.** Ici, ajouter une compétence = écrire un fichier en
français. Demain, un·e professionnel·le *décrira son métier* et l'IA **fabriquera ses propres
skills** — du *no-code* radical. Le code ne disparaît pas : il devient le « sous-sol » exact sous une
couche de langage.

**b) Des agents personnels et spécialisés.** Plutôt qu'une IA géante généraliste, des **agents de
domaine** outillés de connaissances et de garde-fous précis (un pour le diététicien, un pour le
garagiste, un pour le notaire). La valeur se déplace du modèle vers **le harnais métier** : données,
procédures, sécurité.

**c) Des écosystèmes multi-agents.** Le chaînage `planifier → acheter → facturer` devient une équipe
d'agents qui négocient via des formats partagés. Question d'ingénierie : *les contrats d'interface*
(comme notre fichier « commande ») plus que l'intelligence brute.

**d) L'IA ambiante et continue.** Un agent qui, chaque dimanche, propose le menu de la semaine,
commande chez le grossiste au meilleur prix, ajuste selon la météo et les invendus, et apprend des
retours clients. Le présent projet en contient déjà les briques (planification, coûts, stocks à venir).

**Mais aussi des implications à regarder en face :**
- **Responsabilité et sûreté.** En santé, en droit, en finance, une erreur d'agent peut nuire. D'où
  l'importance des **garde-fous**, de l'**humain dans la boucle** et de la **traçabilité**. Notre
  projet le montre en miniature (encarts, « ne pas servir en cas de doute »).
- **Qualité et biais des données.** L'agent ne vaut que ses référentiels. Une table nutritionnelle
  approximative produit des conseils approximatifs. *Garbage in, garbage out* reste vrai.
- **Vie privée.** Connecté « chez soi », un tel agent voit des données sensibles (santé des clients,
  finances). Qui les héberge ? Qui y accède ? Le *local-first* devient un argument éthique.
- **Dépendance et désapprentissage.** Si l'humain ne sait plus *pourquoi* l'agent décide, il ne peut
  plus le superviser. D'où l'exigence de **transparence** (montrer les calculs, citer les sources).
- **Régulation.** Les 14 allergènes INCO encodés dans nos prompts illustrent un futur où **la loi
  s'écrit aussi dans les agents** — et où il faudra *prouver* qu'ils la respectent.
- **Écologie et accès.** Faire tourner des modèles a un coût. Concevoir *sobre* (déléguer au code ce
  qui peut l'être, comme notre `generer.py`) est aussi une responsabilité.

> 🔭 **Thèse.** Le futur de l'IA utile ne sera pas « un cerveau toujours plus gros », mais **des
> agents bien harnachés** : bonnes données, bonnes procédures, bons garde-fous, bonne place laissée à
> l'humain. *Ce projet est une démonstration à l'échelle 1:1000 de cette thèse.*

---

## 14. Connecter « ça » à un humain non technique

Le diététicien ou le traiteur **n'écrira jamais de code**. Voici comment il vit le système — et
comment on le lui « branche » chez lui.

### Au quotidien : il parle, l'agent fait

Il dit, en langage naturel :
> « Fais-moi un menu de la semaine sans fruits à coque pour une cliente en surpoids, et la liste de
> courses pour 4 personnes. »

L'agent déclenche les bons skills, produit le plan, l'Excel des courses, la fiche allergènes — **sans
qu'il ait à connaître les rouages**. Son rôle : **fournir le savoir métier** (ses recettes, ses prix,
ses contraintes clients) et **valider**.

### Comment l'intégrer « chez soi » (du plus simple au plus abouti)

1. **Application de chat** (le plus accessible). Une interface où il tape ou *dicte* sa demande ;
   l'agent répond et joint les fichiers. Aucune installation technique de sa part.
2. **Assistant vocal.** « Ok, prépare la commande des Martin pour samedi » en cuisine, mains
   occupées. La voix devient l'interface naturelle d'un agent.
3. **Intégration à ses outils existants.** Les sorties sont déjà des **fichiers Excel** : il les
   ouvre, imprime sa liste, l'envoie au grossiste. On peut connecter l'agent à son agenda, sa boîte
   mail, son logiciel de caisse.
4. **Local-first / hébergé.** Pour des données de santé, on privilégie un fonctionnement **sur sa
   machine** ou un hébergement de confiance, avec sauvegardes. Ses recettes restent *son* patrimoine.

### Le contrat humain ↔ agent (essentiel)

| L'humain apporte | L'agent apporte |
|------------------|-----------------|
| Le savoir-faire (recettes, tours de main) | La mise à l'échelle, les calculs exacts |
| Les contraintes réelles (prix, clients, allergies) | La cohérence, la traçabilité, les rappels réglementaires |
| **Le jugement et la responsabilité finale** | Des propositions transparentes et vérifiables |
| La validation des actions sensibles | L'exécution rapide du fastidieux |

### Garde-fous pensés pour un non-technique

- **Confirmations** avant toute action à conséquence (enregistrer, supprimer, commander).
- **Transparence** : l'agent montre *d'où viennent* les chiffres et *ce qu'il a improvisé* (les `*`).
- **Filets de sécurité métier** : « valeurs indicatives », « vérifier les étiquettes », « en cas de
  doute, ne pas servir » — écrits noir sur blanc, à chaque fois.
- **Réversibilité** : tout est en fichiers texte qu'on peut relire, corriger, versionner.

> 🤝 **Message clé pour le pro.** L'agent n'est pas là pour *remplacer* votre expertise, mais pour
> **porter le fastidieux** (calculer, agréger, recopier, vérifier la réglementation) afin que vous
> vous concentriez sur ce qu'aucune IA ne fait : *le goût, la relation client, le jugement.*

---

## 15. Travaux pratiques

Pour s'approprier les concepts, dans l'ordre de difficulté :

1. **Lire un skill « comme l'agent ».** Ouvre `apports-nutritionnels/SKILL.md`. Sans rien exécuter,
   déroule mentalement les étapes sur la recette `curry-pois-chiches`. Où l'agent risque-t-il
   d'hésiter ? *(Objectif : comprendre le routage et la procédure.)*
2. **Enrichir une donnée partagée.** Ajoute l'ingrédient « tofu » dans `nutrition-100g.csv`,
   `allergenes.csv` (→ `soja`) et `rayons.csv`. Vérifie qu'aucune logique de skill n'a bougé.
   *(Objectif : vivre le principe « données partagées ».)*
3. **Casser puis réparer le grounding.** Modifie un chemin d'un skill pour le rendre ambigu, observe
   l'agent se tromper, puis corrige via le contexte. *(Objectif : déboguer un agent par le contexte.)*
4. **Mesurer le déterminisme.** Lance `generer.py` deux fois sur la même commande : les totaux sont
   identiques. Demande ensuite au modèle de faire la même somme « de tête » : compare. *(Objectif :
   stochastique vs déterministe.)*
5. **Ajouter un garde-fou.** Crée un profil `hypertension` (limiter le sodium) avec son encart, et
   fais-le respecter par `creer-recette`. *(Objectif : encoder une contrainte/règle.)*
6. **Concevoir un nouveau skill.** Écris `devis-traiteur` (prix de vente = coût × marge + main
   d'œuvre). Quels outils ? Quelles confirmations ? Quel format de sortie ? *(Objectif : penser
   moindre privilège + autonomie.)*
7. **Réflexion ouverte.** À quel niveau d'autonomie (§10) confierais-tu la *commande automatique chez
   le grossiste* ? Quels garde-fous ajouter ? *(Objectif : doser l'autonomie.)*

---

## 16. Glossaire

- **Agent** : modèle de langage placé dans une boucle perception → décision → action → observation,
  capable d'utiliser des outils.
- **Harness / harnais** : l'infrastructure (ici Claude Code) qui exécute les outils et orchestre la
  boucle ; par extension, l'ensemble outillé d'un projet.
- **Outil (tool)** : capacité primitive appelable par l'agent (lire, écrire, exécuter du shell…).
- **Skill** : procédure de haut niveau écrite en langage naturel (`SKILL.md`) qui dit *quand* et
  *comment* accomplir une tâche métier.
- **Frontmatter** : l'en-tête YAML d'un skill (`name`, `description`, `allowed-tools`…).
- **Context engineering** : l'art de composer le contexte (instructions + données) fourni au modèle.
- **Grounding** : ancrer les réponses dans des sources de vérité fournies, plutôt que dans la mémoire
  du modèle.
- **RAG** (*Retrieval-Augmented Generation*) : fournir au modèle des données récupérées (ici, lire
  des CSV) pour qu'il réponde sur des faits, pas de mémoire.
- **Déterminisme** : même entrée → même sortie (propriété du code, pas du modèle).
- **Idempotence** : ré-exécuter une opération laisse le système dans un état cohérent.
- **Human-in-the-loop** : insertion d'une validation humaine aux étapes sensibles.
- **Moindre privilège** : ne donner que les capacités strictement nécessaires.
- **Alignement** : faire en sorte que l'agent poursuive ce que l'humain veut *et* ce qui est sûr.
- **Source unique de vérité** : une information vit à un seul endroit, référencée partout.

---

## 17. Annexe : utiliser ce fichier comme méta-prompt

Ce document peut **briefer un agent** qui reprendrait le projet. Exemple d'amorce à lui donner :

> « Tu vas travailler sur le harness *Diététicien & Traiteur*. Avant toute action, lis
> `docs/meta-prompt-pedagogique.md` (esprit, architecture, garde-fous) et `CLAUDE.md` (hiérarchie des
> fichiers). Respecte les principes suivants : **skills fins / données partagées** ; **déléguer le
> calcul exact au code** ; **demander confirmation avant toute action à effet de bord** ; **conserver
> les encarts d'avertissement médicaux et allergènes** ; **signaler explicitement toute estimation**.
> Les chemins `recettes/`, `data/`, `profils/`, `commandes/` sont relatifs à la racine du harness. »

En une phrase, l'**intention** du projet, à transmettre à toute IA — ou à tout humain — qui le
reprend :

> **Construire un agent utile, c'est moins entraîner un cerveau que bâtir un bon harnais : des
> données fiables, des procédures claires, des garde-fous explicites, et la juste place laissée à
> l'humain.**
