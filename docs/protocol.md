# Cognitive Traversal Protocol — Layer 1 Specification

> Version: 0.4.0
> Scope: session-local discipline (TRIAGE, ANCHOR, IMPACT_MAP, verification)
> Companion: [`knowledge-layer.md`](knowledge-layer.md) — Layer 2 specification

CTP is a two-layer, context-first protocol for LLM-assisted intervention.

This document specifies **Layer 1** — the discipline that runs inside a single agent session: understand just enough, preserve what matters, declare impact, change minimally, and verify honestly.

Layer 2 — the persistent knowledge substrate that provides cross-session memory and mechanical anti-drift — is specified in [`knowledge-layer.md`](knowledge-layer.md). The two layers are complementary; Layer 1 alone is sufficient for one-off tasks, both are required for sustained work on real codebases.

## Principles

### 1. The Current Source Wins

External references can orient the agent, but they do not prove current behavior.

Precedence is operational: **`code > spec > knowledge layer`**. The knowledge layer (Layer 2) and any other external reference is **navigation, not authority**. See [`knowledge-layer.md`](knowledge-layer.md) for the full Pattern A vs Pattern B distinction.

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

## Depth Decision

CTP does not reward reading everything. It rewards reading the minimum needed to protect real contracts.

| Situation | Read first | Do not |
| --- | --- | --- |
| Small visual or narrative change | Affected artifact, style guide, channel constraints | Audit unrelated areas |
| Error with direct evidence | Error message, indicated file or log, minimal reproduction | Infer cause from names or assumptions |
| Contract, promise, or interface change | Current definition, consumers, examples, tests, validators | Edit only the provider of the change |
| Data, identity, permissions, or compliance | Data model, access rules, authority sources, critical history | Continue with critical open questions |
| Cross-cutting restructure | Inputs, outputs, shared modules, invariants, quality criteria | Mix cleanup with hidden behavior changes |

**Heuristic:** if a conclusion appears only in a secondary source (wiki, ticket, note), it is a hint. If it appears in the current artifact, tests, data, configuration, or authoritative documentation, it can enter the anchor. If it appears nowhere verifiable, it remains an open question.

**Reading budget:** cap reading at the smaller of 15 sources or 3,000 lines per section. Sample representative artifacts — entry points, contracts, schemas, interfaces — and record what was sampled vs. skipped intentionally.

## Section Model

Each section of a traversal runs four sub-steps in order:

1. **Observe** — read only what is needed to characterize the section.
2. **Compress** — turn reading into an anchor: invariants, dependencies, risks, contradictions, open questions.
3. **Consolidate** — cross-check against prior anchors. A critical conflict stops the work until resolved.
4. **Advance** — proceed to the next section using the anchor, not a re-derived intuition.

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

## Intervention Principles

The correct intervention is the smallest change that satisfies the task and respects the consolidated anchors.

- **Preserve contracts** — do not change signatures, schemas, routes, permissions, or public semantics as a side effect. If you must, declare it as a direct change in the IMPACT_MAP.
- **Respect the local pattern** — use existing patterns from the system before inventing abstractions. Coherence is worth more than novelty.
- **Do not hide uncertainty** — if an unexpected dependency appears during intervention, pause and update the IMPACT_MAP. Do not keep acting with unresolved impact.
- **Separate refactor from behavior** — avoid mixing structural cleanup with changes in meaning. If they must mix, explain why it was necessary.

**Operational close checklist:**

1. The change matches the declared scope.
2. No unknowns remain in the IMPACT_MAP.
3. Critical invariants remain intact.
4. Dependencies still point in the same direction.
5. The solution adds no complexity without a clear benefit.

## Post-Modification Check

After intervention, the agent should verify:

1. Relevant invariants still hold.
2. No unresolved critical or important contradiction was introduced.
3. Dependencies still point in the expected direction.
4. The change did not mutate unrelated contracts.
5. Verification was run or limitations were clearly stated.
6. Remaining risks were reported.

**Verification by change type:**

| Change type | Minimum verification | Report if missing |
| --- | --- | --- |
| Document or static page | Structure inspection, full read-through, internal links | No viewer or validator available |
| Application or automation | Lint, typecheck, relevant tests, local run if applicable | Dependencies not installed, blocked environment |
| Contract, API, or integration | Expected input/output, consumers, errors, compatibility | External services unavailable |
| Data, models, or business rules | Verifiable sample, rollback, compatibility with current source | Data source inaccessible or insufficient sample |

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
- Using the Layer 2 knowledge layer as authority instead of navigation (see [`knowledge-layer.md`](knowledge-layer.md) Pattern B)
- Auto-updating the knowledge layer from agent edits without explicit human authorization

## Layer 2 — Where To Go Next

This document covers Layer 1 only. For sustained work on a real codebase, also read:

- [`knowledge-layer.md`](knowledge-layer.md) — full normative spec for the persistent knowledge layer, including the external-by-design architecture, the Obsidian + MCP canonical implementation, the typed YAML schema, the `linked_code_entities` mechanical anti-drift mechanism, and the precedence/update rules.

Layer 1 prevents drift inside a session. Layer 2 prevents drift between sessions. Both layers are part of the protocol.
