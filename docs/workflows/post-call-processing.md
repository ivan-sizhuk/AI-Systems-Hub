# Post-Call Processing Workflow

## Purpose

The Post-Call Processing workflow performs all required cleanup and finalization tasks after a customer conversation has ended.

Its purpose is to ensure that conversation data, appointment records, customer information, analytics, and system state remain synchronized for future interactions.

This workflow executes after the customer disconnects or the conversation is otherwise completed.

It does not interact directly with the customer.

---

# Trigger

The workflow begins automatically after the conversation ends.

The workflow executes regardless of whether the call resulted in:

- Appointment booked
- Appointment rescheduled
- Appointment cancelled
- Human handoff
- Information request only
- Unsuccessful conversation

---

# Purpose Within the System

The Post-Call Processing workflow is responsible for:

- Finalizing the conversation
- Saving the conversation summary to the appointment record (update-only, matched by caller phone)
- Saving a timestamped conversation summary to the customer record
- Writing one structured [Call Record](../data-model/call-records.md) per conversation (added V27.0)
- Preparing the system for future interactions

Session cleanup is not performed here; it runs on the scheduled reminder cycle.

It is **not responsible** for:

- Booking appointments
- Modifying appointments
- Customer conversations
- Sending reminders
- Sending appointment notifications

Those responsibilities belong to other workflows.

---

# Preconditions

Before this workflow executes:

- The conversation has ended.
- The final workflow state is known.
- Any active tool executions have completed.
- Session information is available.

---

# Required Inputs

The workflow uses:

## Conversation

- Session ID
- Conversation status
- Conversation outcome

## Customer

- Customer ID (if available)
- Phone number

## Appointment

- Appointment ID (if applicable)
- Final appointment status

## Workflow

- Executed workflow
- Tool execution results
- Error information (if applicable)

---

# Business Rules

The workflow follows these rules.

## Notes Rules

The conversation summary comes from the voice platform's post-call webhook (call summary field).

Appointment notes are updated only for an existing appointment row matched by caller phone; no new appointment rows are created by this workflow.

Customer notes are written with a timestamp prefix.

Matching by phone means the notes land on the first matching record when a caller has multiple records.

---

## Cleanup Rules

Temporary workflow data should not persist after the conversation completes.

Only permanent business records should remain.

---

## Logging Rules

Important workflow events should be recorded for troubleshooting and future analysis.

---

## Call Record Rules

One [Call Record](../data-model/call-records.md) row is appended per completed conversation, after note writing, whether or not the call booked anything.

Fields that cannot be determined are written as sentinels (`unknown`, `not_configured`) and never left blank — a blank is indistinguishable from a zero, and every operational rate is calculated from these rows.

Call Record writing must never affect note writing. Both the record-building and the append step continue on error, so an instrumentation failure loses that row and nothing else.

---

# Conversation Contract

There is no active customer conversation during this workflow.

The AI has already completed the interaction.

All processing occurs entirely in the background.

---

# Processing Sequence

## Step 1

Detect conversation completion.

---

## Step 2

Determine final conversation outcome.

---

## Step 3

Save any required conversation information.

---

## Step 4

Update business records.

---

## Step 5

Record analytics.

---

## Step 6

Clean up temporary session data.

---

## Step 7

Release system resources.

---

## Step 8

Return completion status.

---

# External Systems

This workflow communicates with:

## Google Sheets

Responsible for:

- Conversation records
- Analytics
- Workflow history
- Business reporting

---

## Session Storage

Responsible for:

- Session cleanup
- Temporary data removal

---

# Outputs

Successful execution returns:

- Conversation finalized
- Session cleaned
- Analytics updated
- Business records synchronized
- Structured completion response

---

# Success Response

A successful execution records:

- Final conversation outcome
- Completion timestamp
- Workflow summary
- Analytics status
- Cleanup status

---

# Failure Conditions

The workflow may fail when:

## Session Failures

- Session cannot be located.
- Session cleanup fails.

---

## Data Failures

- Business records unavailable.
- Analytics unavailable.

---

## System Failures

- Google Sheets unavailable.
- Internal workflow error.
- External API failure.

---

# Error Handling

When post-call processing fails:

- Customer-facing workflows remain unaffected.
- Partial cleanup should be avoided whenever possible.
- Structured error information is recorded.
- Failed processing may be retried according to business rules.

---

# Dependencies

This workflow depends on:

- Session Management
- Google Sheets
- Analytics System
- Conversation Records

---

# Related Workflows

## Upstream

- Booking
- Availability
- Service Estimates
- Customer Lookup
- Rescheduling
- Cancellation
- Human Handoff
- Reminder Engine

Every workflow ultimately concludes with Post-Call Processing.

---

# Engineering Notes

## Source of Truth

Conversation records and business data are the authoritative source for completed interactions.

Post-Call Processing ensures that all completed workflows leave the system in a consistent state.

---

## Design Principles

The workflow follows several architectural principles.

- Single responsibility
- Background execution
- Reliable cleanup
- Structured logging
- Session lifecycle management
- Separation of customer interaction and backend processing
- Consistent system state after every conversation

Future modifications should preserve these principles unless the post-processing architecture is intentionally redesigned.
