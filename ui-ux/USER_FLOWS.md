# Talent Mesh User Flows

## Overview

This document defines the key user flows through Talent Mesh, covering all primary user journeys.

---

## Flow Index

| Flow | Actors | Description |
|------|--------|-------------|
| [F1: Candidate Onboarding](#f1-candidate-onboarding) | Candidate | LinkedIn-only signup to profile completion |
| [F2: Verification Upload](#f2-verification-upload) | Candidate | Upload verification documents for blue tick |
| [F3: Schedule Assessment](#f3-schedule-assessment) | Candidate | Booking an assessment slot |
| [F4: Take Assessment](#f4-take-assessment) | Candidate | Complete AI assessment |
| [F5: Assessment Retake](#f5-assessment-retake) | Candidate | Request retake after 2-week wait (max 3) |
| [F6: View Spider Map](#f6-view-spider-map) | Candidate | View personal assessment spider map |
| [F7: Hybrid Interview](#f7-hybrid-interview) | Candidate, Recruiter | AI assessment with optional human interviewer |
| [F8: Recruiter Review](#f8-recruiter-review) | Recruiter | Review candidate results |
| [F9: Template Creation](#f9-template-creation) | Admin | Create assessment template |
| [F10: Organization Setup](#f10-organization-setup) | Admin | Initial org configuration |

---

## F1: Candidate Onboarding

### Flow Diagram

```mermaid
flowchart TD
    START([Candidate visits site]) --> LANDING[Landing Page]
    LANDING --> LINKEDIN_ONLY[LinkedIn Login Page - Single Button]
    LINKEDIN_ONLY --> OAUTH[LinkedIn OAuth Consent]
    OAUTH --> CALLBACK[OAuth Callback]
    CALLBACK --> PROFILE_CHECK{Profile exists?}

    PROFILE_CHECK -->|No| CREATE[Create User & Profile]
    PROFILE_CHECK -->|Yes| UPDATE[Update Profile]

    CREATE --> IMPORT[Import LinkedIn Data]
    UPDATE --> IMPORT

    IMPORT --> ENRICH{Additional Info?}
    ENRICH -->|Yes| CV_UPLOAD[CV Upload Option]
    ENRICH -->|No| VERIFY_PROMPT

    CV_UPLOAD --> PARSE[Parse CV with LLM]
    PARSE --> MERGE[Merge with Profile]
    MERGE --> VERIFY_PROMPT{Verify Identity?}

    VERIFY_PROMPT -->|Yes| VERIFY[Go to Verification Flow]
    VERIFY_PROMPT -->|Skip| DASHBOARD[Candidate Dashboard]

    VERIFY --> DASHBOARD
    DASHBOARD --> END([Onboarding Complete])
```

### Screen Sequence

| Step | Screen | Actions | Data |
|------|--------|---------|------|
| 1 | Landing | View value proposition | - |
| 2 | LinkedIn Login | Click single "Continue with LinkedIn" button | - |
| 3 | LinkedIn OAuth | Grant permissions | r_liteprofile, r_emailaddress |
| 4 | Processing | Auto redirect | LinkedIn data |
| 5 | Profile Setup | Review imported data, upload CV | Name, title, skills |
| 6 | Verification Prompt | Choose to verify or skip | - |
| 7 | Dashboard | View available assessments | Profile, assessments |

### State Transitions

```mermaid
stateDiagram-v2
    [*] --> Anonymous
    Anonymous --> LinkedInAuth: Click LinkedIn (only option)
    LinkedInAuth --> ProfileCreation: New User
    LinkedInAuth --> Dashboard: Existing User
    ProfileCreation --> ProfileEnrichment: Basic Profile Created
    ProfileEnrichment --> VerificationPrompt: Complete/Skip CV
    VerificationPrompt --> Dashboard: Skip Verification
    VerificationPrompt --> VerificationFlow: Start Verification
    VerificationFlow --> Dashboard: Verification Submitted
    Dashboard --> [*]
```

### Authentication Note

**LinkedIn-Only Authentication**: Talent Mesh uses LinkedIn as the sole authentication method. This ensures:
- Professional identity verification
- Reduced fake accounts
- Streamlined onboarding with pre-populated profile data
- No password management overhead

### Error Handling

| Error | Handling | User Message |
|-------|----------|--------------|
| OAuth denied | Return to landing | "Sign-in cancelled. Try again with LinkedIn." |
| LinkedIn API error | Retry with backoff | "Unable to connect to LinkedIn. Retrying..." |
| LinkedIn account issue | Show help | "Please ensure your LinkedIn account is active." |
| CV parse failure | Allow manual entry | "Couldn't read CV. Enter details manually." |

---

## F2: Verification Upload

### Flow Diagram

```mermaid
flowchart TD
    START([Profile Page]) --> VERIFY_BTN[Click 'Get Verified']
    VERIFY_BTN --> INFO[Verification Info Screen]
    INFO --> DOC_SELECT[Select Document Type]

    subgraph "Document Types"
        GOV[Government ID]
        CERT[Professional Certification]
        EDU[Education Credential]
    end

    DOC_SELECT --> GOV & CERT & EDU

    GOV --> UPLOAD_GOV[Upload ID Front/Back]
    CERT --> UPLOAD_CERT[Upload Certificate]
    EDU --> UPLOAD_EDU[Upload Degree/Transcript]

    UPLOAD_GOV --> REVIEW[Review Upload]
    UPLOAD_CERT --> REVIEW
    UPLOAD_EDU --> REVIEW

    REVIEW --> SUBMIT[Submit for Verification]
    SUBMIT --> PENDING[Verification Pending]
    PENDING --> NOTIFY{Verification Result}

    NOTIFY -->|Approved| BADGE[Blue Tick Badge Added]
    NOTIFY -->|Rejected| RETRY[Retry with Different Document]

    BADGE --> END([Verified Profile])
    RETRY --> DOC_SELECT
```

### Document Requirements

| Document Type | Accepted Formats | Max Size | Processing Time |
|---------------|------------------|----------|-----------------|
| Government ID | JPG, PNG, PDF | 10MB | 24-48 hours |
| Professional Cert | PDF | 10MB | 24-48 hours |
| Education Credential | PDF | 10MB | 24-48 hours |

### Verification States

```mermaid
stateDiagram-v2
    [*] --> Unverified
    Unverified --> Pending: Submit Documents
    Pending --> Verified: Approved
    Pending --> Rejected: Failed Verification
    Rejected --> Pending: Resubmit
    Verified --> [*]
```

### Screen Sequence

| Step | Screen | Actions | Data |
|------|--------|---------|------|
| 1 | Profile | Click "Get Verified" button | - |
| 2 | Verification Info | Read requirements, continue | - |
| 3 | Document Type | Select document category | Type selection |
| 4 | Upload | Drag/drop or select files | Document files |
| 5 | Review | Confirm uploads, submit | - |
| 6 | Pending | View status, wait | Submission ID |
| 7 | Result | View approval/rejection | Badge or retry option |

### Error Handling

| Error | Handling | User Message |
|-------|----------|--------------|
| File too large | Reject upload | "File exceeds 10MB limit. Please compress." |
| Invalid format | Reject upload | "Please upload JPG, PNG, or PDF files." |
| Blurry image | Warn user | "Image may be too blurry. Consider retaking." |
| Document expired | Reject | "Document has expired. Please upload valid ID." |

---

## F3: Schedule Assessment

### Flow Diagram

```mermaid
flowchart TD
    START([Dashboard]) --> BROWSE[Browse Available Assessments]
    BROWSE --> SELECT[Select Assessment Type]
    SELECT --> CHECK{Prerequisites met?}

    CHECK -->|No| PREREQ[Show Prerequisites]
    PREREQ --> BROWSE

    CHECK -->|Yes| CALENDAR[Show Available Slots]
    CALENDAR --> SLOT{Select slot?}

    SLOT -->|Scheduled| SELECT_SLOT[Select Time Slot]
    SLOT -->|On-Demand| CHECK_AGENTS{Agents available?}

    CHECK_AGENTS -->|Yes| START_NOW[Start Now]
    CHECK_AGENTS -->|No| QUEUE[Join Queue]

    SELECT_SLOT --> CONFIRM[Confirm Booking]
    CONFIRM --> BOOKED[Assessment Booked]

    QUEUE --> WAIT[Show Wait Time]
    WAIT --> AGENT_READY{Agent Ready?}
    AGENT_READY -->|Yes| START_NOW
    AGENT_READY -->|No| WAIT

    START_NOW --> LAUNCH[Launch Assessment]
    BOOKED --> EMAIL[Send Confirmation Email]
    EMAIL --> REMINDER[Schedule Reminders]

    LAUNCH --> END([Assessment Session])
    REMINDER --> END2([Awaiting Assessment])
```

### Calendar View States

```mermaid
graph TB
    subgraph "Calendar Component"
        MONTH[Month View]
        WEEK[Week View]
        DAY[Day View]

        MONTH --> WEEK --> DAY
    end

    subgraph "Slot States"
        AVAILABLE[Available<br/>Green]
        LIMITED[Limited<br/>Yellow]
        FULL[Full<br/>Red]
        SELECTED[Selected<br/>Blue]
    end

    subgraph "Time Display"
        LOCAL[Local Timezone]
        DURATION[30/45/60 min]
    end
```

### Booking Confirmation

```json
{
  "booking": {
    "id": "bkg_abc123",
    "assessmentType": "DevOps Assessment",
    "dateTime": "2024-01-20T10:00:00Z",
    "duration": 45,
    "timezone": "America/New_York",
    "assessmentUrl": "https://assess.talentmesh.io/session/abc123",
    "lobbyOpensMinutesBefore": 10,
    "reminders": ["24h", "1h", "10m"]
  }
}
```

---

## F4: Take Assessment

### Flow Diagram

```mermaid
flowchart TD
    START([Assessment Time]) --> LOBBY[Assessment Lobby]
    LOBBY --> SYSTEM_CHECK[System Compatibility Check]

    subgraph "Pre-Assessment Lobby"
        SYSTEM_CHECK --> BROWSER{Browser OK?}
        BROWSER -->|No| BROWSER_MSG[Show Browser Requirements]
        BROWSER -->|Yes| PERMISSIONS[Request Permissions]
        PERMISSIONS --> MIC[Camera/Mic/Screen Access]
        MIC --> TEST[Device Testing UI]
        TEST --> CONSENT[Consent & Terms]
    end

    CONSENT --> READY{Ready?}
    READY -->|No| TEST
    READY -->|Yes| WEBRTC[Connect to AI Agent Pod via WebRTC]

    subgraph "WebRTC Connection"
        WEBRTC --> SIGNAL[Signaling Handshake]
        SIGNAL --> ICE[ICE Candidate Exchange]
        ICE --> P2P{P2P Connected?}
        P2P -->|Yes ~80%| DIRECT[Direct P2P Stream]
        P2P -->|No| TURN[STUNner Relay Fallback]
        TURN --> DIRECT
    end

    DIRECT --> INTRO[AI Introduction via TTS]

    subgraph "Assessment Loop"
        INTRO --> QUESTION[AI Asks Question - TTS]
        QUESTION --> LISTEN[Candidate Responds]
        LISTEN --> STT[Speech-to-Text in Pod]
        STT --> LLM[LLM Processing in Pod]
        LLM --> FOLLOWUP{Follow-up needed?}
        FOLLOWUP -->|Yes| QUESTION
        FOLLOWUP -->|No| NEXT{More topics?}
        NEXT -->|Yes| TOPIC[Next Topic]
        TOPIC --> QUESTION
        NEXT -->|No| WRAP
    end

    WRAP[Wrap-up Message] --> END_SESSION[End Assessment]
    END_SESSION --> UPLOAD[Upload Recording to Central]
    UPLOAD --> SCORE[Calculate Scores]
    SCORE --> RESULTS[Show Results Summary]
    RESULTS --> FEEDBACK[Request Feedback]
    FEEDBACK --> END([Complete])
```

### Assessment Interface (WebRTC Portal)

The assessment runs directly in the browser, connecting peer-to-peer with an AI Agent Pod that handles STT, TTS, and LLM processing.

```
┌─────────────────────────────────────────────────────────────────┐
│  Talent Mesh Assessment                          [00:23:45]     │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌──────────────────────┐  ┌────────────────────────────────┐  │
│  │                      │  │  Current Question:              │  │
│  │   Candidate Video    │  │  "Explain how Kubernetes       │  │
│  │   (Self-view)        │  │   handles pod scheduling..."   │  │
│  │                      │  │                                 │  │
│  └──────────────────────┘  │  Topic: Kubernetes              │  │
│                            │  Progress: ████████░░ 40%       │  │
│  ┌──────────────────────┐  └────────────────────────────────┘  │
│  │                      │                                      │
│  │   AI Interviewer     │  ┌────────────────────────────────┐  │
│  │   [Avatar/Waveform]  │  │  Code Editor (when needed)     │  │
│  │                      │  │  ─────────────────────────────  │  │
│  └──────────────────────┘  │  function example() {          │  │
│                            │    // Write your solution      │  │
│  Connection: P2P to AI Pod    │  }                              │  │
│                            └────────────────────────────────┘  │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  🎤 Listening...                                        │   │
│  │  ░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░ (Audio Waveform)    │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  [Mute]  [Camera]  [Screen Share]  [Help]  [End Session]       │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Pre-Assessment Lobby

```
┌─────────────────────────────────────────────────────────────────┐
│  Assessment Lobby                                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  DevOps Technical Assessment                                    │
│  Scheduled: Today at 2:00 PM (starts in 5 minutes)              │
│                                                                 │
├─────────────────────────────────────────────────────────────────┤
│  System Check                                                   │
│  ─────────────────────────────────────────────────────────────  │
│                                                                 │
│  [x] Browser: Chrome 120 (supported)                            │
│  [x] Camera: Logitech HD Webcam                                 │
│  [x] Microphone: Built-in Microphone                            │
│  [ ] Permissions: Click to allow camera/mic                     │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                                                         │   │
│  │              Camera Preview                             │   │
│  │              (Your video will appear here)              │   │
│  │                                                         │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  [Test Audio]  [Test Video]                                     │
│                                                                 │
│  [ ] I agree to the assessment terms and recording consent      │
│                                                                 │
│  [Join Assessment]                                              │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Screen Sequence

| Step | Screen | Actions | Technical Details |
|------|--------|---------|-------------------|
| 1 | Assessment Lobby | View booking details | Load session info |
| 2 | System Check | Browser/device compatibility | WebRTC support check |
| 3 | Permissions | Grant camera/mic/screen | MediaDevices API |
| 4 | Device Test | Test audio/video | Preview media streams |
| 5 | Consent | Accept terms | Recording consent |
| 6 | Connecting | WebRTC handshake | Signaling → ICE → P2P/TURN |
| 7 | Assessment | Interactive AI interview | Audio/video via WebRTC |
| 8 | Complete | View summary | Upload recording |
| 9 | Results | View scores/feedback | Spider map display |

### Connection States

```mermaid
stateDiagram-v2
    [*] --> Lobby
    Lobby --> SystemCheck: Enter
    SystemCheck --> PermissionRequest: Compatible
    SystemCheck --> Incompatible: Not supported
    PermissionRequest --> DeviceTest: Granted
    PermissionRequest --> PermissionDenied: Denied
    DeviceTest --> Consent: Ready
    Consent --> Connecting: Accepted
    Connecting --> Connected: P2P Success
    Connecting --> TURNRelay: Direct Failed
    TURNRelay --> Connected: Relay Success
    Connected --> Assessment: Start
    Assessment --> Complete: Finished
    Complete --> [*]
```

### Real-time Transcript View (Optional)

```
┌─────────────────────────────────────────────────────────────────┐
│  Live Transcript                                     [Toggle]   │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  [10:23:15] AI: Can you explain how Kubernetes handles         │
│  pod scheduling and what factors influence placement            │
│  decisions?                                                     │
│                                                                 │
│  [10:23:45] You: Sure, Kubernetes uses the scheduler           │
│  component to assign pods to nodes. It considers several        │
│  factors like resource requests, node affinity...               │
│  (typing indicator)                                             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Sentiment Indicators (Internal)

```mermaid
graph LR
    subgraph "Real-time Metrics"
        CONF[Confidence<br/>0.72]
        STRESS[Stress<br/>0.35]
        ENTH[Enthusiasm<br/>0.68]
    end

    subgraph "Indicators"
        HIGH[High ≥ 0.7<br/>Green]
        MED[Medium 0.4-0.7<br/>Yellow]
        LOW[Low < 0.4<br/>Red]
    end
```

---

## F5: Assessment Retake

### Flow Diagram

```mermaid
flowchart TD
    START([Assessment Results]) --> VIEW[View Score & Feedback]
    VIEW --> CHECK{Eligible for Retake?}

    CHECK -->|Not Eligible| REASON[Show Ineligibility Reason]
    REASON --> WAIT_INFO[Display Wait Period or Max Attempts]

    CHECK -->|Eligible| RETAKE_BTN[Show Retake Button]
    RETAKE_BTN --> CONFIRM[Confirm Retake Request]
    CONFIRM --> POLICY[Display Retake Policy]

    POLICY --> ACCEPT{Accept Terms?}
    ACCEPT -->|No| CANCEL[Cancel Retake]
    ACCEPT -->|Yes| SCHEDULE[Schedule Retake Assessment]

    SCHEDULE --> BOOK[Booking Confirmed]
    BOOK --> END([Awaiting Retake])

    WAIT_INFO --> END2([Return to Dashboard])
    CANCEL --> END2
```

### Retake Policy

| Rule | Value | Description |
|------|-------|-------------|
| Waiting Period | 14 days | Minimum time between assessment attempts |
| Maximum Attempts | 3 | Total attempts allowed per assessment type |
| Score Retention | Latest | Most recent score is used for evaluation |
| Cooling Period | 6 months | After 3 attempts, wait 6 months to reset |

### Eligibility Check Logic

```mermaid
flowchart TD
    START([Check Eligibility]) --> COUNT{Attempts < 3?}
    COUNT -->|No| INELIGIBLE_MAX[Ineligible: Max Attempts]
    COUNT -->|Yes| TIME{14+ Days Since Last?}
    TIME -->|No| INELIGIBLE_WAIT[Ineligible: Waiting Period]
    TIME -->|Yes| ELIGIBLE[Eligible for Retake]

    INELIGIBLE_MAX --> CALC_RESET[Calculate Reset Date]
    INELIGIBLE_WAIT --> CALC_WAIT[Calculate Days Remaining]
```

### Screen Sequence

| Step | Screen | Actions | Data |
|------|--------|---------|------|
| 1 | Results | View score, check retake eligibility | Score, attempt count |
| 2 | Retake Info | Review policy, confirm intent | Waiting period status |
| 3 | Confirmation | Accept terms, proceed | - |
| 4 | Scheduling | Select new assessment slot | Available times |
| 5 | Confirmed | View booking details | Retake booking |

### Assessment History Display

```
┌─────────────────────────────────────────────────────────────────┐
│  DevOps Assessment History                                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Attempt 1    Jan 5, 2026      Score: 62      [View Details]    │
│  Attempt 2    Jan 20, 2026     Score: 71      [View Details]    │
│  ─────────────────────────────────────────────────────────────  │
│  Retake Eligibility: 1 attempt remaining                        │
│  Next eligible date: Feb 3, 2026 (14 days)                      │
│                                                                 │
│  [Request Retake - Available Feb 3]                             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Error Handling

| Error | Handling | User Message |
|-------|----------|--------------|
| Max attempts reached | Show reset date | "Maximum attempts reached. Eligible again on [date]." |
| Within waiting period | Show countdown | "Please wait [X] days before retaking." |
| Assessment unavailable | Show alternatives | "This assessment is currently unavailable." |

---

## F6: View Spider Map

### Flow Diagram

```mermaid
flowchart TD
    START([Candidate Dashboard]) --> RESULTS[My Results]
    RESULTS --> SELECT[Select Assessment]
    SELECT --> DETAIL[Assessment Detail View]

    DETAIL --> SPIDER[View Spider Map]

    subgraph "Spider Map Components"
        OVERALL[Overall Score]
        DIMS[Dimension Scores]
        BENCH[Benchmark Comparison]
        TREND[Historical Trend]
    end

    SPIDER --> OVERALL & DIMS & BENCH & TREND

    DIMS --> EXPAND[Expand Dimension]
    EXPAND --> BREAKDOWN[View Sub-scores]
    BREAKDOWN --> FEEDBACK[AI Feedback per Area]

    BENCH --> COMPARE[Compare to Role Average]
    TREND --> HISTORY[View Past Attempts]

    FEEDBACK --> IMPROVE[Improvement Suggestions]
    IMPROVE --> RESOURCES[Learning Resources]

    RESOURCES --> END([Action Items])
```

### Spider Map Display

```
┌─────────────────────────────────────────────────────────────────┐
│  Your DevOps Assessment Results                    Score: 78    │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│                      Technical Skills                           │
│                           85                                    │
│                            *                                    │
│                          * | *                                  │
│                        *   |   *                                │
│                      *     |     *                              │
│           Problem  *       |       * Communication              │
│           Solving *        |        *  72                       │
│              82   *        |        *                           │
│                    *       |       *                            │
│                      *     |     *                              │
│                        *   |   *                                │
│                          * | *                                  │
│                            *                                    │
│                      Leadership                                 │
│                          71                                     │
│                                                                 │
│  ━━━ Your Score    ┄┄┄ Role Benchmark (75)                      │
│                                                                 │
├─────────────────────────────────────────────────────────────────┤
│  Strengths: Technical Skills, Problem Solving                   │
│  Areas to Improve: Leadership, Communication                    │
│                                                                 │
│  [View Detailed Breakdown]  [Download Report]  [Share]          │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Screen Sequence

| Step | Screen | Actions | Data |
|------|--------|---------|------|
| 1 | Dashboard | Click "My Results" | - |
| 2 | Results List | Select assessment | Assessment list |
| 3 | Spider Map | View overall visualization | Dimension scores |
| 4 | Dimension Detail | Click dimension for breakdown | Sub-scores, feedback |
| 5 | Improvement | View suggestions and resources | Learning links |

### Dimension Breakdown View

```
┌─────────────────────────────────────────────────────────────────┐
│  Technical Skills Breakdown                        Score: 85    │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Kubernetes & Orchestration        ████████████████░░░░  88%    │
│  CI/CD Pipelines                   ███████████████░░░░░  82%    │
│  Infrastructure as Code            █████████████████░░░  90%    │
│  Monitoring & Observability        ██████████████░░░░░░  78%    │
│                                                                 │
├─────────────────────────────────────────────────────────────────┤
│  AI Feedback:                                                   │
│  "Strong understanding of Kubernetes concepts. Consider         │
│   deepening knowledge in monitoring and alerting strategies."   │
│                                                                 │
│  Suggested Resources:                                           │
│  - Prometheus & Grafana Fundamentals (Udemy)                    │
│  - SRE Book - Monitoring Chapter (Google)                       │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## F7: Hybrid Interview

### Flow Diagram

```mermaid
flowchart TD
    START([Assessment in Progress]) --> AI_RUNNING[AI Conducting Interview]
    AI_RUNNING --> JOIN_REQ{Human wants to join?}

    JOIN_REQ -->|Yes| RECRUITER_JOIN[Recruiter Clicks Join Link]
    JOIN_REQ -->|No| CONTINUE[Continue AI-Only]

    RECRUITER_JOIN --> AUTH[Authenticate via LinkedIn]
    AUTH --> LOBBY[Wait in Lobby]
    LOBBY --> NOTIFICATION[Candidate Notified]
    NOTIFICATION --> ACCEPT{Candidate Accepts?}

    ACCEPT -->|Yes| WEBRTC_JOIN[Join WebRTC Mesh]
    ACCEPT -->|No| STAY_LOBBY[Stay in Lobby]

    WEBRTC_JOIN --> MULTI[Multi-Party Session]

    subgraph "Multi-Party Mode (up to 5)"
        CANDIDATE[Candidate]
        AI[AI Interviewer]
        HUMAN1[Human Interviewer 1]
        HUMAN2[Human Interviewer 2]
    end

    MULTI --> MODES{Interview Mode}
    MODES -->|AI Led| AI_LED[AI Asks, Human Observes]
    MODES -->|Human Led| HUMAN_LED[Human Takes Over]
    MODES -->|Mixed| MIXED[AI + Human Both Ask]

    HUMAN_LED --> MUTE_AI[Mute AI Interviewer]
    MUTE_AI --> HUMAN_CONTROL[Human Controls Session]

    AI_LED --> END([Session Complete])
    HUMAN_LED --> END
    MIXED --> END
    CONTINUE --> END
```

### Hybrid Interview Interface

```
┌─────────────────────────────────────────────────────────────────┐
│  Talent Mesh Assessment (Hybrid)                   [00:23:45]   │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐             │
│  │  Candidate  │  │ AI Agent    │  │ Sarah Chen  │             │
│  │  [Video]    │  │ [Waveform]  │  │ [Video]     │             │
│  │             │  │             │  │  (Recruiter)│             │
│  └─────────────┘  └─────────────┘  └─────────────┘             │
│                                                                 │
│  Current Speaker: AI Interviewer                                │
│  Mode: [AI Led ▼]                                               │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  Current Question:                                      │   │
│  │  "Can you explain your approach to designing a          │   │
│  │   microservices architecture?"                          │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  [Mute AI] [Take Over] [Ask Question] [Leave]                  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Recruiter Join Flow

```
┌─────────────────────────────────────────────────────────────────┐
│  Join Live Assessment                                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  DevOps Assessment - John Smith                                 │
│  Started: 10:15 AM | Duration: 23:45                            │
│  Current Topic: Kubernetes                                      │
│                                                                 │
│  ────────────────────────────────────────────────────────────   │
│                                                                 │
│  Status: AI Interview in Progress                               │
│                                                                 │
│  You will join as an observer. The candidate will be            │
│  notified and must accept your presence.                        │
│                                                                 │
│  Options:                                                       │
│  ○ Join as Observer (listen only)                               │
│  ● Join as Co-Interviewer (can ask questions)                   │
│                                                                 │
│  [Cancel]                              [Join Assessment]        │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Screen Sequence

| Step | Screen | Actions | Data |
|------|--------|---------|------|
| 1 | Recruiter Dashboard | Click "Join Live" on active assessment | Assessment ID |
| 2 | Join Options | Select observer or co-interviewer mode | Role |
| 3 | Lobby | Wait for candidate acceptance | - |
| 4 | Assessment | Join multi-party session | WebRTC mesh |
| 5 | Controls | Use AI controls, ask questions | - |
| 6 | Complete | Session ends, scores generated | Report |

### Multi-Party WebRTC Mesh

```mermaid
graph TB
    subgraph "WebRTC P2P Mesh (Max 5)"
        C[Candidate]
        AI[AI Agent Pod]
        H1[Human 1]
        H2[Human 2]

        C <-->|Media| AI
        C <-->|Media| H1
        C <-->|Media| H2
        AI <-->|Media| H1
        AI <-->|Media| H2
        H1 <-->|Media| H2
    end
```

### Error Handling

| Error | Handling | User Message |
|-------|----------|--------------|
| Session full | Prevent join | "Maximum 5 participants reached." |
| Candidate declined | Return to dashboard | "Candidate declined your join request." |
| Connection failed | Retry or fallback | "Connection failed. Retrying via relay..." |
| AI takeover conflict | Queue request | "Another interviewer is controlling AI. Wait for handover." |

---

## F8: Recruiter Review

### Flow Diagram

```mermaid
flowchart TD
    START([Recruiter Dashboard]) --> FILTER[Filter Candidates]

    subgraph "Filter Options"
        F1[Assessment Type]
        F2[Score Range]
        F3[Date Range]
        F4[Status]
        F5[Skills Match]
    end

    FILTER --> LIST[Candidate List View]
    LIST --> SELECT[Select Candidate]
    SELECT --> DETAIL[Candidate Detail View]

    subgraph "Detail View"
        SPIDER[Spider Map]
        SCORES[Score Breakdown]
        TRANS[Full Transcript]
        RECORD[Recording if available]
        SENTIMENT[Sentiment Timeline]
    end

    DETAIL --> ACTION{Take Action}
    ACTION -->|Advance| ADVANCE[Move to Next Stage]
    ACTION -->|Reject| REJECT[Archive with Reason]
    ACTION -->|Hold| HOLD[Mark for Later]
    ACTION -->|Compare| COMPARE[Compare with Others]

    COMPARE --> SIDE_BY_SIDE[Side-by-Side View]
    SIDE_BY_SIDE --> ACTION

    ADVANCE --> NEXT_STAGE[Schedule Next Assessment]
    REJECT --> FEEDBACK_OPT[Send Feedback Optional]

    NEXT_STAGE --> END([Updated])
    FEEDBACK_OPT --> END
    HOLD --> END
```

### Candidate List View

```
┌─────────────────────────────────────────────────────────────────────────┐
│  Candidates                                              [+ New Invite] │
├─────────────────────────────────────────────────────────────────────────┤
│  Filters: [DevOps ▼] [Score ≥70 ▼] [This Week ▼] [All Status ▼]        │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ┌─────┬──────────────────┬──────────────┬───────┬─────────┬─────────┐ │
│  │ ☐   │ Candidate        │ Assessment   │ Score │ Date    │ Status  │ │
│  ├─────┼──────────────────┼──────────────┼───────┼─────────┼─────────┤ │
│  │ ☐   │ 👤 John Smith    │ DevOps       │ 85    │ Jan 15  │ ✅ Pass │ │
│  │ ☐   │ 👤 Sarah Chen    │ DevOps       │ 78    │ Jan 14  │ ✅ Pass │ │
│  │ ☐   │ 👤 Mike Johnson  │ DevOps       │ 65    │ Jan 14  │ ⏳ Review│ │
│  │ ☐   │ 👤 Lisa Park     │ DevOps       │ 72    │ Jan 13  │ ✅ Pass │ │
│  │ ☐   │ 👤 David Brown   │ DevOps       │ 45    │ Jan 13  │ ❌ Fail │ │
│  └─────┴──────────────────┴──────────────┴───────┴─────────┴─────────┘ │
│                                                                         │
│  Showing 5 of 23 candidates                        [< 1 2 3 4 5 >]     │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### Spider Map Visualization

```
                    Technical Skills
                         100
                          │
                          │
              80 ─────────┼───────── 80
            ╱             │             ╲
          ╱               │               ╲
Problem   ╲               │               ╱  Communication
Solving    ╲              │              ╱
             ╲            │            ╱
               ╲          │          ╱
                 ╲        │        ╱
                   ╲      │      ╱
                     ╲    │    ╱
                       ╲  │  ╱
              Adaptability─┴─Leadership


Legend:
━━━ Candidate Score
┄┄┄ Role Benchmark
░░░ Team Average
```

---

## F9: Template Creation

### Flow Diagram

```mermaid
flowchart TD
    START([Admin Dashboard]) --> TEMPLATES[Template Library]
    TEMPLATES --> NEW[Create New Template]

    NEW --> BASIC[Basic Information]
    BASIC --> TYPE[Select Assessment Type]

    subgraph "Configure Template"
        TYPE --> TOPICS[Define Topics]
        TOPICS --> QUESTIONS[Question Bank]
        QUESTIONS --> WEIGHTS[Set Weights]
        WEIGHTS --> CRITERIA[Evaluation Criteria]
        CRITERIA --> DURATION[Set Duration]
    end

    DURATION --> PREVIEW[Preview Template]
    PREVIEW --> TEST{Test Run?}

    TEST -->|Yes| MOCK[Mock Assessment]
    MOCK --> ADJUST{Adjustments?}
    ADJUST -->|Yes| TOPICS
    ADJUST -->|No| PUBLISH

    TEST -->|No| PUBLISH[Publish Template]
    PUBLISH --> ASSIGN[Assign to Roles]

    ASSIGN --> END([Template Active])
```

### Template Configuration

```
┌─────────────────────────────────────────────────────────────────────────┐
│  Create Assessment Template                                             │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  Template Name: [Senior DevOps Engineer Assessment        ]             │
│                                                                         │
│  Type: [Technical ▼]    Duration: [45 min ▼]    Difficulty: [Senior ▼] │
│                                                                         │
├─────────────────────────────────────────────────────────────────────────┤
│  TOPICS & WEIGHTS                                            [+ Add]   │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ┌──────────────────────────────────────────────────────────────────┐  │
│  │  1. Kubernetes & Orchestration                                   │  │
│  │     Weight: [25%]  Questions: 5-7                                │  │
│  │     ☐ Pod lifecycle  ☐ Networking  ☐ Security  ☐ Scaling        │  │
│  ├──────────────────────────────────────────────────────────────────┤  │
│  │  2. CI/CD Pipelines                                              │  │
│  │     Weight: [20%]  Questions: 4-5                                │  │
│  │     ☐ GitHub Actions  ☐ Jenkins  ☐ ArgoCD  ☐ Testing            │  │
│  ├──────────────────────────────────────────────────────────────────┤  │
│  │  3. Infrastructure as Code                                       │  │
│  │     Weight: [20%]  Questions: 4-5                                │  │
│  │     ☐ Terraform  ☐ Ansible  ☐ CloudFormation                    │  │
│  ├──────────────────────────────────────────────────────────────────┤  │
│  │  4. Problem Solving                                              │  │
│  │     Weight: [20%]  Questions: 3-4                                │  │
│  │     ☐ Debugging  ☐ Incident Response  ☐ Optimization            │  │
│  ├──────────────────────────────────────────────────────────────────┤  │
│  │  5. Communication & Soft Skills                                  │  │
│  │     Weight: [15%]  Questions: 2-3                                │  │
│  │     ☐ Explanation  ☐ Collaboration  ☐ Documentation             │  │
│  └──────────────────────────────────────────────────────────────────┘  │
│                                                                         │
│  [Cancel]                                    [Save Draft] [Preview]     │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## F10: Organization Setup

### Flow Diagram

```mermaid
flowchart TD
    START([Sign Up]) --> ADMIN_REG[Admin Registration]
    ADMIN_REG --> VERIFY[Email Verification]
    VERIFY --> ORG_CREATE[Create Organization]

    subgraph "Organization Setup"
        ORG_CREATE --> ORG_INFO[Organization Details]
        ORG_INFO --> BRANDING[Upload Logo/Colors]
        BRANDING --> INVITE[Invite Team Members]
    end

    INVITE --> ROLES[Assign Roles]

    subgraph "Configure Assessments"
        ROLES --> TEMPLATES[Select/Create Templates]
        TEMPLATES --> SETTINGS[Assessment Settings]
        SETTINGS --> CALENDAR[Calendar Setup]
    end

    CALENDAR --> INTEGRATE{Integrations?}
    INTEGRATE -->|Yes| ATS[Connect ATS]
    INTEGRATE -->|No| READY

    ATS --> READY[Organization Ready]
    READY --> LAUNCH[Go Live]

    LAUNCH --> END([Dashboard])
```

### Setup Wizard

```
┌─────────────────────────────────────────────────────────────────────────┐
│  Welcome to Talent Mesh!                                                │
│  Let's set up your organization                          Step 2 of 5   │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ● Organization  ● Team  ○ Templates  ○ Calendar  ○ Go Live            │
│  ━━━━━━━━━━━━━━━━━━━━━━━━━░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░   │
│                                                                         │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  INVITE TEAM MEMBERS                                                    │
│                                                                         │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  Email                              │  Role                     │   │
│  ├─────────────────────────────────────┼───────────────────────────┤   │
│  │  john@company.com                   │  [Admin ▼]                │   │
│  │  sarah@company.com                  │  [Recruiter ▼]            │   │
│  │  mike@company.com                   │  [Recruiter ▼]            │   │
│  │  [+ Add another member]             │                           │   │
│  └─────────────────────────────────────┴───────────────────────────┘   │
│                                                                         │
│  💡 Team members will receive an email invitation to join.              │
│                                                                         │
│  [← Back]                                              [Skip] [Next →]  │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## Navigation Map

```mermaid
graph TB
    subgraph "Public"
        LANDING[Landing Page]
        LOGIN[Login]
        SIGNUP[Sign Up]
    end

    subgraph "Candidate Portal"
        C_DASH[Dashboard]
        C_PROFILE[Profile]
        C_SCHEDULE[Schedule]
        C_ASSESS[Assessment Room]
        C_RESULTS[Results]
    end

    subgraph "Recruiter Portal"
        R_DASH[Dashboard]
        R_CANDIDATES[Candidates]
        R_DETAIL[Candidate Detail]
        R_COMPARE[Compare]
        R_REPORTS[Reports]
    end

    subgraph "Admin Portal"
        A_DASH[Dashboard]
        A_TEMPLATES[Templates]
        A_USERS[Users]
        A_SETTINGS[Settings]
        A_ANALYTICS[Analytics]
    end

    LANDING --> LOGIN --> C_DASH & R_DASH & A_DASH
    LANDING --> SIGNUP --> C_DASH

    C_DASH --> C_PROFILE & C_SCHEDULE & C_RESULTS
    C_SCHEDULE --> C_ASSESS

    R_DASH --> R_CANDIDATES --> R_DETAIL
    R_DETAIL --> R_COMPARE
    R_DASH --> R_REPORTS

    A_DASH --> A_TEMPLATES & A_USERS & A_SETTINGS & A_ANALYTICS
```

---

---

## HTML Prototypes

Interactive HTML/CSS wireframe prototypes are available in the `/wireframes/` folder. These provide clickable mockups for user testing and stakeholder review.

See [WIREFRAMES.md](WIREFRAMES.md) for the complete component library and screen specifications.

---

*Document Version: 3.1*
*Last Updated: 2026-01-07*
*Owner: Product Team*
