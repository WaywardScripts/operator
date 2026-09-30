# Reporting back to Project:Survival

When you reach a milestone (or hit a wall), produce an **export** and bring it
back to the parent system. The parent is a separate agentic OS that ingests
this to learn what worked, update its findings, and adjust the plan.

## How to produce the export
Run (Phase 6):
```powershell
Export-OperatorData -Path .\operator-export
```
If Phase 6 isn't built yet, manually copy the workspace:
`%USERPROFILE%\.operator\<workspace>\*.jsonl` plus any outbox/pending files.

## What to bring back
An `operator-export/` folder containing:
- `memory/episodic.jsonl`, `procedural.jsonl`, `semantic.jsonl`, `guardrail.jsonl`
- `results.md` — what the agent did, and how well
- `guardrails.md` — what failed and why
- `skills.md` — proposed/promoted automations
- `manifest.json` — see schema below

## manifest.json schema
```json
{
  "product": "operator",
  "version": "0.1.0",
  "workspace": "<slug>",
  "generated_at": "<iso8601>",
  "phase_completed": 6,
  "stats": {
    "episodes": 0, "skills_promoted": 0, "skills_proposed": 0,
    "emails_processed": 0, "drafts_created": 0, "actions_pending": 0,
    "guardrails": 0, "integrity_ok": true
  },
  "notes": "free-text summary of what worked and what didn't"
}
```

## results.md should answer
1. What tasks did the agent actually automate? (with counts)
2. Where did it fail or need a human? (link guardrail entries)
3. What did it learn? (new skills/facts)
4. Time saved (rough) and any cost.
5. The single biggest blocker to the next phase.

## Privacy
Redact or exclude any real client content you don't want in the parent
system. The parent only needs **patterns, counts, failures, and skills** — not
the raw emails. When in doubt, summarize rather than include raw bodies.

## Ingest on the parent side
The parent will run something like:
`bin/survival ingest-operator <path-to-export>` (to be built) which imports the
export into its findings/memory and updates the Operator venture plan.
