# Security Rules Architecture

## Security Objective
The security model for CARE-AI must protect sensitive child support data, prevent unauthorized doctor access, and ensure that moderation and verification actions happen only through trusted server-side flows.

Because this is a collaborative care platform, security is not just authentication. It must enforce:
- identity verification
- role boundaries
- relationship-based access
- sensitive data protection
- auditability
- safe AI invocation

---

## Primary Security Principles

### 1. Authentication is necessary but not sufficient
A signed-in user should still only access data that belongs to their role and relationship scope.

### 2. Role changes must be server-controlled
A parent cannot become a doctor by changing a client-side field.
A doctor cannot self-approve verification.
A moderator cannot be spoofed from the app.

### 3. Relationship-based access is mandatory
Access should depend on whether:
- the parent owns the child
- the doctor is linked to the child
- the moderator has admin privileges

### 4. Sensitive data should be minimized
Only store what the app actually needs.
If a field is highly sensitive, encrypt it or isolate it from general query paths.

### 5. Every critical action should be traceable
Plan creation, verification approval, plan assignment, and moderation actions should write audit events.

---

## Role-Based Access Control (RBAC)

### Parent / Caregiver
Allowed:
- read and update own profile
- create and update own child profile
- view assigned therapy plans
- submit completion evidence
- send messages in own collaboration threads
- read own progress dashboards

Not allowed:
- access unrelated children
- access unlinked doctors' patient data
- approve doctor verification
- edit doctor-authored therapy plans without permission
- read audit logs

### Doctor / Therapist
Allowed:
- read own doctor profile
- read assigned child data
- create therapy plans for linked children
- update therapy plans they created
- review completion data for assigned patients
- send collaboration notes
- view progress snapshots for linked families

Not allowed:
- access unassigned children
- modify parent account data
- self-approve verification
- access moderation queue unless also moderator

### Moderator / Admin
Allowed:
- approve or reject doctor verification
- review flagged content
- manage abusive accounts
- view audit logs
- manage moderation queue
- disable unsafe content or suspicious accounts

Not allowed:
- alter therapy plan content casually outside moderation scope
- impersonate a doctor or parent account
- bypass audit logging

---

## Firestore Security Rule Strategy

Rules should validate:
- `request.auth != null`
- the authenticated UID matches the document owner or relationship
- role field matches expected role
- linked doctor-patient relationship exists for clinical records
- moderator access is restricted to admin-bound collections

### Example rule concepts
- `isOwner(uid)`
- `isDoctor(uid)`
- `isModerator(uid)`
- `isLinkedDoctor(doctorId, childId)`
- `isChildOwner(parentId, childId)`

### Rule enforcement examples
- Parent can read `children/{childId}` only if `parentId == request.auth.uid`
- Doctor can read `children/{childId}` only if `doctor_patient_links` contains an active link
- Therapy plans can be created only when the doctor is approved
- Completion records can be written only by the child owner or caregiver
- Moderation queue can be read only by moderators

---

## Sensitive Data Handling

### Data that should be protected carefully
- child diagnosis details
- medical notes
- therapist comments
- uploaded evidence files
- phone numbers
- doctor verification documents

### Recommended protection layers
1. **Firestore access rules**
   - prevent unauthorized reads before data reaches the client

2. **Storage path isolation**
   - store evidence and verification documents in structured private paths

3. **Client-side encryption for selected fields**
   - only if implemented carefully and consistently
   - avoid reusing IVs or weak encryption patterns

4. **Server-side validation**
   - Cloud Functions should verify role and relationship before sensitive writes

---

## Cloud Function Security

All privileged operations should go through trusted backend functions.

Examples:
- doctor verification approval
- role promotion or verification state changes
- AI calls requiring protected API keys
- audit event creation
- notification fan-out
- bulk cleanup or delete operations

### Cloud function rules
- never trust raw client payloads
- validate the caller role
- reject unauthorized calls immediately
- sanitize all text inputs
- write audit logs for important actions

---

## Storage Security

Firebase Storage should be used for:
- therapy media
- proof uploads
- verification documents
- profile images

### Storage rules should enforce
- parent-only access to their uploaded media
- linked doctor access to supported patient evidence where appropriate
- moderator access to flagged or verification documents
- no public access to private medical attachments

---

## Audit Logging

A dedicated audit trail is recommended for:
- login anomalies
- role changes
- doctor verification decisions
- plan publish/update/delete events
- moderation actions
- suspicious access attempts

### Audit log fields
- actorId
- actorRole
- actionType
- targetType
- targetId
- timestamp
- metadata
- ip or device hint where available

Audit logs help with security review and compliance readiness.

---

## AI Safety Controls

The AI assistant must be bounded.

### Allowed AI behavior
- explain doctor-created plans in simpler language
- help parents understand next steps
- give encouragement and reminders
- summarize progress in plain language

### Disallowed AI behavior
- diagnose disease
- prescribe medication
- override doctor decisions
- present itself as a licensed professional
- generate unsafe treatment instructions

### AI guardrails
- call AI from Cloud Functions where possible
- restrict system prompts
- log AI requests in a minimal privacy-safe way
- add disclaimers in the UI
- require doctor-created context for care instructions

---

## Account Safety

### Suspension policy
Suspended accounts should:
- lose write access
- lose messaging access
- retain only the ability to view safe account status data

### Verification policy
Doctors remain in pending state until moderation approval.
Pending doctors should not be allowed to assign active plans.

### Abuse prevention ideas
- limit repeated verification attempts
- rate-limit messaging bursts
- detect suspicious content uploads
- report abuse with moderator queue entries

---

## Recommended Security Flow

### Parent access flow
1. user signs in
2. auth token is validated
3. role is read from `users/{uid}`
4. ownership checks are applied
5. data is returned only if the parent owns the child or thread

### Doctor access flow
1. user signs in
2. doctor profile is checked for approval
3. active doctor-patient link is confirmed
4. plan and progress data are returned

### Moderator flow
1. moderator signs in
2. admin role is validated
3. moderation queue and audit views are enabled
4. all moderation actions are logged

---

## Threats to Watch

- client-side role spoofing
- direct document path guessing
- overexposed public Storage paths
- weak verification approval logic
- AI prompt injection through user messages
- accidental sharing of child health data
- unauthorized plan edits by linked but inactive doctors

---

## Hardening Recommendations

- use strict Firestore rules everywhere
- keep server-side checks for privileged writes
- use custom claims only if the implementation is kept synchronized with database state
- review Storage rules separately from Firestore rules
- add audit logging to every admin/doctor approval flow
- validate file upload types and sizes
- redact unsafe content in logs

---

## Summary
The security architecture should make CARE-AI safe enough for a real care workflow. The app should always know:
- who the user is
- what role the user has
- which child or family the user is connected to
- whether the action is allowed
- whether the event must be audited

That is the foundation for trustworthy doctor-parent collaboration.
