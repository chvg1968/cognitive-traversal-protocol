# Cognitive Traversal Protocol v1.0

CTP is a context-first protocol for LLM-assisted intervention.

Its goal is simple: make the agent understand just enough, preserve what matters, declare impact, change minimally, and verify honestly.

## Principles

### 1. The Current Source Wins

External references can orient the agent, but they do not prove current behavior.

Examples of current sources:

- The artifact being edited
- Current tests or validators
- Current configuration
- Current data samples
- Current contracts
- Current user instructions

### 2. Unknown Impact Blocks Action

If the agent suspects an affected area but cannot classify it as direct or indirect, the intervention must stop.

The agent can continue only after it:

- Reads more evidence
- Asks a targeted question
- Narrows the change
- Or reports that the task is blocked

### 3. Anchors Are Working Memory

An anchor compresses evidence into reusable state. It prevents the agent from repeatedly re-deriving context or drifting between sections.

Every anchor should include:

- Summary
- Invariants
- Dependencies
- Risks
- Contradictions
- Open questions
- Sources sampled
- Sources skipped intentionally

### 4. Minimal Intervention Is A Feature

The correct intervention is the smallest change that satisfies the task while preserving the relevant invariants.

Avoid:

- Hidden contract changes
- Opportunistic refactors
- Unverified assumptions
- Scope expansion
- Reporting certainty without verification

## Priority System

Use priority to decide what blocks work.

| Priority | Meaning | Effect |
| --- | --- | --- |
| critical | Breaks security, data integrity, irreversible state, or a core invariant | Stop until resolved |
| important | Breaks workflow, public contract, correctness, or user expectation | Must be handled before completion |
| soft | Local, cosmetic, or low-risk issue | Report or continue |

Priority is not the same as severity. Priority is the order in which a human should care.

## Standard Flow

```text
TRIAGE
  -> ANCHOR selected sections
  -> IMPACT_MAP
  -> intervention
  -> post-modification check
  -> final findings
```

## TRIAGE

The first formal block. It sets scope and prevents uncontrolled exploration.

```yaml
TRIAGE:
  context_size: small | medium | large
  complexity: low | medium | high
  stack: [languages, frameworks, tools, or artifact types]
  core_purpose: "one verifiable sentence"
  task_scope: narrow | broad | full-audit
  external_refs_consulted: yes | no
  external_refs_sampled: [sources] | []
  external_refs_confidence: low | medium | high
  source_verification_required: yes | no
  required_sections: ["identity", "domain", "ux", "quality"]
  estimated_sources_to_read: 3
  rationale: "why this scope is enough"
```

## Anchor Format

```yaml
ANCHOR[section-name]:
  summary: "2 to 4 lines about what was observed"

  invariants:
    - text: "property that must be preserved"
      priority: critical | important | soft

  dependencies:
    - "source -> consumer"

  risks:
    - text: "specific risk"
      priority: critical | important | soft
      severity: low | med | high

  contradictions:
    - against: "ANCHOR[previous-section]"
      text: "previous finding vs current finding"
      priority: critical | important | soft
    # or: none

  open_questions:
    - "question that still matters"
    # or: none

  sources_sampled: ["verified sources"]
  sources_skipped_intentionally: ["sources or patterns"]
```

## Impact Map

The impact map is mandatory before intervention.

```yaml
IMPACT_MAP:
  change_summary: "what will change"
  direct:
    - "artifact / module / contract directly modified"
  indirect:
    - "behavior or consumer affected through dependency"
  unknown:
    - "suspected area that cannot yet be classified"
  affected_invariants:
    - text: "affected invariant"
      priority: critical | important | soft
      from_anchor: "ANCHOR[section-name]"
  minimal_modification_justification: "why this change is enough"
```

If `unknown` contains anything, do not intervene.

## Post-Modification Check

After intervention, the agent should verify:

1. Relevant invariants still hold.
2. No unresolved critical or important contradiction was introduced.
3. Dependencies still point in the expected direction.
4. The change did not mutate unrelated contracts.
5. Verification was run or limitations were clearly stated.
6. Remaining risks were reported.

## Final Report Shape

```yaml
Critical findings:
  - none | finding

Important findings:
  - none | finding

Changes made:
  - artifact: concrete change

Verification:
  - command, check, or review performed
  - limitation if any check could not run

Recommended next actions:
  - short concrete action
```

## Anti-Patterns

- Acting before TRIAGE
- Reading everything without a reason
- Treating stale docs as proof
- Editing while `unknown` is non-empty
- Hiding behavior changes inside refactors
- Reporting verification that was not performed
- Creating abstractions that do not reduce real complexity
