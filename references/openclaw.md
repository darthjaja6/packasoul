# OpenClaw Deploy Reference

Use this reference when deploying a `packasoul` snapshot into OpenClaw.

The goal is not to hard-map manifest fields one-to-one.
The goal is to reconstruct an OpenClaw-native agent workspace from the canonical snapshot.

## Core Principle

OpenClaw expects a workspace made of several cooperating markdown files.

The deploy skill should:

1. read the canonical `manifest`
2. read the actual snapshot contents
3. understand what each piece means in OpenClaw terms
4. synthesize an OpenClaw-native workspace
5. keep `SOUL.md` and `AGENTS.md` distinct when identity and process are meaningfully different

Do not blindly copy everything into fixed destinations.

## OpenClaw Workspace Files

### `SOUL.md`

Purpose:

- core identity
- durable behavior
- non-negotiable principles
- the agent's center of gravity

Good inputs:

- canonical `soul`
- stable behavioral rules from state
- compact identity statements

When the source snapshot comes from a flatter runtime, `SOUL.md` should usually become the stable identity anchor rather than a full dump of the source entrypoint.

Do not dump every state file into `SOUL.md`.

### `AGENTS.md`

Purpose:

- working style
- execution rules
- repo or workflow discipline
- coordination / process rules

Good inputs:

- process-oriented parts of the canonical `soul`
- `project_rules`
- workflow-heavy state

If a source soul is mostly a development workflow document, some of it belongs here instead of all of it staying in `SOUL.md`.

For cross-runtime deploys, a thin synthesized `AGENTS.md` is often better than renaming another runtime's entry file verbatim.

### `IDENTITY.md`

Purpose:

- name
- outward identity
- creature / vibe / emoji / avatar style

Good inputs:

- explicit identity state
- clear persona markers inside canonical `soul`

If the source agent has no strong named persona, synthesize a minimal stable identity from role and repo context.

Do not leave a mature deployed agent with the default OpenClaw bootstrap identity template unless there is no better signal.

### `USER.md`

Purpose:

- what the agent knows about the human
- how to address them
- recurring preferences or context

Good inputs:

- `user_profile`
- explicit user notes
- stable preference state

### `TOOLS.md`

Purpose:

- tool usage rules
- connector / integration boundaries
- operational constraints

Good inputs:

- `tool_notes`
- tool-related parts of the source soul

### `MEMORY.md`

Purpose:

- curated long-term memory
- distilled context worth carrying forward

Good inputs:

- `memory_curated`
- durable summaries synthesized from state

### `memory/`

Purpose:

- memory log files
- timestamped or rolling history

Good inputs:

- `memory_log`

### `skills/`

Purpose:

- per-agent skills available inside the OpenClaw workspace

Deploy all canonical `skills[]` into:

```text
<workspace>/skills/<name>/
```

## Bootstrap Rule

For a mature deployed agent:

- `BOOTSTRAP.md` should not remain the primary source of identity
- if OpenClaw auto-generated a generic bootstrap file, the deploy skill should remove it or make it irrelevant

Otherwise the agent may answer like a fresh generic OpenClaw assistant instead of the packaged agent.

## Reconstruction Heuristic

Recommended kind-to-shape mapping:

- `identity` -> `IDENTITY.md` or `SOUL.md`
- `user_profile` -> `USER.md`
- `tool_notes` -> `TOOLS.md`
- `memory_curated` -> `MEMORY.md`
- `memory_log` -> `memory/<...>`
- `project_rules` -> `AGENTS.md` or another workflow doc referenced from it
- `plan` -> keep as ordinary workspace file unless it clearly belongs in `AGENTS.md`
- `activity` -> keep as ordinary workspace file
- `custom` -> keep the canonical logical path unless OpenClaw has a clearly better native home

Cross-runtime deploy rule:

- if canonical `soul` already separates identity and process in an OpenClaw-native way, preserve that shape
- otherwise treat the canonical `soul` as source material, not as a file to rename blindly
- synthesize a compact `SOUL.md` for durable identity
- synthesize a compact `AGENTS.md` for workflow and operating rules
- keep the fuller source material as supporting files when needed

Use this order:

1. restore state files and skills
2. classify which parts of the canonical `soul` are identity versus process
3. synthesize or preserve `SOUL.md` as the durable identity anchor
4. synthesize or preserve `AGENTS.md` as the workflow entrypoint
5. place state into the most natural OpenClaw files
6. keep unmatched state as ordinary workspace files
7. remove or neutralize conflicting bootstrap defaults

Recommended startup graph after deploy:

1. `SOUL.md` says who the agent is
2. `AGENTS.md` says how the agent works
3. `USER.md`, `TOOLS.md`, `MEMORY.md`, and `memory/` hold long-lived supporting state
4. ordinary files remain available for narrower future work

## What To Avoid

- do not let empty default `IDENTITY.md` define the agent
- do not let generic `BOOTSTRAP.md` override a mature packaged soul
- do not treat the runtime model name as the agent's identity
- do not assume every source file deserves a first-class OpenClaw file
- do not rename a Claude or Codex entry file directly to `SOUL.md` and call that reconstruction
- do not collapse both identity and workflow into a single huge OpenClaw markdown file unless the source agent is truly that simple

## Verification Checklist

After deployment, check:

- does the agent describe itself using the packaged role/persona rather than generic OpenClaw bootstrap language
- does it know the durable user/project context it should know
- are the expected skills present under `skills/`
- do normal OpenClaw conversations feel like the packaged agent rather than a blank assistant

For cross-runtime deploys, also check:

- is identity clearly anchored in `SOUL.md`
- is workflow clearly anchored in `AGENTS.md`
- were curated user/tool/memory files placed into the right OpenClaw-native homes
- does the restored workspace avoid depending on generic bootstrap defaults
