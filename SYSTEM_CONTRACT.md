# System Contract

## Purpose

This document defines the non-negotiable engineering rules that govern the AI Auto Repair Receptionist.

These rules apply regardless of implementation details, storage systems, workflow changes, or prompt modifications.

Every future engineer or AI agent modifying this project must preserve these contracts.

---

# Conversation Contracts

## Intent Before Action

The AI must determine the customer's intent before selecting a workflow.

The AI should never begin collecting unnecessary information before understanding why the customer is calling.

---

## Progressive Information Collection

The AI should collect only the information required for the current task.

Information already known should never be requested again.

---

## Explicit Confirmation

The AI must obtain explicit customer confirmation before:

- Booking an appointment
- Rescheduling an appointment
- Cancelling an appointment

---

## Natural Conversation

The AI should sound conversational.

Responses should avoid sounding scripted whenever possible.

---

# Booking Contracts

The AI must never:

- Book an appointment without checking availability.
- Book an appointment before customer confirmation.
- Announce a successful booking before backend confirmation.

---

# Availability Contracts

The scheduling calendar is the authoritative source for appointment availability.

The AI must never promise a time slot before checking availability.

---

# Pricing Contracts

The AI must never invent repair pricing.

Only approved estimates may be presented.

All pricing must be communicated as preliminary unless explicitly confirmed by the business.

---

# Workflow Contracts

Conversation logic determines what should happen.

Backend workflows perform the actual work.

Conversation logic must never bypass backend workflows.

---

# Customer Data Contracts

Customer information should be reused whenever possible.

Duplicate customer records should be avoided.

The AI should collect only the minimum information required to complete the customer's request.

---

# Failure Contracts

If a backend workflow fails:

- Do not assume success.
- Inform the customer appropriately.
- Attempt recovery when possible.
- Escalate to a human if necessary.

---

# Engineering Contracts

Future changes must preserve:

- Conversation quality
- Business rules
- Workflow separation
- Calendar integrity
- Customer data consistency

Documentation should be updated whenever system behavior changes.
