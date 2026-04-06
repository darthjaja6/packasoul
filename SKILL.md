---
name: packasoul
description: Packs an agent project into canonical agent semantics and an optional portable archive, or reconstructs that packed agent into a target runtime. Use when the task is to capture or reconstruct an agent across frameworks such as Claude Code, Codex, and OpenClaw.
---

# /packasoul

## Mission

Use this skill when the user wants to pack an agent project, inspect what belongs in a manifest, or deploy that packed agent into another runtime.

This skill owns the full pack and deploy workflow. The goal is not to move filenames around. The goal is to preserve how the agent actually works.

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

- `soul`: the single core file that most directly defines the agent's identity, behavior, and operating rules
- `state`: non-soul, non-skill files the agent must keep in order to continue working as the same agent
- `skills`: skill directories the agent directly or indirectly uses in its normal workflow
- `operating semantics`: the source agent's actual working logic, including how it uses files, what it reads before speaking, what it updates, and how it brings skills into the loop

These categories are containers, not the main point.

The main point is to preserve the source agent's operating semantics.

`state` is not just "important files in the repo." A file matters because the source agent's workflow depends on it. A deploy is correct only if the target runtime still makes the agent work through those dependencies in a coherent way.

## Classification Framework

Classify the project in this order:

1. identify the source runtime
2. read the matching runtime reference
3. find the single `soul` file
4. read the `soul` carefully before proposing anything else
5. extract the agent's working workflow
6. classify every candidate file or directory by its relationship to that workflow

Use these classes:

- `soul`: the one core file
- `state`: non-soul, non-skill files the workflow must read, reference, maintain, or preserve for future work
- `skills`: skill directories the workflow directly calls, plus their indirect dependencies
- `neither`: everything else

When extracting the workflow, capture these relationships explicitly:

- how the agent starts a conversation
- which files it relies on and why
- what it updates over time
- how and when it uses skills
- what ordering or priority matters inside that workflow
- which files the runtime auto-loads versus which files only work because the source instructions tell the agent to read them

Use the relationship test when proposing `state` or `skills`:

- explain which part of the workflow points to this item
- explain whether the item is runtime-loaded, explicitly read later, or only written during maintenance
- explain whether the item is used directly by the soul or by a soul-invoked skill
- exclude items that only look important to the repository but are not part of the working loop

Do not force the source workflow into a rigid predeclared schema. If the source agent has a distinctive way of sequencing reads, consulting notes, or updating memory, preserve that logic in plain language first and only then classify the files.

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

Templates, rule files, HTML or Markdown assets, and similar materials should count as `state` if the workflow needs them again in normal future work.

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
- during `deploy`, to understand how the target runtime actually needs to express the agent

Do not reason about runtime-specific files without consulting the matching reference.

## Pack Workflow

### 1. Inspect

Source runtime selection rules:

- if one runtime is clearly dominant, say which one you inferred and why
- if multiple runtimes are present or confidence is low, do not guess
- instead, ask the user to choose the source runtime with a short single-choice confirmation
- keep the choices minimal: `codex`, `claude-code`, `openclaw`
- after the user chooses, read that runtime's reference before classifying `soul`, `state`, or `skills`

Then inspect in this order:

- the runtime reference
- the `soul`
- the files and directories the `soul` depends on

Before proposing `state` or `skills`, summarize the workflow in a few short bullets:

- how the agent starts work
- which files are indispensable and why
- what it writes back over time
- how skills fit into the loop
- what would break if those relationships were lost

### 2. Confirm

Before writing files, present a short checklist:

- soul
- operating semantics
- state
- skills

For each proposed state file or skill, briefly explain its relationship to the workflow.
Call out especially:

- required startup files
- runtime-loaded files
- required files that are not auto-loaded by the runtime
- expected missing items

The user should be able to see not just which files were selected, but what source-runtime behavior those files are preserving.

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

If the user wants a portable artifact for transfer, sharing, or backup:

- create the snapshot directly in the workspace
- include `state/` only if the user asked for it
- use normal file operations and zip creation as needed

Do not treat snapshot generation as mandatory if the user only asked to pack and deploy in place.

## Deploy Workflow

### 1. Inspect

Inspect:

- the packed archive contents
- the manifest inside the packed archive
- the requested target runtime
- the requested target location, workspace, or agent id

Then read the target runtime reference and reconstruct the target runtime's actual startup model:

- which files are automatically loaded
- which files must be referenced from those startup files
- which defaults may conflict with the packaged agent
- which state files must stay as ordinary files but still be explicitly read during normal work

### 2. Plan

Before executing anything, explain:

- which target files will become the runtime entrypoints
- where the canonical soul material will land
- where the state will land
- where the skills will land
- how the source operating semantics will be preserved in the target runtime
- whether any target-runtime helper files must be synthesized, merged, downgraded, or removed
- whether any stale or conflicting files must be neutralized

Then ask the user to confirm the deploy plan.

Treat deploy as reconstruction, not field-by-field copying.

### 3. Execute

After confirmation, perform the reconstruction directly.

Default:

- create or rewrite the target runtime files directly
- preserve important supporting files
- synthesize runtime-native entrypoints when needed
- preserve the source agent's operating semantics, not just the file inventory
- remove or neutralize files that would contradict the packaged agent's identity or workflow in the target runtime

Do not introduce an extra execution layer just to preserve an artificial boundary.

### 4. Verify

After deployment, tell the user how to verify:

- the identity entrypoint
- the workflow entrypoint
- the required state files
- the skill directories
- the working-memory behavior

Verification should include live-behavior checks whenever practical. For example:

- can the agent describe itself correctly
- does it know prior user or project context it should know
- does it name the expected skills
- does it reference the expected state files when asked how it works
- does it behave as though the original file-dependent workflow actually survived
