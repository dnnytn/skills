# skills

My curated Claude Code skills — extracted from real projects and refined over time.

## What's here

Each directory is a Claude Code skill: a `SKILL.md` file that teaches Claude patterns specific to a project or domain.

| Skill | Description |
|-------|-------------|
| [course-creation](./course-creation/SKILL.md) | Self-contained HTML courses, Playwright E2E via `file://`, Vercel deployment, scroll-to-unlock gating |

## Installing a skill globally

```bash
cp -r <skill-dir> ~/.claude/skills/
```

## Adding a new skill

Run `/skill-create` inside Claude Code on any repo to generate a new skill from git history, then copy it here.
