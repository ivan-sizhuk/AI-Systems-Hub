# TODO

Documentation gaps identified by the Documentation Audit and not yet created:


- BUSINESS_CONFIG.md — full schema of the Business Config node and the three-clone update rule (partially covered by docs/integrations/n8n.md).
- Multi-vehicle support decision — the data model documents the single-vehicle-per-customer limitation; decide whether to lift it.
- Keep production/ artifacts synchronized on every deploy (see OPERATIONS.md).

Instrumentation gaps identified by the Production Monitoring Framework (monitoring/METRICS.md):

- Per-call outcome record — ADDRESSED by workflow V27.0 Call Record instrumentation (docs/data-model/call-records.md). Pending QA and release.
- ElevenLabs Analysis tab configuration — add call_outcome, call_intent, customer_frustration, and tool_failures data-collection fields. Console change only; the workflow already reads them and records not_configured until they exist.
- Verify transcript tool-call extraction — the transcript shape is platform-defined and undocumented here. The Call Record parser scans defensively, reports unknown on mismatch, and records the actual structure in the Payload Shape column. Read that column after the first live calls, confirm or correct the extraction, then remove the diagnostic column as schema v2.
- Call_Records duplicate rows — the append is unconditional, so a retried post-call webhook writes a second row for the same Conversation ID. Duplicates are detectable and removable by Conversation ID at analysis time. Decide whether deduplication is worth the added read and failure mode.
- Call_Records retention — one row per call, unbounded. Decide a retention or archival policy before the sheet approaches Google Sheets cell limits.
- Hallucination detection — covered by sampled manual transcript review until automated checking exists.
