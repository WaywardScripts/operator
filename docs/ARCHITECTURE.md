# Operator — Architecture

## 1. Purpose and constraints
An employer-approved job aid that reads Office data (Outlook first), reasons
about the worker's tasks, learns to automate them, and gates irreversible
actions behind approval.

Constraints: **Windows**, **PowerShell scripts only (no .exe)**, **no installed
dependencies**, **Outlook COM**, **transparent + audited**, **approval-gated**,
**isolated memory per workspace**.

## 2. Repository layout
```
Invoke-Operator.ps1          # entrypoint (task orchestration)
OperatorConfig.ps1           # config + workspace resolution
OperatorMemory.psm1          # the memory core (5 stores, hash chain)
OperatorOutlook.ps1          # read Outlook; apply approved Outlook actions
OperatorAgent.ps1            # the reasoning loop
OperatorApproval.ps1         # pending queue + apply
OperatorLearning.ps1         # repetition detection + skill lifecycle
OperatorReport.ps1           # export for the parent system
library/skills/*.json        # front-loaded automation library
config.example.json          # template (real config is git-ignored)
docs/                        # this folder
tests/*.Tests.ps1            # acceptance tests
```

## 3. Configuration and workspace
`config.json` (git-ignored; copy from `config.example.json`):
```json
{
  "workspace_root": "%USERPROFILE%\\.operator",
  "workspace": "work",
  "llm": { "endpoint": "https://.../v1/chat/completions", "model": "...", "api_key_env": "OPERATOR_LLM_KEY" },
  "outlook": { "folder": "Inbox", "count": 50 },
  "approval": { "irreversible": ["send", "delete", "move", "submit", "pay", "external_contact"] }
}
```
Workspace layout:
```
<workspace_root>/<workspace>/
  episodic.jsonl  procedural.jsonl  semantic.jsonl  guardrail.jsonl
  state.json
<workspace_root>/<workspace>/outbox/     # safe actions produced
<workspace_root>/<workspace>/pending/    # irreversible actions awaiting approval
```

## 4. Memory core (the moat)
Five stores. Four are **append-only JSONL**; every record is **hash-chained**:
`hash = SHA256(prev_hash + "|" + canonical_json(record_without_hash))`.
Any rewrite/delete breaks the chain and is detectable by `Test-OperatorMemory`.

Record shape (episodic example):
```json
{"seq":1,"ts":"2026-09-30T12:00:00Z","kind":"observation",
 "dedup_key":"<outlook-entry-id>","data":{...},
 "prev_hash":"GENESIS","hash":"<sha256>"}
```

Stores:
- **episodic** — observations, decisions, actions, outcomes. Supports
  **dedup_key** (e.g. Outlook `EntryID`) so nothing is processed twice.
- **procedural** — skills: `{name, trigger, steps[], preconditions, validation,
  status, use_count, source}`. Lifecycle: `proposed → trialing → promoted`.
  Promotion/use are appended as new records (append-only snapshots).
- **semantic** — facts: `{fact, confidence, provenance, status}`. Facts are
  never edited; a stale fact is a new record with `status:"stale"`.
- **guardrail** — failures: `{attempt, reason, ref}`. Checked before acting so
  a failed approach is never repeated.
- **state.json** — small rewritable state (last run, cursors, counters).

Functions (PowerShell):
- `Add-OperatorRecord -Store <name> -Kind <k> -Data <hashtable> [-DedupKey <k>]`
- `Get-OperatorRecords -Store <name> [-Last <n>]`
- `Test-OperatorMemory` → verifies every chain; throws on break.
- `Test-OperatorSeen -DedupKey <k>` → dedup check.
- `Add-OperatorGuard -Attempt <s> -Reason <s>` / `Test-OperatorGuarded -Attempt <s>`
- `Add-OperatorFact`, `Get-OperatorFacts -Query`, `Set-OperatorFactStale`
- `Add-OperatorSkill`, `Get-OperatorSkill`, `Set-OperatorSkillStatus`,
  `Add-OperatorSkillUse`
- `Save-OperatorState`, `Get-OperatorState`

## 5. Outlook reader
`Read-OperatorOutlook -Folder Inbox -Count 50` → returns mail objects:
`{entry_id, subject, sender, received, unread, body_excerpt, attachments[]}`.
- Uses Outlook COM; filters `Class -eq 43`; excerpts body to 2000 chars.
- Read-only in Phase 1; acting on mail is Phase 4 via the approval queue.
- Always releases COM objects.

## 6. LLM client
`Invoke-OperatorLlm -Prompt <string>` → string.
- POSTs an OpenAI-compatible chat completion to `llm.endpoint` with the key from
  `$env:<api_key_env>`.
- On failure: log to guardrail, return empty, and let the loop degrade safely.
- Keep prompts small (send only new observations + relevant facts/skills).

## 7. Agent loop
`Invoke-OperatorAgent -Task <string> [-ObsFile <json>]`:
1. Load new observations (from `Read-OperatorOutlook` or a JSON file).
2. For each: if `Test-OperatorSeen` → skip (dedup); else `Add-OperatorRecord`
   as an observation.
3. If `Test-OperatorGuarded -Attempt $Task` → abort with a clear message.
4. Build prompt: task + live facts + available (promoted) skills + new items.
5. Ask the LLM for **JSON only**: `{"actions":[{type,entry_id,subject,note,body}]}`.
6. Split actions: safe (`flag`, `draft_reply`, `summarize`) → outbox;
   irreversible (see config) → pending queue.
7. Record decision + outcome in episodic memory.

## 8. Approval queue
- `Get-OperatorPending`, `Approve-OperatorAction -Id`, `Deny-OperatorAction -Id`.
- `Apply-OperatorApproved` executes approved actions (Phase 4: create an
  Outlook draft; send only if the employer explicitly enables sending).
- Every apply is recorded in episodic memory (audit trail).

## 9. Learning loop (Phase 5)
- `Get-OperatorRepetition`: scan episodic for action sequences repeated ≥ N times
  (same type + similar subject/body via normalized comparison).
- `New-OperatorSkillProposal -From <pattern>`: append a `proposed` skill.
- `Approve-OperatorSkill -Name`: `proposed → trialing`.
- `Update-OperatorSkillStatus`: after K successful trials → `promoted`.
- Front-loaded `library/skills/*.json` (email_triage, form_fill, meeting_notes,
  data_entry, report_generation) loaded on first run.

## 10. Reporting / export (Phase 6)
`Export-OperatorData -Path <dir>` writes:
`memory/*.jsonl`, `results.md`, `guardrails.md`, `skills.md`, `manifest.json`.
This is the artifact returned to the parent Project:Survival system
(see `docs/REPORTING.md`).

## 11. Security and safety
- Secrets via env vars / git-ignored config only.
- Approval gate is mandatory for irreversible actions; default config has
  sending **disabled**.
- No `Invoke-Expression` on external data.
- Integrity failures are fatal (`$ErrorActionPreference='Stop'`).
- Workspace isolation enforced by path.
- The employer can inspect everything: episodic log + guardrails are plain JSONL.
