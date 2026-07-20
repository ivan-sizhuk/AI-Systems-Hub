# Customer Lookup Workflow

## Purpose

The Customer Lookup workflow identifies whether the caller is an existing customer and retrieves any available customer and appointment information before continuing the conversation.

This workflow reduces unnecessary questions by reusing previously stored customer information whenever possible.

The workflow serves as the primary entry point for customer identification before booking, rescheduling, cancellation, or appointment lookup.

This workflow **never creates or modifies customer data**. Its responsibility is to retrieve existing information only.

---

# Trigger

The workflow begins when the AI calls the `lookup_customer` tool.

The caller's phone number is injected automatically at call start (caller_phone dynamic variable). The AI calls this tool immediately without asking for a phone number.

Typical triggers include:

- Beginning of a booking conversation
- Existing customer calls
- Customer wants to reschedule
- Customer wants to cancel
- Customer asks about an existing appointment
- Customer requests service history

---

# Purpose Within the System

The Customer Lookup workflow is responsible for:

- Identifying existing customers
- Retrieving stored customer information
- Retrieving previous vehicle information
- Retrieving customer notes
- Returning customer identifiers
- Reducing duplicate data collection

It is **not responsible** for:

- Creating customers
- Updating customers
- Booking appointments
- Checking availability
- Cancelling appointments
- Rescheduling appointments

Those responsibilities belong to other workflows.

---

# Preconditions

Before this workflow executes:

## Customer Information

No information needs to be collected. The caller's phone number is available automatically (caller_phone), and the workflow also resolves it from stored session data when needed.

The AI asks for a phone number only when the lookup returns no result AND caller_phone is empty. In that case it asks for the number the customer originally booked with.

The phone number is the primary identifier used to search for customer records.

---

# Required Inputs

The workflow accepts:

## Customer

- Phone number

Additional information may be used for validation if required, but the primary lookup key is the customer's phone number.

---

# Business Rules

The workflow follows these rules.

## Lookup Rules

Customer searches should always be performed using the caller's phone number (automatic first, manually provided only as the fallback described above).

If multiple customer records exist, the workflow should return the best matching record.

If no customer is found, the workflow returns a "customer not found" response instead of creating a new customer.

---

## Customer Data Rules

Existing customer information should be reused whenever possible.

Previously stored information may include:

- Customer name
- Phone number
- Vehicles
- Previous appointments
- Customer notes

The AI should avoid asking for information that already exists unless confirmation is required.

---

## Privacy Rules

Only information required to continue the current conversation should be presented.

The workflow should never expose unnecessary customer information.

---

# Conversation Contract

The AI must:

1. Ask for the customer's phone number.
2. Call the Customer Lookup workflow.
3. Wait for the lookup response.
4. Reuse existing customer information whenever appropriate.
5. Continue naturally based on whether the customer exists.

If the customer exists:

- Welcome them back naturally.
- Confirm stored information only when necessary.

If the customer does not exist:

- Continue gathering customer information as a new customer.

The AI must **never**:

- Assume a customer exists.
- Invent customer information.
- Tell the customer they are in the system before the lookup succeeds.

---

# Processing Sequence

The workflow executes the following sequence.

## Step 1

Receive customer phone number.

---

## Step 2

Search customer records.

---

## Step 3

Determine whether a matching customer exists.

---

## Step 4

If found:

Retrieve:

- Customer information
- Stored vehicles
- Previous appointments
- Customer notes

---

## Step 5

Prepare structured lookup response.

---

## Step 6

Return customer information to the AI.

---

# External Systems

This workflow communicates with:

## Google Sheets

Responsible for:

- Customer records
- Vehicle records
- Customer notes
- Customer identifiers

Google Sheets serves as the customer database.

---

# Outputs

Successful execution returns:

- Customer found status
- Customer identifier
- Customer name
- Stored phone number
- Stored vehicles
- Previous appointments (when available)
- Customer notes (when available)

---

# Success Response

If the customer exists, the response may include:

- Customer found
- Customer ID
- Name
- Vehicles
- Previous appointment information
- Customer history

If the customer does not exist:

- Customer not found

The AI uses this information to determine the next step in the conversation.

---

# Failure Conditions

The workflow may fail when:

## Lookup Failures

- Phone number is missing.
- Phone number format is invalid.
- Customer records cannot be searched.

---

## Data Failures

- Customer database unavailable.
- Customer records corrupted.

---

## System Failures

- Google Sheets unavailable.
- Internal workflow error.
- External API failure.

---

# Error Handling

When lookup fails:

- No customer information should be assumed.
- No customer records should be modified.
- Structured error information is returned.
- The AI should continue assisting the customer naturally by requesting the required information again or proceeding as a new customer if appropriate.

---

# Dependencies

This workflow depends on:

- Google Sheets
- Customer Database
- Session Management

---

# Related Workflows

## Downstream

- Service Estimates
- Availability
- Booking
- Rescheduling
- Cancellation
- Appointment Lookup

Most customer-facing workflows use the customer information returned by this workflow.

---

# Engineering Notes

## Source of Truth

Google Sheets is the authoritative source for:

- Customer records
- Vehicle records
- Customer notes
- Customer identifiers

Customer Lookup is responsible only for retrieving existing information.

Customer creation and updates occur during the Booking workflow or other dedicated customer update processes.

---

## Design Principles

The workflow follows several architectural principles.

- Single responsibility
- Phone number as primary customer identifier
- Structured tool responses
- Read-only customer retrieval
- Conversation separated from implementation
- Minimize duplicate customer data collection
- Preserve customer privacy

Future modifications should preserve these principles unless the customer management architecture is intentionally redesigned.
