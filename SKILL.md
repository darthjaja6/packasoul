---
name: packasoul
description: Packs an agent project into canonical agent semantics and an optional portable archive, or reconstructs that packed agent into a target runtime. Use when the task is to capture or reconstruct an agent across frameworks such as Claude Code, Codex, and OpenClaw.
---

# /packasoul

## Mission

Use this skill when the user wants to:

- pack an agent project
- inspect what should go into a manifest
- deploy a packed agent into another agent runtime

This skill owns the full pack and deploy workflow.

## Inputs And Outputs

Inputs:

- a source project or a previously packed archive
- a source or target runtime
- the runtime references in `references/`

Outputs for `pack`:

- `manifest.yaml`
- `binding.json`
- optionally a portable snapshot zip

Outputs for `deploy`:

- a deploy plan for the target runtime
- after confirmation, direct target-runtime reconstruction in the workspace

## Core Concepts

- `soul`: the single core file that defines the agent's identity, behavior, and operating rules
- `skills`: skill directories the agent directly or indirectly uses in its working workflow
- `state`: non-soul, non-skill files the agent must keep in order to continue working as the same agent

## Classification Framework

Classify the project in this order:

1. Find the single `soul` file.
2. Read the `soul` carefully before proposing anything else.
3. Summarize the soul's core workflow in a few short bullets.
4. Classify every candidate file or directory by its relationship to that workflow.

Use these classes:

- `soul`: the one core file
- `skills`: skill directories the workflow directly calls, plus their indirect dependencies
- `state`: non-soul, non-skill files the workflow must read, reference, maintain, or preserve for future work
- `neither`: everything else

Use the relationship test when proposing `state` or `skills`:

- explain which part of the soul workflow points to this item
- explain whether the item is used directly by the soul or by a soul-invoked skill
- exclude items that only look important to the repository but are not part of the workflow

Only write items that currently exist into `manifest.yaml` and `binding.json`.
If the workflow clearly expects a future state file, directory, or skill that is not present yet, mention it during confirmation as an expected missing item, but do not write it into the current manifest.

Treat these as `neither` unless the workflow clearly depends on them:

- transient artifacts
- caches
- build outputs
- noisy logs
- one-off outputs
- direct deliverables of the current task
- skill inventory or skill products the agent manages but does not itself use

Templates, rule files, HTML/Markdown assets, and similar materials should count as `state` if the workflow needs them again in normal future work.

When the workflow refers to a local skill and it is not present in the expected project-local skill path, check the source runtime's global skill location before excluding it.

For Claude Code projects, that fallback is typically:

- `~/.claude/skills/<name>`

## Runtime References

Before reasoning about a runtime, read the matching reference:

- `references/codex.md`
- `references/claude-code.md`
- `references/openclaw.md`

Use references in both directions:

- during `pack`, to understand how the source runtime expresses the agent
- during `deploy`, to understand how the target runtime should express the agent

Do not reason about runtime-specific files without consulting the matching reference.

## Pack Workflow

### 1. Inspect

Inspect the current project in this order:

- the most likely source runtime
- the `soul`

Then read the `soul` and extract its core workflow before proposing:

- the `state`
- the `skills`

Source runtime selection rules:

- if one runtime is clearly dominant, say which one you inferred and why
- if multiple runtimes are present or confidence is low, do not guess
- instead, ask the user to choose the source runtime with a short single-choice confirmation
- keep the choices minimal: `codex`, `claude-code`, `openclaw`
- after the user chooses, read that runtime's reference before classifying `soul`, `state`, or `skills`

### 2. Confirm

Before writing files, present a short checklist:

- soul
- state
- skills

For each proposed state file or skill, briefly explain its relationship to the soul workflow.
If a workflow expects a missing future item, call it out separately as missing rather than writing it into the manifest.

Ask the user to confirm or correct it.

### 3. Write

After confirmation, write:

- `manifest.yaml`
- `binding.json`

Rules:

- `manifest.yaml` must be machine-independent
- `binding.json` must map manifest entries to real local paths
- manifest `path` values must be relative to the soul directory
- manifest `path` values must not escape the project root
- each skill entry must include `name`, `path`, and `description`

### 4. Optional Archive

After writing the files, always ask:

`Do you want me to generate a snapshot now?`

If yes:

- create the snapshot directly in the workspace
- include `state/` only if the user asked for it
- use normal file operations and zip creation as needed

## Deploy Workflow

### 1. Inspect

Inspect:

- the packed archive contents
- the manifest inside the packed archive
- the requested target runtime
- the requested target location, workspace, or agent id

### 2. Plan

Before executing anything, explain:

- where the soul will land
- where the state will land
- where the skills will land
- whether any target-runtime helper files must be synthesized, merged, or removed

Then ask the user to confirm the deploy plan.

Treat deploy as reconstruction, not field-by-field copying.

### 3. Execute

After confirmation, perform the reconstruction directly.

Default:

- create or rewrite the target runtime files directly
- preserve important supporting files
- synthesize thin runtime entrypoints when needed

Do not introduce an extra execution layer just to preserve an artificial boundary.

### 4. Verify

After deployment, tell the user how to verify:

- the soul file
- the state files
- the skill directories
