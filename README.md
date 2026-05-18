# Cognitive Traversal Protocol (CTP)

> Status: **early draft** — gathering community feedback and empirical validation
> Version: **0.3.0**
> Scope: LLM-assisted work across code, documents, data, prompts, automations, and operational processes

CTP is a lightweight operating protocol that forces an LLM (or an agentic tool) to **declare what it knows, what it does not know, and what will change — before it changes anything.**

It is not a prompt trick. It is a discipline for reducing drift, hallucinated edits, and false-completion reports.

---

## The core idea, in one rule

> **If the agent cannot name what it is changing, what depends on it, and what remains unknown — it is not ready to intervene.**

CTP turns that rule into an executable sequence:

```text
TRIAGE  →  ANCHOR  →  IMPACT_MAP  →  intervene  →  VERIFY
```

And it enforces one structural invariant most agentic frameworks lack:

> **`IMPACT_MAP.unknown` is blocking.**
> If the agent cannot eliminate `unknown`, it must read more, ask a targeted question, or stop.
> No silent assumptions. No "best-effort" edits over unresolved areas.

This single rule is the protocol's main contribution. Everything else exists to support it.

---

## Why CTP exists

LLMs and agentic tools fail in predictable ways:

| Failure mode | What it looks like | CTP step that intercepts it |
|---|---|---|
| Premature action | Edits code before understanding context | TRIAGE |
| Context loss | Reads too much, forgets the task | ANCHOR (compress per section) |
| Stale-doc bias | Trusts outdated wiki/comments over current source | TRIAGE rule: external refs are orientation, not proof |
| Impact-blindness | Changes a contract without mapping consumers | IMPACT_MAP |
| False completion | Reports success without verifying the artifact | Post-modification checklist |

CTP does not improve the underlying LLM. It improves the **governance layer** between humans and LLMs.

---

## Quick start

Paste this in any LLM session:

```text
Use the Cognitive Traversal Protocol.
1. Emit TRIAGE first.
2. Create ANCHOR blocks for the sections you touch.
3. Before changing anything, emit IMPACT_MAP.
4. Do not proceed if IMPACT_MAP.unknown is non-empty.
5. After the change, verify affected invariants.
6. End with: critical findings, important findings, changes, verification, remaining limitations.
```

That's it. Full templates are below; deeper specs are in [`docs/protocol.md`](docs/protocol.md).

---

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
  external_refs_confidence: low | medium | high
  source_verification_required: yes | no
  required_sections: ["identity", "domain", ...]
  estimated_sources_to_read: <integer>
  rationale: "why this scope is enough"
```

### ANCHOR

```yaml
ANCHOR[section-name]:
  summary: "2–4 lines"
  invariants:
    - text: "property that must be preserved"
      priority: critical | important | soft
  dependencies: ["source -> consumer"]
  risks:
    - text: "specific risk"
      priority: critical | important | soft
      severity: low | med | high
  contradictions: none   # or against prior anchor with priority
  open_questions: none
  sources_sampled: [...]
  sources_skipped_intentionally: [...]
```

### IMPACT_MAP

```yaml
IMPACT_MAP:
  change_summary: "what will change"
  direct:    ["artifact / module / contract directly modified"]
  indirect:  ["behavior or consumer affected through dependency"]
  unknown:   ["suspected area that cannot yet be classified"]
  affected_invariants:
    - text: "affected invariant"
      priority: critical | important | soft
      from_anchor: "ANCHOR[section-name]"
  minimal_modification_justification: "why this change is enough"
```

→ **`unknown` is blocking.** Read more, ask, or stop.

---

## Limits and failure modes of CTP

CTP is not a silver bullet. Used badly it can hurt more than help. Honest list:

**1. Overhead on trivial tasks.**
Running TRIAGE + ANCHOR + IMPACT_MAP for renaming a variable is friction without payoff. The protocol explicitly allows narrow scope, but agents tend to over-apply it. Use the "When NOT to use" guidance.

**2. False-precision risk.**
The structured YAML output can give a false sense of rigor. A `priority: critical` tag does not make a finding critical; it just labels it. The protocol does not protect against an agent that simply mislabels.

**3. Calibration of `unknown` is hard.**
Agents may underreport `unknown` to appear competent, or overreport it to seem cautious. We do not yet have empirical data on calibration drift across model families.

**4. Not validated at scale.**
CTP has been used in small-to-medium repos (under 500 source files) during early drafting. It has **not yet been empirically tested** on:
  - Repositories of thousands of files or hundreds of thousands of LOC.
  - Long-running multi-session agent workflows.
  - Adversarial prompting or jailbreak conditions.
  - Comparative A/B against the same task without CTP.

**5. Anchor reuse across sessions is unsolved.**
Anchors are session-local. There is no canonical persistence format yet for cumulative anchors across conversations, branches, or team members. This is a known gap.

**6. Model-family dependence is unknown.**
The protocol was drafted against Claude and GPT-class models. Whether smaller open models follow the protocol's discipline (especially the `unknown` blocking rule) is untested.

**7. The protocol governs humans-instructing-LLMs, not LLMs themselves.**
If the human operator does not enforce the discipline (e.g., accepts an edit with a non-empty `unknown`), the protocol provides no safety net.

If you hit any of these, please open an issue with the trace — they are exactly the cases that make the protocol better.

---

## Validation status

Honest snapshot of what has and has not been done.

| Category | Status |
|---|---|
| Internal coherence | ✅ Reviewed across 3 protocol revisions (v1 → v2 → v3) |
| Small repos (<500 files) | 🟡 Used in author's own projects; informal, no logged traces |
| Medium repos (500–5000 files) | ⚪ Not yet tested |
| Large repos (>5000 files) | ⚪ Not yet tested |
| A/B comparison (with vs without CTP, same model, same task) | ⚪ Not yet performed |
| Cross-model comparison (Claude / GPT / Gemini / open models) | ⚪ Not yet performed |
| Multi-session anchor persistence | ⚪ Not yet designed |
| Adversarial / jailbreak robustness | ⚪ Not yet tested |

**Help needed:** if you run CTP in a real project, please capture the TRIAGE + ANCHOR + IMPACT_MAP traces (sanitized) and submit them via PR or issue. Even one annotated trace from a real repo is more valuable than ten more synthetic examples.

---

## When to use, when NOT to use

**Good fits**
- Code changes touching contracts, schemas, or shared modules.
- Technical documents with cross-references.
- Data cleanups where identifiers or keys must be preserved.
- Prompt or agent-instruction updates.
- API or schema changes.
- Operational runbooks.

**Skip CTP for**
- One-off factual questions.
- Single-file formatting with no dependencies.
- Tasks where the user provides the full artifact and expected output.
- Throwaway prototypes the user explicitly marks as disposable.

**Do not use CTP as decoration.**
If you emit a TRIAGE block but then ignore it, the protocol is providing false assurance — worse than not using it at all.

---

## Documentation

- [Protocol specification](docs/protocol.md) — full normative spec
- [Examples](docs/examples.md) — illustrative cases (synthetic; real traces wanted)
- [LLM quickstart](docs/llm-quickstart.md) — paste-ready prompts
- [Visual guide](index.html) — diagram and walk-through
- [Changelog](CHANGELOG.md)

---

## Contributing

Community feedback is welcome. See [CONTRIBUTING.md](CONTRIBUTING.md).

**Highest-value contributions right now**

1. **Real-world traces.** A sanitized TRIAGE + ANCHOR + IMPACT_MAP from an actual task — especially in large codebases.
2. **A/B case studies.** Same task, same model, with vs without CTP. Even one is valuable.
3. **Failure cases.** Where CTP overshoots, underdelivers, or is silently bypassed. Honest negative results are as useful as success stories.
4. **Tool-specific adapters.** Cursor, Claude Code, ChatGPT custom GPT, Copilot Chat, Aider, etc.
5. **Translations and one-page cheat sheets.**

---

## License

MIT. See [LICENSE](LICENSE).
