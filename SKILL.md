---
name: plan-build-review
description: Plan and execute explicitly requested scoped work, from a feature to a milestone, through independent plan review, user approval, task-level build and review checkpoints, integration review, and controlled publication. Use only when the user explicitly invokes this skill by name (for example, $plan-build-review in Codex or /plan-build-review:plan-build-review in Claude Code) or clearly asks for this workflow in other words; do not invoke it for ordinary coding, review, or documentation tasks.
---

# Plan, Build, Review

Coordinate scoped work without mixing planning, implementation, and review.
The main agent owns scope, sequencing, global coherence, user communication,
and approval gates. It must not outsource product, architecture, or complexity
judgment to role verdicts.

Spawn every named role with a fresh, isolated context and a minimal,
authoritative task packet. Self-contained does not mean copying the full plan
or parent conversation.

- In Codex, always call `spawn_agent` with `fork_turns="none"` — the default
  is `fork_turns="all"`, which inherits the entire parent conversation.
- In Claude Code, use a normal non-fork custom subagent invocation. Each
  role's reasoning effort comes from its agent definition; do not pass the
  Agent tool's `effort` parameter.

Do not rely on or inherit parent conversation history. Give each role a short
decision summary: project purpose, target user and deployment model, owner
decisions, superseded requirements, explicit exclusions, relevant roadmap
delta, and the current approval or release state. Add only the plan slice and
evidence needed for the role's assigned judgment. If this summary conflicts
with the source plan or repository evidence, stop and escalate.

## 1. Preflight

1. Read applicable repository instructions and status documents.
2. Inspect the base branch, working tree, relevant code, tests, and public
   contracts.
3. Preserve unrelated changes. Stop if they prevent safe work.
4. Identify a lowercase hyphenated scope slug. Record a version only when the
   work is a versioned milestone or release.

## 2. Plan and Review

1. Spawn `pbr-explorer` only for a bounded investigation that materially
   improves planning.
2. Spawn `pbr-planner` with the request, decision summary, repository
   instructions, preflight evidence, and explorer findings. Require a
   decision-complete plan with ordered, reviewable tasks. The plan must state
   which material requirements come from the owner and which are inferred;
   compare its scope with the roadmap; and explicitly flag any new public
   contract, compatibility promise, deployment model, or security posture for
   owner decision before the plan can pass. Choose validation by actual
   consumer. Review human-consumed artifacts against owner intent and
   change-specific claims without generic structural or semantic policy. Give
   machine-consumed artifacts proportionate structural and behavioral checks.
   When work includes asynchronous, concurrent, polling, or other time-sensitive
   coordination, state its temporal invariants and consumer-observable behavior.
   Give local, PR, main, release, and scheduled validation distinct purposes;
   justify equivalent coverage.
3. Reconcile the draft with repository evidence in the main thread.
4. Before hashing or freezing the draft, verify expected encoding and line
   endings, a final newline, no trailing whitespace, and diff hygiene. Hash
   only after all checks pass.
5. Spawn a fresh `pbr-reviewer` with the decision summary, relevant
   evidence, and draft. Require one leading verdict: `PASS`,
   `CHANGES_REQUIRED`, or `BLOCKED`. The reviewer must make two separate
   judgments: a premise review (purpose, user, architecture, deployment,
   compatibility, security, and roadmap delta) and a specification review
   (scope, dependencies, actionability, acceptance, and validation). `PASS`
   requires both judgments to pass.
6. Return accepted blocking findings to the same planner, then use a fresh
   reviewer. Ask the user when a material decision or missing evidence blocks
   planning, or the same blocker survives two review cycles.
7. Present only a passed plan, its premise summary, assumptions, and
   non-blocking notes. Stop for explicit user approval.

Do not create a branch, save the plan, or implement before approval. Semantic
revisions return to full review and approval. A proven byte-only correction may
use focused independent review: rerun byte-hygiene checks, compute and approve
a new hash, reuse still-valid evidence, and do not reconstruct history solely
to make the corrected bytes its first commit.

## 3. Establish the Approved Work

1. Recheck the branch and working tree.
2. Create a branch and plan path that follow repository conventions. By
   default, use `work/<slug>` and `docs/plans/<slug>.md`; for a versioned
   milestone, `milestone/<version>-<slug>` and
   `docs/plans/<version>-<slug>.md` remain suitable examples.
3. Save the approved plan and commit it before implementation.
4. Keep the plan, task checkpoints, and release-ready documentation inside the
   semantic work branch and PR. Avoid plan-only, checkpoint-only, or closeout
   PRs unless independently valuable and mergeable.
5. Create a task ledger from the approved tasks. Track `pending`, `building`,
   `reviewing`, `accepted`, or `blocked`, plus checkpoint and validation
   evidence.

Keep approved plan bytes immutable except through that byte-only correction
path. Record progress in the ledger and commits.

## 4. Execute One Task at a Time

Do not begin a later task until the current task passes review, validation, and
has a stable checkpoint commit.

For each task:

1. Mark it `building` and record its review base.
2. Spawn a fresh `pbr-builder` with the decision summary, the approved
   plan slice for one task, repository instructions, review base,
   accepted-task summaries, working-tree state, and validation commands. When
   the approved task has temporal risk, the builder must derive and exercise a
   bounded temporal scenario matrix from the approved invariants and real
   consumer interactions. The builder must stop and report rather than
   mechanically implement when a task needs a new premise, crosses an unplanned
   abstraction boundary, creates unexpected producer-consumer coupling, depends
   on incidental environment state, adds validation that matches no actual
   consumer or distinct purpose, or cannot leave the branch correct on its own.
3. Inspect the returned delta and evidence. A task or PR is the smallest
   semantically complete, independently mergeable, independently verifiable
   unit; do not split only by file type, code layer, or a fixed template. Split
   or combine work only when the resulting units each preserve a clear
   invariant and do not rely on a future task to repair a known incomplete
   state. Otherwise return to planning and approval. A feature normally uses
   one PR for implementation, tests, current public descriptions, and
   release-ready documentation. Treat post-merge defects as new corrective
   work.
4. Mark it `reviewing`. Select the smallest sufficient review charter for the
   change: implementation, final surface, integration, or another explicitly
   stated risk-based charter. For temporal risk, the charter must challenge the
   bounded scenario matrix against the approved invariants and
   consumer-observable behavior. Spawn a fresh `pbr-reviewer` with the
   decision summary, approved plan slice, task, review-base delta, repository
   instructions, accepted-task summaries, validation evidence, selected
   charter, and what that review is not intended to prove.
5. On `CHANGES_REQUIRED`, send accepted blocking findings to the same builder,
   rerun validation, and use a fresh reviewer. On `BLOCKED`, obtain the missing
   evidence, capability, or user decision. Stop when the same blocker survives
   two review cycles.
6. After `PASS`, confirm the reviewed delta and validation inputs are unchanged.
   Reuse evidence with available command, result, provenance, and applicability.
   Rerun only invalidated or charter-specific checks, and review changed tracked
   output.
7. Commit the accepted delta with its task ID, mark it `accepted`, and report
   the checkpoint and evidence.

A material change invalidates `PASS`. A bounded finding fix that changes no
scope, premise, or charter needs targeted validation and explicit closure
against the new head; otherwise use a fresh reviewer.

Reviewer independence means independent judgment, not automatic repetition of
the builder's complete validation suite.

## 5. Integration Review

1. After all tasks are accepted, decide whether a distinct integration gate
   has a purpose not already covered by task review.
2. For multiple tasks or real cross-task, cross-component, or cumulative risk,
   run the relevant cumulative validation and spawn a fresh `pbr-reviewer`
   with the request, approved plan, ledger, checkpoints, verdicts, evidence,
   repository instructions, and full branch diff.
3. For one task with no distinct integration risk, let its final task review
   serve as cumulative review only when the charter explicitly covers the
   complete base-to-head acceptance criteria. Do not repeat the same review
   under a second label.
4. Reuse applicable validation evidence. Run additional checks only for the
   cumulative risk this gate is intended to cover.
5. Treat required fixes as a narrow task: build, validate, review, and
   checkpoint it before repeating integration review.

The approved work is complete only after a cumulative `PASS`, whether supplied
by the final task review or a distinct integration review.

## 6. Visibility

Use the native plan display when available. Show setup, each task, integration
review, and publication; keep it synchronized with the ledger.

Keep internal role and ledger detail available for coordination, but report to
the human at each meaningful state change: current state, what happened, actual
impact, next expected event, and whether a decision is needed. Distinguish a
product or data correctness issue from a merge-gate failure and a workflow
issue. Include task ID, role, and review pass in internal task names. During
long operations, post a brief human-readable heartbeat at least once per
minute.

## 7. Publication Gate

Before requesting publication approval, confirm the required cumulative review
passed, validation is green, the branch contains only accepted work, and PR or
release prerequisites are available.

Keep release-ready repository documentation durable enough that publication
state can be owned by the release platform. Publication alone does not justify
a follow-up closeout PR.

At release, validate the delivered object instead of repeating earlier gates.
For a source-only release, verify the tag target, release metadata, and source
archive. Run tag CI only for tag-specific execution, assets, signing, or other
new coverage.

Summarize the outcome, checkpoints, tests, final verdict, non-blocking notes,
and working-tree state. Ask before pushing or opening a draft pull request.
Never merge automatically.
