# lookup_customer

## Purpose

Identifies the caller from the Customers records and returns their stored details, including the Last Appointment ID used by reschedule and cancel.

Documented workflow: [customer-lookup.md](../workflows/customer-lookup.md)

---

# Business Rules

- Identification is automatic: the caller's phone is read from the session. The AI calls this tool directly without asking for a phone number.
- Ask for a phone number only when this tool returns no result AND caller_phone is empty — then ask for the number the customer originally booked with.
- Read-only: never creates or modifies customer data.

---

# Inputs

| Field | Required | Description |
|---|:---:|---|
| caller_phone | Yes | The caller_phone dynamic variable, exactly as-is. The workflow also resolves phone from stored session data. |

---

# Outputs

| Field | Description |
|---|---|
| success | Lookup executed. |
| found | true when a customer record matched. |
| message | Natural-language summary including the Last Appointment ID when present. |
| customerName, customerPhone | Stored identity. |
| appointmentId | Last Appointment ID — use as eventId for reschedule/cancel. |
| vehicleYear, vehicleMake, vehicleModel, vehicle | Stored vehicle. |
| lastService | Most recent service. |
| notes | Stored customer notes. |

---

# Success Responses

found=true with the record fields. Address the caller by name and confirm the found appointment before proceeding.

---

# Failure Responses

found=false with a not-found message. Follow the fallback rule in Business Rules; do not invent a customer.

---

# MISSING INPUT Protocol

Not applicable. This tool has no MISSING INPUT protocol.

---

# Retry Behavior

Retry once after obtaining the originally-booked phone number via the documented fallback. Otherwise do not retry.

---

# Limitations

- One record per phone number; the stored vehicle is the most recent one (single-vehicle limitation, see [vehicles.md](../data-model/vehicles.md)).
- appointmentId may be stale if the appointment was changed outside the system.

---

# Data Touched

| Tab | Access | Detail |
|---|---|---|
| Customers | Read | Row matched by Phone; returns identity, vehicle, Last Service, Last Appointment ID, Notes |
| Sessions | Read | caller_phone resolution (most recent row by stored_at) |

No writes. Appointments, Services, and Call_Log are not touched.

---

# Related Workflows

- [customer-lookup.md](../workflows/customer-lookup.md)
- Downstream: [reschedule_appointment](reschedule_appointment.md), [cancel_appointment](cancel_appointment.md)

---

# Example

Request:

```json
{
  "caller_phone": "+14165551234"
}
```

Response:

```json
{
  "success": true,
  "found": true,
  "message": "Found customer Ivan. Last appointment ID is abc123def456.",
  "customerName": "Ivan",
  "customerPhone": "+14165551234",
  "appointmentId": "abc123def456",
  "vehicleYear": "2005",
  "vehicleMake": "Audi",
  "vehicleModel": "A4",
  "vehicle": "2005 Audi A4",
  "lastService": "cv axle replacement",
  "notes": ""
}
```
