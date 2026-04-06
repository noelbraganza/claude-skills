# Contributing

## Adding a new skill

1. Fork this repo
2. Create a directory: `your-skill-name/`
3. Add `your-skill-name/SKILL.md` with this frontmatter:

```yaml
---
name: your-skill-name
description: >
  One or two sentences describing when Claude should activate this skill.
  Be specific — Claude uses this to decide relevance.
---
```

4. Write the skill content — aim for:
   - Concrete code examples, not just descriptions
   - Comparison tables (service A vs service B)
   - Known gotchas and how to fix them
   - Integration patterns for common frameworks

5. Open a pull request

## What makes a good skill?

- **Specific trigger**: The description should clearly describe what situations activate the skill. Vague descriptions mean the skill won't load when it should.
- **Dense information**: Skills are loaded into context — make every line count. Remove filler.
- **No private data**: No API keys, personal project IDs, internal domain names, or credentials.
- **Generic examples**: Use `your-domain.com`, `your-app-name`, `example@example.com` — not real project details.
- **Stays current**: If an API changes, update the skill.

## Skill quality checklist

- [ ] Frontmatter `name` matches the directory name
- [ ] `description` is specific enough to trigger correctly
- [ ] All code examples use generic placeholders (no real domains, IDs, keys)
- [ ] No references to personal projects, internal tooling, or private infrastructure
- [ ] Includes at least one comparison table or gotcha section
