---
name: orchestrate-skills
description: Decompose a multi-step goal into an approved workflow of installed or discoverable Skills, find missing Skills, present ordered dependencies and an installation manifest, then install and execute only after confirmation. Use when a user asks to combine, chain, select, or sequence multiple Skills; asks which Skills are needed for a goal; or requests plan-first execution of a reusable multi-step workflow.
---

# Orchestrate Skills

Turn a goal into a verified Skill workflow. Separate planning from execution and preserve the user's approval boundary.

## Operating contract

Use two phases:

1. **Plan phase:** clarify, decompose, resolve Skills, and present a complete workflow. Do not install or execute candidate Skills.
2. **Execution phase:** begin only after the user explicitly approves the frozen workflow revision. Install and execute only what that revision authorizes.

Read [references/workflow-contract.md](references/workflow-contract.md) when rendering a plan, recording execution state, or handling the WeChat example.

## Plan phase

### 1. Frame the goal

Extract:

- Goal and final deliverables
- Completion criteria
- Constraints, supplied inputs, and assumptions
- Permitted local and external side effects
- Choices that materially change the workflow

Ask one concise question at a time only when the answer changes task boundaries, Skill selection, approval scope, or the final deliverable. Otherwise state reasonable assumptions and continue.

### 2. Decompose the work

Create bounded steps that each produce an inspectable output. Declare dependencies, required inputs, expected outputs, validation, and side effects. Mark a step parallel-eligible only when it has no unmet dependency, shared mutable output, or conflicting external effect.

Avoid both monolithic steps and coordination-heavy micro-steps. Every required final criterion must map to at least one step validation.

### 3. Resolve a Skill for each step

Use this order:

1. Inspect the active or installed Skill catalog and match on description, triggers, and task fit.
2. Use `find-skills` when it is available and follow its current instructions.
3. If `find-skills` is unavailable, use the embedded discovery fallback below.
4. Prefer built-in general capability over an untrusted or poorly matched external Skill.
5. Mark the step unresolved when neither a Skill nor built-in capability can meet its validation.

Do not choose by name similarity alone. Explain why the selected Skill fits the specific task and output.

#### Embedded discovery fallback

When `find-skills` is unavailable:

1. Generate specific English and Chinese queries from the task and expected output.
2. Check the skills.sh leaderboard for strong known candidates.
3. Run `npx skills find "<query>"`; try alternative terms when results are weak.
4. Open the candidate documentation and verify functional fit.
5. Record install count, source reputation, GitHub stars, and available security audits.
6. Prefer candidates with at least 1,000 installs and repositories with at least 100 GitHub stars. Treat these as signals, not automatic acceptance.
7. Reject candidates aimed at a different domain even when their names match.
8. If network, browsing, or CLI access is unavailable, report the search as unresolved and never invent candidates or evidence.

### 4. Render the workflow

Follow the contract reference exactly. Include:

- Goal, deliverables, completion criteria, and assumptions
- Ordered steps and dependencies
- Skill name, source, installed state, and rationale for every step
- Inputs, outputs, validation, execution mode, side effects, and approved fallback
- `install_manifest` with package, install command, evidence, and direct source link
- Unresolved items and explicit alternatives
- Exact approval scope and approval prompt

Assign a stable `workflow_id` and increment `revision` whenever an approved field changes. End in `awaiting-approval` and stop. Design approval, general positive feedback, or discussion does not approve execution.

## Approval boundary

Accept execution approval only when the user unmistakably confirms the displayed workflow ID and revision. Confirmation authorizes:

- Packages listed verbatim in that revision's `install_manifest`
- Non-destructive local execution of that revision's steps

Require separate just-in-time approval for every high-impact action, including authentication, secret entry, payment, public publishing, message sending, production modification, deletion, or irreversible overwrite. Plan approval never implies high-impact approval.

Set the workflow to `replan-required` and request revised approval when execution needs an unlisted install, a materially different Skill, a new step, a wider goal, or a new external effect.

## Execution phase

### 1. Verify approval and prerequisites

Confirm that the approval matches the current `workflow_id` and `revision`. Refuse to execute a stale or ambiguous revision. Verify required user inputs and local prerequisites.

### 2. Install the approved manifest

Install only packages in `install_manifest`, using the recorded command. Verify that each new Skill is discoverable before continuing. On failure, verify the identifier and connectivity, retry once, then stop and propose a revised plan.

### 3. Execute by dependency order

Move ready steps through `running` and `validating`. Before each step, state which Skill is being used and what inputs it receives. Follow that Skill's instructions faithfully. Capture the output in the execution ledger and pass only validated outputs to dependent steps.

Run parallel-eligible steps concurrently only when safe. Pause immediately at a high-impact gate.

### 4. Validate and recover

- On execution failure, correct the input and retry once.
- Use a fallback only when the approved step names it.
- On validation failure, revise the current output once and validate again.
- Do not advance invalid output.
- Do not automatically retry an ambiguous external action; it may already have succeeded.
- Resume interrupted work after the last validated step without repeating completed external effects.
- Enter `replan-required` whenever recovery exceeds the approved workflow.

### 5. Complete the workflow

Mark `completed` only when every required step and final completion criterion passes. Report:

- Final deliverables and their locations
- Completed, skipped, blocked, or revised steps
- Newly installed Skills
- Validation results
- Unresolved warnings
- External actions deliberately left unperformed

Never claim completion from installation success or intermediate output alone.
