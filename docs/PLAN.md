# Operator — Build Plan (phased, issue-ready)

Work top to bottom. Each phase must pass its acceptance criteria before the
next begins. Keep `CHANGELOG.md` updated and tick boxes here as you go.

---

## Phase 1 — Config + Memory core
**Deliverables:** `OperatorConfig.ps1`, `OperatorMemory.psm1`, `tests/OperatorMemory.Tests.ps1`

- [ ] `config.example.json` and a config loader resolving workspace_root + workspace.
- [ ] `ConvertTo-CanonicalJson` (recursively sorted keys) for deterministic hashing.
- [ ] `Add-OperatorRecord` with hash chaining (`prev_hash`, `hash`) per store.
- [ ] `Get-OperatorRecords`, `Test-OperatorSeen` (dedup by key).
- [ ] `Test-OperatorMemory` verifies all chains and **throws** on tamper.
- [ ] Guardrail: `Add-OperatorGuard`, `Test-OperatorGuarded`.
- [ ] Facts: `Add-OperatorFact`, `Get-OperatorFacts`, `Set-OperatorFactStale`.
- [ ] Skills: `Add-OperatorSkill`, `Get-OperatorSkill`, `Set-OperatorSkillStatus`,
      `Add-OperatorSkillUse` (append-only snapshots).
- [ ] State: `Save-OperatorState`, `Get-OperatorState`.
- [ ] Workspace isolation (path contains the workspace slug).

**Acceptance**
- [ ] Appending then reading returns records with valid, linked hashes.
- [ ] Manually editing a JSONL record makes `Test-OperatorMemory` fail with the
      broken sequence number.
- [ ] Same `DedupKey` twice stores once.
- [ ] Facts/skills from workspace A are invisible to workspace B.
- [ ] `pwsh -File tests/OperatorMemory.Tests.ps1` (or `powershell -File ...`)
      prints PASS and exits 0.

---

## Phase 2 — Outlook reader
**Deliverables:** `OperatorOutlook.ps1`, `tests/OperatorOutlook.Tests.ps1` (mockable)

- [ ] `Read-OperatorOutlook -Folder -Count` via COM; filter `Class -eq 43`.
- [ ] Fields: `entry_id, subject, sender, received, unread, body_excerpt, attachments`.
- [ ] Release COM objects; handle missing Outlook gracefully.

**Acceptance**
- [ ] Returns a JSON-serializable array for a real inbox.
- [ ] `EntryID` is stable across runs (dedup works end to end).

---

## Phase 3 — LLM client + Agent loop
**Deliverables:** `OperatorAgent.ps1`, `Invoke-Operator.ps1`, `tests/OperatorAgent.Tests.ps1`

- [ ] `Invoke-OperatorLlm` (OpenAI-compatible POST; key from env).
- [ ] `Invoke-OperatorAgent -Task [-ObsFile]`: dedup → guardrail → prompt →
      parse JSON actions → route safe vs irreversible.
- [ ] `Invoke-Operator.ps1` entrypoint (task in, summary out).
- [ ] Prompt sends only new observations + live facts + promoted skills.

**Acceptance**
- [ ] Given a JSON of 3 sample emails, the agent writes safe actions to the
      outbox and irreversible ones to pending.
- [ ] Re-running with the same emails does nothing (dedup).
- [ ] A task in the guardrail store is refused.

---

## Phase 4 — Approval queue + Apply
**Deliverables:** `OperatorApproval.ps1`, `tests/OperatorApproval.Tests.ps1`

- [ ] `Get-OperatorPending`, `Approve-OperatorAction`, `Deny-OperatorAction`.
- [ ] `Apply-OperatorApproved` (default: create Outlook **draft** only; sending
      disabled unless config explicitly enables it).
- [ ] Every apply recorded in episodic memory.

**Acceptance**
- [ ] Approved drafts appear in the Outlook Drafts folder; denied ones do not.
- [ ] Sending is impossible unless `approval.allow_send = true`.

---

## Phase 5 — Learning loop + library
**Deliverables:** `OperatorLearning.ps1`, `library/skills/*.json`, tests

- [ ] `Get-OperatorRepetition` (repeated action sequences ≥ N).
- [ ] `New-OperatorSkillProposal`, `Approve-OperatorSkill`,
      `Update-OperatorSkillStatus` (promote after K successful trials).
- [ ] Ship `library/skills/`: `email_triage.json`, `form_fill.json`,
      `meeting_notes.json`, `data_entry.json`, `report_generation.json`.

**Acceptance**
- [ ] Feeding repeated similar actions produces a `proposed` skill.
- [ ] A skill promoted after K trials appears in the agent prompt's skills list.

---

## Phase 6 — Reporting / export
**Deliverables:** `OperatorReport.ps1`, `tests/OperatorReport.Tests.ps1`

- [ ] `Export-OperatorData -Path <dir>` → `memory/*.jsonl`, `results.md`,
      `guardrails.md`, `skills.md`, `manifest.json`.

**Acceptance**
- [ ] Export runs on a populated workspace and the manifest validates.
- [ ] Matches the format in `docs/REPORTING.md`.
