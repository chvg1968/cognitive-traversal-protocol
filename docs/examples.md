# CTP Examples

These examples are intentionally compact. They show the shape of the protocol, not every possible detail.

## Example 1: Static Page Copy Update

```yaml
TRIAGE:
  context_size: small
  complexity: low
  stack: [HTML, CSS]
  core_purpose: "Static page explaining a protocol"
  task_scope: narrow
  external_refs_consulted: no
  external_refs_sampled: []
  external_refs_confidence: low
  source_verification_required: yes
  required_sections: ["identity", "ux", "quality"]
  estimated_sources_to_read: 1
  rationale: "Only visible copy is changing."
```

```yaml
ANCHOR[ux]:
  summary: "The page is a single static artifact. The change should preserve layout and navigation."
  invariants:
    - text: "Internal links must keep working."
      priority: important
  dependencies:
    - "nav anchors -> section ids"
  risks:
    - text: "Longer copy may overflow on mobile."
      priority: soft
      severity: low
  contradictions: none
  open_questions: none
  sources_sampled: ["index.html"]
  sources_skipped_intentionally: []
```

```yaml
IMPACT_MAP:
  change_summary: "Update wording to make the page more universal."
  direct:
    - "index.html copy"
  indirect:
    - "reader interpretation"
  unknown: []
  affected_invariants:
    - text: "Internal links must keep working."
      priority: important
      from_anchor: "ANCHOR[ux]"
  minimal_modification_justification: "Only copy needs to change."
```

Verification:

- Count section ids and nav links.
- Search for outdated terms.
- Open or render the page if a browser is available.

## Example 2: Dataset Cleanup

```yaml
TRIAGE:
  context_size: medium
  complexity: medium
  stack: [CSV, validation rules]
  core_purpose: "Normalize customer records for import"
  task_scope: narrow
  external_refs_consulted: yes
  external_refs_sampled: ["import-spec.md"]
  external_refs_confidence: medium
  source_verification_required: yes
  required_sections: ["domain", "quality"]
  estimated_sources_to_read: 3
  rationale: "Only the import dataset and validation rules are relevant."
```

Critical invariant:

```yaml
- text: "Do not alter customer identifiers."
  priority: critical
```

Impact map:

```yaml
IMPACT_MAP:
  change_summary: "Normalize phone and country fields."
  direct:
    - "customers.csv"
  indirect:
    - "import validation"
  unknown: []
  affected_invariants:
    - text: "Do not alter customer identifiers."
      priority: critical
      from_anchor: "ANCHOR[domain]"
  minimal_modification_justification: "Only formatting fields fail validation."
```

Verification:

- Compare record count before and after.
- Confirm identifiers are unchanged.
- Run import validator or explain why it cannot run.

## Example 3: Agent Instruction Update

```yaml
TRIAGE:
  context_size: small
  complexity: medium
  stack: [prompt, policy, examples]
  core_purpose: "Guide an assistant's response style"
  task_scope: narrow
  external_refs_consulted: no
  external_refs_sampled: []
  external_refs_confidence: low
  source_verification_required: yes
  required_sections: ["identity", "domain", "quality"]
  estimated_sources_to_read: 2
  rationale: "The prompt and examples define the behavior."
```

Important invariant:

```yaml
- text: "The assistant must not claim actions it did not perform."
  priority: important
```

Impact map:

```yaml
IMPACT_MAP:
  change_summary: "Clarify final-answer verification language."
  direct:
    - "agent-instructions.md"
  indirect:
    - "example responses"
  unknown:
    - "production eval expectations"
```

Because `unknown` is non-empty, the agent must stop, inspect the eval expectations, or ask the owner before editing.

## Example 4: API Contract Change

```yaml
TRIAGE:
  context_size: medium
  complexity: high
  stack: [OpenAPI, service, tests]
  core_purpose: "Expose account data to clients"
  task_scope: broad
  external_refs_consulted: yes
  external_refs_sampled: ["client-request.md"]
  external_refs_confidence: medium
  source_verification_required: yes
  required_sections: ["domain", "contracts", "quality"]
  estimated_sources_to_read: 8
  rationale: "Changing a public response affects consumers."
```

Critical invariant:

```yaml
- text: "Existing clients must continue to parse required fields."
  priority: critical
```

Impact map:

```yaml
IMPACT_MAP:
  change_summary: "Add optional account status field."
  direct:
    - "openapi.yaml"
    - "account serializer"
  indirect:
    - "client SDK generation"
    - "contract tests"
  unknown: []
  affected_invariants:
    - text: "Existing clients must continue to parse required fields."
      priority: critical
      from_anchor: "ANCHOR[contracts]"
  minimal_modification_justification: "The field is additive and optional."
```
