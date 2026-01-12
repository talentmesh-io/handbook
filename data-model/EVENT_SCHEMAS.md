# Talent Mesh Event Schemas

## Overview

This document defines the event payload schemas for Redpanda (Kafka-compatible) topics in Talent Mesh. All events follow a standard envelope format with domain-specific payloads.

---

## Event Envelope

All events share a common envelope structure:

```typescript
interface EventEnvelope<T> {
  // Event metadata
  event_id: string;           // UUID v4
  event_type: string;         // e.g., "assessment.completed"
  schema_version: string;     // Semantic version, e.g., "1.0.0"
  timestamp: string;          // ISO 8601 format
  source: string;             // Service name that produced the event

  // Tracing
  correlation_id: string;     // Request trace ID
  causation_id?: string;      // ID of event that caused this one

  // Payload
  payload: T;
}
```

### Example

```json
{
  "event_id": "550e8400-e29b-41d4-a716-446655440000",
  "event_type": "assessment.completed",
  "schema_version": "1.0.0",
  "timestamp": "2024-01-15T10:30:00.000Z",
  "source": "agent-service",
  "correlation_id": "req-abc123",
  "causation_id": "evt-xyz789",
  "payload": {
    // Event-specific data
  }
}
```

---

## Topic: `user-events`

### user.registered

Emitted when a new user registers via LinkedIn OAuth.

```typescript
interface UserRegisteredPayload {
  user_id: string;
  linkedin_id: string;        // Primary identifier
  email: string;
  name: string;
  registration_method: "linkedin";  // LinkedIn-only auth
  organization_id?: string;   // If invited to org
}
```

**Example:**
```json
{
  "event_type": "user.registered",
  "payload": {
    "user_id": "usr_abc123",
    "linkedin_id": "john-doe-12345",
    "email": "john@example.com",
    "name": "John Doe",
    "registration_method": "linkedin"
  }
}
```

**Consumers:**
- `user-service`: Creates profile document

---

### user.profile_updated

Emitted when user profile is updated.

```typescript
interface UserProfileUpdatedPayload {
  user_id: string;
  updated_fields: string[];
  update_source: "manual" | "linkedin_refresh" | "cv_analysis";
}
```

---

### user.cv_analyzed

Emitted when CV analysis completes.

```typescript
interface UserCVAnalyzedPayload {
  user_id: string;
  cv_file_path: string;
  skills_extracted: string[];
  experience_years: number;
  seniority_level: string;
  suggested_assessments: string[];
  analysis_model: string;     // LLM model used
}
```

**Consumers:**
- `scoring-service`: May trigger job matching

---

### user.linkedin_imported

Emitted when LinkedIn data is imported.

```typescript
interface UserLinkedInImportedPayload {
  user_id: string;
  linkedin_id: string;
  skills_count: number;
  experience_count: number;
  education_count: number;
}
```

---

### user.skill_verified

Emitted when a skill is verified via assessment.

```typescript
interface UserSkillVerifiedPayload {
  user_id: string;
  skill_name: string;
  verified_score: number;
  assessment_id: string;
  assessment_type: string;
}
```

---

## Topic: `assessment-events`

### assessment.scheduled

Emitted when assessment is scheduled.

```typescript
interface AssessmentScheduledPayload {
  assessment_id: string;
  booking_id: string;
  user_id: string;
  organization_id: string;
  type: string;
  template_id: string;
  scheduled_at: string;       // ISO 8601
  duration_minutes: number;
}
```

**Consumers:**
- `notification-service`: Sends confirmation email
- `agent-service`: Pre-warms pods if needed

---

### assessment.started

Emitted when assessment begins.

```typescript
interface AssessmentStartedPayload {
  assessment_id: string;
  booking_id: string;
  linkedin_id: string;        // Candidate's LinkedIn ID
  pod_id: string;             // AI Agent Pod ID
  session_url: string;        // WebRTC session URL
  started_at: string;
  scheduled_at: string;       // For comparison
  delay_seconds: number;      // Time from scheduled to actual start
}
```

**Consumers:**
- `scheduling-service`: Updates booking status

---

### assessment.completed

Emitted when assessment finishes normally.

```typescript
interface AssessmentCompletedPayload {
  assessment_id: string;
  user_id: string;
  organization_id: string;
  type: string;

  // Timing
  started_at: string;
  completed_at: string;
  duration_seconds: number;

  // Summary
  questions_answered: number;
  questions_total: number;

  // Recording
  has_recording: boolean;
  recording_path?: string;

  // Pod
  pod_id: string;
}
```

**Consumers:**
- `scoring-service`: Triggers scoring
- `notification-service`: Sends completion email
- `scheduling-service`: Frees slot

---

### assessment.scored

Emitted when scoring completes.

```typescript
interface AssessmentScoredPayload {
  assessment_id: string;
  user_id: string;
  organization_id: string;
  type: string;

  // Scores
  overall_score: number;
  passed: boolean;
  pass_threshold: number;

  // Dimension scores
  dimensions: {
    [dimension: string]: {
      score: number;
      weight: number;
    };
  };

  // Spider map update
  spider_map_updated: boolean;
  new_dimension_score?: {
    dimension: string;
    score: number;
  };

  scored_at: string;
}
```

**Consumers:**
- `user-service`: Updates spider map
- `notification-service`: Sends results notification
- `report-service`: Generates report

---

### assessment.abandoned

Emitted when candidate abandons assessment.

```typescript
interface AssessmentAbandonedPayload {
  assessment_id: string;
  user_id: string;

  reason: "disconnected" | "timeout" | "user_left" | "pod_failed";

  // Progress
  duration_seconds: number;
  questions_answered: number;
  last_activity: string;

  // Recovery
  resumable: boolean;
  resume_deadline?: string;

  pod_id: string;
}
```

**Consumers:**
- `scheduling-service`: May allow rescheduling
- `notification-service`: Sends follow-up

---

## Topic: `scheduling-events`

### booking.created

Emitted when a booking is created.

```typescript
interface BookingCreatedPayload {
  booking_id: string;
  user_id: string;
  organization_id: string;
  assessment_config_id: string;
  assessment_type: string;

  scheduled_at: string;
  duration_minutes: number;
  timezone: string;

  booking_method: "scheduled" | "on_demand";
}
```

**Consumers:**
- `notification-service`: Sends confirmation
- `scheduling-service`: Schedules reminders

---

### booking.rescheduled

Emitted when booking is rescheduled.

```typescript
interface BookingRescheduledPayload {
  booking_id: string;
  user_id: string;

  old_scheduled_at: string;
  new_scheduled_at: string;

  reschedule_count: number;
  reason?: string;
}
```

**Consumers:**
- `notification-service`: Sends updated confirmation

---

### booking.cancelled

Emitted when booking is cancelled.

```typescript
interface BookingCancelledPayload {
  booking_id: string;
  user_id: string;

  scheduled_at: string;
  cancelled_at: string;
  cancelled_by: string;       // user_id or "system"
  reason?: string;
}
```

**Consumers:**
- `notification-service`: Sends cancellation notice
- `agent-service`: Releases reserved capacity

---

### reminder.sent

Emitted when a reminder is sent.

```typescript
interface ReminderSentPayload {
  reminder_id: string;
  booking_id: string;
  user_id: string;

  reminder_type: "24_hour" | "1_hour" | "15_minute";
  channel: "email" | "sms";

  sent_at: string;
  delivery_status: "sent" | "delivered" | "failed";
}
```

---

## Topic: `pod-events`

### pod.ready

Emitted when an AI Agent Pod becomes ready.

```typescript
interface PodReadyPayload {
  pod_id: string;
  k8s_pod_name: string;
  k8s_node: string;
  ready_at: string;

  configuration: {
    stt_model: string;        // whisper-medium
    tts_voice: string;        // en_US-amy-medium
    llm_backend: string;      // cli | api
  };

  resources: {
    cpu_allocated: number;
    memory_mb: number;
  };
}
```

---

### pod.busy

Emitted when pod starts an assessment session.

```typescript
interface PodBusyPayload {
  pod_id: string;
  k8s_pod_name: string;
  assessment_id: string;
  session_id: string;

  started_at: string;
  expected_duration_minutes: number;

  participants: Array<{
    id: string;
    type: "candidate" | "ai_interviewer" | "human_interviewer";
    name: string;
  }>;
}
```

**Consumers:**
- `scheduling-service`: Updates capacity
- `agent-service`: Queue processing

---

### pod.draining

Emitted when pod is draining before shutdown.

```typescript
interface PodDrainingPayload {
  pod_id: string;
  k8s_pod_name: string;
  reason: "scale_down" | "node_maintenance" | "update";
  drain_started_at: string;
  current_assessment_id?: string;
}
```

---

### pod.terminated

Emitted when pod has been terminated.

```typescript
interface PodTerminatedPayload {
  pod_id: string;
  k8s_pod_name: string;
  terminated_at: string;
  reason: string;
  assessments_completed: number;
  uptime_seconds: number;
}
```

---

### pod.failed

Emitted when pod encounters an error.

```typescript
interface PodFailedPayload {
  pod_id: string;
  k8s_pod_name: string;

  error_type: string;
  error_message: string;
  error_stack?: string;

  // Impact
  assessment_id?: string;     // If failed during assessment
  recovery_action: "restart" | "replace" | "manual";

  failed_at: string;
}
```

**Consumers:**
- `agent-service`: Handles recovery, triggers pod replacement
- `notification-service`: Alerts ops team

---

### session.human_joined

Emitted when a human interviewer joins a session.

```typescript
interface SessionHumanJoinedPayload {
  session_id: string;
  assessment_id: string;
  pod_id: string;

  human_interviewer: {
    user_id: string;
    name: string;
    role: string;
  };

  joined_at: string;
  participant_count: number;
}
```

---

### session.human_left

Emitted when a human interviewer leaves a session.

```typescript
interface SessionHumanLeftPayload {
  session_id: string;
  assessment_id: string;
  pod_id: string;

  human_interviewer_id: string;
  left_at: string;
  remaining_participant_count: number;
}
```

---

## Topic: `audit-events`

### audit.action

Generic audit event for compliance logging.

```typescript
interface AuditActionPayload {
  actor_id: string;
  actor_type: "user" | "system" | "admin";

  action: string;             // e.g., "view_candidate", "export_data"
  resource_type: string;
  resource_id: string;

  details: {
    [key: string]: any;
  };

  ip_address: string;
  user_agent?: string;
  geo_location?: {
    country: string;
    region: string;
  };
}
```

---

## Event Versioning

### Schema Evolution Rules

1. **Adding fields**: Add with default or make optional
2. **Removing fields**: Deprecate first, remove in next major version
3. **Changing types**: New field name, deprecate old
4. **Renaming fields**: New field name, deprecate old

### Version Header

```json
{
  "schema_version": "1.0.0",
  "min_consumer_version": "1.0.0"
}
```

### Migration Example

```typescript
// v1.0.0 → v1.1.0: Added new_field
interface PayloadV1_0 {
  existing_field: string;
}

interface PayloadV1_1 {
  existing_field: string;
  new_field?: string;         // Optional for backwards compat
}

// Consumer handling
function processEvent(event: EventEnvelope<any>) {
  const version = semver.parse(event.schema_version);

  if (semver.gte(version, '1.1.0')) {
    // Can use new_field
  }
}
```

---

## Consumer Guidelines

### Idempotency

```typescript
async function handleEvent(event: EventEnvelope<any>) {
  // Check if already processed
  const processed = await db.events.findOne({
    event_id: event.event_id
  });

  if (processed) {
    return; // Skip duplicate
  }

  // Process event
  await processEventPayload(event.payload);

  // Mark as processed
  await db.events.insertOne({
    event_id: event.event_id,
    processed_at: new Date()
  });
}
```

### Error Handling

```typescript
async function consumeWithRetry(event: EventEnvelope<any>) {
  const maxRetries = 3;
  const backoff = [1000, 5000, 30000]; // ms

  for (let attempt = 0; attempt < maxRetries; attempt++) {
    try {
      await handleEvent(event);
      return;
    } catch (error) {
      if (attempt < maxRetries - 1) {
        await sleep(backoff[attempt]);
      } else {
        // Send to DLQ
        await publishToDLQ(event, error);
      }
    }
  }
}
```

---

*Document Version: 3.0*
*Last Updated: 2026-01-04*
*Owner: Architecture Team*
