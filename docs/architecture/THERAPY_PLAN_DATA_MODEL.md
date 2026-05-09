# Therapy Plan Data Model

## Purpose
This document defines the data model for therapy planning in CARE-AI.

A therapy plan is the central clinical/collaborative artifact in the system. It connects the doctor, the parent, the child, the exercises, the completion history, and the progress analytics.

The model must support:
- doctor-created structured plans
- repeatable exercises
- versioning and updates
- task completion logs
- milestone tracking
- evidence uploads
- parent/doctor feedback

---

## Core Object Model

### 1. Therapy Plan
This is the top-level object created by a doctor.

#### Recommended fields
```json
{
  "planId": "string",
  "doctorId": "string",
  "parentId": "string",
  "childId": "string",
  "title": "string",
  "description": "string",
  "category": "speech | occupational | behavioral | physical | mixed",
  "status": "draft | active | paused | completed | archived",
  "version": 1,
  "goals": ["string"],
  "targetOutcomes": ["string"],
  "tags": ["string"],
  "startDate": "timestamp",
  "endDate": "timestamp | null",
  "timezone": "Asia/Kolkata",
  "createdAt": "timestamp",
  "updatedAt": "timestamp"
}
```

#### Notes
- `version` should increase whenever the doctor republishes a meaningful update.
- `status` should reflect lifecycle state, not UI labels.
- `category` helps filter plans in dashboards and analytics.

---

### 2. Exercise
Exercises are the repeatable actions inside a therapy plan.

#### Recommended fields
```json
{
  "exerciseId": "string",
  "planId": "string",
  "title": "string",
  "instructions": "string",
  "sessionType": "home | guided | practice | review",
  "durationMinutes": 10,
  "repeatCount": 3,
  "frequency": "daily | alternate_days | weekly",
  "difficulty": "easy | medium | hard",
  "orderIndex": 1,
  "mediaUrls": ["string"],
  "completionCriteria": "string",
  "safetyNotes": "string | null",
  "active": true,
  "createdAt": "timestamp",
  "updatedAt": "timestamp"
}
```

#### Notes
- Keep exercises small and specific.
- If an exercise needs explanation media, attach images or videos.
- Safety notes are important for caregiver confidence.

---

### 3. Assignment
An assignment links a plan to a child and parent for execution.

#### Recommended fields
```json
{
  "assignmentId": "string",
  "planId": "string",
  "doctorId": "string",
  "parentId": "string",
  "childId": "string",
  "status": "assigned | acknowledged | in_progress | paused | completed | cancelled",
  "assignedAt": "timestamp",
  "acknowledgedAt": "timestamp | null",
  "lastViewedAt": "timestamp | null",
  "dueDate": "timestamp | null",
  "priority": "low | medium | high",
  "notes": "string | null"
}
```

#### Notes
- A plan can have one assignment per child or multiple assignment records over time.
- `acknowledgedAt` is useful for measuring parent engagement.

---

### 4. Completion Event
Every task completion should create a record.

#### Recommended fields
```json
{
  "completionId": "string",
  "assignmentId": "string",
  "planId": "string",
  "exerciseId": "string",
  "parentId": "string",
  "childId": "string",
  "completedBy": "string",
  "status": "completed | partial | skipped",
  "parentNote": "string | null",
  "doctorFeedback": "string | null",
  "evidenceUrls": ["string"],
  "completedAt": "timestamp",
  "reviewedAt": "timestamp | null",
  "reviewState": "unreviewed | reviewed | needs_followup"
}
```

#### Notes
- This record should not overwrite previous completions.
- Each session should remain traceable.

---

### 5. Milestone
Milestones define the expected checkpoints in the therapy path.

#### Recommended fields
```json
{
  "milestoneId": "string",
  "planId": "string",
  "title": "string",
  "description": "string",
  "dueDate": "timestamp | null",
  "targetMetric": "string",
  "targetValue": "string | number | null",
  "status": "pending | in_progress | achieved | missed",
  "orderIndex": 1,
  "createdAt": "timestamp",
  "updatedAt": "timestamp"
}
```

#### Notes
- Milestones help doctors and parents understand progress at a higher level than individual exercises.

---

## Collection Layout

Recommended Firestore layout:

```text
therapy_plans/{planId}
therapy_plans/{planId}/exercises/{exerciseId}
therapy_plans/{planId}/milestones/{milestoneId}
therapy_assignments/{assignmentId}
therapy_completions/{completionId}
```

This layout works well for:
- reading a plan with its exercises
- tracking completions separately
- showing history in a timeline
- storing analytics outside the core object

---

## Versioning Strategy

The therapy plan should be versioned when:
- the doctor changes exercise structure
- the schedule changes materially
- the goal definition changes
- the plan is republished after review

### Suggested fields for version tracking
```json
{
  "version": 4,
  "previousVersionId": "string | null",
  "changeSummary": "string",
  "publishedAt": "timestamp | null",
  "publishedBy": "string | null"
}
```

Versioning makes the plan history auditable and helps avoid confusion when changes happen mid-week.

---

## Status Lifecycle

### Plan status
- `draft` -> not visible to parent yet
- `active` -> currently used by family
- `paused` -> temporarily stopped
- `completed` -> finished successfully or as decided by doctor
- `archived` -> kept for history only

### Assignment status
- `assigned`
- `acknowledged`
- `in_progress`
- `paused`
- `completed`
- `cancelled`

### Completion status
- `completed`
- `partial`
- `skipped`

### Milestone status
- `pending`
- `in_progress`
- `achieved`
- `missed`

---

## Validation Rules

The app should validate the following before write:
- title is not empty
- childId exists
- doctorId exists and is approved
- exercises contain instructions
- due dates are logical
- plan references belong together
- completion records refer to valid assignment and exercise IDs

---

## UI Mapping

### Therapy plan card
Should show:
- title
- category
- status
- next due item
- progress summary

### Exercise card
Should show:
- exercise title
- duration
- repeat count
- completion state

### Completion timeline
Should show:
- date
- status
- note
- evidence icon

### Milestone card
Should show:
- milestone title
- target date
- current state
- achievement indicator

---

## Analytics Derived From This Model

From these objects you can compute:
- completion rate
- adherence rate
- session streak
- missed tasks
- weekly progress
- milestone success rate
- doctor response latency
- parent acknowledgement rate

These metrics power the dashboard and make the product feel clinically meaningful.

---

## Summary
The therapy plan model is the backbone of CARE-AI. It should stay simple enough for mobile use but structured enough to support real collaboration between doctors and parents.
