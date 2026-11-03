---
name: retro
description: "After a feature ships, propose edits to rules, agents, and CLAUDE.md from what went wrong in its review and QA. Usage: /retro <slug>"
disable-model-invocation: true
argument-hint: "<slug>"
---

Arguments: `$ARGUMENTS`. The first token is `<slug>`; anything after it is `<args>`. If `$ARGUMENTS` is empty, ask for the slug and stop.

# Retro: <slug>

1. Run role `retro` with slug `<slug>`.
2. Show the user the proposal table.
3. For each proposal the user accepts, apply the edit exactly as proposed (minimal change). Commit as `chore(pipeline): retro <slug>`.
4. Append a line to `docs/pipeline/RETROS.md`: date, slug, rounds needed, proposals accepted.
