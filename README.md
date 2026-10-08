# Plan, Build, Review

[English](README.md) | [简体中文](README.zh-CN.md)

A role-based workflow for delivering meaningful scoped work, from features to
milestones, through
independent review, explicit approval, and task-level checkpoints — works
with both [Codex](https://developers.openai.com/codex/) and
[Claude Code](https://code.claude.com/).

## Why use it

- A planner creates an evidence-grounded, minimal plan; a separate reviewer
  must pass both its premises and specification.
- Nothing is implemented before explicit user approval.
- Work proceeds one task at a time through build, validation, review, and
  checkpoint commit.
- Every review pass uses a fresh reviewer and a risk-based charter, not a
  generic checklist.
- Validation follows the artifact's real consumer, and local and remote gates
  must have distinct purposes.
- A distinct integration review checks cumulative behavior when cross-task
  risk warrants it.
- Publication requires explicit approval, and the workflow never merges
  automatically.

```text
preflight → plan → review → approve → build task → review task → checkpoint
          → repeat → integration review when needed → publication approval
```

Each role receives a short decision summary plus the plan and evidence needed
for its assignment. The workflow does not use a whole plan as default
context for every handoff.

## What it prevents, and where it helps

This workflow prevents a plausible plan from being treated as a sufficient
reason to build. It exposes unchallenged product assumptions, scope expansion,
unjustified compatibility or security work, incomplete merge units, and tests
that accidentally depend on their development environment.

It is most useful when a feature or milestone crosses several components,
needs an owner decision before implementation, or must leave a trustworthy
review and release trail. Its value is not more agents or more ceremony: it
gives each role a distinct challenge, a small authoritative context, and a
clear escalation path while the main agent maintains the whole outcome.

## Roles

| Role | Responsibility | Access | Codex model · effort | Claude model · effort |
| --- | --- | --- | --- | --- |
| `pbr-explorer` | Bounded investigation | Read only | Terra · medium | Haiku |
| `pbr-planner` | Minimal, decision-complete plan | Read only | Sol · xhigh | Opus · xhigh |
| `pbr-builder` | One approved task | Workspace write | Sol · medium | Sonnet · high |
| `pbr-reviewer` | Premise, plan, task, and risk-based integration review | Read only | Sol · high | Opus · high |

The role definitions are bundled and can be tuned for either runtime. Haiku
doesn't support a configurable reasoning-effort level, so the Claude column
is left blank for `pbr-explorer`.

## Review model

Plan review has two required judgments. The premise review challenges the
purpose, target user, architecture, deployment model, compatibility posture,
security posture, and roadmap delta. The specification review checks whether
the chosen plan is complete and executable. A plan passes only when both pass.
Plan bytes pass mechanical hygiene checks before hashing; a proven byte-only
fix needs only focused re-review and a newly approved hash, while a semantic
revision returns to full review.

For implementation work, the main agent selects the smallest sufficient review
charter: for example, implementation behavior, a final public surface, or
cross-task integration. For work with asynchronous, concurrent, polling, or
other time-sensitive coordination, the plan states temporal invariants and
consumer-observable behavior; implementation and review then use a bounded
scenario matrix derived from them. A task must be a semantically complete unit
that can be merged and verified on its own. The workflow does not split work
mechanically by file type or code layer. Each verdict also states its charter,
reviewed surfaces, and coverage limits. One complete task review may also be
the final cumulative review when no distinct integration risk exists. A feature
normally keeps implementation, tests, current public descriptions, and
release-ready documentation in one PR; publication alone does not require a
closeout PR.

## Validation model

The workflow asks who consumes an artifact before choosing its checks.
Review human-consumed artifacts against owner intent and change-specific
claims, without generic structural or semantic policy. Use only
change-relevant mechanical checks. Give machine-consumed artifacts
proportionate structural and behavioral checks.

Local, PR, main, release, and scheduled gates serve different purposes. The
plan must explain equivalent coverage rather than repeat expensive checks by
default. Evidence remains reusable while the candidate and relevant inputs are
unchanged; reviewer independence does not require repeating the builder's full
suite. Source-only releases verify the tag, release metadata, and archive, and
add tag CI only for tag-specific behavior or artifacts.

## Install

### Codex

Clone the repository into the Codex skills directory using the internal skill
name:

```sh
git clone https://github.com/jameswei/plan-build-review.git \
  ~/.codex/skills/plan-build-review
```

Install the four custom agent definitions:

```sh
mkdir -p ~/.codex/agents
cp ~/.codex/skills/plan-build-review/assets/agents/*.toml ~/.codex/agents/
```

Review existing files before copying if you already have agents with the same
names. The `pbr-` prefix avoids collisions because Codex installs agents into a
flat global directory. Start a new Codex session after installation so the skill and agents are
discovered. (Codex also supports a project-scoped `.agents/skills` location
alongside the user-scoped `~/.codex/skills/` path used above, if you'd rather
install per-repository.)

### Claude Code

Requires Claude Code 2.1.287 or later, so each role runs at the reasoning
effort set in its agent definition.

Add this repository as a plugin marketplace, then install the plugin:

```text
/plugin marketplace add jameswei/plan-build-review
/plugin install plan-build-review@plan-build-review
/reload-plugins
```

This installs the skill and all four subagent definitions in one step, with
no manual file copying. Once installed, the skill and roles are namespaced
under the plugin name, e.g. `pbr-planner` becomes
`plan-build-review:pbr-planner`.

## Use

Invoke the skill explicitly:

```text
Use $plan-build-review to deliver feature export-history for this repository.
```

in Codex, or in Claude Code:

```text
Use /plan-build-review:plan-build-review to deliver feature export-history for this repository.
```

The workflow stops after the reviewed plan and waits for approval before it
creates a branch or edits implementation files. It also asks again before
pushing or opening a draft pull request.

Use it for scoped feature or milestone work where scope, correctness, and review traceability
matter. It intentionally does not trigger for ordinary coding or documentation
tasks.

## Compatibility

This package supports both Codex and Claude Code; no other runtime is
targeted. Installation isn't equally frictionless on both: Claude Code's
plugin marketplace installs the skill and all four subagents in one step,
with zero manual file copying. Codex has no mechanism to bundle custom agent
definitions with a skill install, so its custom agents still require the
one-time manual copy described above.

Read-only enforcement also isn't equivalent. Codex's `sandbox_mode:
read-only` blocks filesystem writes at the OS level even inside a shell
command. Claude Code has no equivalent execution sandbox, so
`pbr-explorer`, `pbr-planner`, and `pbr-reviewer` enforce
read-only access by omitting Bash and MCP tools from their `tools` allowlist
entirely, rather than sandboxing them — they simply aren't granted the
tools that could write.

## License

[MIT](LICENSE)
