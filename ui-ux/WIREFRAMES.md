# Talent Mesh Wireframes & Component Library

## Overview

This document provides wireframe specifications and reusable component definitions for Talent Mesh UI.

### HTML Prototypes

Interactive HTML/CSS wireframe prototypes are available in the `/wireframes/` folder. These clickable prototypes provide:
- Realistic user interactions for testing
- Stakeholder review and feedback collection
- Developer reference for implementation

---

## Design System

### Color Palette

| Color | Hex | Usage |
|-------|-----|-------|
| Primary | #2563EB | CTAs, links, active states |
| Primary Dark | #1D4ED8 | Hover states |
| Secondary | #64748B | Secondary text, icons |
| Success | #10B981 | Pass, success states |
| Warning | #F59E0B | Attention, pending |
| Error | #EF4444 | Errors, failures |
| Background | #F8FAFC | Page background |
| Surface | #FFFFFF | Card backgrounds |
| Border | #E2E8F0 | Borders, dividers |

### Typography

| Style | Font | Size | Weight | Usage |
|-------|------|------|--------|-------|
| H1 | Inter | 32px | 700 | Page titles |
| H2 | Inter | 24px | 600 | Section headers |
| H3 | Inter | 20px | 600 | Card headers |
| H4 | Inter | 16px | 600 | Subsections |
| Body | Inter | 14px | 400 | Main content |
| Small | Inter | 12px | 400 | Captions, meta |
| Code | JetBrains Mono | 14px | 400 | Code snippets |

### Spacing Scale

```
4px  - xs
8px  - sm
16px - md
24px - lg
32px - xl
48px - 2xl
64px - 3xl
```

### Breakpoints

| Name | Width | Target |
|------|-------|--------|
| sm | 640px | Mobile landscape |
| md | 768px | Tablet |
| lg | 1024px | Desktop |
| xl | 1280px | Large desktop |
| 2xl | 1536px | Extra large |

---

## Core Components

### Buttons

```
┌─────────────────────────────────────────────────────────────────┐
│  BUTTONS                                                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Primary:                                                       │
│  ┌────────────────┐  ┌────────────────┐  ┌────────────────┐    │
│  │  Get Started   │  │  Get Started   │  │  Get Started   │    │
│  └────────────────┘  └────────────────┘  └────────────────┘    │
│  Default             Hover              Disabled               │
│  bg: #2563EB         bg: #1D4ED8        bg: #94A3B8            │
│                                                                 │
│  Secondary:                                                     │
│  ┌────────────────┐  ┌────────────────┐  ┌────────────────┐    │
│  │  Learn More    │  │  Learn More    │  │  Learn More    │    │
│  └────────────────┘  └────────────────┘  └────────────────┘    │
│  Default             Hover              Disabled               │
│  border: #E2E8F0     bg: #F1F5F9        opacity: 0.5           │
│                                                                 │
│  Ghost:                                                         │
│  ┌────────────────┐  ┌────────────────┐                        │
│  │  Cancel        │  │  Cancel        │                        │
│  └────────────────┘  └────────────────┘                        │
│  Default             Hover                                      │
│  text only           bg: #F1F5F9                                │
│                                                                 │
│  Sizes:                                                         │
│  ┌──────┐  ┌──────────┐  ┌────────────────┐                    │
│  │ SM   │  │   MD     │  │      LG        │                    │
│  └──────┘  └──────────┘  └────────────────┘                    │
│  32px h     40px h        48px h                                │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Form Inputs

```
┌─────────────────────────────────────────────────────────────────┐
│  FORM INPUTS                                                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Text Input:                                                    │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  Email address                                          │   │
│  ├─────────────────────────────────────────────────────────┤   │
│  │  john@example.com                                    ✓  │   │
│  └─────────────────────────────────────────────────────────┘   │
│  Label above, placeholder or value, optional icon              │
│                                                                 │
│  States:                                                        │
│  ┌────────────────────┐ ┌────────────────────┐                 │
│  │  Default           │ │  Focused           │                 │
│  │  border: #E2E8F0   │ │  border: #2563EB   │                 │
│  └────────────────────┘ └────────────────────┘                 │
│  ┌────────────────────┐ ┌────────────────────┐                 │
│  │  Error             │ │  Disabled          │                 │
│  │  border: #EF4444   │ │  bg: #F1F5F9       │                 │
│  └────────────────────┘ └────────────────────┘                 │
│                                                                 │
│  Select:                                                        │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  Assessment Type                                        │   │
│  ├─────────────────────────────────────────────────────────┤   │
│  │  DevOps Assessment                                  ▼   │   │
│  ├─────────────────────────────────────────────────────────┤   │
│  │  ○ DevOps Assessment                                    │   │
│  │  ○ Backend Development                                  │   │
│  │  ○ System Design                                        │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  Checkbox & Radio:                                              │
│  ☐ Unchecked    ☑ Checked    ○ Unselected    ● Selected        │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Cards

```
┌─────────────────────────────────────────────────────────────────┐
│  CARDS                                                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Basic Card:                                                    │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                                                         │   │
│  │  Card Title                                             │   │
│  │  ─────────────────────────────────────────────────      │   │
│  │  Card content goes here. This is a basic card           │   │
│  │  component with header and body.                        │   │
│  │                                                         │   │
│  └─────────────────────────────────────────────────────────┘   │
│  border-radius: 8px, shadow: sm, padding: 24px                 │
│                                                                 │
│  Interactive Card:                                              │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  ┌───┐                                                  │   │
│  │  │ 📊│  DevOps Assessment                               │   │
│  │  └───┘  45 minutes • 5 topics                           │   │
│  │         ────────────────────────────                    │   │
│  │         Evaluate Kubernetes, CI/CD, and                 │   │
│  │         infrastructure skills.                          │   │
│  │                                      [Schedule →]       │   │
│  └─────────────────────────────────────────────────────────┘   │
│  cursor: pointer, hover: shadow-md, transition: 200ms          │
│                                                                 │
│  Stat Card:                                                     │
│  ┌─────────────────────┐                                       │
│  │  Total Assessments  │                                       │
│  │  ────────────────── │                                       │
│  │  1,234              │                                       │
│  │  ↑ 12% from last mo │                                       │
│  └─────────────────────┘                                       │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Navigation

```
┌─────────────────────────────────────────────────────────────────┐
│  NAVIGATION                                                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Top Navigation:                                                │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  🔷 Talent Mesh    Dashboard  Candidates  Reports    👤  │   │
│  └─────────────────────────────────────────────────────────┘   │
│  height: 64px, bg: white, border-bottom: 1px                   │
│                                                                 │
│  Side Navigation:                                               │
│  ┌───────────────────────┐                                     │
│  │  🔷 Talent Mesh       │                                     │
│  │  ─────────────────    │                                     │
│  │  ▪ Dashboard          │ ← Active                            │
│  │  ○ Candidates         │                                     │
│  │  ○ Assessments        │                                     │
│  │  ○ Templates          │                                     │
│  │  ○ Analytics          │                                     │
│  │  ─────────────────    │                                     │
│  │  ○ Settings           │                                     │
│  │  ○ Help               │                                     │
│  └───────────────────────┘                                     │
│  width: 240px, bg: #F8FAFC                                     │
│                                                                 │
│  Breadcrumbs:                                                   │
│  Home / Candidates / John Smith / Assessment Results            │
│                                                                 │
│  Tabs:                                                          │
│  ┌──────────┬──────────┬──────────┬──────────┐                 │
│  │ Overview │ Details  │ Timeline │ Notes    │                 │
│  └──────────┴──────────┴──────────┴──────────┘                 │
│  ━━━━━━━━━━━                                                    │
│  Active has underline indicator                                 │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Data Display

```
┌─────────────────────────────────────────────────────────────────┐
│  DATA DISPLAY                                                   │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Table:                                                         │
│  ┌──────┬────────────────┬────────┬────────┬─────────┐         │
│  │  ☐   │ Name           │ Score  │ Date   │ Actions │         │
│  ├──────┼────────────────┼────────┼────────┼─────────┤         │
│  │  ☐   │ John Smith     │ 85     │ Jan 15 │ ⋮       │         │
│  │  ☐   │ Sarah Chen     │ 78     │ Jan 14 │ ⋮       │         │
│  │  ☐   │ Mike Johnson   │ 65     │ Jan 14 │ ⋮       │         │
│  └──────┴────────────────┴────────┴────────┴─────────┘         │
│  Header: bg: #F8FAFC, font-weight: 600                          │
│  Rows: hover bg: #F1F5F9, border-bottom: 1px                   │
│                                                                 │
│  Badges:                                                        │
│  ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐                  │
│  │ Pass │ │ Fail │ │ Pend │ │ New  │ │ Pro  │                  │
│  └──────┘ └──────┘ └──────┘ └──────┘ └──────┘                  │
│  Success   Error    Warning  Primary  Secondary                 │
│                                                                 │
│  Progress Bar:                                                  │
│  Assessment Progress                                            │
│  ████████████░░░░░░░░░░░░░░░░░░░░ 40%                          │
│  height: 8px, border-radius: 4px                                │
│                                                                 │
│  Avatar:                                                        │
│  ┌───┐  ┌─────┐  ┌───────┐                                     │
│  │JS │  │ JS  │  │  JS   │                                     │
│  └───┘  └─────┘  └───────┘                                     │
│  24px    32px     48px                                          │
│  Initials or image, circular                                    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Feedback

```
┌─────────────────────────────────────────────────────────────────┐
│  FEEDBACK COMPONENTS                                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Toast Notifications:                                           │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  ✓ Assessment scheduled successfully              [×]   │   │
│  └─────────────────────────────────────────────────────────┘   │
│  Position: top-right, auto-dismiss: 5s                         │
│                                                                 │
│  Alert Banners:                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  ⚠ Your assessment starts in 10 minutes    [Join Now]   │   │
│  └─────────────────────────────────────────────────────────┘   │
│  Full width, bg varies by type                                  │
│                                                                 │
│  Empty States:                                                  │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │              ┌─────────────┐                            │   │
│  │              │    📭       │                            │   │
│  │              └─────────────┘                            │   │
│  │          No assessments yet                             │   │
│  │    Schedule your first assessment to get started        │   │
│  │              [Schedule Assessment]                      │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  Loading States:                                                │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │              ◠ ◡ ◠ ◡                                    │   │
│  │          Loading candidates...                          │   │
│  └─────────────────────────────────────────────────────────┘   │
│  Spinner + text, centered                                       │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## Page Wireframes

### Screen Index

| Screen | Description | HTML Prototype |
|--------|-------------|----------------|
| W1: Landing Page | Public homepage | `/wireframes/landing.html` |
| W2: LinkedIn Login | Single-button authentication | `/wireframes/login.html` |
| W3: Candidate Dashboard | Candidate home view | `/wireframes/candidate-dashboard.html` |
| W4: Profile with Verification | Profile with blue tick badge | `/wireframes/profile.html` |
| W5: Assessment History | Past assessments with retake eligibility | `/wireframes/assessment-history.html` |
| W6: Candidate Spider Map | Personal assessment results visualization | `/wireframes/spider-map.html` |
| W7a: Assessment Lobby | Pre-assessment system check and permissions | `/wireframes/assessment-lobby.html` |
| W7b: Assessment Permissions | Camera/mic/screen permission dialogs | `/wireframes/assessment-permissions.html` |
| W7c: Assessment Session | Live AI interview with WebRTC | `/wireframes/assessment-session.html` |
| W8: Recruiter Candidate List | Candidate management view | `/wireframes/recruiter-candidates.html` |
| W9: Candidate Detail View | Full candidate profile | `/wireframes/candidate-detail.html` |
| W10: Assessment Scheduling | Calendar booking interface | `/wireframes/scheduling.html` |
| W11: Admin Template Editor | Assessment template configuration | `/wireframes/template-editor.html` |
| W12: Analytics Dashboard | Metrics and reporting | `/wireframes/analytics.html` |

---

### W1: Landing Page

```
┌─────────────────────────────────────────────────────────────────────────────┐
│  🔷 Talent Mesh                           Features  Pricing  [Login]        │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│                                                                             │
│                    AI-Powered Technical Assessments                         │
│                    ─────────────────────────────────                        │
│                                                                             │
│              Evaluate candidates with intelligent, adaptive                 │
│              assessments that go beyond traditional interviews              │
│                                                                             │
│                    [Get Started Free]  [Watch Demo]                         │
│                                                                             │
│                                                                             │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│     ┌─────────────┐    ┌─────────────┐    ┌─────────────┐                  │
│     │  🤖 AI      │    │  📊 Spider  │    │  ⚡ Real-   │                  │
│     │  Driven     │    │  Map        │    │  Time       │                  │
│     │             │    │  Profiling  │    │  Analysis   │                  │
│     └─────────────┘    └─────────────┘    └─────────────┘                  │
│                                                                             │
│     Adaptive questions   Multi-dimensional   Sentiment and                  │
│     based on responses   skill evaluation    confidence tracking            │
│                                                                             │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│                    Trusted by 500+ Companies                                │
│                                                                             │
│     [Logo 1]  [Logo 2]  [Logo 3]  [Logo 4]  [Logo 5]  [Logo 6]             │
│                                                                             │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  🔷 Talent Mesh    Features | Pricing | About | Contact    © 2026          │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### W2: LinkedIn Login Page

```
┌─────────────────────────────────────────────────────────────────────────────┐
│  🔷 Talent Mesh                                                             │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│                                                                             │
│                                                                             │
│                     ┌─────────────────────────────────┐                     │
│                     │                                 │                     │
│                     │      🔷 Talent Mesh             │                     │
│                     │                                 │                     │
│                     │   Welcome to AI-Powered         │                     │
│                     │   Technical Assessments         │                     │
│                     │                                 │                     │
│                     │   ─────────────────────────     │                     │
│                     │                                 │                     │
│                     │  ┌───────────────────────────┐  │                     │
│                     │  │  🔗 Continue with LinkedIn │  │                     │
│                     │  └───────────────────────────┘  │                     │
│                     │                                 │                     │
│                     │   By continuing, you agree to   │                     │
│                     │   our Terms & Privacy Policy    │                     │
│                     │                                 │                     │
│                     │   ─────────────────────────     │                     │
│                     │                                 │                     │
│                     │   Why LinkedIn?                 │                     │
│                     │   • One-click professional sign-in │                  │
│                     │   • Auto-populate your profile  │                     │
│                     │   • Verified professional identity │                  │
│                     │                                 │                     │
│                     └─────────────────────────────────┘                     │
│                                                                             │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

**Note**: LinkedIn is the sole authentication method. No email/password or other OAuth options are provided.

### W3: Candidate Dashboard

```
┌─────────────────────────────────────────────────────────────────────────────┐
│  🔷 Talent Mesh                                          🔔  👤 John Smith │
├───────────────────────┬─────────────────────────────────────────────────────┤
│                       │                                                     │
│  ▪ Dashboard          │  Welcome back, John!                                │
│  ○ My Assessments     │  ────────────────────────────────────────────      │
│  ○ Profile            │                                                     │
│  ○ Schedule           │  ┌─────────────────────────────────────────────┐   │
│  ○ Results            │  │  🔔 Upcoming Assessment                      │   │
│  ─────────────────    │  │  ─────────────────────────────────────────  │   │
│  ○ Settings           │  │  DevOps Assessment                          │   │
│  ○ Help               │  │  Tomorrow, Jan 16 at 10:00 AM EST           │   │
│                       │  │                                             │   │
│                       │  │  [Join Now]  [Reschedule]                   │   │
│                       │  └─────────────────────────────────────────────┘   │
│                       │                                                     │
│                       │  Available Assessments                              │
│                       │  ─────────────────────────────────────────────      │
│                       │                                                     │
│                       │  ┌───────────────────┐  ┌───────────────────┐      │
│                       │  │  📊 Backend       │  │  🔧 System        │      │
│                       │  │  Development      │  │  Design           │      │
│                       │  │  45 min • 5 topics│  │  60 min • 4 topics│      │
│                       │  │  [Schedule]       │  │  [Schedule]       │      │
│                       │  └───────────────────┘  └───────────────────┘      │
│                       │                                                     │
│                       │  Recent Results                                     │
│                       │  ─────────────────────────────────────────────      │
│                       │                                                     │
│                       │  ┌─────────────────────────────────────────────┐   │
│                       │  │  Python Fundamentals  │  Score: 82  │ Pass │   │
│                       │  │  Completed Jan 10     │  [View Details]     │   │
│                       │  └─────────────────────────────────────────────┘   │
│                       │                                                     │
└───────────────────────┴─────────────────────────────────────────────────────┘
```

### W4: Profile with Verification Badge

```
┌─────────────────────────────────────────────────────────────────────────────┐
│  🔷 Talent Mesh                                          🔔  👤 John Smith │
├───────────────────────┬─────────────────────────────────────────────────────┤
│                       │                                                     │
│  ▪ Dashboard          │  My Profile                          [Edit Profile] │
│  ○ My Assessments     │  ─────────────────────────────────────────────────  │
│  ○ Profile            │                                                     │
│  ○ Schedule           │  ┌─────────────────────────────────────────────────┐│
│  ○ Results            │  │  ┌────────┐                                     ││
│  ─────────────────    │  │  │  👤    │  John Smith  ✓ (Blue Tick)          ││
│  ○ Settings           │  │  │        │  Senior DevOps Engineer              ││
│  ○ Help               │  │  └────────┘  San Francisco, CA                   ││
│                       │  │                                                  ││
│                       │  │  ✓ Verified on Jan 3, 2026                       ││
│                       │  │  📧 john.smith@email.com                         ││
│                       │  │  🔗 linkedin.com/in/johnsmith                    ││
│                       │  │                                                  ││
│                       │  └─────────────────────────────────────────────────┘│
│                       │                                                     │
│                       │  Verification Status                                │
│                       │  ─────────────────────────────────────────────────  │
│                       │                                                     │
│                       │  ┌─────────────────────────────────────────────────┐│
│                       │  │  ✓ Identity Verified       Government ID        ││
│                       │  │  ✓ Professional Verified   LinkedIn Profile     ││
│                       │  │  ○ Education (Optional)    [Upload Credentials] ││
│                       │  └─────────────────────────────────────────────────┘│
│                       │                                                     │
│                       │  Skills & Experience                                │
│                       │  ─────────────────────────────────────────────────  │
│                       │                                                     │
│                       │  [Kubernetes] [Docker] [AWS] [Terraform] [CI/CD]   │
│                       │  [Python] [Go] [Linux] [Prometheus] [ArgoCD]       │
│                       │                                                     │
│                       │  Experience: 8 years in DevOps/SRE                  │
│                       │                                                     │
└───────────────────────┴─────────────────────────────────────────────────────┘
```

**Verification Badge Legend:**
- ✓ Blue Tick: Fully verified identity (government ID + LinkedIn)
- Pending: Verification documents under review
- Unverified: No documents submitted

### W5: Assessment History with Retake Eligibility

```
┌─────────────────────────────────────────────────────────────────────────────┐
│  🔷 Talent Mesh                                          🔔  👤 John Smith │
├───────────────────────┬─────────────────────────────────────────────────────┤
│                       │                                                     │
│  ▪ Dashboard          │  Assessment History                                 │
│  ○ My Assessments     │  ─────────────────────────────────────────────────  │
│  ○ Profile            │                                                     │
│  ○ Schedule           │  ┌─────────────────────────────────────────────────┐│
│  ○ Results            │  │  DevOps Assessment                              ││
│  ─────────────────    │  │  ───────────────────────────────────────────── ││
│  ○ Settings           │  │                                                 ││
│  ○ Help               │  │  Attempt 1    Dec 15, 2025    Score: 62   Fail  ││
│                       │  │  Attempt 2    Jan 2, 2026     Score: 71   Pass  ││
│                       │  │                                                 ││
│                       │  │  ───────────────────────────────────────────── ││
│                       │  │  Retake Status:                                 ││
│                       │  │  • Attempts remaining: 1 of 3                   ││
│                       │  │  • Next eligible: Jan 16, 2026 (14 days)        ││
│                       │  │                                                 ││
│                       │  │  [View Results]  [Request Retake - Jan 16]      ││
│                       │  └─────────────────────────────────────────────────┘│
│                       │                                                     │
│                       │  ┌─────────────────────────────────────────────────┐│
│                       │  │  Backend Development Assessment                 ││
│                       │  │  ───────────────────────────────────────────── ││
│                       │  │                                                 ││
│                       │  │  Attempt 1    Nov 20, 2025    Score: 85   Pass  ││
│                       │  │                                                 ││
│                       │  │  ───────────────────────────────────────────── ││
│                       │  │  Retake Status:                                 ││
│                       │  │  • Attempts remaining: 2 of 3                   ││
│                       │  │  • Eligible now (passed 14-day wait)            ││
│                       │  │                                                 ││
│                       │  │  [View Results]  [Request Retake]               ││
│                       │  └─────────────────────────────────────────────────┘│
│                       │                                                     │
│                       │  ┌─────────────────────────────────────────────────┐│
│                       │  │  System Design Assessment                       ││
│                       │  │  ───────────────────────────────────────────── ││
│                       │  │                                                 ││
│                       │  │  Attempt 1-3   (Completed)    Best: 68          ││
│                       │  │                                                 ││
│                       │  │  ───────────────────────────────────────────── ││
│                       │  │  Retake Status:                                 ││
│                       │  │  • Maximum attempts reached                     ││
│                       │  │  • Reset date: Jun 15, 2026 (6 month cooldown)  ││
│                       │  │                                                 ││
│                       │  │  [View Results]                                 ││
│                       │  └─────────────────────────────────────────────────┘│
│                       │                                                     │
└───────────────────────┴─────────────────────────────────────────────────────┘
```

### W6: Candidate Spider Map View

```
┌─────────────────────────────────────────────────────────────────────────────┐
│  🔷 Talent Mesh                                          🔔  👤 John Smith │
├───────────────────────┬─────────────────────────────────────────────────────┤
│                       │  ← Back to History                                  │
│  ▪ Dashboard          │                                                     │
│  ○ My Assessments     │  DevOps Assessment Results            Score: 71     │
│  ○ Profile            │  ─────────────────────────────────────────────────  │
│  ○ Schedule           │  Completed: January 2, 2026 | Attempt: 2 of 3       │
│  ○ Results            │                                                     │
│  ─────────────────    │  ┌─────────────────────────────────────────────────┐│
│  ○ Settings           │  │                                                 ││
│  ○ Help               │  │                 Technical Skills                ││
│                       │  │                      85                         ││
│                       │  │                       *                         ││
│                       │  │                     * | *                       ││
│                       │  │                   *   |   *                     ││
│                       │  │                 *     |     *                   ││
│                       │  │  Problem      *      |      *     Communication ││
│                       │  │  Solving     *       |       *       68         ││
│                       │  │    78       *        |        *                 ││
│                       │  │              *       |       *                  ││
│                       │  │                *     |     *                    ││
│                       │  │                  *   |   *                      ││
│                       │  │                    * | *                        ││
│                       │  │                      *                          ││
│                       │  │               Adaptability                      ││
│                       │  │                    65                           ││
│                       │  │                                                 ││
│                       │  │  ━━━ Your Score    ┄┄┄ Role Benchmark (75)      ││
│                       │  │                                                 ││
│                       │  └─────────────────────────────────────────────────┘│
│                       │                                                     │
│                       │  Dimension Breakdown                                │
│                       │  ─────────────────────────────────────────────────  │
│                       │                                                     │
│                       │  Technical Skills        ████████████████░░░░ 85%   │
│                       │    • Kubernetes          ████████████████████ 92%   │
│                       │    • CI/CD               ██████████████░░░░░░ 78%   │
│                       │    • IaC                 █████████████████░░░ 88%   │
│                       │                                                     │
│                       │  Problem Solving         ███████████████░░░░░ 78%   │
│                       │  Communication           █████████████░░░░░░░ 68%   │
│                       │  Adaptability            ████████████░░░░░░░░ 65%   │
│                       │                                                     │
│                       │  ─────────────────────────────────────────────────  │
│                       │  Strengths: Kubernetes, Infrastructure as Code      │
│                       │  Improve: Communication clarity, Adaptability       │
│                       │                                                     │
│                       │  [Download Report]  [Share]  [View Transcript]      │
│                       │                                                     │
└───────────────────────┴─────────────────────────────────────────────────────┘
```

### W7a: Assessment Lobby

The lobby screen where candidates prepare for their assessment. System checks and permissions are requested before joining.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│  🔷 Talent Mesh                                                             │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│                    Assessment Lobby                                         │
│                    ─────────────────                                        │
│                                                                             │
│     DevOps Technical Assessment                                             │
│     Scheduled: Today at 2:00 PM (starts in 5 minutes)                       │
│     Duration: 45 minutes                                                    │
│                                                                             │
│  ┌───────────────────────────────────────────────────────────────────────┐ │
│  │  System Check                                                         │ │
│  │  ─────────────────────────────────────────────────────────────────── │ │
│  │                                                                       │ │
│  │  ✓ Browser: Chrome 120 (supported)                                    │ │
│  │  ✓ WebRTC: Available                                                  │ │
│  │  ○ Camera: Not checked yet                                            │ │
│  │  ○ Microphone: Not checked yet                                        │ │
│  │  ○ Screen Share: Not checked yet                                      │ │
│  │                                                                       │ │
│  │  [Check Permissions]                                                  │ │
│  │                                                                       │ │
│  └───────────────────────────────────────────────────────────────────────┘ │
│                                                                             │
│  ┌───────────────────────────────────────────────────────────────────────┐ │
│  │  Device Preview                                                       │ │
│  │  ─────────────────────────────────────────────────────────────────── │ │
│  │                                                                       │ │
│  │  ┌─────────────────────────────────────────────────────────────────┐ │ │
│  │  │                                                                 │ │ │
│  │  │              Camera preview will appear here                    │ │ │
│  │  │              after permissions are granted                      │ │ │
│  │  │                                                                 │ │ │
│  │  └─────────────────────────────────────────────────────────────────┘ │ │
│  │                                                                       │ │
│  │  [Test Audio]  [Test Video]                                           │ │
│  │                                                                       │ │
│  └───────────────────────────────────────────────────────────────────────┘ │
│                                                                             │
│  [ ] I agree to the assessment terms and consent to recording              │
│                                                                             │
│  [Join Assessment]  ← Disabled until checks complete                        │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### W7b: Assessment Permissions

Browser permission dialogs flow with guidance for the candidate.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│  🔷 Talent Mesh                                                             │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│                    Grant Permissions                                        │
│                    ─────────────────                                        │
│                                                                             │
│  ┌───────────────────────────────────────────────────────────────────────┐ │
│  │                                                                       │ │
│  │  Step 1 of 3: Camera Access                                           │ │
│  │  ─────────────────────────────────────────────────────────────────── │ │
│  │                                                                       │ │
│  │  ┌───────────────────────────────────────────────────┐               │ │
│  │  │  ┌─────────────────────────────────────────────┐  │               │ │
│  │  │  │ talentmesh.io wants to use your camera     │  │               │ │
│  │  │  │                                             │  │               │ │
│  │  │  │  [Block]  [Allow]                           │  │               │ │
│  │  │  └─────────────────────────────────────────────┘  │               │ │
│  │  │                                                   │               │ │
│  │  │      Browser permission dialog appears here       │               │ │
│  │  │                                                   │               │ │
│  │  └───────────────────────────────────────────────────┘               │ │
│  │                                                                       │ │
│  │  Why we need this:                                                    │ │
│  │  • Video recording for proctoring and review                          │ │
│  │  • Ensures assessment integrity                                       │ │
│  │  • Your video is encrypted and stored securely                        │ │
│  │                                                                       │ │
│  └───────────────────────────────────────────────────────────────────────┘ │
│                                                                             │
│  Progress: ●○○                                                              │
│  Next: Microphone → Screen Share                                            │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### W7c: Assessment Session (WebRTC)

The full assessment interface with WebRTC P2P connection to AI Agent Pod.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│  DevOps Assessment                     ⏱ 32:15      Connection: P2P Direct │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌────────────────────────────────────┐ ┌─────────────────────────────────┐│
│  │                                    │ │  Current Question               ││
│  │                                    │ │  ───────────────────────────── ││
│  │        Candidate Video             │ │  "Explain how Kubernetes       ││
│  │        (Self-view)                 │ │   handles pod scheduling and   ││
│  │                                    │ │   what factors influence       ││
│  │                                    │ │   placement decisions?"        ││
│  │                                    │ │                                 ││
│  └────────────────────────────────────┘ │  Topic: Kubernetes & Orch.      ││
│                                         │  Progress: ████████░░ 45%       ││
│  ┌────────────────────────────────────┐ │  Question: 4 of 9               ││
│  │                                    │ └─────────────────────────────────┘│
│  │        AI Interviewer              │                                    │
│  │        (Avatar/Waveform)           │ ┌─────────────────────────────────┐│
│  │                                    │ │  Code Editor (when needed)      ││
│  │        Speaking...                 │ │  ───────────────────────────── ││
│  │                                    │ │  function solution() {          ││
│  └────────────────────────────────────┘ │    // Write your code here      ││
│                                         │  }                               ││
│                                         │                                  ││
│                                         │  [Run Code]  [Reset]             ││
│                                         └─────────────────────────────────┘│
│                                                                             │
│  ┌───────────────────────────────────────────────────────────────────────┐ │
│  │  🎤 Listening...                                                      │ │
│  │                                                                       │ │
│  │  ░░▒▓██▓▒░░░▒▓████▓▒░░░▒▓██▓▒░░░░▒▓██████▓▒░░░▒▓██▓▒░░             │ │
│  │                                                                       │ │
│  └───────────────────────────────────────────────────────────────────────┘ │
│                                                                             │
│  ┌──────────┐  ┌──────────┐  ┌────────────┐  ┌───────┐  ┌──────────────┐ │
│  │ 🔇 Mute  │  │ 📹 Video │  │ 🖥 Screen  │  │ ❓    │  │ 🚪 End       │ │
│  └──────────┘  └──────────┘  └────────────┘  └───────┘  └──────────────┘ │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

**WebRTC Connection States:**
- `Connecting...` - ICE candidate exchange in progress
- `P2P Direct` - Successfully connected peer-to-peer (~80%)
- `Via Relay` - Connected through TURN server (~20%)
- `Reconnecting...` - Connection dropped, attempting recovery

### W8: Recruiter - Candidate List

```
┌─────────────────────────────────────────────────────────────────────────────┐
│  🔷 Talent Mesh                                          🔔  👤 Sarah (HR) │
├───────────────────────┬─────────────────────────────────────────────────────┤
│                       │                                                     │
│  ▪ Dashboard          │  Candidates                           [+ Invite]   │
│  ○ Candidates         │  ─────────────────────────────────────────────────  │
│  ○ Assessments        │                                                     │
│  ○ Templates          │  ┌─────────────────────────────────────────────┐   │
│  ○ Analytics          │  │ 🔍 Search candidates...                     │   │
│  ─────────────────    │  └─────────────────────────────────────────────┘   │
│  ○ Settings           │                                                     │
│                       │  Filters:                                           │
│                       │  [All Types ▼] [Score ≥70 ▼] [This Week ▼] [All ▼] │
│                       │                                                     │
│                       │  ┌─────┬────────────────┬──────────┬───────┬──────┐│
│                       │  │ ☐   │ Candidate      │ Assess.  │ Score │Status││
│                       │  ├─────┼────────────────┼──────────┼───────┼──────┤│
│                       │  │ ☐   │ 👤 John Smith  │ DevOps   │ 85    │ ✓    ││
│                       │  │ ☐   │ 👤 Sarah Chen  │ DevOps   │ 78    │ ✓    ││
│                       │  │ ☐   │ 👤 Mike J.     │ Backend  │ 92    │ ✓    ││
│                       │  │ ☐   │ 👤 Lisa Park   │ DevOps   │ 65    │ ⏳   ││
│                       │  │ ☐   │ 👤 David B.    │ System   │ 71    │ ✓    ││
│                       │  │ ☐   │ 👤 Emma W.     │ Backend  │ 45    │ ✗    ││
│                       │  └─────┴────────────────┴──────────┴───────┴──────┘│
│                       │                                                     │
│                       │  Selected: 0                                        │
│                       │  [Compare] [Export] [Archive]                       │
│                       │                                                     │
│                       │  Showing 6 of 127               [< 1 2 3 ... 13 >] │
│                       │                                                     │
└───────────────────────┴─────────────────────────────────────────────────────┘
```

### W9: Candidate Detail View

```
┌─────────────────────────────────────────────────────────────────────────────┐
│  🔷 Talent Mesh                                          🔔  👤 Sarah (HR) │
├───────────────────────┬─────────────────────────────────────────────────────┤
│                       │  ← Back to Candidates                               │
│  ▪ Dashboard          │                                                     │
│  ○ Candidates         │  ┌─────────────────────────────────────────────────┐│
│  ○ Assessments        │  │  ┌────┐                                         ││
│  ○ Templates          │  │  │ JS │  John Smith                             ││
│  ○ Analytics          │  │  └────┘  Senior DevOps Engineer                 ││
│  ─────────────────    │  │          john.smith@email.com                   ││
│  ○ Settings           │  │          LinkedIn | GitHub                      ││
│                       │  │                                                 ││
│                       │  │  [Advance to Interview]  [Reject]  [⋮]          ││
│                       │  └─────────────────────────────────────────────────┘│
│                       │                                                     │
│                       │  ┌──────────┬──────────┬──────────┬──────────┐     │
│                       │  │ Overview │ Scores   │ Timeline │ Notes    │     │
│                       │  └──────────┴──────────┴──────────┴──────────┘     │
│                       │  ━━━━━━━━━━                                         │
│                       │                                                     │
│                       │  ┌────────────────────┐  ┌────────────────────────┐│
│                       │  │    Spider Map      │  │  Score Breakdown       ││
│                       │  │                    │  │  ────────────────────  ││
│                       │  │     Technical      │  │  Kubernetes:    88%    ││
│                       │  │         ∧          │  │  CI/CD:         82%    ││
│                       │  │        /|\         │  │  IaC:           85%    ││
│                       │  │       / | \        │  │  Problem Solve: 90%    ││
│                       │  │  Comm   |   Lead   │  │  Communication: 78%    ││
│                       │  │       \ | /        │  │  ────────────────────  ││
│                       │  │        \|/         │  │  Overall:       85%    ││
│                       │  │         ∨          │  │                        ││
│                       │  │      Problem       │  │  ✓ Recommended for     ││
│                       │  │                    │  │    next stage          ││
│                       │  └────────────────────┘  └────────────────────────┘│
│                       │                                                     │
└───────────────────────┴─────────────────────────────────────────────────────┘
```

### W10: Assessment Scheduling (Calendar)

```
┌─────────────────────────────────────────────────────────────────────────────┐
│  🔷 Talent Mesh                                          🔔  👤 John Smith │
├───────────────────────┬─────────────────────────────────────────────────────┤
│                       │  Schedule Assessment                                │
│  ▪ Dashboard          │  ─────────────────────────────────────────────────  │
│  ○ My Assessments     │                                                     │
│  ○ Profile            │  Assessment: DevOps Assessment (45 min)             │
│  ○ Schedule           │                                                     │
│  ○ Results            │  ┌─────────────────────────────────────────────────┐│
│  ─────────────────    │  │  ◀  January 2026  ▶                             ││
│  ○ Settings           │  │                                                 ││
│  ○ Help               │  │  Su   Mo   Tu   We   Th   Fr   Sa               ││
│                       │  │                    1    2    3    4              ││
│                       │  │   5    6    7    8    9   10   11              ││
│                       │  │  12   13  [14]  15   16   17   18              ││
│                       │  │  19   20   21   22   23   24   25              ││
│                       │  │  26   27   28   29   30   31                    ││
│                       │  │                                                 ││
│                       │  │  [14] = Selected                                ││
│                       │  │  Green = Available | Gray = Unavailable         ││
│                       │  └─────────────────────────────────────────────────┘│
│                       │                                                     │
│                       │  Available Times for January 14:                    │
│                       │  ┌──────────┐ ┌──────────┐ ┌──────────┐            │
│                       │  │ 9:00 AM  │ │ 10:00 AM │ │ 11:00 AM │            │
│                       │  └──────────┘ └──────────┘ └──────────┘            │
│                       │  ┌──────────┐ ┌──────────┐ ┌──────────┐            │
│                       │  │ 2:00 PM  │ │ 3:00 PM  │ │ 4:00 PM  │            │
│                       │  └──────────┘ └──────────┘ └──────────┘            │
│                       │                                                     │
│                       │  Or start now:  [Start On-Demand Assessment]        │
│                       │                                                     │
└───────────────────────┴─────────────────────────────────────────────────────┘
```

### W11: Admin - Template Editor

```
┌─────────────────────────────────────────────────────────────────────────────┐
│  🔷 Talent Mesh                                          🔔  👤 Admin      │
├───────────────────────┬─────────────────────────────────────────────────────┤
│                       │  Edit Template: DevOps Assessment                   │
│  ▪ Dashboard          │  ─────────────────────────────────────────────────  │
│  ○ Candidates         │                                                     │
│  ○ Assessments        │  ┌──────────┬──────────┬──────────┬──────────┐     │
│  ○ Templates          │  │ Basics   │ Topics   │ Scoring  │ Preview  │     │
│  ○ Analytics          │  └──────────┴──────────┴──────────┴──────────┘     │
│  ─────────────────    │              ━━━━━━━━━                              │
│  ○ Settings           │                                                     │
│                       │  Topics & Questions                    [+ Add Topic]│
│                       │  ─────────────────────────────────────────────────  │
│                       │                                                     │
│                       │  ┌─────────────────────────────────────────────────┐│
│                       │  │  ≡  Kubernetes & Orchestration        Weight 25%││
│                       │  │     ├─ Pod lifecycle management                 ││
│                       │  │     ├─ Service networking                       ││
│                       │  │     ├─ Security contexts                        ││
│                       │  │     └─ [+ Add subtopic]                         ││
│                       │  │                                        [Edit ⋮] ││
│                       │  ├─────────────────────────────────────────────────┤│
│                       │  │  ≡  CI/CD Pipelines                   Weight 20%││
│                       │  │     ├─ GitHub Actions                           ││
│                       │  │     ├─ Pipeline optimization                    ││
│                       │  │     └─ [+ Add subtopic]                         ││
│                       │  │                                        [Edit ⋮] ││
│                       │  ├─────────────────────────────────────────────────┤│
│                       │  │  ≡  Infrastructure as Code            Weight 20%││
│                       │  │     └─ ...                                      ││
│                       │  └─────────────────────────────────────────────────┘│
│                       │                                                     │
│                       │  [Cancel]                      [Save Draft] [Publish]│
│                       │                                                     │
└───────────────────────┴─────────────────────────────────────────────────────┘
```

### W12: Analytics Dashboard

```
┌─────────────────────────────────────────────────────────────────────────────┐
│  🔷 Talent Mesh                                          🔔  👤 Admin      │
├───────────────────────┬─────────────────────────────────────────────────────┤
│                       │  Analytics                   [Jan 1 - Jan 31 ▼]    │
│  ▪ Dashboard          │  ─────────────────────────────────────────────────  │
│  ○ Candidates         │                                                     │
│  ○ Assessments        │  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐   │
│  ○ Templates          │  │ Assessments │ │ Pass Rate   │ │ Avg Score   │   │
│  ○ Analytics          │  │ ─────────── │ │ ─────────── │ │ ─────────── │   │
│  ─────────────────    │  │    234      │ │    67%      │ │    72       │   │
│  ○ Settings           │  │   ↑ 12%     │ │   ↑ 5%      │ │   ↓ 3pts    │   │
│                       │  └─────────────┘ └─────────────┘ └─────────────┘   │
│                       │                                                     │
│                       │  ┌─────────────────────────────────────────────────┐│
│                       │  │  Assessments Over Time                          ││
│                       │  │                                                 ││
│                       │  │  50│     ╱╲                                     ││
│                       │  │    │    ╱  ╲    ╱╲                              ││
│                       │  │  25│   ╱    ╲  ╱  ╲                             ││
│                       │  │    │  ╱      ╲╱    ╲                            ││
│                       │  │   0└───┴───┴───┴───┴───┴───┴───                 ││
│                       │  │     W1  W2  W3  W4  W5  W6  W7                  ││
│                       │  └─────────────────────────────────────────────────┘│
│                       │                                                     │
│                       │  ┌──────────────────────┐ ┌────────────────────────┐│
│                       │  │ Score Distribution   │ │ By Assessment Type    ││
│                       │  │ ──────────────────── │ │ ────────────────────  ││
│                       │  │      ▄               │ │ DevOps     ████████ 45││
│                       │  │    ▄ █ ▄             │ │ Backend    ██████   32││
│                       │  │  ▄ █ █ █ ▄           │ │ System     ████     18││
│                       │  │  █ █ █ █ █           │ │ Frontend   ██        5││
│                       │  │ 0-20 40 60 80 100    │ │                       ││
│                       │  └──────────────────────┘ └────────────────────────┘│
│                       │                                                     │
└───────────────────────┴─────────────────────────────────────────────────────┘
```

---

## Mobile Wireframes

### M1: Mobile Dashboard

```
┌─────────────────────────┐
│  ☰  Talent Mesh     🔔  │
├─────────────────────────┤
│                         │
│  Hi, John!              │
│                         │
│  ┌─────────────────────┐│
│  │ 🔔 Upcoming         ││
│  │ ─────────────────── ││
│  │ DevOps Assessment   ││
│  │ Tomorrow, 10:00 AM  ││
│  │                     ││
│  │ [Join]  [Reschedule]││
│  └─────────────────────┘│
│                         │
│  Available Assessments  │
│  ─────────────────────  │
│                         │
│  ┌─────────────────────┐│
│  │ 📊 Backend Dev      ││
│  │ 45 min • 5 topics   ││
│  │           [Schedule]││
│  └─────────────────────┘│
│                         │
│  ┌─────────────────────┐│
│  │ 🔧 System Design    ││
│  │ 60 min • 4 topics   ││
│  │           [Schedule]││
│  └─────────────────────┘│
│                         │
├─────────────────────────┤
│ 🏠  📋  📊  👤          │
└─────────────────────────┘
```

### M2: Mobile Assessment Session (WebRTC)

```
┌─────────────────────────┐
│  DevOps   ⏱ 32:15  P2P │
├─────────────────────────┤
│                         │
│  ┌─────────────────────┐│
│  │                     ││
│  │   Your Camera       ││
│  │     [Preview]       ││
│  │                     ││
│  └─────────────────────┘│
│                         │
│  ┌─────────────────────┐│
│  │                     ││
│  │    AI Interviewer   ││
│  │   [Avatar/Waveform] ││
│  │                     ││
│  └─────────────────────┘│
│                         │
│  Kubernetes & Orch.     │
│  ████████░░░░░░░ 45%    │
│                         │
│  ┌─────────────────────┐│
│  │                     ││
│  │  🎤 Listening...    ││
│  │                     ││
│  │  ░▒▓██▓▒░░▒▓██▓▒░   ││
│  │                     ││
│  └─────────────────────┘│
│                         │
│  ┌───┐ ┌───┐ ┌───┐ ┌───┐│
│  │🔇 │ │📹 │ │🖥 │ │🚪 ││
│  └───┘ └───┘ └───┘ └───┘│
│                         │
└─────────────────────────┘
```

**Mobile Notes:**
- Camera preview smaller on mobile
- Swipe gesture to toggle code editor
- Connection status in header (P2P/Relay)

---

## Responsive Behavior

### Breakpoint Adaptations

| Component | Desktop (lg+) | Tablet (md) | Mobile (sm) |
|-----------|---------------|-------------|-------------|
| Navigation | Side + Top | Top only | Bottom + Hamburger |
| Tables | Full columns | Scrollable | Cards |
| Cards | 3-4 per row | 2 per row | 1 per row |
| Forms | Multi-column | Single column | Single column |
| Modals | Centered | Centered | Full screen |
| Calendar | Month view | Week view | Day list |

### Touch Targets

- Minimum touch target: 44x44px
- Spacing between targets: 8px minimum
- Form inputs: 48px height on mobile

---

## Accessibility

### Requirements

| Feature | Implementation |
|---------|----------------|
| Focus indicators | 2px blue outline |
| Color contrast | WCAG AA (4.5:1) |
| Screen reader | ARIA labels |
| Keyboard nav | All interactive elements |
| Alt text | All images |
| Form labels | Associated with inputs |

### Keyboard Shortcuts

| Key | Action |
|-----|--------|
| Tab | Navigate forward |
| Shift+Tab | Navigate backward |
| Enter | Activate button/link |
| Space | Toggle checkbox |
| Esc | Close modal |
| Arrow keys | Navigate menu/options |

---

---

## HTML Prototype Reference

All wireframes have corresponding interactive HTML/CSS prototypes located in the `/wireframes/` folder. These prototypes:

- Provide clickable navigation between screens
- Include responsive behavior for mobile testing
- Use the design system tokens defined above
- Are suitable for user testing and stakeholder review

See [USER_FLOWS.md](USER_FLOWS.md) for the complete user journey documentation.

---

*Document Version: 3.1*
*Last Updated: 2026-01-07*
*Owner: Design Team*
