# Changelog

All notable changes to the Cognitive Traversal Protocol will be documented here.

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
