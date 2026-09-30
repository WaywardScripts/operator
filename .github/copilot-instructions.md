# GitHub Copilot instructions — Operator

Read and follow `AGENTS.md` at the repo root as the primary instruction source.
This file adds Copilot-specific guidance.

## Ground rules (repeat of the hard constraints)
- **Scripts only — no `.exe`/`.msi`/`.dll`.** PowerShell `.ps1`/`.psm1` only.
- **No installed dependencies.** Built-in cmdlets and .NET only.
- **Approval gate**: irreversible actions go to a pending queue; never auto-send.
- **No secrets in the repo.**
- **Windows + Outlook COM** is the target.

## How to work
- Implement in the order given by `docs/PLAN.md`; do not start a phase before
  the previous one's tests pass.
- Prefer small commits with clear messages (`phase1: add memory hash chain`).
- When a requirement is ambiguous, choose the safer option (more logging, more
  approval, no irreversible action) and note the decision in `CHANGELOG.md`.
- For each module, add a matching `tests/<Module>.Tests.ps1`.

## Do not
- Do not add package managers, installers, or build steps that produce binaries.
- Do not use `Invoke-Expression` on data read from files, email, or the network.
- Do not silently swallow errors in the memory-integrity code.
- Do not send email or modify Outlook items without going through the approval
  queue.

## Reference
- Architecture: `docs/ARCHITECTURE.md`
- Plan and acceptance criteria: `docs/PLAN.md`
- Export/report format: `docs/REPORTING.md`
- Config template: `config.example.json`
