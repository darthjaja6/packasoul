# Claude Code Deploy Reference

Use this reference when deploying a `packasoul` snapshot into a Claude Code style project.

The goal is to reconstruct a Claude-native project layout, not to hard-map canonical fields too mechanically.

## Core Principle

Claude Code is much flatter than OpenClaw.

The deploy skill should:

1. preserve ordinary state files in the project tree
2. keep `CLAUDE.md` as the clear runtime entrypoint
3. install project-local skills
4. make sure important state stays reachable from `CLAUDE.md`
5. synthesize a thin `CLAUDE.md` only when the canonical `soul` comes from another runtime

## Claude Code Project Files

### `CLAUDE.md`

Purpose:

- primary soul file
- identity
- instructions
- workflow rules
- references to other project files

Canonical `soul` should land here.

If the source snapshot has additional state that must stay top-of-mind, the deploy skill may:

- keep it as a separate file
- add or preserve `@path` references from `CLAUDE.md`

If the canonical `soul` does not already come from a Claude-native project:

- preserve the source `soul` as a supporting project file
- synthesize a thin `CLAUDE.md` that tells Claude which files to read first
- keep the detailed identity or workflow rules in those supporting files instead of flattening them into the entrypoint

### Ordinary Project State Files

Claude Code does not require the same fixed split as OpenClaw.

So most canonical `state` should remain ordinary files in the project tree:

- `profile.json`
- `log.md`
- `plan.md`
- `activities/...`
- `doc/...`
- other custom files

Use the snapshot's logical paths unless the deploy plan has a better Claude-native structure.

Recommended kind-to-shape mapping:

- `user_profile` -> ordinary project file such as `profile.json` or `USER.md`
- `memory_curated` -> `MEMORY.md` or another durable summary file
- `memory_log` -> `memory/` or `memories/`
- `tool_notes` -> `TOOLS.md` or other tool doc
- `project_rules` -> project doc referenced from `CLAUDE.md`
- `identity` -> `IDENTITY.md`, `SOUL.md`, or another stable persona file
- `custom` -> keep the canonical logical path

### `.claude/skills/`

Purpose:

- project-local skills

Deploy all canonical `skills[]` into:

```text
<project>/.claude/skills/<name>/
```

## Reconstruction Heuristic

Use this order:

1. restore state files into the project tree
2. if canonical `soul` is already `CLAUDE.md`, keep it as the entrypoint
3. if canonical `soul` comes from another runtime, preserve it as a supporting project file and synthesize a thin `CLAUDE.md`
4. install `.claude/skills/<name>/`
5. ensure `CLAUDE.md` still points at important auxiliary state where needed

For cross-runtime deploys, prefer this startup shape:

1. `CLAUDE.md` as the entrypoint
2. source `soul` preserved as a supporting file
3. top-priority state files explicitly referenced from `CLAUDE.md`
4. everything else left as ordinary project files

## What To Avoid

- do not invent OpenClaw-style files like `IDENTITY.md` or `AGENTS.md`
- do not flatten all state into `CLAUDE.md`
- do not lose `@path`-style relationships that help Claude keep important context in view
- do not treat a renamed non-Claude soul as a good Claude entrypoint if a thin `CLAUDE.md` wrapper would be clearer

## Verification Checklist

After deployment, check:

- does `CLAUDE.md` exist and carry the packaged soul
- are important state files restored in the right project locations
- are expected skills under `.claude/skills/`
- does the agent behave like the packaged project agent when opened in Claude Code

For cross-runtime deploys, also check:

- is the source `soul` preserved somewhere readable in the project
- does `CLAUDE.md` explicitly point at that supporting file
- does `CLAUDE.md` stay thin instead of absorbing the whole project state
