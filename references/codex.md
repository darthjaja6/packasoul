# Codex Runtime Reference

Use this reference when packing from or deploying into a Codex-style project.

The goal is to reconstruct a Codex-native project layout, not to hard-map canonical fields too mechanically.

## Core Principle

Codex treats `AGENTS.md` as the repository-local instruction entrypoint.

The runtime may also apply additional global or nested instruction layers, but the portable project expression should center on the repo-local `AGENTS.md`.

The deploy skill should:

1. preserve the canonical entry instruction as `AGENTS.md`
2. keep supporting project files as ordinary files in the workspace
3. keep the instruction graph explicit, so `AGENTS.md` points to the files Codex should read next
4. avoid treating non-Codex runtime files as the primary entrypoint unless the user explicitly chose another runtime

## Codex Project Files

### `AGENTS.md`

Purpose:

- primary project instruction entrypoint
- workflow rules
- repository conventions
- startup reading order
- references to supporting files

Canonical `soul` should land here when the target runtime is Codex.

If the source agent's durable identity or working memory lives in other files, `AGENTS.md` should point at them rather than absorbing everything inline.

### Supporting Project Files

Common supporting files may include:

- `SOUL.md`
- `USER.md`
- `TOOLS.md`
- `HEARTBEAT.md`
- `IDENTITY.md`
- `MEMORY.md`
- `memory/...`
- other repo-local docs the workflow explicitly depends on

These should usually remain separate `state` files.

For Codex, they are not assumed to be auto-loaded just because they exist.
They matter because `AGENTS.md` or a nearer instruction layer tells Codex to read them.

### `AGENTS.override.md`

Purpose:

- narrower instruction override for a subdirectory or work slice

If a project intentionally relies on nested `AGENTS.md` or `AGENTS.override.md` files, preserve them as `state` only when the current agent workflow actually depends on them.

Do not invent extra override files unless the source workflow clearly requires them.

### Global Codex Files

Examples:

- `~/.codex/AGENTS.md`
- `~/.codex/skills/...`

These are runtime-global, machine-local helpers.

Do not pack them by default.
Only include them if the user explicitly wants machine-local Codex state captured and the item belongs to this agent rather than the user's whole Codex setup.

### `skills/`

Purpose:

- project-local skill source directories used by the Codex workflow

Deploy all canonical `skills[]` into:

```text
<project>/skills/<name>/
```

## Pack Heuristic

When the source runtime is Codex:

1. identify the repo-local `AGENTS.md` that defines the current agent workflow
2. treat that file as the source `soul`
3. read the files it explicitly tells Codex to load next
4. classify those supporting files as canonical `state` when they are required for future continuity
5. classify only the directly used skill directories as canonical `skills`

## Deploy Heuristic

When the target runtime is Codex:

1. restore supporting `state` files into their logical project paths
2. if canonical `soul` is already `AGENTS.md`, keep it as the entrypoint
3. if canonical `soul` comes from another runtime, preserve it as a supporting project file and synthesize a thin `AGENTS.md` entrypoint
4. install project-local skills under `skills/<name>/`
5. ensure `AGENTS.md` explicitly points at the important supporting files
6. do not force Codex projects into OpenClaw-style or Claude-style layouts

## What To Avoid

- do not assume `SOUL.md` or `CLAUDE.md` is the Codex entrypoint
- do not pack every markdown file in the repo just because `AGENTS.md` exists
- do not silently absorb global `~/.codex` files into a portable snapshot
- do not guess the source runtime when a repo mixes `AGENTS.md`, `CLAUDE.md`, and `SOUL.md`

## Verification Checklist

After packing or deploying a Codex-style project, check:

- does `AGENTS.md` exist and function as the clear entrypoint
- are required supporting files restored in their expected project locations
- does `AGENTS.md` still point at the files Codex must read next
- are non-portable global Codex files excluded unless explicitly requested
