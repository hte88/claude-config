# claude-config

Skills et configuration Claude personnalisés.

## Installation

### Via le marketplace officiel (recommandé)

```bash
/plugin install hte88/claude-config
```

### Manuellement

```bash
git clone https://github.com/hte88/claude-config.git ~/.claude/plugins/claude-config
```

## Skills inclus

| Skill | Description |
|-------|-------------|
| **vue-code-review** | Code review strict pour Vue 3, Nuxt 4, TypeScript — interdit `any`, enforce best practices |

## Ajouter un skill

Créer un dossier dans `skills/` avec un fichier `SKILL.md` :

```
skills/
└── mon-skill/
    ├── SKILL.md           # Obligatoire
    └── references/        # Optionnel
        └── details.md
```

## Licence

MIT
