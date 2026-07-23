# TODO

Documentation gaps identified by the Documentation Audit and not yet created:


- BUSINESS_CONFIG.md — full schema of the Business Config node and the three-clone update rule (partially covered by docs/integrations/n8n.md).
- Multi-vehicle support decision — the data model documents the single-vehicle-per-customer limitation; decide whether to lift it.
- Keep production/ artifacts synchronized on every deploy (see OPERATIONS.md).

Instrumentation gaps identified by the Production Monitoring Framework (monitoring/METRICS.md):

- Per-call outcome record — post-call processing writes a conversation summary but no structured outcome. A durable per-call record (outcome, intent, tools fired, resolution) would make conversation abandonment, intent misunderstanding, missing tool calls, and customer frustration measurable. Currently none of the four can be measured.
- Hallucination detection — covered by sampled manual transcript review until automated checking exists.
