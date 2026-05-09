# FIRESTORE SCHEMA DETAILED

## Core Collections

### users/{uid}
Role-based user metadata.
Fields:
- uid
- role
- name
- email
- profileComplete
- photoUrl
- createdAt
- updatedAt
- verificationStatus

### children/{childId}
Child profile records.
Fields:
- parentId
- childName
- age
- diagnosis
- therapyGoals
- medicalNotes
- emergencyContacts

### doctors/{doctorId}
Doctor metadata.
Fields:
- specialization
- licenseNumber
- institution
- verificationStatus
- credentials

### doctor_patient_links/{linkId}
Relationship mapping.

### therapy_plans/{planId}
Therapy definitions.

### therapy_assignments/{assignmentId}
Assigned plans.

### therapy_completions/{completionId}
Completion records.

### collaboration_threads/{threadId}
Workspace threads.

### collaboration_threads/{threadId}/messages/{messageId}
Realtime messages.

### progress_snapshots/{snapshotId}
Analytics snapshots.
