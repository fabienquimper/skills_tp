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
