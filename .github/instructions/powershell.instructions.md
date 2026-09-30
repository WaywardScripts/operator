---
applyTo: "**/*.ps1,**/*.psm1"
---
# PowerShell conventions for Operator

- Target **Windows PowerShell 5.1** and be compatible with PowerShell 7.
- Every public function: `[CmdletBinding()]`, a `param()` block with typed
  parameters, and comment-based help (`.SYNOPSIS`, `.DESCRIPTION`, `.PARAMETER`,
  `.EXAMPLE`).
- Use approved verbs (`Get-`, `Set-`, `New-`, `Add-`, `Invoke-`, `Export-`,
  `Test-`).
- JSON: use `-Depth` explicitly with `ConvertTo-Json` (default depth truncates).
  For hashing, serialize a **canonical** (sorted-key) representation — implement
  a `ConvertTo-CanonicalJson` helper and use it everywhere a hash is computed.
- Hashing: `[System.Security.Cryptography.SHA256]::Create()` over UTF-8 bytes;
  return lowercase hex.
- File appends: use `Add-Content -Encoding UTF8`. Never rewrite an append-only
  store in place.
- Outlook COM: always release objects
  (`[Runtime.InteropServices.Marshal]::ReleaseComObject(...)`) and handle
  `$item.Class -eq 43` (olMail) before accessing mail properties.
- HTTP: `Invoke-RestMethod` for LLM calls; read keys from environment variables.
- Errors: `$ErrorActionPreference = 'Stop'` in scripts; use `try/catch` and
  rethrow with context. Integrity failures must be fatal.
- No `Invoke-Expression` on external data.
- Tests: `tests/<Module>.Tests.ps1`, runnable via a documented one-liner; no
  external test framework required.
