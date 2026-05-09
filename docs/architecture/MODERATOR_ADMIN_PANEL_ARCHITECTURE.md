# Moderator Admin Panel Architecture

## Purpose
The moderator/admin panel exists to protect trust, safety, and verification integrity in CARE-AI.

Because CARE-AI handles child-support workflows and doctor-parent collaboration, moderation is not optional. The panel must support clear review queues, doctor verification, abuse handling, and audit visibility.

---

## Moderator Responsibilities

### 1. Doctor verification approval
Review credentials and approve or reject doctor accounts.

### 2. Abuse and content moderation
Handle suspicious messages, reports, or unsafe community posts.

### 3. Audit monitoring
Inspect platform-wide actions when needed.

### 4. Account safety controls
Suspend or restrict abusive accounts.

### 5. Trust and policy enforcement
Ensure the platform remains safe for caregiver use.

---

## Moderator Dashboard Goals

The dashboard should answer these questions immediately:
- What is pending review?
- Which doctor accounts need action?
- Are there urgent abuse reports?
- Are there suspicious or repeated moderation events?
- Is there anything requiring escalation?

The dashboard must be fast, clean, and audit-friendly.

---

## Main Panel Sections

### 1. Overview Summary
This is the landing screen after login.

Widgets:
- pending doctor approvals
- open abuse reports
- flagged content count
- active suspensions
- recent audit events

### 2. Doctor Verification Queue
This is the most important operational section.

Displays:
- doctor name
- specialization
- institution
- uploaded documents
- verification status
- submitted timestamp
- review notes

Actions:
- approve
- reject
- request more info
- flag for manual review

### 3. Abuse Management Queue
Handles user reports and suspicious behavior.

Displays:
- reporter identity
- target account
- report reason
- evidence preview
- severity
- status

Actions:
- dismiss
- warn user
- suspend user
- escalate to admin review

### 4. Flagged Content Review
For reviewing content that may violate policy.

Examples:
- harmful or misleading advice
- abusive chat content
- inappropriate uploads
- spam or harassment

### 5. Audit Log Viewer
Shows important system actions.

Filters:
- actor role
- action type
- date range
- target resource
- severity

### 6. Account Control Tools
For administrative action when needed.

Actions:
- suspend account
- reinstate account
- restrict posting
- restrict messaging
- mark account under review

---

## Verification Workflow

### Step 1: Doctor submits credentials
Doctor uploads:
- license document
- specialization proof
- institution info
- profile bio

### Step 2: Moderator receives queue item
A moderation queue item is created with status `open` or `pending`.

### Step 3: Moderator reviews information
The moderator checks:
- document authenticity
- profile completeness
- specialization match
- policy compliance

### Step 4: Decision is recorded
Possible decisions:
- approved
- rejected
- more_info_required

### Step 5: System updates role access
If approved:
- doctor profile becomes active
- doctor can create therapy plans
- parent linking becomes possible

### Step 6: Audit log is written
Every decision should be logged.

---

## Abuse Handling Workflow

### Stage 1: Report submitted
A parent, doctor, or system rule flags an issue.

### Stage 2: Queue creation
The report enters the moderation queue.

### Stage 3: Moderator review
Moderator inspects:
- context
- evidence
- related conversation or post
- account history

### Stage 4: Action
- dismiss if harmless
- warn if minor
- suspend if severe
- escalate if repeated or complex

### Stage 5: Audit and notification
The system stores the moderation outcome and notifies relevant parties if appropriate.

---

## Suggested UI Layout

### Left navigation
- Overview
- Doctor Verifications
- Abuse Reports
- Flagged Content
- Audit Logs
- Account Controls

### Main content area
- cards for pending work
- table or list views for queues
- right-side detail pane for selected item

### Detail pane
When a queue item is selected, show:
- metadata
- evidence preview
- history
- action buttons
- internal notes field

---

## Design Requirements

The moderator panel should feel:
- serious
- efficient
- admin-grade
- uncluttered
- easy to scan

Avoid:
- playful visuals
- consumer-style clutter
- too many animations
- unstructured cards with no hierarchy

The visual tone should communicate trust and control.

---

## Data Objects Used by the Panel

### `doctor_verification_requests`
Used for doctor credential review.

### `moderation_queue`
Used for all reviewable moderation items.

### `audit_logs`
Used to trace actions and platform state changes.

### `account_actions`
Optional collection for warnings, suspensions, and reinstatements.

---

## Permissions Model

Moderators can:
- read moderation queues
- approve or reject doctor verification
- manage abuse reports
- view audit records
- restrict accounts

Moderators cannot:
- impersonate doctors
- edit therapy plans without a valid admin reason
- override ownership rules casually
- access unrelated private medical data outside their moderation scope

---

## Important Audit Events

Log the following:
- login success/failure
- doctor approval decision
- rejection reason
- user suspension
- content removal
- moderation note creation
- escalation

Each event should capture:
- actor ID
- target ID
- action type
- timestamp
- metadata

---

## Escalation Paths

The moderator panel should support escalation for:
- suspicious doctor documents
- repeated abuse from the same account
- unsafe caregiving advice
- child safety concerns
- policy disputes that require admin judgment

Escalation may create a higher priority queue item or a separate admin review case.

---

## Summary
The moderator panel is the trust layer of CARE-AI. It protects the platform from unsafe access, unverified professionals, and harmful content while keeping the collaboration system reliable for real families.
