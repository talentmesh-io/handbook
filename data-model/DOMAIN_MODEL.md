# Talent Mesh Domain Model

## Overview

This document defines the domain model for Talent Mesh using Domain-Driven Design (DDD) principles. It identifies bounded contexts, aggregates, entities, and value objects.

---

## Strategic Design

### Domain Vision Statement

Talent Mesh is an AI-powered talent assessment platform that evaluates candidates across multiple dimensions through automated video interviews, creating comprehensive skill profiles that match candidates to opportunities.

### Subdomains

```mermaid
graph TB
    subgraph "Core Domain"
        ASSESS[Assessment<br/>Core value - AI interviews]
    end

    subgraph "Supporting Domains"
        PROFILE[Profile<br/>Candidate data]
        SCHEDULE[Scheduling<br/>Booking management]
        SCORING[Scoring<br/>Evaluation engine]
    end

    subgraph "Generic Domains"
        AUTH[Identity<br/>Authentication]
        NOTIFY[Notification<br/>Communications]
    end

    ASSESS --> PROFILE
    ASSESS --> SCHEDULE
    ASSESS --> SCORING
    PROFILE --> AUTH
    SCHEDULE --> NOTIFY
```

| Subdomain | Type | Description |
|-----------|------|-------------|
| Assessment | Core | AI-powered interview execution |
| Scoring | Core | Multi-dimensional evaluation |
| Profile | Supporting | Candidate data management |
| Scheduling | Supporting | Availability and booking |
| Identity | Generic | Authentication/Authorization |
| Notification | Generic | Email and webhooks |

---

## Bounded Contexts

### Context Map

```mermaid
graph LR
    subgraph "Identity Context"
        IC[Identity Context]
    end

    subgraph "Profile Context"
        PC[Profile Context]
    end

    subgraph "Assessment Context"
        AC[Assessment Context]
    end

    subgraph "Scheduling Context"
        SC[Scheduling Context]
    end

    subgraph "Scoring Context"
        SCC[Scoring Context]
    end

    subgraph "Agent Context"
        AGC[Agent Context]
    end

    IC -->|U/D| PC
    PC -->|U/D| AC
    SC -->|U/D| AC
    AC -->|U/D| SCC
    AC -->|Partnership| AGC
```

### Context Relationships

| Upstream | Downstream | Relationship | Description |
|----------|------------|--------------|-------------|
| Identity | Profile | Customer-Supplier | Profile depends on user identity |
| Profile | Assessment | Customer-Supplier | Assessment uses candidate profiles |
| Scheduling | Assessment | Customer-Supplier | Assessment triggered by bookings |
| Assessment | Scoring | Customer-Supplier | Scoring processes completed assessments |
| Assessment | Agent | Partnership | Tight integration for real-time interviews |

---

## Bounded Context Details

### 1. Identity Context

**Responsibility:** User authentication, authorization, and organization management

```mermaid
classDiagram
    class User {
        +UUID id
        +String linkedinId PK
        +Email email
        +String passwordHash
        +Boolean emailVerified
        +VerificationStatus verificationStatus
        +DateTime createdAt
    }

    class VerificationDocument {
        <<entity>>
        +UUID id
        +UUID userId
        +VerificationType type
        +VerificationStatus status
        +String documentUrl
        +DateTime verifiedAt
        +String verifierNotes
    }

    class VerificationType {
        <<enumeration>>
        GOVERNMENT_ID
        LINKEDIN_PROFILE
        EMPLOYMENT_VERIFICATION
        EDUCATION_VERIFICATION
        CERTIFICATION
    }

    class VerificationStatus {
        <<enumeration>>
        PENDING
        IN_REVIEW
        VERIFIED
        REJECTED
        EXPIRED
    }

    User "1" --> "*" VerificationDocument

    class Organization {
        +UUID id
        +String name
        +String slug
        +Settings settings
        +DateTime createdAt
    }

    class Membership {
        +UUID userId
        +UUID organizationId
        +Role role
        +DateTime joinedAt
    }

    class Role {
        <<enumeration>>
        ADMIN
        RECRUITER
        CANDIDATE
    }

    User "1" --> "*" Membership
    Organization "1" --> "*" Membership
```

**Aggregates:**
- **User** (Aggregate Root): User identity and credentials (linkedinId as primary identifier)
- **Organization** (Aggregate Root): Company and team structure
- **VerificationDocument** (Entity): User verification documents

**Domain Events:**
- `UserRegistered`
- `UserLoggedIn`
- `OrganizationCreated`
- `MemberInvited`
- `VerificationDocumentSubmitted`
- `VerificationDocumentApproved`
- `VerificationDocumentRejected`

---

### 2. Profile Context

**Responsibility:** Candidate profile management, CV analysis, skill tracking

```mermaid
classDiagram
    class CandidateProfile {
        +UUID id
        +UUID userId
        +PersonalInfo personalInfo
        +LinkedInData linkedIn
        +CVData cv
        +List~Skill~ skills
        +SpiderMap spiderMap
        +Int completeness
    }

    class PersonalInfo {
        <<value object>>
        +String name
        +String headline
        +String location
        +String phone
    }

    class LinkedInData {
        <<value object>>
        +String linkedInId
        +String profileUrl
        +List~Experience~ experiences
        +List~Education~ education
        +DateTime importedAt
    }

    class CVData {
        <<value object>>
        +String filePath
        +String rawText
        +ParsedCV parsed
        +CVAnalysis analysis
        +DateTime analyzedAt
    }

    class Skill {
        <<entity>>
        +String name
        +SkillSource source
        +Boolean verified
        +Int verifiedScore
        +DateTime verifiedAt
    }

    class SpiderMap {
        <<value object>>
        +Map~String,DimensionScore~ dimensions
        +Float completeness
    }

    CandidateProfile *-- PersonalInfo
    CandidateProfile *-- LinkedInData
    CandidateProfile *-- CVData
    CandidateProfile *-- "*" Skill
    CandidateProfile *-- SpiderMap
```

**Aggregates:**
- **CandidateProfile** (Aggregate Root): Complete candidate profile

**Domain Events:**
- `ProfileCreated`
- `ProfileUpdated`
- `CVUploaded`
- `CVAnalyzed`
- `LinkedInImported`
- `SkillVerified`
- `SpiderMapUpdated`

---

### 3. Assessment Context

**Responsibility:** Assessment configuration, execution, and transcript management

```mermaid
classDiagram
    class AssessmentConfig {
        +UUID id
        +UUID organizationId
        +AssessmentType type
        +UUID templateId
        +Int passingThreshold
        +Int durationMinutes
    }

    class Assessment {
        +UUID id
        +UUID userId
        +UUID configId
        +UUID bookingId
        +AssessmentStatus status
        +DateTime startedAt
        +DateTime completedAt
        +Transcript transcript
        +Recording recording
    }

    class AssessmentTemplate {
        +UUID id
        +String name
        +AssessmentType type
        +List~Section~ sections
        +ScoringConfig scoring
    }

    class Section {
        <<entity>>
        +String name
        +Int durationMinutes
        +List~Question~ questions
    }

    class Question {
        <<entity>>
        +UUID id
        +String topic
        +String questionText
        +Difficulty difficulty
        +String expectedAnswer
        +List~FollowUp~ followUps
    }

    class Transcript {
        <<value object>>
        +List~TranscriptEntry~ entries
    }

    class TranscriptEntry {
        <<value object>>
        +Speaker speaker
        +String content
        +DateTime timestamp
        +Sentiment sentiment
    }

    AssessmentConfig --> AssessmentTemplate
    Assessment --> AssessmentConfig
    Assessment *-- Transcript
    AssessmentTemplate *-- "*" Section
    Section *-- "*" Question
```

#### Assessment History Aggregate

Tracks assessment retakes and historical performance for a candidate.

```mermaid
classDiagram
    class AssessmentHistory {
        +UUID id
        +UUID userId
        +UUID configId
        +AssessmentType type
        +List~AssessmentAttempt~ attempts
        +Int totalAttempts
        +Int passedAttempts
        +Float bestScore
        +DateTime firstAttemptAt
        +DateTime lastAttemptAt
        +CooldownPolicy cooldownPolicy
    }

    class AssessmentAttempt {
        <<entity>>
        +UUID assessmentId
        +Int attemptNumber
        +Float score
        +Boolean passed
        +DateTime completedAt
        +AttemptStatus status
        +String failureReason
    }

    class CooldownPolicy {
        <<value object>>
        +Int cooldownDays
        +Int maxAttempts
        +DateTime nextEligibleAt
        +Boolean isEligible
    }

    class AttemptStatus {
        <<enumeration>>
        COMPLETED
        ABANDONED
        NO_SHOW
        INVALIDATED
    }

    AssessmentHistory *-- "*" AssessmentAttempt
    AssessmentHistory *-- CooldownPolicy
```

#### Assessment State Machine

The Assessment entity follows a comprehensive state machine that handles WebRTC connection, AI/Human modes, and various failure scenarios.

```mermaid
stateDiagram-v2
    [*] --> Scheduled: booking.confirmed

    Scheduled --> Connecting: candidate.join
    Scheduled --> NoShow: timeout(15min)
    Scheduled --> Cancelled: booking.cancelled

    state "Connecting" as Connecting {
        [*] --> WebRTCHandshake
        WebRTCHandshake --> ICEExchange
        ICEExchange --> P2PEstablished
        P2PEstablished --> [*]
    }

    Connecting --> ConnectionFailed: ice.failed
    Connecting --> AIReady: connection.established
    ConnectionFailed --> Connecting: retry(3x)
    ConnectionFailed --> Failed: max_retries

    AIReady --> InProgress: assessment.start

    state "InProgress" as InProgress {
        [*] --> AIMode
        AIMode --> HumanJoining: human.request_join
        HumanJoining --> HybridMode: human.connected
        HybridMode --> AIPaused: ai.pause
        AIPaused --> HybridMode: ai.resume
        HybridMode --> AIMode: human.left
        AIMode --> [*]: questions.complete
        HybridMode --> [*]: questions.complete
    }

    InProgress --> Paused: connection.lost
    Paused --> InProgress: connection.restored
    Paused --> Abandoned: timeout(5min)

    InProgress --> Completed: assessment.finish
    InProgress --> Abandoned: candidate.disconnect

    Completed --> Scoring: transcript.ready
    Scoring --> Scored: score.calculated
    Scored --> [*]

    NoShow --> [*]
    Cancelled --> [*]
    Failed --> [*]
    Abandoned --> [*]

    note right of AIMode
        STT → LLM → TTS
        AI asks questions
        Records responses
    end note

    note right of HybridMode
        Up to 5 participants
        Full mesh WebRTC
        AI can be paused
    end note
```

#### Assessment Status Values

| Status | Description | Transitions To |
|--------|-------------|----------------|
| `SCHEDULED` | Booking confirmed, awaiting candidate | CONNECTING, NO_SHOW, CANCELLED |
| `CONNECTING` | WebRTC handshake in progress | AI_READY, CONNECTION_FAILED |
| `AI_READY` | P2P established, AI ready | IN_PROGRESS |
| `IN_PROGRESS` | Assessment actively running | PAUSED, COMPLETED, ABANDONED |
| `PAUSED` | Connection temporarily lost | IN_PROGRESS, ABANDONED |
| `COMPLETED` | Assessment finished normally | SCORING |
| `SCORING` | Transcript being evaluated | SCORED |
| `SCORED` | Final scores available | (terminal) |
| `NO_SHOW` | Candidate didn't join within timeout | (terminal) |
| `CANCELLED` | Booking cancelled before start | (terminal) |
| `ABANDONED` | Candidate disconnected during assessment | (terminal) |
| `FAILED` | Technical failure prevented completion | (terminal) |

#### Session Mode Transitions

```mermaid
stateDiagram-v2
    [*] --> AIOnly: default

    AIOnly --> HumanJoining: human.request_join
    HumanJoining --> Hybrid: human.connected
    Hybrid --> AIPaused: human.pause_ai
    AIPaused --> Hybrid: human.resume_ai
    Hybrid --> AIOnly: all_humans.left
    AIOnly --> HumanOnly: ai.disable

    HumanOnly --> [*]: session.end
    AIOnly --> [*]: session.end
    Hybrid --> [*]: session.end
```

| Mode | Participants | AI Behavior |
|------|--------------|-------------|
| `AI_ONLY` | Candidate + AI | AI asks questions, STT/TTS active |
| `HUMAN_JOINING` | Candidate + AI + Human (connecting) | AI continues, signaling in progress |
| `HYBRID` | Candidate + AI + 1-3 Humans | AI and humans can both interact |
| `AI_PAUSED` | Candidate + Humans (AI silent) | AI listens but doesn't speak |
| `HUMAN_ONLY` | Candidate + Humans | AI disabled, recording only |

**Aggregates:**
- **AssessmentTemplate** (Aggregate Root): Reusable assessment structure
- **Assessment** (Aggregate Root): Individual assessment instance
- **AssessmentHistory** (Aggregate Root): Candidate's assessment attempt history for retakes

**Domain Events:**
- `AssessmentScheduled`
- `AssessmentStarted`
- `AssessmentCompleted`
- `AssessmentAbandoned`
- `TranscriptUpdated`
- `RetakeRequested`
- `CooldownExpired`
- `HumanInterviewerJoined`
- `HumanInterviewerLeft`
- `AIPaused`
- `AIResumed`
- `ConnectionLost`
- `ConnectionRestored`

---

### 4. Scheduling Context

**Responsibility:** Slot availability, booking management, reminders

```mermaid
classDiagram
    class Booking {
        +UUID id
        +UUID userId
        +UUID configId
        +UUID organizationId
        +DateTime scheduledAt
        +Int durationMinutes
        +BookingStatus status
        +String meetingUrl
        +Int rescheduleCount
    }

    class TimeSlot {
        <<value object>>
        +DateTime startTime
        +DateTime endTime
        +Int capacity
        +Int available
    }

    class Reminder {
        +UUID id
        +UUID bookingId
        +ReminderType type
        +DateTime scheduledFor
        +DateTime sentAt
        +ReminderStatus status
    }

    class BookingStatus {
        <<enumeration>>
        SCHEDULED
        IN_PROGRESS
        COMPLETED
        CANCELLED
        NO_SHOW
    }

    Booking "1" --> "*" Reminder
```

**Aggregates:**
- **Booking** (Aggregate Root): Assessment booking

**Domain Events:**
- `BookingCreated`
- `BookingRescheduled`
- `BookingCancelled`
- `ReminderSent`

---

### 5. Scoring Context

**Responsibility:** Assessment evaluation, scoring, and result generation

```mermaid
classDiagram
    class AssessmentScore {
        +UUID id
        +UUID assessmentId
        +Int overallScore
        +Boolean passed
        +Map~String,DimensionScore~ dimensions
        +Map~String,Int~ topicScores
        +SentimentSummary sentiment
        +String justification
        +DateTime scoredAt
    }

    class DimensionScore {
        <<value object>>
        +String dimensionName
        +Int score
        +Float weight
    }

    class SentimentSummary {
        <<value object>>
        +Float confidence
        +Float stress
        +Float enthusiasm
        +List~SentimentMarker~ markers
    }

    class JobMatch {
        +UUID id
        +UUID candidateId
        +UUID jobId
        +Float matchScore
        +List~String~ missingDimensions
        +DateTime calculatedAt
    }

    class MatchScore {
        <<value object>>
        +Float technicalFit
        +Float cultureFit
        +Float experienceFit
        +CostOfLivingFactors colFactors
        +Float adjustedSalaryRange
        +Float overallScore
    }

    class CostOfLivingFactors {
        <<value object>>
        +String candidateLocation
        +String jobLocation
        +Float colIndex
        +Float salaryAdjustmentFactor
        +String currencyCode
    }

    class BrainDumpDetection {
        <<value object>>
        +Float brainDumpScore
        +List~String~ suspiciousPatterns
        +Float responseTimeVariance
        +Float coherenceScore
        +Boolean flagged
        +String analysisNotes
    }

    AssessmentScore *-- "*" DimensionScore
    AssessmentScore *-- SentimentSummary
    AssessmentScore *-- BrainDumpDetection
    JobMatch *-- MatchScore
    MatchScore *-- CostOfLivingFactors
```

**Aggregates:**
- **AssessmentScore** (Aggregate Root): Assessment evaluation results
- **JobMatch** (Aggregate Root): Candidate-job matching with cost-of-living adjustments

**Domain Events:**
- `AssessmentScored`
- `SpiderMapGenerated`
- `JobMatchCalculated`
- `BrainDumpDetected`

---

### 6. Agent Context

**Responsibility:** AI Agent Pod lifecycle management, capacity scaling, session orchestration

```mermaid
classDiagram
    class PodPool {
        +Int minPods
        +Int maxPods
        +List~AgentPod~ pods
        +Queue assessmentQueue
        +scaleUp()
        +scaleDown()
        +assignAssessment()
    }

    class AgentPod {
        <<entity>>
        +UUID id
        +String k8sPodName
        +String k8sNode
        +PodStatus status
        +UUID currentAssessmentId
        +DateTime createdAt
        +DateTime lastHeartbeat
        +Int assessmentsCompleted
        +PodMetrics metrics
    }

    class AssessmentSession {
        <<entity>>
        +UUID sessionId
        +UUID assessmentId
        +UUID podId
        +List~Participant~ participants
        +SessionState state
        +DateTime startedAt
        +addHuman()
        +removeHuman()
    }

    class Participant {
        <<value object>>
        +UUID id
        +ParticipantType type
        +String name
        +Boolean active
        +DateTime joinedAt
    }

    class ParticipantType {
        <<enumeration>>
        CANDIDATE
        AI_INTERVIEWER
        HUMAN_INTERVIEWER
    }

    class PodMetrics {
        <<value object>>
        +Int totalSessions
        +Int successfulSessions
        +Int failedSessions
        +Float successRate
        +Float avgSttLatencyMs
        +Float avgTtsLatencyMs
        +Float avgLlmLatencyMs
        +DateTime lastUpdated
    }

    class PodStatus {
        <<enumeration>>
        PENDING
        READY
        BUSY
        DRAINING
        TERMINATED
    }

    class SessionState {
        <<enumeration>>
        WAITING
        IN_PROGRESS
        PAUSED
        COMPLETED
        FAILED
    }

    PodPool *-- "*" AgentPod
    AgentPod *-- PodMetrics
    AgentPod --> AssessmentSession
    AssessmentSession *-- "*" Participant
```

**Aggregates:**
- **PodPool** (Aggregate Root): Pod lifecycle and capacity management
- **AgentPod** (Aggregate Root): Kubernetes pod hosting AI interview components
- **AssessmentSession** (Aggregate Root): Multi-participant session coordination

**Domain Events:**
- `PodReady`
- `PodBusy`
- `PodDraining`
- `PodTerminated`
- `PodFailed`
- `AssessmentAssigned`
- `SessionStarted`
- `HumanInterviewerJoined`
- `HumanInterviewerLeft`
- `SessionCompleted`

---

## Ubiquitous Language

| Term | Definition |
|------|------------|
| Assessment | A structured evaluation of a candidate's skills through AI conversation |
| Spider Map | Visual representation of candidate capabilities across multiple dimensions |
| Dimension | A category of skills being evaluated (e.g., DevOps, System Design) |
| AI Agent Pod | Kubernetes pod hosting STT, TTS, and LLM components for AI interviews |
| Pod Pool | Collection of AI Agent Pods managed by the Agent Service |
| Assessment Session | A WebRTC session with candidate and AI (optionally human interviewers) |
| Booking | A scheduled appointment for an assessment |
| Template | A reusable structure defining assessment questions and flow |
| Transcript | Complete record of assessment conversation |
| Scoring | Process of evaluating assessment responses |
| Pass Threshold | Minimum score required to pass an assessment |
| Slot | Available time period for scheduling |
| Profile | Aggregated candidate information from all sources |
| Skill | A verified or declared competency |
| Match Score | Compatibility rating between candidate and job with cost-of-living adjustments |
| Verification Document | Document submitted for identity or credential verification |
| Assessment History | Record of all assessment attempts by a candidate for retake tracking |
| Brain Dump Detection | Analysis to detect inauthentic or memorized responses |
| Cost of Living Factor | Adjustment factor for salary based on candidate/job locations |
| Cooldown Policy | Rules governing waiting periods between assessment retakes |
| Pod Metrics | Performance and health metrics for AI Agent Pods |
| Hybrid Interview | Assessment session with both AI and human interviewers |

---

## Domain Services

### Assessment Execution Service
Coordinates the real-time assessment flow between components.

```
interface AssessmentExecutionService {
    startAssessment(assessmentId: UUID): void
    processResponse(assessmentId: UUID, response: AudioChunk): void
    completeAssessment(assessmentId: UUID): void
}
```

### Scoring Service
Calculates multi-dimensional scores from assessment transcripts.

```
interface ScoringService {
    scoreAssessment(assessmentId: UUID): AssessmentScore
    generateSpiderMap(userId: UUID): SpiderMap
    calculateJobMatch(userId: UUID, jobId: UUID): JobMatch
}
```

### CV Analysis Service
Analyzes CV documents using LLM.

```
interface CVAnalysisService {
    analyzeCV(cvData: CVData): CVAnalysis
    extractSkills(cvData: CVData): List<Skill>
    convertToATS(cvData: CVData): ATSFormat
}
```

---

## Anti-Corruption Layers

### LinkedIn ACL
Translates LinkedIn API responses to domain model.

```
LinkedInAPI -> LinkedInACL -> LinkedInData (Value Object)
```

### WebRTC Signaling ACL
Abstracts WebRTC signaling interactions.

```
WebRTC Signaling -> SignalingACL -> AssessmentSession (Entity)
```

---

*Document Version: 3.1*
*Last Updated: 2026-01-07*
*Owner: Architecture Team*
