# ATLAS.Compliance.Logging

> **Automation Trust Layer for Agency Solutions — Compliance Logging Library**  
> UiPath Library · v0.1.0 · C# · Studio 2026.x STS · Windows

A federal-focused UiPath activity library that gives RPA developers drop-in workflows for NIST 800-53 Rev. 5 audit logging, PII redaction, tamper-evident hash chaining, records retention stamping, and ATO evidence packaging — all without writing a line of compliance infrastructure code.

[![GitHub release](https://img.shields.io/github/v/release/kkapula4/atlas-compliance-logging)](https://github.com/kkapula4/atlas-compliance-logging/releases/tag/v0.1.0)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)

---

## Table of Contents

- [Why ATLAS?](#why-atlas)
- [NIST 800-53 Control Coverage](#nist-800-53-control-coverage)
- [Requirements](#requirements)
- [Installation](#installation)
- [Workflows](#workflows)
  - [Get_RunContext](#1-get_runcontext)
  - [Log_NIST_AuditEvent](#2-log_nist_auditevent)
  - [Redact_PII_Text](#3-redact_pii_text)
  - [Hash_And_Chain_Audit](#4-hash_and_chain_audit)
  - [Build_AuditEvidencePacket](#5-build_auditevidencepacket)
  - [Apply_Retention_Policy](#6-apply_retention_policy)
- [Quick Start](#quick-start)
- [Output Schemas](#output-schemas)
- [Sample Process](#sample-process)
- [Roadmap](#roadmap)
- [License](#license)

---

## Why ATLAS?

Federal RPA programs face recurring compliance friction before every ATO review:

- Audit logs scattered across robot runs with no tamper-evidence
- PII redaction done ad hoc or not at all
- Evidence packages assembled manually the week before an audit
- Retention schedules tracked in spreadsheets

ATLAS.Compliance.Logging solves all four in a single NuGet package. Drop the workflows into any UiPath process, wire 3–4 arguments, and your automation produces machine-readable, auditor-ready evidence on every run.

---

## NIST 800-53 Control Coverage

| Control | Title | Workflows |
|---|---|---|
| AU-2 | Event Logging | Get_RunContext, Log_NIST_AuditEvent |
| AU-3 | Content of Audit Records | Log_NIST_AuditEvent, Redact_PII_Text |
| AU-4 | Audit Log Storage Capacity | Build_AuditEvidencePacket |
| AU-7 | Audit Record Reduction & Report Generation | Log_NIST_AuditEvent |
| AU-9 | Protection of Audit Information | Hash_And_Chain_Audit |
| AU-11 | Audit Record Retention | Apply_Retention_Policy, Build_AuditEvidencePacket |
| AU-12 | Audit Record Generation | Log_NIST_AuditEvent, Hash_And_Chain_Audit, Build_AuditEvidencePacket |
| AC-21 | Information Sharing | Get_RunContext |
| SI-7 | Software, Firmware, and Information Integrity | Hash_And_Chain_Audit, Redact_PII_Text |
| SI-11 | Error Handling | Log_NIST_AuditEvent |
| SI-12 | Information Management and Retention | Apply_Retention_Policy |

---

## Requirements

| Requirement | Version |
|---|---|
| UiPath Studio | 2026.x STS (Windows) |
| UiPath.System.Activities | >= 26.2.4 |
| .NET target | Windows / net6.0-windows |
| Newtonsoft.Json | Transitive via UiPath.System.Activities — no explicit install needed |

---

## Installation

### Option 1 — NuGet local feed (recommended for air-gapped environments)

1. Download `ATLAS.Compliance.Logging.0.1.0.nupkg` from the [v0.1.0 release](https://github.com/kkapula4/atlas-compliance-logging/releases/tag/v0.1.0)
2. In UiPath Studio: **Manage Packages → Settings → Add source**
   - Name: `ATLAS Local`
   - Source: folder containing the `.nupkg`
3. Search for `ATLAS.Compliance.Logging` → Install → Save

### Option 2 — UiPath Marketplace

Search `ATLAS Compliance Logging` in the UiPath Marketplace tab inside Manage Packages.

---

## Workflows

### 1. Get_RunContext

Captures runtime environment metadata (machine name, user, Studio version, process name) into a `Dictionary<String, Object>` for embedding in every audit event.

| Argument | Direction | Type | Default | Description |
|---|---|---|---|---|
| `in_ProcessName` | In | String | `""` | Caller process name; auto-detected if blank |
| `in_AdditionalContext` | In | `Dictionary<String,Object>` | `null` | Caller-supplied extra key-value pairs |
| `out_RunContext` | Out | `Dictionary<String,Object>` | — | Populated runtime context dictionary |

**NIST:** AU-2, AU-3, AC-21

---

### 2. Log_NIST_AuditEvent

Writes a single structured JSONL audit event to a file. Supports optional PII redaction via `Redact_PII_Text` when `in_RedactPII=True`.

| Argument | Direction | Type | Default | Description |
|---|---|---|---|---|
| `in_EventType` | In | String | (required) | Event label e.g. `"DocumentProcessed"` |
| `in_Payload` | In | `Dictionary<String,Object>` | (required) | Event data to log |
| `in_OutputPath` | In | String | `""` | Output `.jsonl` path; auto-generates temp path if blank |
| `in_RedactPII` | In | Boolean | `False` | If True, runs payload through Redact_PII_Text before writing |
| `in_ControlIds` | In | `String[]` | `null` | NIST control IDs to embed e.g. `["AU-2","AU-3"]` |
| `in_RunContext` | In | `Dictionary<String,Object>` | `null` | Run context from Get_RunContext |
| `in_SchemaVersion` | In | String | `"atlas.compliance.audit.v1"` | Schema version stamp |
| `out_AuditEventId` | Out | String | — | GUID of the written event |
| `out_WrittenTimestamp` | Out | DateTime | — | UTC timestamp of the write |
| `out_OutputPath` | Out | String | — | Resolved path to the `.jsonl` file |

**NIST:** AU-2, AU-3, AU-7, AU-9, AU-12, SI-11

---

### 3. Redact_PII_Text

Redacts emails, US phone numbers, SSNs, and account numbers from any string using configurable regex patterns.

| Argument | Direction | Type | Default | Description |
|---|---|---|---|---|
| `in_InputText` | In | String | (required) | Text to redact |
| `in_RedactEmails` | In | Boolean | `True` | Redact email addresses |
| `in_RedactPhones` | In | Boolean | `True` | Redact US phone numbers |
| `in_RedactSSN` | In | Boolean | `True` | Redact Social Security Numbers |
| `in_RedactAccountNumbers` | In | Boolean | `True` | Redact 8–17 digit account numbers |
| `in_Replacement` | In | String | `"[REDACTED]"` | Replacement token |
| `out_RedactedText` | Out | String | — | Redacted output text |
| `out_RedactionCount` | Out | Int32 | — | Number of redactions applied |
| `out_DetectedTypes` | Out | `String[]` | — | Types of PII detected e.g. `["EMAIL","SSN"]` |

**NIST:** AU-3, SI-7, AU-12

---

### 4. Hash_And_Chain_Audit

Computes a SHA-256 hash of an audit log line, chains it to a previous hash, and appends a hash chain record to a `.hashchain` sidecar file.

| Argument | Direction | Type | Default | Description |
|---|---|---|---|---|
| `in_AuditLogLine` | In | String | (required) | The raw JSONL line to hash |
| `in_PreviousHash` | In | String | `""` | Hash of the previous entry; empty string for first entry |
| `in_HashChainPath` | In | String | `""` | Path to `.hashchain` file; auto-derived if blank |
| `out_CurrentHash` | Out | String | — | SHA-256 hex digest of this entry |
| `out_ChainedHash` | Out | String | — | SHA-256 of (PreviousHash + CurrentLine) |
| `out_HashChainPath` | Out | String | — | Resolved path to the `.hashchain` file |

**NIST:** AU-9, SI-7, AU-12

---

### 5. Build_AuditEvidencePacket

Packages an audit log file and a SHA-256 manifest into a single ZIP file for federal records handoff. Refuses to overwrite an existing ZIP — atomic creation only.

| Argument | Direction | Type | Default | Description |
|---|---|---|---|---|
| `in_AuditLogPath` | In | String | (required) | Path to the `.jsonl` audit log |
| `in_OutputZipPath` | In | String | (required) | Destination path for the evidence ZIP |
| `in_PacketId` | In | String | `""` | Optional packet ID; auto-generated if blank |
| `out_PacketId` | Out | String | — | Resolved packet ID |
| `out_ZipPath` | Out | String | — | Path to the created ZIP |
| `out_ManifestJson` | Out | String | — | Full manifest JSON string |

**NIST:** AU-4, AU-11, AU-12

---

### 6. Apply_Retention_Policy

Writes a `.retention.json` sidecar file alongside any target file, stamping it with retention class, effective date, expiry date, and stamping agent. Library-agnostic — no hardcoded NARA values.

| Argument | Direction | Type | Default | Description |
|---|---|---|---|---|
| `in_TargetFilePath` | In | String | (required) | Path to the file being stamped |
| `in_RetentionClass` | In | String | (required) | Retention class e.g. `"GRS-3.1-100"` |
| `in_RetentionYears` | In | Int32 | (required) | Retention period in years; must be > 0 |
| `in_StampedBy` | In | String | `""` | Stamping agent identity; defaults to `Environment.UserName` |
| `out_SidecarPath` | Out | String | — | Path to the written `.retention.json` file |
| `out_ExpiryDate` | Out | DateTime | — | Calculated UTC expiry date |

**NIST:** AU-11, SI-12

---

## Quick Start

```xml
<!-- 1. Capture run context -->
<InvokeWorkflowFile WorkflowFileName="Activities/Get_RunContext.xaml">
  <Arguments>
    <x:Reference>out_RunContext</x:Reference>
  </Arguments>
</InvokeWorkflowFile>

<!-- 2. Log an audit event -->
<InvokeWorkflowFile WorkflowFileName="Activities/Log_NIST_AuditEvent.xaml">
  <Arguments>
    in_EventType="DocumentProcessed"
    in_Payload="{your payload dict}"
    in_RedactPII="True"
    in_RunContext="{out_RunContext}"
    in_OutputPath="C:\audit\audit.jsonl"
  </Arguments>
</InvokeWorkflowFile>

<!-- 3. Hash and chain the event -->
<InvokeWorkflowFile WorkflowFileName="Activities/Hash_And_Chain_Audit.xaml">
  <Arguments>
    in_AuditLogLine="{last written line}"
    in_PreviousHash="{previous chain hash}"
  </Arguments>
</InvokeWorkflowFile>

<!-- 4. Apply retention policy -->
<InvokeWorkflowFile WorkflowFileName="Activities/Apply_Retention_Policy.xaml">
  <Arguments>
    in_TargetFilePath="C:\audit\audit.jsonl"
    in_RetentionClass="GRS-3.1-100"
    in_RetentionYears="7"
  </Arguments>
</InvokeWorkflowFile>

<!-- 5. Package evidence -->
<InvokeWorkflowFile WorkflowFileName="Activities/Build_AuditEvidencePacket.xaml">
  <Arguments>
    in_AuditLogPath="C:\audit\audit.jsonl"
    in_OutputZipPath="C:\audit\evidence.zip"
  </Arguments>
</InvokeWorkflowFile>
```

See the [ATLAS.Sample.AuditLoggingDemo](https://github.com/kkapula4/atlas-sample-audit-logging-demo) repository for a fully working end-to-end example.

---

## Output Schemas

### Audit Event (`atlas.compliance.audit.v1`)

```json
{
  "audit_event_id":  "3f1a9c22-...",
  "written_at":      "2026-05-18T18:16:49.0000000Z",
  "event_type":      "DocumentProcessed",
  "schema_version":  "atlas.compliance.audit.v1",
  "redact_pii":      true,
  "control_ids":     ["AU-2", "AU-3"],
  "payload":         { "customer": "[REDACTED-EMAIL]", "ssn": "[REDACTED-SSN]" },
  "run_context":     { "machine": "WORKSTATION-01", "run_by": "karth" }
}
```

### Evidence Manifest (`atlas.compliance.evidence.v1`)

```json
{
  "schema_version": "atlas.compliance.evidence.v1",
  "packet_id":      "ATLAS-PKT-20260518-181649-A1B2C3D4",
  "built_at":       "2026-05-18T18:16:49.0000000Z",
  "nist_controls":  ["AU-4", "AU-11", "AU-12"],
  "files": [
    { "name": "audit.jsonl", "bytes": 1024, "sha256": "e3b0c44298fc..." }
  ]
}
```

### Retention Sidecar (`atlas.compliance.retention.v1`)

```json
{
  "schema_version":  "atlas.compliance.retention.v1",
  "target_file":     "audit.jsonl",
  "retention_class": "GRS-3.1-100",
  "retention_years": 7,
  "effective_date":  "2026-05-18T18:16:49.0000000Z",
  "expiry_date":     "2033-05-18T18:16:49.0000000Z",
  "stamped_by":      "karth",
  "nist_controls":   ["AU-11", "SI-12"]
}
```

---

## Sample Process

[ATLAS.Sample.AuditLoggingDemo](https://github.com/kkapula4/atlas-sample-audit-logging-demo) demonstrates all 6 workflows end-to-end in a single UiPath process. Clone it, install the ATLAS.Compliance.Logging package, and run `Main.xaml` to see:

```
=== ATLAS Demo Complete ===
Events logged  : 2
PII redacted   : 1 event
Chain hash 1   : bfbabd0a2db849e3...
Chain hash 2   : 9a59d9423da4635c...
Retention file : C:\ATLAS\test-out\audit.jsonl.retention.json
Evidence ZIP   : C:\ATLAS\test-out\evidence.zip
Packet ID      : ATLAS-PKT-DEMO-20260518-181649
Execution time : 00:00:04
```

---

## Roadmap

| Version | Planned |
|---|---|
| v0.2.0 | NARA GRS lookup table (optional); Orchestrator queue output target for Log_NIST_AuditEvent |
| v0.3.0 | Test workflows for all 6 activities; CI/CD pipeline with UiPath CLI |
| v1.0.0 | Marketplace-certified release with full test coverage |

---

## License

MIT License — see [LICENSE](LICENSE) for details.

> **NARA Disclaimer:** Retention class examples (e.g. GRS-3.1-100) are for illustration only. This library does not constitute legal or records-management advice. Agency personnel are responsible for selecting the correct retention schedule for their record series.

---

*Built by [Karthik Kapula](https://github.com/kkapula4) · UiPath MVP · ATLAS Sprint 2 · 2026*
