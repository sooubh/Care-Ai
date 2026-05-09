# Firestore Schema Detailed

## Purpose
This document defines the Firestore data model for CARE-AI. The schema is designed around a collaborative therapy workflow where parents, doctors/therapists, and moderators coordinate child support plans, progress monitoring, messaging, and verification.

The goals of this schema are:
- keep ownership boundaries clear
- prevent cross-user data leakage
- support realtime collaboration
- store therapy plans and completion evidence cleanly
- allow analytics and auditability
- keep reads predictable for mobile clients

---

## Design Principles

1. **Ownership-first modeling**
   - Every record should have a clear owner or governing doctor.
   - Parent-owned child data and doctor-authored plan data are separated.

2. **Role-safe access paths**
   - Parents can only read/write their own family data.
   - Doctors can only read assigned child records.
   - Moderators can only access verification and moderation workflows.

3. **Realtime where needed**
   - Collaboration threads, plan updates, and progress summaries should stream in realtime.
   - Static metadata should be cached aggressively on the client.

4. **Expandable but normalized**
   - Use subcollections for repeated event data such as messages, logs, completions, and audit events.
   - Keep top-level documents small and queryable.

5. **Auditability**
   - Every important transition should store timestamps, actor IDs, and status changes.
   - Avoid updating records without tracking the previous state when it matters.

---

## Top-Level Collections

### 1. `users/{uid}`
Primary identity record for every authenticated account.

#### Fields
```json
{
  "uid": "string",
  "role": "parent | doctor | moderator",
  "displayName": "string",
  "email": "string",
  "phoneNumber": "string | null",
  "photoUrl": "string | null",
  "profileComplete": true,
  "accountStatus": "active | suspended | pending_verification",
  "createdAt": "timestamp",
  "updatedAt": "timestamp",
  "lastLoginAt": "timestamp",
  "themePreference": "light | dark | system"
}
```

#### Notes
- This is the root user profile used for routing after login.
- Do not store private medical data here.
- Role changes should be performed only through server-side admin workflows.

---

### 2. `children/{childId}`
Stores child or dependent profiles created by parents/caregivers.

#### Fields
```json
{
  "childId": "string",
  "parentId": "string",
  "fullName": "string",
  "dateOfBirth": "timestamp | null",
  "age": 8,
  "gender": "string | null",
  "supportCategory": "developmental | physical | speech | autism | other",
  "diagnosisSummary": "string",
  "therapyGoals": ["string"],
  "medicalNotes": "string",
  "allergies": ["string"],
  "emergencyContacts": [
    {
      "name": "string",
      "relation": "string",
      "phone": "string"
    }
  ],
  "assignedDoctorIds": ["string"],
  "createdAt": "timestamp",
  "updatedAt": "timestamp"
}
```

#### Notes
- `assignedDoctorIds` is a convenience list for UI and security checks, but linking should also live in a dedicated relationship collection.
- Sensitive fields should be encrypted or stored carefully depending on the final implementation.

---

### 3. `doctors/{doctorId}`
Stores therapist or doctor professional profile data.

#### Fields
```json
{
  "doctorId": "string",
  "userId": "string",
  "displayName": "string",
  "specialization": "string",
  "licenseNumber": "string",
  "institution": "string",
  "experienceYears": 5,
  "verificationStatus": "unverified | pending | approved | rejected",
  "verificationDocuments": ["string"],
  "bio": "string",
  "languages": ["string"],
  "activePatientCount": 12,
  "createdAt": "timestamp",
  "updatedAt": "timestamp"
}
```

#### Notes
- `verificationStatus` is the gate for doctor access.
- `userId` should match the authenticated Firebase UID.

---

### 4. `moderation_queue/{itemId}`
Queue for verification reviews, reports, and flagged content.

#### Fields
```json
{
  "itemType": "doctor_verification | content_report | abuse_report",
  "targetId": "string",
  "createdBy": "string",
  "status": "open | in_review | resolved | rejected",
  "priority": "low | medium | high",
  "notes": "string",
  "createdAt": "timestamp",
  "updatedAt": "timestamp",
  "resolvedBy": "string | null"
}
```

---

### 5. `doctor_patient_links/{linkId}`
Represents an approved collaboration relationship.

#### Fields
```json
{
  "linkId": "string",
  "doctorId": "string",
  "parentId": "string",
  "childId": "string",
  "status": "pending | active | paused | ended",
  "source": "parent_request | doctor_invitation | admin_assigned",
  "createdAt": "timestamp",
  "updatedAt": "timestamp",
  "lastInteractionAt": "timestamp | null"
}
```

#### Notes
- This collection is critical for access control.
- A doctor should not access a child record unless an active link exists.

---

### 6. `therapy_plans/{planId}`
Master therapy plan document created by a doctor.

#### Fields
```json
{
  "planId": "string",
  "doctorId": "string",
  "childId": "string",
  "parentId": "string",
  "title": "string",
  "description": "string",
  "category": "speech | occupational | behavioral | physical | mixed",
  "status": "draft | active | paused | completed | archived",
  "startDate": "timestamp",
  "endDate": "timestamp | null",
  "timezone": "Asia/Kolkata",
  "version": 3,
  "goals": ["string"],
  "tags": ["string"],
  "createdAt": "timestamp",
  "updatedAt": "timestamp"
}
```

---

### 7. `therapy_plans/{planId}/exercises/{exerciseId}`
Subcollection for repetitive therapy exercises.

#### Fields
```json
{
  "exerciseId": "string",
  "title": "string",
  "instructions": "string",
  "durationMinutes": 15,
  "repeatCount": 3,
  "frequency": "daily | alternate_days | weekly",
  "difficulty": "easy | medium | hard",
  "mediaUrls": ["string"],
  "completionCriteria": "string",
  "orderIndex": 1,
  "active": true,
  "createdAt": "timestamp",
  "updatedAt": "timestamp"
}
```

---

### 8. `therapy_assignments/{assignmentId}`
Tracks the active assignment of a plan to a family.

#### Fields
```json
{
  "assignmentId": "string",
  "planId": "string",
  "doctorId": "string",
  "parentId": "string",
  "childId": "string",
  "status": "assigned | acknowledged | in_progress | paused | completed",
  "assignedAt": "timestamp",
  "acknowledgedAt": "timestamp | null",
  "lastViewedAt": "timestamp | null",
  "dueDate": "timestamp | null"
}
```

---

### 9. `therapy_completions/{completionId}`
Completion event log for each exercise or session.

#### Fields
```json
{
  "completionId": "string",
  "assignmentId": "string",
  "planId": "string",
  "exerciseId": "string",
  "parentId": "string",
  "childId": "string",
  "completedBy": "string",
  "status": "completed | skipped | partial",
  "evidenceUrls": ["string"],
  "parentNote": "string",
  "doctorFeedback": "string | null",
  "completedAt": "timestamp",
  "reviewedAt": "timestamp | null"
}
```

---

### 10. `collaboration_threads/{threadId}`
Represents a shared conversation workspace between doctor and parent.

#### Fields
```json
{
  "threadId": "string",
  "doctorId": "string",
  "parentId": "string",
  "childId": "string",
  "threadType": "collaboration | support | review",
  "lastMessageAt": "timestamp | null",
  "unreadCounts": {
    "doctor": 0,
    "parent": 0
  },
  "createdAt": "timestamp",
  "updatedAt": "timestamp"
}
```

#### Subcollection: `messages/{messageId}`
```json
{
  "messageId": "string",
  "senderId": "string",
  "senderRole": "doctor | parent | moderator",
  "messageType": "text | image | file | system",
  "text": "string | null",
  "attachmentUrl": "string | null",
  "createdAt": "timestamp",
  "readAt": "timestamp | null"
}
```

---

### 11. `progress_snapshots/{snapshotId}`
Materialized analytics for fast dashboards.

#### Fields
```json
{
  "snapshotId": "string",
  "planId": "string",
  "childId": "string",
  "parentId": "string",
  "doctorId": "string",
  "completionRate": 87,
  "weeklyAdherence": 92,
  "missedSessions": 2,
  "milestoneScore": 74,
  "trend": "improving | stable | declining",
  "generatedAt": "timestamp"
}
```

---

## Supporting Collections

### `doctor_verification_requests/{requestId}`
Used for moderation.

### `session_notes/{noteId}`
Doctor notes after review sessions.

### `audit_logs/{logId}`
Security-sensitive action log.

### `notifications/{notificationId}`
Persistent notification records if needed.

### `app_settings/{settingId}`
Global config and feature flags.

---

## Read/Write Ownership Matrix

| Collection | Parent | Doctor | Moderator | Server |
|---|---:|---:|---:|---:|
| users | own record | own record | limited | full |
| children | own children | assigned only | limited | full |
| doctors | own profile | own profile | approve/review | full |
| doctor_patient_links | own links | own links | admin support | full |
| therapy_plans | read assigned | create/update own | limited | full |
| therapy_assignments | read own | manage own | limited | full |
| therapy_completions | own submissions | review assigned | limited | full |
| collaboration_threads | own threads | own threads | limited | full |
| progress_snapshots | own family | assigned patients | limited | full |
| audit_logs | no direct | no direct | limited | full |

---

## Indexing Recommendations

Create composite indexes for:
- `therapy_assignments` by `parentId + status + assignedAt`
- `therapy_plans` by `doctorId + childId + status`
- `collaboration_threads` by `doctorId + lastMessageAt`
- `therapy_completions` by `assignmentId + completedAt`
- `moderation_queue` by `status + priority + createdAt`

These indexes keep dashboards and timelines fast on mobile.

---

## Data Lifecycle Notes

### On doctor approval
1. moderator approves verification request
2. doctor profile status changes to approved
3. role access becomes active
4. notification is sent to doctor

### On plan creation
1. doctor creates draft plan
2. exercises are saved as subcollection docs
3. plan is assigned to a child
4. parent receives a notification
5. assignment status becomes active after acknowledgement

### On completion submission
1. parent marks exercise complete
2. proof or note is attached
3. completion record is written
4. progress snapshot is recalculated
5. doctor is notified if review is needed

---

## Validation Rules

- Every child must have one `parentId` owner.
- Every active plan must reference a valid `childId` and `doctorId`.
- Every doctor must have approved verification before creating active plans.
- Every collaboration thread must map to an active link.
- Every completion must reference an existing assignment.
- Moderator actions must create audit logs.

---

## Recommended Implementation Pattern

Use a service/repository layer with these responsibilities:
- validate document shape before write
- convert timestamps and enums consistently
- centralize document path builders
- avoid duplicate business logic in widgets
- keep security checks on the server whenever possible

---

## Summary
This schema supports the core CARE-AI product vision:
- doctor-led therapy planning
- parent execution
- collaborative review
- moderation and trust
- analytics and progress visibility

It is intentionally designed for a real-world care workflow, not a generic social app.
