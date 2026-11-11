# new-skill

A Claude Code skill for creating and improving other skills. It's a slimmed-down take on Anthropic's `skill-creator`, without the eval harness, benchmarking, viewer and description-optimisation loop.

It walks through:

1. Capturing what the skill should do and when it should trigger
2. Writing the `SKILL.md` (frontmatter, structure, progressive disclosure, style)
3. Installing it as its own repo in `~/Development/<name>/`, symlinked into `~/.claude/skills/`
4. A quick manual try-out on realistic prompts
5. Revising from feedback

## Install

```bash
ln -s ~/Development/new-skill/skills/new-skill ~/.claude/skills/new-skill
```

Then ask Claude to "make a skill for …" or run `/new-skill`.
