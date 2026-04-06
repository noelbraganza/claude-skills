# skills-much

A collection of reusable skills for [Claude Code](https://claude.ai/code) — drop-in context files that make Claude an expert in specific domains, tools, and tech stacks.

Inspired by [garrytan/gstack](https://github.com/garrytan/gstack).

---

## What are skills?

Skills are Markdown files that Claude Code loads automatically when it detects relevant context. Each skill is a directory containing a `SKILL.md` with:

- A YAML frontmatter block (`name`, `description`) that tells Claude when to activate it
- Deep reference material — integration patterns, gotchas, code examples, comparison tables

Claude reads the skill at the start of a conversation and uses it to give more accurate, opinionated answers without you having to explain the same context every time.

---

## Skills in this collection

| Skill | Description |
|---|---|
| [`us-tech-stack`](./us-tech-stack/) | Next.js 14 + Supabase + Vercel + Resend — architecture decisions, RLS patterns, cron gotchas, email flows |
| [`eu-tech-stack`](./eu-tech-stack/) | EU-sovereign alternatives to US cloud services: Hetzner, UpCloud, AppSignal, Brevo, 46elks, OpenCage, Mistral AI, Mollie |

---

## Quick start

Clone this repo directly into your Claude skills directory:

```bash
git clone https://github.com/noelbraganza/skills-much.git ~/.claude/skills/skills-much
```

That's it. Claude Code will pick up the skills automatically on the next conversation.

### Install a single skill

If you only want one skill, copy just that directory:

```bash
# us-tech-stack only
cp -r ~/.claude/skills/claude-skills/us-tech-stack ~/.claude/skills/

# eu-tech-stack only
cp -r ~/.claude/skills/claude-skills/eu-tech-stack ~/.claude/skills/
```

### Keep skills updated

```bash
cd ~/.claude/skills/skills-much && git pull
```

---

## Skill structure

```
skill-name/
└── SKILL.md          # The skill content with YAML frontmatter
```

`SKILL.md` frontmatter:

```yaml
---
name: skill-name
description: >
  One or two sentences that describe when Claude should activate this skill.
  Be specific — Claude uses this to decide relevance.
---
```

---

## Contributing

Contributions welcome. See [CONTRIBUTING.md](./CONTRIBUTING.md).

---

## License

MIT — see [LICENSE](./LICENSE).
