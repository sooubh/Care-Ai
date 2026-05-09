# API Contracts and Cloud Functions

## Purpose
This document defines how CARE-AI should expose secure server-side operations through callable functions and service contracts.

The app should keep sensitive operations off the client wherever possible.

Use Cloud Functions for:
- privileged writes
- AI calls
- verification workflows
- notification dispatch
- role-sensitive actions
- audit logging

---

## API Design Principles

### 1. Server-authoritative writes
The client should request actions, but the server should validate and commit important state changes.

### 2. Clear request and response shapes
Each callable should return a predictable JSON object with success/error metadata.

### 3. Minimal payloads
Only send what the server needs.
Avoid leaking unnecessary child or medical data.

### 4. Idempotent where possible
Repeated calls should not create duplicate bad state.

### 5. Audit every privileged action
Any critical mutation should create an audit record.

---

## Suggested Callable Functions

### 1. `createChildProfile`
Creates a child profile for an authenticated parent.

#### Request
```json
{
  "fullName": "string",
  "dateOfBirth": "timestamp | null",
  "supportCategory": "string",
  "therapyGoals": ["string"],
  "medicalNotes": "string"
}
```

#### Response
```json
{
  "success": true,
  "childId": "string",
  "message": "Child profile created successfully"
}
```

---

### 2. `requestDoctorLink`
Creates a link request between parent and doctor.

#### Request
```json
{
  "doctorId": "string",
  "childId": "string",
  "message": "string"
}
```

#### Response
```json
{
  "success": true,
  "requestId": "string",
  "status": "pending"
}
```

---

### 3. `approveDoctorVerification`
Moderator-only callable for approving doctor accounts.

#### Request
```json
{
  "doctorId": "string",
  "decision": "approve | reject",
  "notes": "string"
}
```

#### Response
```json
{
  "success": true,
  "verificationStatus": "approved"
}
```

---

### 4. `createTherapyPlan`
Doctor-only callable to create a plan.

#### Request
```json
{
  "childId": "string",
  "parentId": "string",
  "title": "string",
  "description": "string",
  "category": "speech | occupational | behavioral | physical | mixed",
  "goals": ["string"],
  "exercises": [
    {
      "title": "string",
      "instructions": "string",
      "durationMinutes": 15,
      "repeatCount": 3
    }
  ]
}
```

#### Response
```json
{
  "success": true,
  "planId": "string",
  "assignmentId": "string",
  "status": "active"
}
```

---

### 5. `updateTherapyPlan`
Doctor-only callable for editing a plan.

#### Request
```json
{
  "planId": "string",
  "title": "string | null",
  "description": "string | null",
  "status": "draft | active | paused | completed | archived",
  "changeSummary": "string"
}
```

#### Response
```json
{
  "success": true,
  "updatedAt": "timestamp"
}
```

---

### 6. `submitCompletion`
Parent/caregiver submits completion status for an exercise.

#### Request
```json
{
  "assignmentId": "string",
  "exerciseId": "string",
  "status": "completed | partial | skipped",
  "parentNote": "string",
  "evidenceUrls": ["string"]
}
```

#### Response
```json
{
  "success": true,
  "completionId": "string",
  "progressUpdated": true
}
```

---

### 7. `postCollaborationMessage`
Adds a message to a collaboration thread.

#### Request
```json
{
  "threadId": "string",
  "messageType": "text | image | file",
  "text": "string | null",
  "attachmentUrl": "string | null"
}
```

#### Response
```json
{
  "success": true,
  "messageId": "string"
}
```

---

### 8. `generateProgressSnapshot`
Recomputes progress metrics for dashboards.

#### Request
```json
{
  "planId": "string",
  "childId": "string"
}
```

#### Response
```json
{
  "success": true,
  "snapshotId": "string",
  "completionRate": 87,
  "weeklyAdherence": 92
}
```

---

### 9. `sendNotification`
Internal notification dispatcher.

#### Request
```json
{
  "userId": "string",
  "type": "plan_update | reminder | verification | message | alert",
  "title": "string",
  "body": "string",
  "route": "string"
}
```

#### Response
```json
{
  "success": true
}
```

---

### 10. `chatWithAI`
Support assistant callable.

#### Request
```json
{
  "threadId": "string | null",
  "prompt": "string",
  "context": {
    "childId": "string | null",
    "planId": "string | null",
    "userRole": "parent | doctor"
  }
}
```

#### Response
```json
{
  "success": true,
  "response": "string"
}
```

---

## Function Categories

### Auth and identity
- createChildProfile
- requestDoctorLink

### Clinical workflow
- createTherapyPlan
- updateTherapyPlan
- submitCompletion
- generateProgressSnapshot

### Trust and safety
- approveDoctorVerification
- moderation actions
- audit logging

### Communication
- postCollaborationMessage
- sendNotification

### AI support
- chatWithAI

---

## Function Security Requirements

Every callable should:
- verify authentication
- verify role
- validate payload shape
- reject unauthorized relationship access
- sanitize text input
- write audit events where relevant

---

## Recommended HTTP/Callable Response Format

A consistent response format helps the frontend simplify error handling.

```json
{
  "success": true,
  "data": {},
  "message": "string",
  "errorCode": null
}
```

On failure:

```json
{
  "success": false,
  "data": null,
  "message": "string",
  "errorCode": "UNAUTHORIZED"
}
```

---

## Cloud Function Implementation Notes

### Keep the handler thin
The function should:
- validate auth
- call service logic
- write to Firestore
- return compact data

### Move business logic to service modules
Examples:
- verification service
- therapy plan service
- notification service
- AI orchestration service
- audit service

### Avoid client-trusted flags
Do not trust client claims such as:
- `isDoctorApproved`
- `isModerator`
- `planPublished`
- `completionVerified`

These must be derived server-side.

---

## Error Handling Strategy

Recommended error classes:
- `UNAUTHENTICATED`
- `FORBIDDEN`
- `NOT_FOUND`
- `INVALID_ARGUMENT`
- `FAILED_PRECONDITION`
- `INTERNAL`

Each error should map to a human-readable message for the app.

---

## Summary
The API layer should stay small, explicit, and secure. CARE-AI is a sensitive workflow app, so every callable must protect the relationship between parent, doctor, and moderator.
