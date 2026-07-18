# AI Systems Hub Architecture

## Purpose

The AI Auto Repair Receptionist is a modular voice automation platform designed to handle inbound customer calls for automotive repair shops.

The platform consists of multiple independent services orchestrated by n8n. Each service has a single responsibility and communicates with external systems such as Twilio, ElevenLabs, Google Sheets, and Google Calendar.

---

# High-Level Architecture

Customer
    │
    ▼
Twilio
    │
    ▼
ElevenLabs AI Agent
    │
    ▼
n8n Workflow Engine
    │
    ├── Business Configuration
    ├── Session Manager
    ├── Knowledge Base
    ├── Appointment Engine
    ├── Calendar Integration
    ├── Customer CRM
    ├── SMS Notification Service
    ├── Human Handoff
    ├── Post-Call Processing
    └── Scheduled Jobs

            │
            ▼

Google Sheets
Google Calendar
Twilio API

---

# System Components

## 1. Twilio

Responsibilities

- Receives inbound calls
- Routes calls to ElevenLabs
- Sends SMS notifications
- Transfers calls to humans
- Terminates calls
- Owns the business phone number

Twilio is responsible only for telecommunications.

Business logic never lives inside Twilio.

---

## 2. ElevenLabs Voice Agent

Responsibilities

- Speech recognition
- Speech synthesis
- Natural conversation
- Intent recognition
- Information collection
- Tool invocation

The AI agent never performs business logic directly.

Whenever information must be retrieved or modified, it calls an n8n workflow.

---

## 3. n8n Workflow Engine

n8n is the central orchestration layer.

Every business operation passes through n8n.

Responsibilities include

- booking appointments
- rescheduling
- cancellations
- service lookup
- pricing lookup
- availability lookup
- customer updates
- SMS automation
- call logging
- human handoff
- workflow coordination

No permanent business data should originate inside n8n.

---

# Internal Services

## Business Configuration

Contains shop-specific configuration.

Examples

- business name
- phone number
- address
- hours
- timezone
- owner information
- booking rules

Allows one workflow to support multiple repair shops.

---

## Knowledge Base

Primary source of truth for services.

Contains

- services
- descriptions
- estimated duration
- starting prices
- categories
- keywords

Current implementation

Google Sheets

Future

Database

---

## Session Manager

Tracks active phone calls.

Responsibilities

- caller phone number
- call SID
- temporary session state
- active workflow state

Uses

- n8n static data
- Sessions Google Sheet

The Session Manager exists only for the duration of a conversation.

---

## Appointment Engine

Responsible for all booking logic.

Functions

- create appointments
- reschedule appointments
- cancel appointments
- validate requests
- prevent duplicate bookings
- generate confirmations

The Appointment Engine never speaks directly to customers.

It only returns structured data to ElevenLabs.

---

## Calendar Integration

Current provider

Google Calendar

Responsibilities

- availability lookup
- create events
- update events
- delete events

Google Calendar is the scheduling authority.

---

## Customer CRM

Stores customer history.

Current implementation

Google Sheets

Contains

- customer information
- phone number
- notes
- previous interactions
- last visit

Future implementation

Dedicated database.

---

## SMS Notification Service

Uses Twilio.

Handles

- booking confirmations
- reminder messages
- cancellation messages
- appointment updates

---

## Human Handoff

Transfers active calls to shop staff.

Responsibilities

- determine transfer eligibility
- initiate transfer
- log transfer result
- request callback when unavailable

---

## Post-Call Processing

Runs after every completed conversation.

Responsibilities

- save conversation summary
- update customer notes
- log interaction
- prepare future context

---

## Scheduled Jobs

Background automations.

Current jobs

- appointment reminders
- cleanup tasks

Future jobs

- follow-up reminders
- missed-call outreach
- maintenance reminders
- customer reactivation

---

# Data Sources

## Google Sheets

Current source of truth for

- services
- customer records
- call logs
- sessions
- business configuration

---

## Google Calendar

Source of truth for

- appointment availability
- scheduled appointments

---

# Guiding Principles

Business logic belongs in n8n.

Voice belongs in ElevenLabs.

Telephony belongs in Twilio.

Scheduling belongs in Google Calendar.

Business data belongs in Google Sheets.

Each component has a single responsibility.

No component should duplicate another component's responsibility.

---

# Future Architecture

Planned improvements

- PostgreSQL / Supabase
- Vector database
- Automated testing
- Regression testing
- AI Workflow Auditor
- Prompt Reviewer
- Conversation Evaluator
- Analytics Dashboard
- Deployment Pipeline
