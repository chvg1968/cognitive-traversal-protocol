# Cognitive Traversal Protocol (CTP)

> Status: **early draft** — gathering community feedback and empirical validation
> Version: **0.5.0**
> Scope: LLM-assisted work across code, documents, data, prompts, automations, and operational processes

CTP is an operating protocol that forces an LLM (or an agentic tool) to **declare what it knows, what it does not know, and what will change — before it changes anything.**

It is a two-layer protocol: a session-local discipline that runs inside a conversation, and a persistent knowledge layer that survives between sessions and provides mechanical anti-drift. Both layers matter. Presenting only the first is selling a half-protocol.

It is not a prompt trick. It is a discipline for reducing drift, hallucinated edits, and false-completion reports — across single sessions and over time.

---

## Architecture: two layers

CTP is intentionally a two-layer protocol.

**Layer 1 — Session-local discipline.**
TRIAGE, ANCHOR, IMPACT_MAP, the `unknown is blocking` rule, priorities. This is what runs inside a single LLM conversation. It prevents premature action, context loss, and false completion *within* the session.

**Layer 2 — Persistent knowledge layer.**
A structured knowledge substrate that lives outside the agent's context and outside the codebase. Provides cross-session memory, mechanical anti-drift verification, and graph-based navigation. Specified in [`docs/knowledge-layer.md`](docs/knowledge-layer.md).

| Capability | Layer 1 alone | Both layers |
|---|---|---|
| Discipline inside a session | ✓ | ✓ |
| Refuses to act on unresolved unknowns | ✓ | ✓ |
| Invariants survive between sessions | — | ✓ |
| Anti-drift verifiable mechanically | — | ✓ |
| Fast structural navigation (graph queries) | — | ✓ |

Layer 1 is enough for one-off tasks. Sustained work on real codebases requires Layer 2. Pretending otherwise is what makes most "agentic workflows" leak drift over time.

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

This single rule is the protocol's main contribution at Layer 1. Everything else exists to support it.

---

## Why CTP exists

LLMs and agentic tools fail in predictable ways:

| Failure mode | What it looks like | CTP step that intercepts it |
|---|---|---|
| Premature action | Edits code before understanding context | TRIAGE (Layer 1) |
| Context loss | Reads too much, forgets the task | ANCHOR (Layer 1) |
| Stale-doc bias | Trusts outdated wiki/comments over current source | Precedence rule: code > knowledge layer |
| Impact-blindness | Changes a contract without mapping consumers | IMPACT_MAP (Layer 1) |
| False completion | Reports success without verifying the artifact | Post-modification checklist (Layer 1) |
| Cross-session drift | Invariants forgotten when conversation ends | Persistent knowledge layer (Layer 2) |
| Documentation drift | Wiki diverges silently from code | `linked_code_entities` mechanical verification (Layer 2) |

CTP does not improve the underlying LLM. It improves the **governance layer** between humans and LLMs — both within a session and over time.

---

## Origin

This protocol started with a short video about Peano curves, Koch snowflakes, and Hausdorff dimensions — the idea that a line can be folded infinitely to fill a plane, and that nature builds efficient systems through hierarchical self-similarity. That sparked a question: can a repository be "folded" so an LLM ingests, understands, and modifies it without losing structural awareness?

The literal answer is no — tokenizers already linearize code, and the real problem is not linearization but preserving hierarchy through it. The intuition still translated into something operational: **apply the same cognitive pattern at every scale of the system.** Observe, compress, consolidate, advance — at the function level, the file level, the module level, the service level. A small repo runs the same pattern, fewer times.

CTP is the engineering form of that intuition. The fractal framing is the spark, not the proof. The protocol stands on its own operational merits, described in the rest of this document.

---

## Quick start (Layer 1)

You can apply Layer 1 today, even before setting up the persistent knowledge layer. Paste this in any LLM session:

```text
Use the Cognitive Traversal Protocol.
1. Emit TRIAGE first.
2. Create ANCHOR blocks for the sections you touch.
3. Before changing anything, emit IMPACT_MAP.
4. Do not proceed if IMPACT_MAP.unknown is non-empty.
5. After the change, verify affected invariants.
6. End with: critical findings, important findings, changes, verification, remaining limitations.
```

That gives you the session-local benefits. For sustained work on a real codebase, also set up Layer 2 — see [`docs/knowledge-layer.md`](docs/knowledge-layer.md).

Full Layer 1 spec: [`docs/protocol.md`](docs/protocol.md).

---

## Layer 1 templates

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

## Layer 2 in one page

Full specification: [`docs/knowledge-layer.md`](docs/knowledge-layer.md). The headlines:

- **External by design.** The knowledge layer lives outside the repository. The repo contains only what is deployed and tested; the knowledge corpus evolves on its own pace and keeps its own history.
- **Canonical implementation.** Obsidian vault exposed via Model Context Protocol (MCP). Nodes carry typed YAML frontmatter with `relations`, `linked_code_entities`, and `semantic_constraints`.
- **Pattern A — navigation (recommended).** Query the graph to orient. Then read code to verify. The graph answers "where does X live?", not "what is true today?".
- **Pattern B — authority (anti-pattern).** Trusting the graph as ground truth for current behavior. Knowledge layers drift; code is the source of truth.
- **Precedence is non-negotiable.** `code > spec (CLAUDE.md / protocol.md) > knowledge layer`. If a query returns something the code contradicts, the code wins.
- **No auto-updates.** The agent reports drift; it never edits the knowledge layer without explicit human authorization.
- **`linked_code_entities` is the anti-drift anchor.** Each typed node lists files and symbols. A simple verifier checks they still exist — drift is caught mechanically, not by human review.

---

## Limits and failure modes of CTP

CTP is not a silver bullet. Used badly it can hurt more than help. Honest list:

**1. Overhead on trivial tasks.**
Running TRIAGE + ANCHOR + IMPACT_MAP for renaming a variable is friction without payoff. The protocol explicitly allows narrow scope, but agents tend to over-apply it. Use the "When NOT to use" guidance.

**2. False-precision risk.**
The structured YAML output can give a false sense of rigor. A `priority: critical` tag does not make a finding critical; it just labels it. The protocol does not protect against an agent that simply mislabels.

**3. Calibration of `unknown` is hard.**
Agents may underreport `unknown` to appear competent, or overreport it to seem cautious. We do not yet have empirical data on calibration drift across model families.

**4. Layer 1 without Layer 2 is incomplete.**
The session-local discipline alone prevents drift *within* a session. It does not prevent invariants from being forgotten across sessions, and it cannot mechanically verify documentation against code. For sustained work on real codebases, Layer 2 is not optional in practice.

**5. Layer 2 requires investment.**
Setting up an Obsidian vault + MCP server + typed knowledge nodes is real work. For small or short-lived projects, that investment is not worth it. CTP does not pretend otherwise.

**6. Not validated at scale.**
CTP has been used in small-to-medium repos (under 500 source files) during early drafting. It has **not yet been empirically tested** on:
  - Repositories of thousands of files or hundreds of thousands of LOC.
  - Long-running multi-session agent workflows.
  - Adversarial prompting or jailbreak conditions.
  - Comparative A/B against the same task without CTP.

**7. Multi-session anchor persistence is still maturing.**
Layer 2 specifies the persistent substrate, but canonical schemas for cumulative anchors across conversations, branches, or team members are still being explored. Expect iteration.

**8. Model-family dependence is unknown.**
The protocol was drafted against Claude and GPT-class models. Whether smaller open models follow the protocol's discipline (especially the `unknown` blocking rule and the Pattern A/B distinction) is untested.

**9. The protocol governs humans-instructing-LLMs, not LLMs themselves.**
If the human operator does not enforce the discipline (e.g., accepts an edit with a non-empty `unknown`, or treats the knowledge layer as authority), the protocol provides no safety net.

If you hit any of these, please open an issue with the trace — they are exactly the cases that make the protocol better.

---

## Validation status

Honest snapshot of what has and has not been done.

| Category | Status |
|---|---|
| Internal coherence | ✅ Reviewed across 4 protocol revisions (v1 → v2 → v3 → v0.4) |
| Small repos (<500 files) | 🟡 Used in author's own projects; informal, no logged traces |
| Medium repos (500–5000 files) | ⚪ Not yet tested |
| Large repos (>5000 files) | ⚪ Not yet tested |
| Layer 2 in production | 🟡 External wiki pattern used informally; MCP exposure not yet wired |
| `linked_code_entities` verifier | ⚪ Schema designed, verifier script not yet shipped |
| A/B comparison (with vs without CTP, same model, same task) | ⚪ Not yet performed |
| Cross-model comparison (Claude / GPT / Gemini / open models) | ⚪ Not yet performed |
| Adversarial / jailbreak robustness | ⚪ Not yet tested |

**Help needed:** if you run CTP in a real project, please capture the TRIAGE + ANCHOR + IMPACT_MAP traces (sanitized) and submit them via PR or issue. Even one annotated trace from a real repo is more valuable than ten more synthetic examples.

---

## When to use, when NOT to use

**Use Layer 1 only** when:
- The task is a one-off (no follow-up sessions expected).
- The artifact is self-contained (no cross-references that drift over time).
- You want to try CTP discipline before investing in infrastructure.

**Use both layers** when:
- You work on a real codebase across many sessions.
- Multiple people or agents touch the same artifacts.
- Documentation drift has bitten you before, or will.
- Contracts, schemas, or invariants matter and must survive turnover.

**Skip CTP entirely** for:
- One-off factual questions.
- Single-file formatting with no dependencies.
- Tasks where the user provides the full artifact and expected output.
- Throwaway prototypes the user explicitly marks as disposable.

**Do not use CTP as decoration.**
If you emit a TRIAGE block but then ignore it, the protocol is providing false assurance — worse than not using it at all. Same applies to Layer 2: a knowledge layer nobody reads or verifies is theater.

---

## Documentation

- [Protocol specification](docs/protocol.md) — Layer 1 full normative spec
- [Knowledge layer specification](docs/knowledge-layer.md) — Layer 2 full normative spec
- [Examples](docs/examples.md) — illustrative cases (synthetic; real traces wanted)
- [LLM quickstart](docs/llm-quickstart.md) — paste-ready prompts for Layer 1
- [Visual guide](index.html) — diagram and walk-through
- [Changelog](CHANGELOG.md)

---

## Contributing

Community feedback is welcome. See [CONTRIBUTING.md](CONTRIBUTING.md).

**Highest-value contributions right now**

1. **Real-world traces.** A sanitized TRIAGE + ANCHOR + IMPACT_MAP from an actual task — especially in large codebases.
2. **A/B case studies.** Same task, same model, with vs without CTP. Even one is valuable.
3. **Layer 2 reference implementations.** A working Obsidian + MCP setup, or an alternative substrate (database-backed graph, custom MCP server, etc.) with a writeup.
4. **A `linked_code_entities` verifier.** A small script (Python, TS, anything) that reads typed knowledge nodes and confirms the referenced files and symbols still exist. Catches drift mechanically.
5. **Failure cases.** Where CTP overshoots, underdelivers, or is silently bypassed. Honest negative results are as useful as success stories.
6. **Tool-specific adapters.** Cursor, Claude Code, ChatGPT custom GPT, Copilot Chat, Aider, etc.
7. **Translations and one-page cheat sheets.**

---

## License

MIT. See [LICENSE](LICENSE).
