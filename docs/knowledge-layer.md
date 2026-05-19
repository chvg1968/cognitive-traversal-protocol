# The Persistent Knowledge Layer

> Status: **normative — part of CTP architecture**
> Version: 0.4.0
> Layer: 2 of 2 (see [README](../README.md) for the two-layer overview)

This document specifies Layer 2 of CTP — the persistent knowledge substrate that provides cross-session memory, mechanical anti-drift verification, and structural navigation.

Without this layer, CTP only protects within a single conversation. Anchors die when the session ends, invariants are not enforceable across turns, and drift between code and documentation is detectable only by human review. Layer 2 is what makes CTP a protocol for sustained work, not a prompt trick.

---

## Why this layer exists

Three problems Layer 1 cannot solve alone:

1. **Session boundary erases memory.** Anchors compress evidence within a session, but disappear at its end. The next session starts blind.
2. **Documentation drift is silent.** Wikis, READMEs, comments and architecture notes diverge from code over time. By the time a human notices, drift has already shaped multiple decisions.
3. **Navigation by grep does not scale.** In large codebases, "where does X live?" or "what depends on Y?" becomes prohibitive without structured graph queries.

Layer 2 addresses all three.

---

## Architectural principle: external by design

**The knowledge layer lives outside the repository.** This is not an accident; it is a design choice with two operational reasons:

1. **The repository contains only what is deployed and tested.** The knowledge corpus is not a build artifact. Mixing them pollutes the repo with content that has different change cadence, different review criteria, and different ownership.
2. **The knowledge corpus evolves independently.** It may be consulted by multiple repositories. It maintains its own git history. It changes when *understanding* changes, not when *code* changes.

Concrete consequences:

- Changes to the knowledge layer never enter commits of the repository.
- The repository does not assume the knowledge layer is present in order to build, test, or deploy.
- The knowledge layer is a **consultable reference**, not an operational dependency.

If the project ships, the knowledge layer can be absent. If the knowledge layer is rich, the project still ships.

---

## Canonical implementation: Obsidian vault + MCP

The recommended substrate is an [Obsidian](https://obsidian.md) vault exposed via [Model Context Protocol (MCP)](https://modelcontextprotocol.io).

**Why Obsidian:**
- Native wikilinks and YAML frontmatter — already the shape of a typed knowledge graph.
- Local-first markdown — readable without Obsidian if needed.
- Vault structure (folders, tags, frontmatter) is preserved and queryable.

**Why MCP:**
- The agent gets programmatic graph queries instead of full-text grep across markdown.
- Queries like "list all nodes with `type: business_domain`" or "what depends on `base_geospatial_layer`?" become first-class operations.
- Composable with the other MCP servers an agent uses.

**Alternative substrates** are acceptable when justified:

- Plain markdown without MCP — works, but slower; relies on grep over filesystem.
- A database-backed knowledge graph (Neo4j, etc.) — heavyweight; appropriate for large organizations.
- A custom MCP server over an existing wiki — viable if the wiki already has structured data.

The choice of substrate matters less than respecting the rules below.

---

## Use patterns

### Pattern A (recommended): Navigation layer

The agent queries the knowledge graph **to orient**, then reads code **to verify**.

Examples of legitimate queries:

- "Where does the maturity module live? What files implement it?"
- "What nodes depend on `base_geospatial_layer`?"
- "List all nodes with `status: deprecated`."
- "What are the `semantic_constraints` documented for the dashboard module?"

The output of these queries is **orientation, not proof**. The agent always confirms current behavior by reading the code itself.

### Pattern B (anti-pattern): Authority layer

The agent treats the knowledge graph as the source of truth for current behavior, and modifies code to match the graph.

Examples of illegitimate use:

- "The node says this function must be pure; the code has a side effect; the code is wrong."
- "The graph says module X consumes module Y; therefore I will refactor module X without reading Y."
- "The `semantic_constraints` say input vectors are normalized; therefore I will skip input validation."

This inverts the precedence. Knowledge layers drift. Treating them as authority propagates the drift into code.

**Avoid Pattern B unconditionally.**

---

## Typed YAML schema

For artifacts whose drift cost is high — typically domains with active code and shared contracts — adopt typed frontmatter. The schema is intentionally minimal.

```yaml
---
id: smart_city_maturity_module
type: business_domain | service | contract | integration
status: active | deprecated | experimental
relations:
  depends_on:
    - node_id: base_geospatial_layer
      reason: "Requires municipal polygons for cross-referencing"
  consumed_by:
    - node_id: smart_cities_dashboard
linked_code_entities:
  - file: src/analytics/maturity_model.py
    symbol: MaturityCalculator
  - file: src/analytics/scoring.py
    symbol: normalize_input_vector
semantic_constraints:
  - "Statistical calculations must be pure (no DB side-effects)."
  - "Input vectors must undergo structural normalization before scoring."
---
```

**Field semantics**

| Field | Purpose | Verifiable? |
|---|---|---|
| `id` | Stable node identifier | No |
| `type` | Coarse classification | No |
| `status` | Lifecycle stage | No |
| `relations.depends_on` | Outgoing edges | Yes — can compare against actual imports |
| `relations.consumed_by` | Incoming edges | Yes — can compare against actual references |
| `linked_code_entities` | Concrete code anchors | **Yes — mechanically verifiable** |
| `semantic_constraints` | Documented invariants | Partial — can be referenced in `affected_invariants` |

**Adoption guidance**

- Start with **at most 3 critical pages** as a pilot. Do not retype the whole vault upfront.
- Choose pages where drift has bitten the team before, or where contracts are shared widely.
- Evaluate after one month: did typed pages drift less? did `linked_code_entities` catch real changes? If yes, expand. If not, the schema is not earning its keep on this corpus.

---

## `linked_code_entities` as mechanical anti-drift

This is the single most important field in the schema. It is the only mechanism in the entire CTP architecture that catches drift **without requiring human review**.

**The contract:**
Every entry in `linked_code_entities` claims that a specific file exists, and that a specific symbol (class, function, constant, type) is defined in that file.

**The verifier:**
A small script (50–100 lines in any language) walks every typed node in the knowledge layer, opens each referenced file, and confirms each referenced symbol is still defined. The output is a report: green if all references hold; red with specific drift locations otherwise.

**When it runs:**
- As a CI step (cheap and frequent).
- As a pre-commit hook on the knowledge vault.
- On demand before a major refactor.

**What it cannot do:**
- It cannot verify `semantic_constraints` — those are natural-language claims.
- It cannot detect *behavioral* drift (e.g., a function that still exists but no longer is pure).
- It cannot reconcile renamings automatically.

What it *does* do is catch the most common drift: code moves, symbols are renamed, files are split — and the knowledge layer silently lies about it. The verifier converts that silent lie into a noisy alert.

A reference implementation is on the roadmap. Contributions welcome.

---

## Precedence rules

The precedence rules from Layer 1 carry forward unchanged:

> **`code > spec (CLAUDE.md / protocol.md) > knowledge layer`**

If the knowledge layer contradicts the spec, the spec wins.
If the spec contradicts the code, the code wins.
If anything contradicts the code, the code wins.

These are not aspirations; they are operational rules. A CTP agent that violates them is not doing CTP.

---

## Update flow

**Authorization is required.** The agent never updates the knowledge layer on its own. Detecting drift authorizes a **report**, not a correction.

When the user does authorize an update, the **tripod** is mandatory:

1. Change the affected page.
2. Log the change in `log.md` with date and tag.
3. If the change introduces a new page, link it from `index.md`.

**Atomicity:** one log entry per topic. Two independent changes get two entries.

**Page health:** if a page exceeds ~200 lines or mixes two domains, split it before continuing to grow it.

**Deprecation, not deletion:** mark deprecated pages with `> Deprecada YYYY-MM-DD — ver [[nueva-pagina]]` and log the change. Do not delete history.

These rules exist because knowledge layers, like wikis, become useless when they accumulate silent edits. Discipline at update time is what preserves their value.

---

## What this layer does NOT do

To prevent miscategorization, the boundaries are explicit:

- It does **not** replace code as the source of truth.
- It does **not** auto-update from agent edits.
- It does **not** enforce `semantic_constraints` — only documents them.
- It does **not** provide business reasoning — only structure.
- It is **not** a database, a cache, or a queryable replacement for production data.
- It is **not** a substitute for tests, linters, or type checkers.

If you find yourself asking "should the agent use the knowledge layer to *decide* X?", the answer is almost certainly no. The knowledge layer answers **where to look**, never **what is true today**.

---

## Anti-patterns to avoid

- Using the knowledge layer as source of intent for code changes (Pattern B).
- Auto-updating the knowledge layer from agent edits.
- Treating `semantic_constraints` as enforceable contracts instead of documented claims.
- Halting an agent on a contradiction between code and knowledge layer where the code is correct — the resolution is to fix the knowledge layer, not the code.
- Building a knowledge layer nobody reads or verifies. A stale knowledge layer is worse than no knowledge layer at all, because it lends false authority to wrong claims.

---

## Relationship to Layer 1

Layer 2 does not replace Layer 1. It surrounds it.

- TRIAGE consults the knowledge layer to scope the task. It does not trust it.
- ANCHOR may cite `linked_code_entities` from the knowledge layer as a starting point — but the invariants in the anchor come from the code.
- IMPACT_MAP may use the knowledge layer's `relations` to enumerate `indirect` and `unknown` items — then verifies them in code before classifying as `direct`.
- Verification after intervention may include re-running the `linked_code_entities` verifier to confirm no drift was introduced.

The two layers are complementary. Each catches failures the other cannot.
