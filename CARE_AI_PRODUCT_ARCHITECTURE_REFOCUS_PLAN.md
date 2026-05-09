# CARE-AI Product Architecture Refocus Plan

## Vision
CARE-AI is a collaborative therapy management platform connecting parents, therapists/doctors, and moderators for structured therapy execution and progress tracking for children with developmental or physical support needs.

Core principle: existing code/features remain intact. This plan refocuses product priority, architecture, and workflows.

---

## Product Roles

### Parent / Caregiver
Responsibilities:
- register and manage child profile
- request doctor/therapist connection
- receive therapy plans
- execute assigned therapy tasks
- upload completion proof
- communicate with doctor
- track progress
- use AI helper assistant

### Doctor / Therapist
Responsibilities:
- complete verification
- manage assigned patients
- create/edit therapy plans
- monitor progress
- send notes and instructions
- collaborate with parents

### Moderator / Admin
Responsibilities:
- verify doctors
- approve/reject doctor onboarding
- manage abuse reports
- moderate community features
- audit platform activity

---

## Core Product Direction

### Keep Existing Features
Preserve:
- AI chat
- voice assistant
- community module
- emergency tools
- notifications
- caching
- current architecture

### Product Repositioning
Primary product:
Doctor-parent therapy collaboration platform.

Secondary product:
AI-assisted caregiver support.

---

## Core Modules

### 1. Authentication & Role Management
Flow:
Signup/Login -> role detection -> profile completion -> role dashboard

Collections:
- users
- doctors
- moderators
- role_requests

Security:
Role assignment enforced server-side only.

---

### 2. Child Profile Management
Parent creates child profile:
- name
- age
- diagnosis/support category
- therapy goals
- developmental observations
- emergency contacts
- notes

Collections:
- children
- child_profiles

---

### 3. Doctor Verification Workflow
Flow:
Doctor signup -> credential upload -> moderator review -> approval/rejection -> access granted

Fields:
- license number
- specialization
- institution
- certifications
- proof docs

Collections:
- doctor_verification_requests

---

### 4. Doctor-Patient Linking
Flow A:
Parent requests doctor

Flow B:
Doctor invites parent

Lifecycle:
pending -> accepted -> linked -> active collaboration

Collections:
- doctor_patient_requests
- doctor_patient_links

Rule:
Doctors can only access linked patients.

---

### 5. Therapy Plan Builder (Flagship)
Doctor creates structured plans.

Plan structure:
- title
- description
- goal
- therapy type
- duration
- frequency
- start/end dates
- milestone checkpoints
- exercises
- attachments

Exercise structure:
- title
- instructions
- duration
- repetition count
- media attachment
- completion criteria

Collections:
- therapy_plans
- therapy_plan_templates
- therapy_assignments

---

### 6. Parent Execution Dashboard
Parent sees:
- today's tasks
- overdue tasks
- progress summary
- doctor updates
- reminders

Actions:
- mark task complete
- upload photo/video proof
- add notes
- request clarification

Collections:
- therapy_completions
- task_logs

---

### 7. Doctor Collaboration Workspace (Killer Feature)
Single shared collaboration screen.

Contains:
- chat
- progress timeline
- plan updates
- task feedback
- attachment sharing
- session notes
- adherence graphs

Doctor sees:
- missed tasks
- completion rates
- child progress
- parent notes

Parent sees:
- instructions
- doctor recommendations
- latest changes

Collections:
- collaboration_threads
- collaboration_messages
- session_notes

---

### 8. Progress Tracking Engine
Metrics:
- daily completion %
- weekly adherence %
- missed sessions
- milestone progress
- improvement logs

Collections:
- progress_snapshots
- milestone_logs

---

### 9. AI Support Assistant
AI role:
Support assistant only.

Allowed:
- explain therapy instructions
- simplify doctor guidance
- motivation support
- caregiver Q&A
- activity suggestions

Blocked:
- diagnosis
- prescriptions
- replacing doctor decisions

Architecture:
Gemini via secure Cloud Functions.

---

### 10. Notifications Engine
Triggers:
- new assignment
- missed task reminder
- doctor message
- milestone alert
- plan updated

Channels:
- FCM push
- local notifications

---

## System Architecture

```text
Flutter App
   |
Provider State Layer
   |
Service Layer
   |
Repository Layer
   |
Firebase Backend
   |
Firestore + Storage + Functions + FCM
   |
Gemini AI
```

---

## Frontend Architecture

Flutter modules:
- auth
- parent dashboard
- doctor dashboard
- moderator dashboard
- collaboration workspace
- therapy planner
- AI assistant
- notifications

Existing feature-first structure remains.

---

## Backend Architecture

Firebase Auth:
- authentication
- session management

Firestore:
- primary database

Storage:
- documents
- media uploads
- proof uploads

Cloud Functions:
- role validation
- AI requests
- notification triggers
- audit logic

FCM:
- push notifications

---

## Firestore Schema

### users/{uid}
Fields:
- role
- name
- email
- profileComplete
- createdAt

### doctors/{doctorId}
Fields:
- specialization
- verificationStatus
- credentials

### children/{childId}
Fields:
- parentId
- profileData
- goals

### doctor_patient_links/{linkId}
Fields:
- doctorId
- parentId
- childId
- status

### therapy_plans/{planId}
Fields:
- doctorId
- childId
- metadata
- exercises

### therapy_assignments/{assignmentId}
Fields:
- planId
- assignee
- status

### therapy_completions/{completionId}
Fields:
- taskId
- proofUrl
- notes

### collaboration_threads/{threadId}
Fields:
- doctorId
- parentId
- childId

### collaboration_threads/{threadId}/messages/{messageId}
Messages collection

### progress_snapshots/{snapshotId}
Progress analytics

---

## Security Architecture

Must enforce:
- strict RBAC
- server-side validation
- doctor verification gating
- audit logging
- secure callable AI endpoints
- encrypted sensitive fields

Rules:
Parent -> own child only
Doctor -> linked patients only
Moderator -> admin actions only

---

## UX Flow

### Parent Journey
Signup -> child profile -> request doctor -> receive plan -> execute tasks -> chat with doctor -> track progress

### Doctor Journey
Signup -> verification -> patient accepted -> create therapy plan -> monitor progress -> adjust plan

### Moderator Journey
Login -> review doctor requests -> approve/reject -> moderation oversight

---

## Existing Feature Repositioning

Voice assistant:
therapy voice guidance assistant

Community:
moderated caregiver support network (future priority)

Emergency tools:
secondary support utility

AI chat:
caregiver assistant, not main product

---

## Development Phases

### Phase 1
- role auth hardening
- doctor verification
- child profile system

### Phase 2
- doctor-patient linking
- therapy plan builder
- parent dashboard

### Phase 3
- collaboration workspace
- progress analytics
- reminders

### Phase 4
- AI assistant enhancement
- voice integration refinement

### Phase 5
- community evolution
- advanced analytics
- scaling optimizations

---

## Success Metrics
KPIs:
- therapy adherence rate
- weekly active caregivers
- doctor response time
- plan completion rate
- retention
- engagement

---

## Final Positioning
CARE-AI is not an AI healthcare chatbot.

CARE-AI is a collaborative therapy care execution platform with structured doctor-parent coordination, progress tracking, and AI-assisted caregiver support.
