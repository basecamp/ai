# 37signals AI Skills

This repo contains skills for AI coding assistants.

## Structure

```
skills/           # Agent skills (SKILL.md + references/)
install.md        # Agent-guided installation
```

## Skills

Skills follow the [Agent Skills](https://agentskills.io/specification) spec:
- `SKILL.md` with YAML frontmatter (name, description, triggers)
- Optional `references/` directory for supporting context

## Internal vs General

- **Skills** section in README: General-purpose, designed for broad use
- **Internal** section: 37signals-specific, references internal paths and conventions

## Contributing

When adding a skill:
1. Create `skills/SKILL_NAME/SKILL.md` with required frontmatter
2. Add supporting files to `skills/SKILL_NAME/references/` if needed
3. Add entry to appropriate README section (Skills or Internal)
