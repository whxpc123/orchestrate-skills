# Workflow Contract

Use this reference to render plans consistently and preserve approval state during execution.

## Contents

- [Render order](#render-order)
- [Plan schema](#plan-schema)
- [Ordered step table](#ordered-step-table)
- [Candidate evidence](#candidate-evidence)
- [Approval rules](#approval-rules)
- [Execution ledger](#execution-ledger)
- [Recovery matrix](#recovery-matrix)
- [WeChat article example](#wechat-article-example)

## Render order

Present every plan in this order:

1. Goal, deliverables, and completion criteria
2. Assumptions and unresolved choices
3. Ordered step table
4. Detailed steps
5. Installation manifest and candidate evidence
6. Unresolved gaps and alternatives
7. Approval scope
8. Exact approval prompt

Keep the summary readable, but do not omit contract fields.

## Plan schema

```yaml
workflow_id: goal-derived-stable-id
revision: 1
state: awaiting-approval
goal: One outcome-focused sentence
deliverables:
  - Inspectable final artifact
completion_criteria:
  - Verifiable condition
assumptions:
  - Explicit assumption

steps:
  - id: step-01
    state: pending
    task: One bounded task
    skill: skill-name | built-in-general-capability | unresolved
    skill_status: installed | to-install | built-in-fallback | unresolved
    source: local | skills.sh | built-in
    source_link: URL | local-path | null
    reason: Why this choice fits the task and output
    depends_on: []
    inputs:
      - Named input
    outputs:
      - Named output
    validation: Observable pass condition
    side_effects: none | local-write | external-write | high-impact
    execution: sequential | parallel-eligible
    fallback: approved-skill-name | built-in-general-capability | null

install_manifest:
  - package: owner/repo@skill
    skill: skill-name
    source_link: https://skills.sh/owner/repo/skill-name
    evidence:
      installs: 1200
      source_reputation: concise assessment
      github_stars: 250
      security_audits: named audits | unknown
      fit: concise functional-fit assessment
    command: npx skills add owner/repo@skill -g -y

unresolved:
  - step: step-id
    reason: Why no reliable option exists
    alternatives:
      - built-in or create-Skill option

approval_scope:
  installs:
    - Exact packages authorized by confirmation
  local_actions:
    - Non-destructive local steps authorized by confirmation
  excluded_high_impact_actions:
    - Actions requiring separate just-in-time approval
approval_prompt: Confirm execution of workflow_id revision-N, including the listed installs and local actions?
```

Use `revision: 1` for the first displayed plan. Increment it whenever any step, Skill, install, fallback, validation, side effect, deliverable, or completion criterion changes.

## Ordered step table

Render a compact overview before detailed YAML:

| Order | Task | Skill | Status | Depends on | Output | Side effect |
|---|---|---|---|---|---|---|
| 1 | Bounded task | `skill-name` | installed | none | Artifact | none |

Use the detailed schema after the table when fields would otherwise be ambiguous.

## Candidate evidence

For each external candidate, record:

- Functional fit based on its documentation
- Install count from skills.sh or the Skills CLI
- Author or organization reputation
- GitHub star count
- Available security-audit results
- Direct Skills page and repository links

Prefer at least 1,000 installs and 100 GitHub stars. A lower-count candidate can be proposed only when fit is strong, risk is low, the weakness is explicit, and the user approves it. Reject a popular but domain-mismatched Skill.

## Approval rules

- Planning may read local metadata, browse, search, and evaluate candidates.
- Planning may not install or execute candidate Skills.
- Approval must identify the current workflow and revision unmistakably.
- Approval covers only exact manifest packages and listed non-destructive local actions.
- Require another plan revision for unlisted installs, Skill substitutions, new steps, broader outputs, or new side effects.
- Require just-in-time approval for authentication, secrets, payment, public publishing, sending messages, production changes, deletion, or irreversible overwrite even when the step appears in the plan.

## Execution ledger

Use this compact ledger during execution:

```yaml
workflow_id: stable-id
revision: 1
state: approved | running | completed | blocked | replan-required
approved_at: timestamp-or-conversation-reference
installed:
  - package@skill
steps:
  step-01:
    state: pending | ready | running | validating | completed | failed | skipped
    skill: skill-name
    inputs: []
    outputs: []
    validation_result: not-run | passed | failed
    attempts: 0
    external_effect_record: none | exact-effect-description
blockers: []
```

Keep the ledger in the conversation for short workflows. For a long workflow, write it to a working artifact when file access is available. Update it after installation, each state transition, each validation, and each approval.

## Recovery matrix

| Event | Allowed response | Re-plan required |
|---|---|---|
| Install identifier or network failure | Verify and retry once | After retry fails |
| Skill execution failure | Correct input and retry once | If still failing |
| Validation failure | Revise current output once | If still failing |
| Approved fallback needed | Use named fallback | No |
| Unlisted fallback or Skill needed | Stop | Yes |
| New side effect discovered | Stop | Yes |
| Ambiguous external failure | Do not retry | Ask for inspection or direction |
| User interruption | Resume after last validated step | Only if scope changed |

## WeChat article example

Use actual runtime discovery rather than hard-coding these illustrative Skill names.

| Order | Task | Candidate type | Output | Validation | Side effect |
|---|---|---|---|---|---|
| 1 | Define audience and topic | Content strategy Skill | Topic brief | Audience, angle, and promise are explicit | none |
| 2 | Research claims and sources | Research Skill | Source notes | Each material claim has a reliable source | none |
| 3 | Draft the article | WeChat or article-writing Skill | Markdown draft | Structure matches the brief | local-write |
| 4 | Edit and proofread | Editing and humanization Skills | Final copy | No known factual or language defects | local-write |
| 5 | Create cover and illustrations | Image Skills | Image assets | Required dimensions and topics match | local-write |
| 6 | Convert formatting | WeChat HTML Skill | Compatible HTML | Preview preserves headings, images, and links | local-write |
| 7 | Publish | WeChat publishing Skill | Published post | Publish only after separate high-impact approval | high-impact |

Example approval scope:

```yaml
approval_scope:
  installs:
    - Only packages explicitly shown in the real resolved manifest
  local_actions:
    - Research, drafting, editing, image creation, and local HTML conversion
  excluded_high_impact_actions:
    - WeChat login
    - Credential entry
    - Public publication
approval_prompt: Confirm execution of wechat-article revision-1, excluding login and publication until separately approved?
```
