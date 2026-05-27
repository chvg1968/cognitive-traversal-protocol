# Changelog

All notable changes to the Cognitive Traversal Protocol will be documented here.

## [0.5.0] - 2026-05-27

Socratic extension. The protocol gains an élenchos sub-step and a maieutic phrasing rule. Both are minimal additions, not a rewrite — they formalize practices that were implicitly desirable but not enforced under 0.4.0.

Changed
- Section Model: from four sub-steps to five. New step 3 "Challenge (élenchos)" forces the agent to actively seek refutations of the tentative invariants before sealing them. The previous step 3 "Consolidate" becomes step 4; "Advance" becomes step 5.
- Anchor Format: new `elenchic_challenge` block (`attempted_refutation`, `result`, `notes`) records the refutation attempt. Sits between `summary` and `invariants` so that what gets sealed is the post-challenge invariant list, not the original.
- "Anchors Are Working Memory" contents list now includes "Elenchic challenge (refutation attempt and result)".
- "Unknown Impact Blocks Action": "Asks a targeted question" replaced with "Asks a maieutic question — phrased to surface the user's implicit invariants, not to extract a binary decision".

Rationale
- CTP 0.4.0 already operationalized *docta ignorantia* through the `unknown` bucket of IMPACT_MAP — élenchos applied to the impact analysis. It did not apply élenchos to invariants themselves: anchors could be sealed comfortably without any refutation attempt. The new sub-step closes that gap and hardens anchors against confirmation bias under tight reading budgets.
- CTP 0.4.0 already required asking the user when reading alone could not resolve `unknown`. It did not specify *how* to ask. Binary questions extract decisions; maieutic questions extract tacit invariants. The latter is what is actually scarce when human knowledge is the missing input.

Preserved
- All existing fields, sub-steps, and semantics. Anchors emitted under 0.4.0 remain valid; the new field is additive and the renumbered sub-steps preserve order.
- Priority system, IMPACT_MAP semantics, `unknown`-blocking rule, precedence rule (`code > spec > knowledge layer`), Layer 2 architecture — all unchanged.
- The "trust code over external references" rule, the human-in-the-loop authorization for the knowledge layer, and the limits/validation framework.

Versioning
- Minor bump (0.4.0 → 0.5.0). Backward-compatible additive change. README, protocol.md, and knowledge-layer.md bumped in lockstep, following the convention established in 0.4.0.

## [0.4.0] - 2026-05-19

Architectural reframing. CTP is now explicitly presented as a two-layer protocol, not a session-local prompt discipline. The persistent knowledge layer is lifted from "optional extension" to "inherent part of the protocol" — the previous framing oversold what Layer 1 alone delivers.

Changed
- README restructured around the two-layer architecture. New "Architecture: two layers" section makes explicit that Layer 1 (session-local discipline: TRIAGE, ANCHOR, IMPACT_MAP, unknown-blocking) and Layer 2 (persistent knowledge substrate) are both part of CTP. Layer 1 alone is enough for one-off tasks; sustained work on real codebases requires Layer 2.
- "Why CTP exists" failure-mode table extended to include cross-session drift and documentation drift, both intercepted by Layer 2.
- Quick start section renamed "Quick start (Layer 1)" to set the right expectation. The pasted prompt is the on-ramp, not the full protocol.
- Templates section renamed "Layer 1 templates" for the same reason.
- "When to use, when NOT to use" rewritten to distinguish single-layer use (one-off tasks) from full two-layer use (sustained work).
- Documentation list extended with the new Layer 2 spec.
- Contributing reordered: real-world traces first, then A/B case studies, Layer 2 reference implementations, `linked_code_entities` verifiers, failure cases, tool adapters, translations.
- protocol.md retitled "Layer 1 Specification" with scope clarified at the top. References Layer 2 explicitly via the new knowledge-layer.md document.

Added
- docs/knowledge-layer.md: full normative specification of Layer 2. Covers the external-by-design architecture, the Obsidian + MCP canonical implementation, Pattern A (navigation, recommended) vs Pattern B (authority, anti-pattern), the typed YAML schema with `relations`, `linked_code_entities`, and `semantic_constraints`, the `linked_code_entities` mechanical anti-drift verifier (specified; reference implementation pending), the precedence rules (`code > spec > knowledge layer`), and the update flow with human-in-the-loop authorization.
- New limit (#4): "Layer 1 without Layer 2 is incomplete" — explicit acknowledgement that session-local discipline alone does not prevent cross-session drift or mechanically verify documentation against code.
- New limit (#5): "Layer 2 requires investment" — honest about the setup cost.
- New validation-status row: "Layer 2 in production" (informally used; MCP exposure not yet wired) and "`linked_code_entities` verifier" (schema designed, script pending).
- protocol.md anti-pattern: "Using Layer 2 as authority instead of navigation".
- protocol.md anti-pattern: "Auto-updating the knowledge layer without explicit human authorization".

Preserved
- All Layer 1 mechanics (TRIAGE, ANCHOR, IMPACT_MAP, unknown-blocking, priorities) unchanged.
- Origin section unchanged.
- Limits and validation framework unchanged in structure, only extended.
- The human-in-the-loop rule for the knowledge layer: the agent reports drift, it does not auto-correct.
- Precedence rule (`code > spec > knowledge layer`) is now stated explicitly in both protocol.md and knowledge-layer.md.

## [0.3.0] - 2026-05-17

README reorganization and honest scoping. Version downgraded from 1.0.0 to 0.3.0 to reflect "early draft" status more accurately while empirical validation is pending.

Changed
- Reordered README so the core rule and the `unknown is blocking` invariant lead the document. Previously buried.
- Added a "Why CTP exists" table mapping LLM failure modes (premature action, context loss, stale-doc bias, impact-blindness, false completion) to the CTP step that intercepts each.
- Expanded "When to use, when NOT to use" with the explicit warning that emitting CTP blocks without enforcing them is worse than not using the protocol.
- Reframed CTP as a governance layer between humans and LLMs, not as a method to improve the underlying model.

Added
- "Limits and failure modes of CTP" section listing 7 honest limitations: overhead on trivial tasks, false-precision risk, `unknown` calibration drift, lack of large-scale validation, no multi-session anchor persistence, unknown model-family dependence, and human-operator enforcement gap.
- "Validation status" table making explicit what has and has not been tested: internal coherence reviewed, small repos used informally, medium/large repos untested, A/B comparison not performed, cross-model comparison not performed, multi-session persistence not designed, adversarial robustness not tested.
- Reorganized "Contributing" section by value: real-world traces first, A/B case studies second, failure cases third.

Versioning
- Downgraded from 1.0.0 to 0.3.0. The previous 1.0.0 tag was inconsistent with the "draft for community feedback" status. 0.x signals that breaking changes are expected as the protocol is validated.

## [1.0.0] - 2026-05-17

Initial community draft. Superseded by 0.3.0 (same date) — see entry above for the version correction rationale.

- Added the core CTP flow: TRIAGE, anchors, IMPACT_MAP, intervention, verification.
- Added universal templates for LLM-assisted work across code, documents, data, prompts, automations, and operations.
- Added examples for static pages, datasets, agent instructions, and API contracts.
- Added a standalone visual guide for GitHub Pages.
