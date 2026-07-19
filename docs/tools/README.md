# Tool Contracts

The interface between the ElevenLabs agent and the n8n workflow engine. Nine tools are exposed through the MCP server trigger. Parameter names are exact; response fields listed are the ones the AI must act on.

After changing any tool's parameters or description in n8n, re-sync the tool list in the ElevenLabs agent settings (schemas are cached).

---

## estimate_job_ballpark

Purpose: starting price and duration for a service.

Inputs: service (customer's own words, required — never empty), serviceCategory (best-guess snake_case key, optional), vehicleYear, vehicleMake, vehicleModel, serviceDetails.

Response: estimatedMinutes, startingAtPrice, startingAtText, serviceCategory, diagnosticOnly, needsService, servicesLoaded, message.

Protocol: if needsService is true (MISSING INPUT), call again with the service field filled. Relay message as the reply basis — it already contains duration and price.

---

## get_availability

Purpose: check open slots for a requested day/time.

Inputs: request (date/time in the caller's words), estimatedMinutes (default 90), service, serviceCategory, serviceDetails, vehicleYear, vehicleMake, vehicleModel.

Response: message, available, scenario, confirmedStartTime, confirmedEndTime, estimatedMinutes.

Protocol: when a specific time is confirmed, pass confirmedStartTime and confirmedEndTime unchanged (exact ISO strings) into book_appointment or reschedule_appointment.

---

## book_appointment

Purpose: create the appointment after explicit customer confirmation.

Inputs: request, name, vehicleYear, vehicleMake, vehicleModel, service, serviceCategory, serviceDetails, estimatedMinutes, startingAtPrice, startingAtText, confirmedStartTime, confirmedEndTime, caller_phone.

Response: success, booked, message, appointmentId, appointment date/time fields.

Protocol: call exactly once; booked is true only on real success. Rebooking replaces any existing active appointment for the same phone.

---

## reschedule_appointment

Purpose: move an existing appointment.

Inputs: eventId (from lookup_customer), confirmedStartTime, confirmedEndTime, caller_phone.

Response: success, message, new appointment fields.

Protocol: availability must be confirmed first; call exactly once. The workflow resolves a missing eventId from records by caller phone.

---

## cancel_appointment

Purpose: cancel an existing appointment.

Inputs: eventId (from lookup_customer), caller_phone, cancellationReason (optional).

Response: success, message.

Protocol: never call with an empty or unknown eventId; explicit customer confirmation first.

---

## lookup_customer

Purpose: identify the caller and return their record including Last Appointment ID.

Inputs: caller_phone (automatic).

Response: found, customer fields, Last Appointment ID (used as eventId downstream).

Protocol: never ask the caller for their phone number; ask only if this returns nothing and caller_phone is empty.

---

## lookup_appointment

Purpose: retrieve appointment details from the calendar.

Inputs: eventId when known; otherwise matches by caller phone parsed from the event description.

Response: found, appointment details.

Protocol: no cross-customer fallback — an unmatched lookup returns not found.

---

## human_handoff

Purpose: transfer the call to shop staff.

Inputs: caller_phone, reason.

Response: success.

Protocol: two-step confirmation before transfer; on failure, collect name and number for callback.

---

## end_call

Purpose: terminate the call after a farewell.

Inputs: reason.

Response: success.

Protocol: farewell first, then call immediately; required after silence timeout, natural conclusion, abuse, or robocall.
