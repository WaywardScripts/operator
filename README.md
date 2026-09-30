# Operator — Copilot handoff package

This directory is a **self-contained starter repo** for building **Operator**:
an autonomous work-agent that reads Office data (starting with Outlook),
reasons about the job, learns to automate it, and asks for approval before
anything irreversible.

**You build it on the work machine with GitHub Copilot; results come back here.**

## How to use this package

1. On the work machine, create a new **private** repo (e.g. `operator`).
2. Copy the **contents** of this folder to the repo root:
   - `AGENTS.md`, `README.md`, `config.example.json`
   - `.github/copilot-instructions.md`
   - `.github/instructions/powershell.instructions.md`
   - `docs/ARCHITECTURE.md`, `docs/PLAN.md`, `docs/REPORTING.md`
3. Open it in VS Code with Copilot (agent mode), or use the GitHub Copilot
   coding agent. Copilot will read `AGENTS.md` and
   `.github/copilot-instructions.md` automatically.
4. Work through `docs/PLAN.md` phase by phase. Each phase has tasks and
   acceptance criteria — turn them into issues or prompts.
5. When you have results, follow `docs/REPORTING.md` and bring the export
   back here.

## Hard constraints (also in AGENTS.md)

- **Scripts only. No executables** (`.exe`, `.msi`, `.dll` are forbidden).
  Deliverable is PowerShell (`.ps1`/`.psm1`), optionally VBA.
- **No external dependencies that require installation.** Use built-in
  PowerShell 5.1+ cmdlets only. (Pester optional; plain `.Tests.ps1` otherwise.)
- **Employer-approved job aid**: transparent, auditable, approval-gated for
  irreversible actions. No stealth.
- **No secrets in the repo.** API keys come from environment variables or a
  config file outside version control.
- **Windows + Outlook** is the first target.

## What "done" looks like for the pilot

`Invoke-Operator.ps1 -Task "triage today's inbox and draft replies"` reads
Outlook, dedups against memory, reasons, and writes drafts to an outbox —
with anything irreversible waiting in a pending-approval queue.
