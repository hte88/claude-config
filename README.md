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

### Méthodologie (superpowers core)

| Skill | Description |
|-------|-------------|
| **brainstorming** | Explorer l'intention et les requirements avant l'implémentation |
| **dispatching-parallel-agents** | Paralléliser des tâches indépendantes via sous-agents |
| **executing-plans** | Exécuter un plan d'implémentation avec checkpoints |
| **finishing-a-development-branch** | Guider la finalisation d'une branche (merge, PR, cleanup) |
| **receiving-code-review** | Recevoir et traiter du feedback de code review |
| **requesting-code-review** | Vérifier le travail avant merge |
| **subagent-driven-development** | Exécuter des plans via sous-agents dans la session courante |
| **systematic-debugging** | Debugging en 4 phases avec analyse root cause |
| **test-driven-development** | Workflow RED-GREEN-REFACTOR |
| **using-git-worktrees** | Isolation via git worktrees pour le feature work |
| **using-superpowers** | Découverte et utilisation des skills disponibles |
| **verification-before-completion** | Vérifier avant de déclarer le travail terminé |
| **writing-plans** | Planification structurée multi-étapes |
| **writing-skills** | Créer et éditer des skills |

### Stack-specific

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
