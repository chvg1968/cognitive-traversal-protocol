# Cognitive Traversal Protocol

Status: draft for community feedback  
Version: 1.0.0  
Scope: LLM-assisted work across code, documents, data, prompts, automations, and operational processes

The Cognitive Traversal Protocol (CTP) is a lightweight operating protocol for LLMs and agentic tools. It helps an assistant inspect a context, preserve critical invariants, declare impact, make the smallest responsible intervention, and verify the result.

CTP is not a prompt trick. It is a disciplined workflow for reducing drift before an LLM changes anything.

## Why This Exists

LLMs often fail in predictable ways:

- They act before understanding the local context.
- They over-read irrelevant material and lose the task.
- They trust stale docs over the current source of truth.
- They make changes without mapping indirect impact.
- They report success without verifying the artifact.

CTP gives the agent a compact loop:

```text
orient -> anchor -> map impact -> intervene -> verify
```

## Core Rule

If the agent cannot name what changes, what depends on it, and what remains unknown, it is not ready to intervene.

## Quick Start

Ask the LLM to use CTP before acting:

```text
Use the Cognitive Traversal Protocol.
Start with TRIAGE.
Create anchors for the sections you touch.
Before changing anything, emit an IMPACT_MAP.
Do not proceed if IMPACT_MAP.unknown is non-empty.
After the change, run the post-modification checklist.
```

Then require this sequence:

1. `TRIAGE`
2. Relevant `ANCHOR[...]` blocks
3. `IMPACT_MAP`
4. Minimal intervention
5. Verification and final findings

## Templates

### TRIAGE

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

### ANCHOR

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

### IMPACT_MAP

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

`unknown` is blocking. The agent must read more, ask a targeted question, or stop.

## Documentation

- [Protocol specification](docs/protocol.md)
- [Examples](docs/examples.md)
- [LLM quickstart](docs/llm-quickstart.md)
- [Visual guide](index.html)
- [Changelog](CHANGELOG.md)

## Publishing With GitHub Pages

This project is static. To publish it:

1. Create a GitHub repository.
2. Push these files to the default branch.
3. In GitHub, go to `Settings -> Pages`.
4. Select `Deploy from a branch`.
5. Choose the default branch and `/root`.
6. Open the generated Pages URL.

GitHub Pages will serve `index.html` as the visual guide.

## When To Use CTP

Use CTP when an LLM will inspect, modify, summarize, migrate, refactor, review, or verify an artifact where context matters.

Good fits:

- Code changes
- Technical documents
- Data cleanup
- Prompt or agent instruction updates
- API or schema changes
- Operational runbooks
- Product or design artifacts

Not necessary:

- One-off factual questions
- Tiny formatting changes with no dependencies
- Tasks where the user explicitly provides the full artifact and expected output

## Contributing

Community feedback is welcome. See [CONTRIBUTING.md](CONTRIBUTING.md).

Useful contribution types:

- Better examples
- More precise templates
- Translations
- Tool-specific adaptations
- Case studies from real workflows

## License

MIT. See [LICENSE](LICENSE).
