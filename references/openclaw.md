# OpenClaw Runtime Reference

Use this reference when `packasoul` needs to understand or deploy an agent into OpenClaw.

The goal is not to rename files into an OpenClaw-looking tree.
The goal is to reconstruct an OpenClaw workspace whose startup behavior matches the packaged agent.

The controlling principle is simple:

Preserve the source agent's operating semantics first. Use OpenClaw's files as the expression layer for that behavior, not as a fixed mold that the source agent must be squeezed into.

## What Matters In OpenClaw

OpenClaw is not just a folder layout. It is a startup graph.

Some files are commonly injected into the runtime prompt.
Other files merely exist in the workspace and must be read later because the injected files tell the agent to read them.

That distinction is critical.

If deployment preserves the files but breaks the startup graph, the agent may keep its persona while forgetting its actual working memory.

This is why OpenClaw deploy must be based on semantic fit, not file renaming.

## Observed Runtime Shape

In a mature OpenClaw workspace, the runtime commonly treats these files as startup-facing workspace docs:

- `AGENTS.md`
- `SOUL.md`
- `TOOLS.md`
- `IDENTITY.md`
- `USER.md`
- `HEARTBEAT.md`
- sometimes `BOOTSTRAP.md`

Do not assume ordinary state files are auto-loaded just because they exist in the workspace.

Files such as:

- `profile.json`
- `log.md`
- `plan.md`
- `activities/*`
- `memory/*`

may need to remain ordinary files that are explicitly read through startup instructions.

This means OpenClaw deployment has two jobs:

1. place files in sensible locations
2. make sure the injected startup files point to the right non-injected files

## OpenClaw File Roles

### `SOUL.md`

Use `SOUL.md` for:

- durable identity
- stable behavioral principles
- non-negotiable role constraints
- the agent's long-lived center of gravity

`SOUL.md` should answer:

- who am I
- what kind of agent am I
- what values or style should remain stable across sessions

Do not turn `SOUL.md` into a dump of every remembered fact or every workflow file.

### `AGENTS.md`

Use `AGENTS.md` for:

- startup instructions
- working process
- execution discipline
- explicit file-read and file-write workflow
- how the runtime should traverse supporting state

`AGENTS.md` should answer:

- what should I read before replying
- what should I update after working
- what workspace rules override generic defaults

For OpenClaw, this is the most important bridge file.

If the packaged agent depends on ordinary files such as `profile.json`, `log.md`, or `activities/*`, `AGENTS.md` must make that dependency explicit unless another injected file already does so cleanly.

In other words, `AGENTS.md` is not just "process notes." It is often the bridge that preserves the original agent's file-usage logic inside OpenClaw.

### `IDENTITY.md`

Use `IDENTITY.md` for:

- name
- outward persona markers
- presentation details such as vibe, creature, emoji, or avatar style

Do not let an empty or generic `IDENTITY.md` outrank a mature packaged identity.

### `USER.md`

Use `USER.md` for:

- who the human is
- how to address them
- stable personal preferences
- recurring user-specific context

### `TOOLS.md`

Use `TOOLS.md` for:

- tool usage rules
- connectors and integrations
- operational boundaries
- local environment notes

### `MEMORY.md`

Use `MEMORY.md` for:

- curated long-term memory
- distilled context that should be reloaded as summarized memory

### `memory/`

Use `memory/` for:

- rolling or dated memory logs
- append-heavy history files that match OpenClaw's memory conventions

### Ordinary State Files

Not every important file should be converted into an OpenClaw-native filename.

These often remain ordinary files:

- `profile.json`
- `log.md`
- `plan.md`
- `activities/*`
- project-specific ledgers or reports

That is fine, as long as the startup graph makes the agent read them when needed.

Do not promote an ordinary file into an OpenClaw-native filename unless that improves semantic fit. Many source agents work best when these files stay exactly what they are.

### `skills/`

Deploy canonical skills into:

```text
<workspace>/skills/<name>/
```

Skills should not be the only place where critical memory or workflow rules live.

## Bootstrap Rule

`BOOTSTRAP.md` is dangerous in mature deploys.

If it still contains generic "you just woke up" or "there is no memory yet" language, it can directly contradict the packaged agent.

For mature deployed agents:

- remove `BOOTSTRAP.md`, or
- rewrite it to be harmless, or
- make it clearly irrelevant from the actual startup graph

Do not leave a generic bootstrap file active beside a mature packaged identity.

## Reconstruction Model

Think in two layers.

### Layer 1: file placement

Place state where it naturally belongs:

- `identity` -> `IDENTITY.md` or `SOUL.md`
- `user_profile` -> `USER.md` or ordinary profile file plus startup reference
- `tool_notes` -> `TOOLS.md`
- `memory_curated` -> `MEMORY.md`
- `memory_log` -> `memory/<...>` or ordinary log file if that exact file matters to the workflow
- `project_rules` -> `AGENTS.md`
- `plan` -> ordinary workspace file unless it is truly startup-critical
- `activity` -> ordinary workspace files
- `custom` -> keep the logical path unless OpenClaw has a clearly better native home

### Layer 2: startup graph

After placing files, decide:

- which files OpenClaw will likely inject automatically
- which files the agent still needs to read explicitly
- where those read instructions should live
- which stale files must be removed so they do not compete with the intended startup path

If a source agent's continuity depends on files like `profile.json`, `log.md`, and `activities/*`, deployment is incomplete unless an injected startup file tells the agent to read them.

The question is not "where can I place this file?"
The question is "how will the deployed agent continue to use this file the way the source agent used it?"

## Cross-Runtime Deploy Rules

When the source agent comes from a flatter runtime such as Claude Code or Codex:

- treat the canonical `soul` as source material, not as a file to rename blindly
- synthesize a compact `SOUL.md` for durable identity
- synthesize a compact `AGENTS.md` for workflow and file dependency instructions
- preserve supporting state in natural OpenClaw locations
- explicitly bridge any must-read ordinary files into the startup graph

When the source agent already has a strong file workflow, preserve that logic even if the filenames change.

If the source soul says, in effect, "before I answer, I read these files in this order," then the OpenClaw deploy is only correct if the new workspace still causes that behavior to happen.

What must survive is not just:

- identity
- state
- skills

It is also:

- startup reads
- ongoing reads
- write-back rules
- conflict resolution between startup-facing files

## Recommended Deploy Order

1. read the canonical `soul` and required state files
2. identify the agent's startup reads, working reads, writes, and skills
3. restore state files and skills into their OpenClaw locations
4. synthesize or preserve `SOUL.md` as the durable identity anchor
5. synthesize or preserve `AGENTS.md` as the startup and workflow bridge
6. place user, tool, identity, and memory state into native OpenClaw files where appropriate
7. keep project-specific state as ordinary files when that is the better fit
8. remove or neutralize stale bootstrap defaults and contradictory startup instructions

## What To Avoid

- do not rename a source entry file directly to `SOUL.md` and call that deployment
- do not assume every important state file is auto-loaded by OpenClaw
- do not assume that placing a file in the workspace means the agent will read it
- do not let `AGENTS.md`, `SOUL.md`, and `BOOTSTRAP.md` give conflicting startup instructions
- do not let a generic bootstrap file imply the agent has no memory when packaged state exists
- do not convert every ordinary state file into an OpenClaw-native file if that destroys the original workflow
- do not keep startup instructions that point to nonexistent files
- do not preserve the persona while dropping the source file-usage logic that actually makes the agent useful

## Verification Checklist

After deployment, check both structure and behavior.

Structural checks:

- are the expected skills present under `skills/`
- is `SOUL.md` a real identity anchor rather than a file rename
- is `AGENTS.md` a real startup and workflow bridge rather than a generic template
- are stale bootstrap defaults removed or neutralized
- do startup instructions point to files that actually exist

Behavioral checks:

- does the agent describe itself using the packaged role rather than generic OpenClaw language
- does it know prior user or project context it should know
- if asked what it reads before answering, does it name the expected files
- can it recall prior ideas or project history that live in preserved state files
- do normal OpenClaw conversations feel like the packaged agent rather than a blank assistant
