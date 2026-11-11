---
name: new-skill
description: Create new Claude skills and edit or improve existing ones. Use whenever the user wants to make a skill, write a SKILL.md, turn a workflow or this conversation into a skill, capture a repeated process as a reusable skill, or fix a skill that isn't triggering or behaving well, even if they don't say the word "skill" outright (e.g. "can you remember how to do this every time", "package this up so Claude always does it this way").
---

# New Skill

Help the user go from "I want Claude to do X" to a working, installed skill, then refine it with them.

The loop is:

1. Work out what the skill should do and when it should trigger
2. Write a draft
3. Install it so it's live
4. Try it on a realistic prompt or two and look at what happens
5. Revise based on what the user thinks, repeat until they're happy

Figure out where the user is in this loop and pick up from there. Someone saying "make a skill for X" starts at step 1. Someone with a draft that misbehaves starts at step 5. Stay flexible: if the user just wants a quick draft with no back-and-forth, do that.

## Communicating with the user

Users range from seasoned developers to people who opened a terminal for the first time last week. Read the cues. Terms like "frontmatter" or "YAML" are fine with someone who clearly codes; otherwise give a short plain-English gloss the first time you use them.

---

## 1. Capture intent

The conversation may already contain the workflow (e.g. "turn this into a skill"). If so, mine it first: the tools used, the order of steps, corrections the user made, the input and output formats. Then fill gaps with the user and get them to confirm before drafting.

Pin down:

1. What should the skill let Claude do?
2. When should it trigger? What would the user actually type?
3. What does good output look like? (format, length, files produced)
4. Where should it live? (see Installing below; default to the user's own setup)

Ask about edge cases, inputs, example files, success criteria and dependencies. Don't over-interview: if a sensible default exists, propose it and move on. If MCP tools or docs would help (looking up an API, checking how a library works), do that research yourself so the user doesn't have to.

## 2. Write the SKILL.md

### Frontmatter

- **name**: lowercase, hyphenated, matches the folder name.
- **description**: the main thing that decides whether Claude loads the skill. Say both what it does and the specific situations it applies to. All "when to use" guidance belongs here, not in the body, because the body isn't read until after the skill triggers.

  Claude tends to *under*-trigger skills, so make descriptions a little pushy. Instead of "Builds dashboards for internal data", write "Builds dashboards for internal data. Use whenever the user mentions dashboards, metrics, charts of company data, or wants to visualise any internal numbers, even if they don't say 'dashboard'."

- Optional Claude Code fields, only when they earn their place:
  - `allowed-tools`: tools the skill may use without prompting
  - `disable-model-invocation: true`: only runs when the user types `/name`, never auto-triggers (good for deploys, sends, anything with side effects)
  - `argument-hint`: shown in the slash-command menu, e.g. `[issue-number]`

### Anatomy

```
skill-name/
├── SKILL.md          (required: frontmatter + instructions)
├── scripts/          (optional: code for deterministic or repetitive steps)
├── references/       (optional: docs Claude reads only when needed)
└── assets/           (optional: templates, icons, fonts used in output)
```

### Progressive disclosure

Skills load in three layers:

1. **Name + description**: always in context (~100 words), so keep it tight
2. **SKILL.md body**: loaded when the skill triggers; aim for under 500 lines
3. **Bundled files**: loaded or run only when needed; size is effectively unlimited

So:

- If SKILL.md is getting long, move detail into `references/` and point to it with clear guidance on *when* to read each file.
- Give reference files over ~300 lines a table of contents.
- When a skill covers several variants (AWS/GCP/Azure, React/Vue), put each in its own reference file and have SKILL.md say how to pick. Claude then reads only the one it needs.
- If a step is mechanical and the same every time, write a script in `scripts/` rather than describing it in prose. Scripts run without being loaded into context and don't drift.

### Writing style

- Use the imperative: "Read the config", not "You should read the config".
- **Explain why.** Models are smart; give them the reasoning and they'll handle cases you didn't anticipate. A wall of ALWAYS/NEVER in caps is a yellow flag. Reframe it as the reason the thing matters.
- Keep it general. The skill will run on many prompts you'll never see, so describe the pattern, not just the one example in front of you.
- Show examples where format matters:

  ```markdown
  ## Commit message format
  **Example:**
  Input: Added user authentication with JWT tokens
  Output: feat(auth): implement JWT-based authentication
  ```

- Pin down output formats explicitly when the user cares about them:

  ```markdown
  ## Report structure
  Use this template:
  # [Title]
  ## Summary
  ## Findings
  ## Recommendations
  ```

Write a draft, then reread it as if you were the model receiving it cold. Cut anything that isn't pulling its weight and clarify anything ambiguous.

### Lack of surprise

A skill's contents should match what the user would expect from its description. Don't write skills that hide behaviour, exfiltrate data, or facilitate unauthorised access. Roleplay or persona skills are fine.

## 3. Installing

Claude Code finds skills in:

- `~/.claude/skills/<name>/`: personal, available in every project
- `<repo>/.claude/skills/<name>/`: project-specific, shared with anyone using the repo

Default for this user: keep each personal skill as its own git repo in `~/Development/<name>/` with the skill at `skills/<name>/SKILL.md` and a short `README.md` at the repo root, then symlink it in:

```bash
ln -s ~/Development/<name>/skills/<name> ~/.claude/skills/<name>
```

This keeps the skill version-controlled and editable in place; changes are live immediately. Commit the initial version with a Conventional Commits message (e.g. `feat: add <name> skill`). If the skill is clearly project-specific, put it in that repo's `.claude/skills/` instead and say so.

Before creating anything, check `~/.claude/skills/` for a name clash. Confirm the symlink resolves (`ls ~/.claude/skills/<name>/SKILL.md`).

## 4. Try it

No formal eval harness. Just a sanity check:

- Come up with one or two realistic prompts, the kind the user would actually type, with real detail.
- Run through them following the skill's instructions (or spawn a subagent pointed at the skill if the user wants an independent run) and show the user what came out.
- Watch for: the skill doing busywork that doesn't help, steps being skipped, instructions that were ambiguous, and the same helper code being written each time (a sign it belongs in `scripts/`).

If the skill isn't triggering when it should, the description is almost always the culprit. Rewrite it to name the situations and phrasings more concretely. New skills may need a fresh session to be picked up.

## 5. Improve

When revising based on feedback:

1. **Generalise from the feedback.** You're iterating on a couple of examples, but the skill will run on far more. Avoid fiddly, overfitted fixes and rigid MUSTs. If something keeps going wrong, try a different explanation, metaphor or approach.
2. **Keep it lean.** Remove instructions that aren't changing behaviour. Shorter skills are followed more reliably.
3. **Explain the why.** Even if the user's feedback is terse ("no, wrong"), work out what they actually wanted and why, and write that understanding into the skill.
4. **Bundle repeated work.** If every run reinvents the same helper, write it once in `scripts/` and point to it.

Draft the revision, reread it fresh, tighten, apply. Commit meaningful changes. Stop when the user is happy or further changes stop helping.

## Updating an existing skill

- Keep the original `name` and folder name.
- If it's a symlinked skill in `~/Development`, edit it in place and commit. If it's a third-party skill (cloned from GitHub), don't edit it without asking: local changes get clobbered on the next pull.
- If the installed path is read-only, copy it somewhere writable, edit there, and tell the user where it is.
