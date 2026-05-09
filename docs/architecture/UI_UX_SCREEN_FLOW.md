# UI UX Screen Flow

## Goal
This document defines how users move through CARE-AI across the three major roles: parent/caregiver, doctor/therapist, and moderator/admin.

The UI is designed around a small number of high-value workflows:
- onboarding and role selection
- doctor-parent collaboration
- therapy plan creation and execution
- progress review and communication
- moderation and verification

The product should feel calm, trustworthy, and structured rather than noisy or social-media-like.

---

## Navigation Principles

### 1. Role-based shell after login
The app should not show every feature to every user.
Instead, each role lands in its own dashboard shell.

### 2. Task-first navigation
Primary actions should be visible from the dashboard:
- view plans
- create plan
- message doctor
- review progress
- approve request

### 3. Minimal cognitive load
The app should avoid clutter.
Important information should be surfaced in cards, timelines, and actionable lists.

### 4. Realtime collaboration area
The collaboration screen should feel like the center of the product, not an optional extra.

---

## Global App Structure

### Splash / Boot Flow
1. app launches
2. Firebase and local services initialize
3. auth state is checked
4. profile and role are loaded
5. user is routed to the correct shell

### First-time user routing
- no account -> login/signup screen
- account but incomplete profile -> onboarding flow
- role verified -> role dashboard
- pending doctor verification -> verification pending screen

---

## Parent / Caregiver Journey

### Screen 1: Login / Sign Up
Purpose:
- authenticate the parent
- capture account identity

UI elements:
- email/password
- Google sign-in if supported
- phone login if supported
- forgot password

### Screen 2: Parent Onboarding
Purpose:
- collect child and family context

Steps:
- parent profile details
- child profile creation
- support needs selection
- emergency contacts
- preferred communication settings

### Screen 3: Parent Home Dashboard
Purpose:
- show current therapy state

Main widgets:
- today’s tasks
- active therapy plan summary
- progress ring
- doctor messages preview
- reminders and alerts
- next milestone

### Screen 4: Child Profile Screen
Purpose:
- manage child data

Features:
- edit child details
- view assigned doctor(s)
- view therapy history
- attach notes or documents

### Screen 5: Therapy Plan Detail Screen
Purpose:
- explain what to do today

Features:
- plan summary
- exercise list
- due dates
- media instructions
- completion status
- doctor notes

### Screen 6: Task Execution Screen
Purpose:
- perform or log therapy activity

Features:
- instructions
- timer or repetition counter
- upload proof
- mark complete
- add parent note

### Screen 7: Collaboration Workspace
Purpose:
- communicate with doctor

Features:
- chat thread
- session notes
- plan updates
- attachments
- plan clarification requests

### Screen 8: Progress Screen
Purpose:
- show adherence and improvement

Features:
- weekly chart
- completion rate
- milestone timeline
- missed activity list
- doctor observations

---

## Doctor / Therapist Journey

### Screen 1: Doctor Login / Sign Up
Purpose:
- authenticate the professional account

### Screen 2: Doctor Verification Pending
Purpose:
- show verification status if not approved

Features:
- document upload status
- approval progress
- retry/edit verification submission

### Screen 3: Doctor Dashboard
Purpose:
- show patient workload and key actions

Main widgets:
- assigned patients count
- pending approvals
- recent messages
- plan drafts
- urgent reviews

### Screen 4: Patient List Screen
Purpose:
- browse linked patients

Features:
- patient cards
- filtering by status
- search by name
- quick access to child profile and plan history

### Screen 5: Patient Detail Screen
Purpose:
- inspect one child’s therapy context

Features:
- profile summary
- linked parent account
- active plan(s)
- recent completions
- risk flags or missed tasks
- notes timeline

### Screen 6: Therapy Plan Builder
Purpose:
- create structured therapy plans

Main sections:
- plan metadata
- goals
- schedule
- exercises
- media attachments
- checkpoints
- publish/assign button

### Screen 7: Plan Review / Edit Screen
Purpose:
- manage drafts and updates

Features:
- plan version history
- duplicate plan
- pause / resume
- edit exercise order
- republish with change summary

### Screen 8: Collaboration Workspace
Purpose:
- communicate with parent

Features:
- chat thread
- completion evidence review
- feedback comments
- quick plan adjustment actions

### Screen 9: Progress Analytics Screen
Purpose:
- review adherence and response patterns

Features:
- charts
- missed sessions
- improvement trends
- child-by-child filtering
- export or snapshot options

---

## Moderator / Admin Journey

### Screen 1: Moderator Login
Purpose:
- secure access to moderation functions

### Screen 2: Moderation Dashboard
Purpose:
- show pending tasks and alerts

Main widgets:
- pending doctor verifications
- flagged content count
- abuse reports
- account actions needed
- audit event summary

### Screen 3: Doctor Verification Queue
Purpose:
- review new professional accounts

Features:
- document viewer
- specialization info
- approve / reject buttons
- notes field
- decision audit log

### Screen 4: Flagged Content Review
Purpose:
- inspect community posts, messages, or uploads that violate policy

Features:
- content preview
- reporting reason
- action buttons
- escalation notes

### Screen 5: Abuse Management Screen
Purpose:
- manage abuse cases and user restrictions

Features:
- user suspension toggle
- case timeline
- internal moderation notes
- resolution status

### Screen 6: Audit Dashboard
Purpose:
- inspect platform actions for trust and safety

Features:
- filtered event log
- action type search
- role-based filtering
- exportable records if needed

---

## Shared Components

### Header / App Bar
Should show:
- role label
- notifications icon
- profile avatar
- search if relevant

### Bottom Navigation
Recommended for parent and doctor roles:
- dashboard
- plans
- collaboration
- progress
- profile

Moderator shell may use a side menu or tabbed admin layout.

### Common cards
- therapy plan card
- patient card
- progress card
- message card
- verification card

### Common states
- loading
- empty
- error
- permission denied
- pending verification
- no assigned patient

---

## Screen Interaction Guidelines

### Empty states
Every empty screen should explain what to do next.
Example:
- no therapy plan yet -> “Your doctor will create a plan here.”
- no linked doctor -> “Request or wait for doctor approval.”

### Confirmation states
Important actions should ask for confirmation:
- delete plan
- pause plan
- suspend user
- reject verification

### Error states
Errors should be human-readable, not technical.
Examples:
- could not load plan
- no internet connection
- permission denied
- verification still pending

---

## Key User Flows

### Parent flow
Login -> onboarding -> child profile -> doctor link -> receive plan -> execute tasks -> chat -> progress review

### Doctor flow
Login -> verification -> patient list -> build plan -> monitor execution -> message parent -> update plan

### Moderator flow
Login -> dashboard -> verify doctor -> review abuse report -> audit review -> action taken

---

## Visual Direction

The interface should use:
- clean cards
- soft spacing
- clear hierarchy
- calm colors
- readable typography
- status chips
- timeline layouts
- progress visualizations

Avoid:
- noisy gradients
- overly playful visuals
- crowded dashboards
- unnecessary animation overload

The design should communicate:
trust, care, clarity, and medical seriousness.

---

## Summary
The UI/UX system should make CARE-AI easy to understand for stressed caregivers and efficient for doctors.
The navigation must always answer three questions quickly:
- what is happening now
- what should be done next
- who is responsible for it
