# LLM Quickstart

Paste this into an LLM session when you want the assistant to use CTP.

```text
Use the Cognitive Traversal Protocol (CTP) before acting.

Rules:
1. Emit TRIAGE first.
2. Read only the context needed for this task.
3. Create anchors for the sections you touch.
4. Treat external references as orientation, not proof.
5. The current source of truth wins.
6. Before changing anything, emit IMPACT_MAP.
7. Do not proceed if IMPACT_MAP.unknown is non-empty.
8. Make the smallest responsible intervention.
9. After the change, verify affected invariants.
10. In the final answer, report critical findings, important findings, changes, verification, and remaining limitations.
```

## Minimal Prompt

```text
Apply CTP. Start with TRIAGE. Do not change anything until you provide an IMPACT_MAP with unknown: [].
```

## Review Prompt

```text
Review this artifact using CTP. Prioritize bugs, contradictions, broken contracts, missing verification, and risky assumptions. Do not rewrite unless I ask.
```

## Editing Prompt

```text
Use CTP to modify this artifact. Preserve existing contracts and style. If the impact map has unknowns, stop and ask a targeted question.
```

## Final Answer Contract

```text
End with:
- Critical findings
- Important findings
- Changes made
- Verification
- Recommended next actions
```
