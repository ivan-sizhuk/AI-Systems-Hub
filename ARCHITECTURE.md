# Architecture

## Purpose

The AI Auto Repair Receptionist is a modular voice automation platform that handles inbound customer phone calls for automotive repair shops.

Its primary responsibility is to answer calls, understand customer requests, retrieve business information, book appointments, update customer records, send notifications, and transfer callers to a human when necessary.

The system is designed to automate front-desk operations while maintaining reliability, accuracy, and a natural conversational experience.

---

# High-Level Architecture

```
Customer
    │
    ▼
Twilio Voice
    │
    ▼
ElevenLabs AI Agent
    │
    ▼
n8n Workflow Engine
    │
    ├── Business Configuration
    ├── Service Estimate Engine
    ├── Customer Lookup
    ├── Appointment Lookup
    ├── Booking Engine
    ├── Availability Engine
    ├── Rescheduling Engine
    ├── Cancellation Engine
    ├── Session Manager
    ├── SMS Notification Service
    ├── Human Handoff
    ├── Post-Call Processor
    └── Reminder Engine
            │
            ▼
Google Sheets
Google Calendar
Twilio API
```

---

# External Services

## Twilio

Twilio provides all telecommunications functionality.

Responsibilities:

- Receives inbound phone calls
- Transfers calls to shop staff
- Sends SMS notifications
- Terminates phone calls
- Owns the business phone number

Twilio contains no business logic.

---

## ElevenLabs

ElevenLabs provides the conversational AI layer.

Responsibilities:

- Speech-to-text
- Text-to-speech
- Natural language conversation
- Intent recognition
- Tool invocation

ElevenLabs is responsible only for conversation. All business logic is delegated to the workflow engine.

The language model used for reasoning and tool selection is configured inside the ElevenLabs agent. There is no separate LLM integration in the workflow engine.

---

## Google Calendar

Google Calendar is the scheduling authority.

Responsibilities:

- Store appointments
- Check availability
- Create appointments
- Update appointments
- Delete appointments

Appointment availability is never assumed and must always be verified.

---

## Google Sheets

Google Sheets currently serves as the application's operational data store.

Stores:

- Services
- Customers
- Appointments
- Sessions
- Call logs

Business configuration is not stored in Google Sheets. See Business Configuration below.

Future versions may migrate to a relational database.

---

# Internal Components

## Business Configuration

Provides shop-specific configuration used throughout the workflow.

Configuration lives in an n8n Set node ("Business Config") as a JSON object, with two clones ("Business Config - Init" for the call-initiation chain and "Business Config - Reminder" for the scheduled reminder chain). All three must be updated together when configuration changes.

Contains:

- Shop information
- Business hours
- Appointment settings
- Pricing configuration
- General settings

---

## Booking Engine

Responsible for creating new appointments.

Responsibilities:

- Validate booking requests
- Collect required information
- Check calendar availability
- Create appointments
- Update customer records
- Send confirmation messages

---

## Availability Engine

Determines available appointment times.

Responsibilities:

- Parse requested dates
- Query Google Calendar
- Resolve service duration from the service catalog when not supplied
- Calculate available time slots
- Return availability

---

## Service Estimate Engine

Provides approximate repair estimates.

Responsibilities:

- Retrieve service information
- Return approximate pricing
- Return estimated duration
- Include pricing disclaimers

---

## Customer Lookup

Retrieves existing customer information.

Responsibilities:

- Search customer records
- Retrieve previous history
- Return customer details

---

## Appointment Lookup

Finds existing appointments.

Responsibilities:

- Locate appointments
- Validate appointment information
- Return appointment details

---

## Rescheduling Engine

Updates existing appointments.

Responsibilities:

- Find appointment
- Verify new availability
- Update calendar
- Update customer records
- Send confirmation

---

## Cancellation Engine

Cancels appointments.

Responsibilities:

- Locate appointment
- Remove calendar event
- Update records
- Notify customer

---

## Session Manager

Maintains temporary state during active phone calls.

Responsibilities:

- Track active sessions
- Store temporary conversation data
- Associate calls with customers
- Clean expired sessions

---

## SMS Notification Service

Sends automated text messages.

Supports:

- Booking confirmations
- Reschedule confirmations
- Cancellation confirmations
- Appointment reminders

---

## Human Handoff

Transfers callers to shop personnel.

Responsibilities:

- Initiate transfers
- Detect transfer outcome
- Log transfer status
- Return call outcome

---

## Reminder Engine

Runs scheduled background tasks.

Responsibilities:

- Monitor upcoming appointments
- Send reminder SMS
- Prevent duplicate reminders

---

## Post-Call Processor

Processes completed conversations.

Responsibilities:

- Store conversation summaries
- Update customer notes
- Update appointment notes
- Save call records

---

# Data Flow

A typical customer interaction follows this sequence:

1. Customer calls the shop.
2. Twilio receives the call.
3. ElevenLabs conducts the conversation.
4. ElevenLabs invokes tools through the n8n workflow.
5. The workflow retrieves or updates business data.
6. Google Calendar is consulted whenever scheduling information is required.
7. Google Sheets stores customer, appointment, and operational data.
8. Twilio sends SMS notifications when appropriate.
9. After the conversation ends, post-call processing updates customer records and conversation summaries.

---

# Design Principles

The architecture follows several guiding principles:

- Business logic belongs in n8n.
- Conversation belongs in ElevenLabs.
- Telephony belongs in Twilio.
- Scheduling belongs in Google Calendar.
- Business data belongs in Google Sheets.
- Components should have a single responsibility.
- Workflows should remain modular and reusable.
- Reliability is prioritized over unnecessary complexity.
- Customer data should have a single source of truth.

---

# Future Architecture

Planned improvements include:

- PostgreSQL or Supabase for structured data
- Vector database for semantic knowledge retrieval
- Automated regression testing
- AI workflow auditing
- Conversation quality evaluation
- Analytics dashboard
- Multi-shop architecture
- CRM integrations
- Customer history improvements
- Deployment pipeline
- Monitoring and alerting
