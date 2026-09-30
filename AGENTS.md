# AGENTS.md — Operator build instructions

You are the coding agent building **Operator**: an autonomous work-agent that
reads Office data (Outlook first), reasons about the job, learns to automate
it, and requires human approval before irreversible actions. The full design is
in `docs/ARCHITECTURE.md`; the build order is in `docs/PLAN.md`.

## Mission
Build a **portable, dependency-free PowerShell toolkit** that lets an
employee automate their own work on an **employer-approved** basis, with a
tamper-evident memory that lets it learn the job over time.

## Non-negotiable constraints
1. **Scripts only. No executables.** Never produce `.exe`, `.msi`, `.dll`, or
   anything that must be compiled/installed. Deliver `.ps1` / `.psm1`
   (and `.vba` only if explicitly needed). This is an employer rule.
2. **No installed dependencies.** Use built-in PowerShell 5.1+ cmdlets and
   .NET types only. Do not require external modules. Tests must run without
   installing anything (plain `.Tests.ps1`; use Pester only if already present).
3. **Windows + Outlook** are the first target. Use Outlook COM
   (`New-Object -ComObject Outlook.Application`).
4. **Transparency, not stealth.** This is an approved job aid. Everything it
   does must be logged and inspectable. Never hide activity from the employer.
5. **Approval gate.** Irreversible actions (send, delete, move, submit, pay,
   external contact) must be written to a **pending-approval queue** and never
   executed automatically. Safe actions (flag, draft, summarize, read) may
   proceed.
6. **No secrets in the repo.** API keys/tokens come from environment variables
   or a config file that is git-ignored. Never commit credentials.
7. **Workspace isolation.** All memory/data lives under its own workspace
   directory. Nothing crosses workspaces.

## Coding standards
- PowerShell 5.1 compatible (also run on PowerShell 7 if present).
- Prefer `[CmdletBinding()]` advanced functions with `param()` blocks and
  comment-based help for public functions.
- Deterministic, sorted output where hashing or comparison is involved.
- Fail loudly on integrity errors; never silently continue on a broken memory
  chain.
- No `Invoke-Expression` on untrusted input. Validate before acting.
- Keep functions small and single-purpose; one module per component (see
  architecture).

## Required workflow
1. Read `docs/ARCHITECTURE.md` and `docs/PLAN.md` before writing code.
2. Implement one phase at a time; do not skip ahead.
3. After each phase: run its acceptance tests (`tests/*.Tests.ps1`), then
   summarize what changed and any deviations.
4. Update `docs/PLAN.md` checkboxes as you complete tasks.
5. Keep `CHANGELOG.md` current (one line per meaningful change).

## Definition of done (per phase)
- Code is `.ps1`/`.psm1`, no exe, no installed deps.
- Tests exist and pass, runnable with a single command documented in the phase.
- The phase's acceptance criteria in `docs/PLAN.md` are met.
- No secrets committed; `.gitignore` covers config/outbox/state.

## Reporting back
When a milestone is reached, produce the export described in
`docs/REPORTING.md` (memory stores + results + metrics). That artifact is how
the parent "Project:Survival" system ingests your work.

## First task
Start at **Phase 1** in `docs/PLAN.md`: the memory core
(`OperatorMemory.psm1`) with hash-chained append-only JSONL stores, workspace
isolation, dedup, guardrails, facts, and the skills lifecycle — plus tests.
