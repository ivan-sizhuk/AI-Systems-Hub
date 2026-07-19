# estimate_job_ballpark

## Purpose

Returns the starting price and estimated duration for a requested service, matched against the Services catalog.

Documented workflow: [service-estimates.md](../workflows/service-estimates.md)

---

# Business Rules

- Call only when the customer asks about price, cost, or timing — or when a duration is needed before checking availability.
- Estimates are starting points, never quotes. The response message already frames this.
- Matching runs against the Services catalog: exact key match, then unambiguous keyword routing (synthetic oil, rotors, all-four brake bundle), then scored word matching with stemming and a rare-word bonus.
- A named service missing from the catalog returns a bookable no-price response, not an error.

---

# Inputs

| Field | Required | Description |
|---|:---:|---|
| service | Yes | The job in the customer's own words, keeping descriptors like front/rear/left/right/all four. Example: "front ball joint replacement". Never empty. |
| vehicleYear | Yes | Vehicle year, e.g. "2012". |
| vehicleMake | Yes | Vehicle make, e.g. "Infiniti". |
| vehicleModel | Yes | Vehicle model, e.g. "G37". |
| serviceCategory | No | Best-guess snake_case catalog key, e.g. "ball_joint". Leave empty if unsure. |
| serviceDetails | No | Short summary of intake answers. Used as a raw-text fallback when service is empty. |
| brakeAxle, brakeParts, tireServiceType, tiresOnRims | No | Legacy clarifier fields; matching works without them. |

---

# Outputs

| Field | Description |
|---|---|
| success | Always true in the response wrapper; failures are signaled through message. |
| fallback | true when the diagnostic default was used because no service could be identified. |
| diagnosticOnly | true when a vague symptom was routed to a diagnostic recommendation. |
| estimatedMinutes | Duration in minutes. 90 when unknown. |
| startingAtPrice | Starting price number. Empty when no price is available. |
| startingAtText | Spoken price text, e.g. "$449 CAD and up". Empty when no price. |
| serviceCategory | Matched catalog key, e.g. "cv_joint". |
| disclaimer | Pricing disclaimer text. |
| message | Natural-language reply basis. Contains duration and price when matched. |

---

# Success Responses

Matched service: message contains the duration and starting price, ends with the technician confirmation line, and offers to book.

Vague symptom (diagnosticOnly=true): message recommends a diagnostic appointment.

Named service not in catalog: message offers to book the service with technician confirmation at the shop; no price is given.

---

# Failure Responses

There is no error-shaped response. Two degraded states are delivered through message:

- MISSING INPUT — see protocol below.
- Pricing sheet unavailable: message offers booking with technician confirmation and states the pricing sheet was unavailable for this call.

---

# MISSING INPUT Protocol

If the tool is called with both service and serviceCategory empty, the message begins with "MISSING INPUT" and instructs a re-call with the service field filled from the conversation.

The AI must immediately call the tool again with the service field populated, without telling the customer.

---

# Retry Behavior

- MISSING INPUT: retry immediately, once, with fields filled.
- Any other response: do not retry; relay the message.

---

# Limitations

- One service per call. Multiple requested jobs require separate calls.
- Quantity is not priced: "two ball joints" returns the per-unit starting price (labels marked "(each)").
- Matching quality depends on catalog label wording; see [services.md](../data-model/services.md).

---

# Data Touched

| Tab | Access | Columns used |
|---|---|---|
| Services | Read (all rows, every call) | key, label, duration_minutes, starting_at |

No writes. Appointments, Customers, Sessions, and Call_Log are not touched.

---

# Related Workflows

- [service-estimates.md](../workflows/service-estimates.md)
- Downstream: [availability.md](../workflows/availability.md), [booking.md](../workflows/booking.md)

---

# Example

Request:

```json
{
  "service": "replace both front axles",
  "serviceCategory": "cv_joint",
  "vehicleYear": "2005",
  "vehicleMake": "Audi",
  "vehicleModel": "A4"
}
```

Response:

```json
{
  "success": true,
  "fallback": false,
  "diagnosticOnly": false,
  "estimatedMinutes": 150,
  "startingAtPrice": 379,
  "startingAtText": "$379 CAD and up",
  "serviceCategory": "cv_joint",
  "disclaimer": "Ballpark starting point only. Final time and price depend on inspection, parts availability, rust, vehicle condition, and shop approval.",
  "message": "Got it — a cv joint / axle shaft for a 2005 Audi A4 usually takes about 150 minutes, starting around $379 CAD. That's a rough starting point — the technician will confirm the final price after inspecting the vehicle. Would you like to book an appointment?"
}
```
