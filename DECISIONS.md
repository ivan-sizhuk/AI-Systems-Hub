# Decisions

## Google Sheets as the operational datastore

Chosen for owner visibility and zero-cost editing. Services pricing is owner-editable with no deploys. Revisit if multi-shop or reporting needs outgrow it.

## Business configuration in an n8n Set node, not Sheets

The call-initiation webhook must respond within the voice platform's timeout; reading config from Sheets on every call start is too slow and adds a failure mode. Cost: the config exists as three synchronized clones (main / Init / Reminder).

## Create-before-delete rescheduling

The new calendar event is created and verified before the old one is deleted. If creation fails, the customer keeps their original slot. Never invert this order.

## One active appointment per phone number

A new booking from a phone with an existing active appointment replaces it; the old record is marked "Rebooked" and its calendar event deleted. Prevents duplicate bookings from repeated calls; the cost is that one phone cannot hold two future appointments.

## Appointments record as identifier source of truth

Cancellation and rescheduling resolve the calendar event ID from the Appointments record by caller phone (status Booked/Rescheduled only). Identifiers spoken by the AI are fallback only — language models garble IDs.

## Customer ID derived from phone (CUST-<digits>)

Deterministic derivation makes customer writes idempotent create-or-update operations and requires no ID storage in conversation state.

## Estimate not-found returns a bookable response, not an error

A caller naming a service the catalog lacks is offered a booking with technician confirmation instead of an apology. Revenue-preserving by design.
