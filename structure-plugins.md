Oui, exactement — c'est bien ce niveau de simplicité, et je vais te montrer que c'est même plus simple que ce que tu imagines : pas besoin d'un vrai script, une seule commande suffit.

## Le concept en 2 fichiers

```
plugin/
├── knowledge/
│   └── CORE-RULES.md          ← les ~50 règles, en Markdown pur
└── hooks/
    └── hooks.json              ← déclare le hook
```

**`knowledge/CORE-RULES.md`** — un fichier Markdown normal, rien de spécial :
```markdown
## Règles de développement (framework interne)
- Toute route API doit passer par le middleware `authGuard`.
- Les migrations DB sont écrites en SQL brut, jamais via l'ORM.
- ...(≈50 règles, en bullets courts)
```

**`hooks/hooks.json`** — le hook ne fait qu'un `cat` :
```json
{
  "hooks": {
    "SessionStart": [
      {
        "hooks": [
          { "type": "command", "command": "cat ${CLAUDE_PLUGIN_ROOT}/knowledge/CORE-RULES.md" }
        ]
      }
    ]
  }
}
```

C'est tout. Pas de script `.sh` intermédiaire, pas de JSON à construire : la doc confirme que pour l'événement `SessionStart`, **le stdout brut est directement ajouté au contexte de Claude** — pas besoin d'enrober dans du JSON si tu ne fais que charger du contenu statique. `cat` suffit comme commande.

## Deux précisions qui comptent

**1. Le plafond de 10 000 caractères.** La sortie de n'importe quel hook (stdout ou `additionalContext`) est plafonnée à 10 000 caractères — au-delà, elle est tronquée et remplacée par un aperçu + chemin de fichier. Pour ~50 règles courtes en bullets, tu es largement en dessous (quelques milliers de caractères). C'est justement pour ça que Tier A (règles) et Tier B (les 400+ fichiers de doc) doivent rester séparés : Tier A doit tenir dans ce plafond, Tier B non — et c'est pour ça qu'il reste en lecture à la demande plutôt qu'injecté.

**2. `cat` brut vs JSON structuré — un vrai piège à connaître.** Il existe un bug documenté (issue GitHub récente) où le format JSON structuré (`hookSpecificOutput.additionalContext`) émis par un hook **de plugin** ne remonte pas correctement à Claude — celui-ci ne voit qu'un message générique de succès, pas le contenu. Le `cat` en stdout brut, lui, est confirmé fonctionnel par la doc officielle. Donc pour ton cas d'usage précis (contenu statique, pas de champ additionnel comme `systemMessage`), **reste sur du stdout brut** — c'est non seulement plus simple, mais aussi actuellement plus fiable pour un hook livré dans un plugin.

Si un jour tu veux aussi qu'un message visible s'affiche au développeur dans le terminal (pas juste dans le contexte de Claude), il faudra passer par le JSON avec un champ `systemMessage` en plus — mais garde ça pour plus tard, vu le bug ci-dessus.

## Maintenance

`CORE-RULES.md` vit dans le repo du plugin, versionné avec lui. Une mise à jour du framework = un commit dans ce fichier + publication d'une nouvelle version du plugin. Les devs n'ont rien à faire : à la prochaine session (ou après `/reload-plugins`), le `cat` sert la version à jour.




problème initial 

Voici la synthèse de toute la réflexion qu'on a menée.

## Le problème de départ

Comment distribuer un référentiel d'entreprise volumineux (400-500 fichiers Markdown de règles/framework) à 500 développeurs via un plugin Claude Code, sachant que :
- **Hygiène Git** : interdiction de copier ces fichiers dans chaque dépôt applicatif.
- **Coût token** : interdiction de charger tout le corpus en mémoire à chaque session.
- **DX** : le moins d'actions manuelles possible pour les 500 devs.
- **Infra simple** : pas de serveur distant, pas de base vectorielle.
- **Le blocage technique** : Claude Code ne charge nativement que les règles situées dans le dossier local du projet — jamais celles rangées passivement à l'intérieur d'un plugin. Il fallait un "pont" pour contourner ça.

## Première proposition (que j'ai fait évoluer)

J'ai d'abord proposé une architecture assez lourde : des dizaines de Skills documentaires (un par domaine), un éventuel serveur MCP local pour la recherche, et un `@import` dans le **CLAUDE.md global** du développeur (`~/.claude/CLAUDE.md`) pour faire le pont.

## Ta première objection, décisive

Tu as pointé que le CLAUDE.md global pollue **tous** les projets du développeur, pas seulement ceux qui utilisent ce framework — un dev avec 10 projets dont 3 seulement concernés se retrouverait avec ce contexte chargé partout. Il fallait un scoping **par projet**, pas par machine. Ça nous a amenés à découvrir que Claude Code permet justement d'activer un plugin en **scope "project"** via `.claude/settings.json` (un fichier commité, mais qui ne contient qu'une référence au plugin — pas les 500 fichiers). Ça réglait le scoping *et* renforçait la DX : c'est même une action unique par repo (pas par dev), faite une fois par un tech lead.

## Ta deuxième objection : la mutualisation des règles

Tu ne voulais pas que chaque skill répète les mêmes ~50 règles transverses. Il fallait un mécanisme "comme un CLAUDE.md, mais qui vivrait dans le plugin" — une base commune chargée une fois, sur laquelle les skills s'appuient sans redite.

## L'architecture finale, à deux niveaux

- **Niveau A (socle commun)** : les ~50 règles fondamentales, dans un simple fichier `CORE-RULES.md`, injectées **verbatim** à chaque session via un hook `SessionStart` qui ne fait qu'un `cat` de ce fichier — sous le plafond de 10 000 caractères, donc toujours présentes, sans qu'aucun skill n'ait à les répéter.
- **Niveau B (corpus détaillé)** : les 400+ fichiers de doc restent sur disque dans le plugin, jamais injectés d'office. Claude les découvre via un index pointeur et les lit à la demande avec ses outils natifs Read/Grep — pas besoin de MCP ni de base vectorielle pour ça.
- **Skills** : réservées aux vraies procédures outillées (scaffolding, checks), pas à la documentation — elles s'appuient sur le Niveau A déjà en contexte.
- **Application forcée** : un hook `PreToolUse` pour les règles vraiment bloquantes — seule couche réellement déterministe, le reste restant consultatif.
- **Distribution/scoping** : plugin déclaré en scope "project" dans `.claude/settings.json`, commité une fois par repo — zéro action répétée pour les 500 devs, zéro pollution des projets non concernés.




règle pour mettre en avant des choses


## Règles d'or — SKILL.md efficace

1. **Balises XML** ✅ — Utiliser `<instructions>`, `<example>`, `<formatting>` pour séparer les parties du prompt. Évite que Claude mélange contexte/instructions/exemples.

2. **Pas de MAJUSCULES criées** ❌ — Anthropic déconseille ALWAYS/NEVER en majuscules. Préférer expliquer le *pourquoi* plutôt que crier une règle rigide.

3. **Titres vs gras** ❓ — Pas de mécanisme documenté. Utiliser des titres pour la lisibilité, point.

4. **Instructions affirmatives** ✅ — Dire quoi faire, pas quoi éviter. Test simple : un collègue sans contexte doit pouvoir suivre le prompt sans confusion.

5. **Concision = priorité** — Chaque token du SKILL.md entre en compétition avec le reste du contexte. Ne mettre que ce que Claude ne sait pas déjà.

6. **La `description` est reine** — C'est elle qui décide si le skill se déclenche parmi 100+ skills possibles. Doit être précise et couvrir les cas d'usage réels.

7. **Tester en conditions réelles** — Faire écrire le skill par Claude, le faire tester par une instance fraîche sur des vraies tâches, avant de figer.

Tu veux que je transforme ça en template SKILL.md prêt à l'emploi ?
