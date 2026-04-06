# packasoul

**Pack your agent once. Rebuild it for Codex, Claude Code, or OpenClaw.**

`packasoul` is a skill for moving agents between systems like Codex, Claude Code, and OpenClaw without reducing them to a pile of copied files.

If you've ever had an agent that worked beautifully in one runtime and felt strangely hollow after being moved to another, this is the problem `packasoul` is trying to solve.

Instead of saying "copy these files over there," it asks a more useful question:

**What makes this agent itself?**

That usually means separating:

- the core identity and operating rules
- the durable state that gives the agent continuity
- the actual skills it uses to do its work
- the original operating semantics that tell the runtime how this agent actually works

Then it can reconstruct that agent in another runtime using that runtime's native shape.

It has two top-level actions:

- `pack`: inspect a source agent project, identify its canonical `soul`, `state`, `skills`, and workflow dependencies, then write `manifest.yaml` and `binding.json`
- `deploy`: reconstruct that packed agent into a target runtime using that runtime's native conventions

This repo intentionally stays minimal:

- `SKILL.md`
- `references/`
- this `README.md`

## Why It Exists

Agent runtimes all express the same underlying thing differently.

- Codex leans on `AGENTS.md`
- Claude Code leans on `CLAUDE.md`
- OpenClaw often splits identity, workflow, user context, and memory across several files

If you migrate an agent by filename alone, you often keep the shell and lose the person.

`packasoul` treats migration as semantic reconstruction:

1. understand the source runtime
2. capture the agent in canonical form
3. rebuild it in the target runtime's best-practice layout

## Core Model

`packasoul` treats an agent as three canonical parts:

- `soul`: the single core file that defines identity, behavior, and operating rules
- `state`: persistent non-soul, non-skill files the agent must keep in order to continue working as the same agent
- `skills`: skill directories the agent directly or indirectly uses in its normal workflow

It also tracks something just as important:

- `operating semantics`: the source agent's real file usage, skill usage, and working flow

Supporting artifact:

- `snapshot`: an optional portable archive produced during `pack`

`snapshot` is not a separate top-level action. It is just an optional output of `pack`.

## What It Does

`packasoul` is not a blind file copier.

It should:

1. understand how the source runtime expresses the agent
2. preserve the source agent's operating semantics in canonical form
3. reconstruct those semantics in the target runtime's best-practice startup shape

## Runtime References

Current references:

- [references/codex.md](references/codex.md)
- [references/claude-code.md](references/claude-code.md)
- [references/openclaw.md](references/openclaw.md)

These references define:

- runtime entrypoints
- native file responsibilities
- startup behavior and file-loading expectations
- cross-runtime reconstruction rules
- verification checklists
- anti-patterns

If source runtime detection is ambiguous, `packasoul` should not guess. It should ask the user to choose:

- `codex`
- `claude-code`
- `openclaw`

## Quick Example

You point `packasoul` at an agent project.

It should come back with something like:

- `soul`: this is the one core file
- `operating semantics`: this is how the source agent actually works
- `state`: these files are necessary for continuity
- `skills`: these are the skills the agent actually uses

Then you confirm the proposal.

From there it can:

- write `manifest.yaml`
- write `binding.json`
- optionally produce a snapshot
- reconstruct the agent into another runtime

## Pack Workflow

1. inspect the project
2. infer or confirm the source runtime
3. read the source runtime reference
4. identify `soul`, `workflow`, `state`, and `skills`
5. show the proposal to the user
6. write `manifest.yaml` and `binding.json`
7. optionally generate a snapshot

## Deploy Workflow

1. inspect the packed project or snapshot
2. confirm the target runtime
3. read the target runtime reference
4. show a deploy plan
5. reconstruct the target runtime startup graph and files directly
6. verify soul, state, skills, and memory behavior in the target workspace

Deploy should be runtime-native reconstruction of both file layout and startup behavior, not naive filename remapping.

## Repository Layout

- [SKILL.md](SKILL.md)
- [references/codex.md](references/codex.md)
- [references/claude-code.md](references/claude-code.md)
- [references/openclaw.md](references/openclaw.md)

## Installation

Copy this repo's `SKILL.md` and `references/` into whatever skill system you use.

The exact installation path depends on the host runtime. The important part is:

- keep `SKILL.md` as the skill entrypoint
- keep `references/` next to it so the skill can read them during `pack` and `deploy`

## What This Repo Is Not

This is not:

- a CLI tool
- a background service
- a runtime adapter implementation
- a framework for syncing private runtime internals

It is a skill plus the reference knowledge that makes the skill useful.

## Design Principles

- agent-first: the skill does the semantic work
- reference-driven: runtime behavior comes from explicit runtime references
- canonical-first: normalize before reconstructing
- explicit confirmation: ambiguous runtime detection or fuzzy classification must be confirmed with the user
- minimal artifact model: `pack` and `deploy` are the only top-level actions
