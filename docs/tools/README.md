# Tool Contracts

The interface between the ElevenLabs agent and the n8n workflow engine. Nine tools are exposed through the MCP server trigger.

Each tool is documented as an API contract in its own file:

- [estimate_job_ballpark](estimate_job_ballpark.md)
- [get_availability](get_availability.md)
- [book_appointment](book_appointment.md)
- [reschedule_appointment](reschedule_appointment.md)
- [cancel_appointment](cancel_appointment.md)
- [lookup_customer](lookup_customer.md)
- [lookup_appointment](lookup_appointment.md)
- [human_handoff](human_handoff.md) — the call-transfer tool
- [end_call](end_call.md)

There is no tool named transfer_call; call transfer is implemented by human_handoff.

---

# Data Access Matrix

Which tools touch which Google Sheets tabs (R = read, W = write):

| Tool | Services | Appointments | Customers | Sessions | Call_Log | Calendar |
|---|:---:|:---:|:---:|:---:|:---:|:---:|
| estimate_job_ballpark | R | — | — | — | — | — |
| get_availability | R | — | — | — | — | R |
| book_appointment | R | R/W | R/W | R | — | W |
| reschedule_appointment | — | R/W | W | R | — | W |
| cancel_appointment | — | R/W | W | R | — | W |
| lookup_customer | — | — | R | R | — | — |
| lookup_appointment | — | — | — | — | — | R |
| human_handoff | — | — | — | — | W | — |
| end_call | — | — | — | R | — | — |

Sessions rows are written at call start by the initiation webhook (not by a tool) and expired rows are deleted by the hourly scheduled cleanup. See the [data model](../data-model/README.md) for tab schemas.

---

# Shared Rules

- Every tool call carries a fixed `action` value that routes it inside the workflow engine. The AI never sets it.
- `caller_phone` is injected automatically at call start. Tools that accept it receive the dynamic variable exactly as-is; the AI never asks the caller for it except the documented fallback in [customer-lookup](../workflows/customer-lookup.md).
- Tool responses are structured JSON. The `message` field is always the natural-language basis for the AI's reply.
- Never announce success unless the response says so (`success`, `booked`, `rescheduled`, `canceled`, `transfer_initiated`).
- After changing any tool's parameters or description in n8n, re-sync the tool list in the ElevenLabs agent settings (schemas are cached). See [OPERATIONS.md](../../OPERATIONS.md).
