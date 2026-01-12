# Talent Mesh MongoDB Document Schemas

## Overview

This document defines the MongoDB collection schemas for Talent Mesh. MongoDB is used for flexible document storage where schema variations are expected.

---

## Database Structure

```
talent_mesh (database)
├── profiles              # Candidate profiles (with verified fields)
├── assessments           # Assessment instances
├── templates             # Assessment templates
├── questions             # Question bank
├── jobs                  # Job definitions (for matching)
├── verification_documents # User verification documents
├── agent_pod_metrics     # AI Agent Pod performance metrics
└── cost_of_living        # Cost of living data by location
```

---

## Collection: `profiles`

Stores comprehensive candidate profile data aggregated from multiple sources.

### Schema

```javascript
{
  // Identifiers
  _id: ObjectId,
  user_id: UUID,              // Reference to PostgreSQL auth.users
  linkedin_id: String,        // Primary identifier (synced from auth.users)
  organization_id: UUID,      // For recruiter-created profiles

  // Personal Information
  personal_info: {
    name: String,
    headline: String,
    location: String,
    phone: String,
    email: String,            // May differ from auth email
    photo_url: String
  },

  // Verification Status (synced from PostgreSQL)
  verification: {
    status: String,           // pending, in_review, verified, rejected, expired
    verified_at: ISODate,
    verified_documents: [{
      type: String,           // government_id, linkedin_profile, employment, etc.
      status: String,
      verified_at: ISODate
    }],
    identity_verified: Boolean,
    employment_verified: Boolean,
    education_verified: Boolean
  },

  // LinkedIn Data
  linkedin: {
    id: String,
    profile_url: String,

    experience: [{
      title: String,
      company: String,
      company_linkedin_id: String,
      location: String,
      start_date: ISODate,
      end_date: ISODate,      // null if current
      is_current: Boolean,
      description: String
    }],

    education: [{
      school: String,
      degree: String,
      field_of_study: String,
      start_year: Number,
      end_year: Number,
      activities: String
    }],

    skills: [String],
    certifications: [{
      name: String,
      authority: String,
      license_number: String,
      url: String
    }],

    imported_at: ISODate,
    raw_data: Object          // Original API response
  },

  // CV Data
  cv: {
    file_path: String,        // MinIO S3 path
    file_name: String,
    file_type: String,        // pdf, docx, etc.
    file_size: Number,

    raw_text: String,         // Extracted text

    parsed: {
      name: String,
      email: String,
      phone: String,
      summary: String,

      experience: [{
        title: String,
        company: String,
        duration: String,
        start_date: String,
        end_date: String,
        description: String,
        achievements: [String]
      }],

      education: [{
        degree: String,
        school: String,
        year: String,
        gpa: String
      }],

      skills: [String],
      certifications: [String],
      languages: [String],
      projects: [{
        name: String,
        description: String,
        technologies: [String]
      }]
    },

    ats_format: {
      // Standardized ATS-compatible format
      // Structure varies by ATS target
    },

    analysis: {
      strengths: [String],
      gaps: [String],
      suggested_assessments: [String],
      years_of_experience: Number,
      seniority_level: String,  // junior, mid, senior, lead
      career_trajectory: String,
      red_flags: [String]
    },

    uploaded_at: ISODate,
    analyzed_at: ISODate
  },

  // Merged/Unified Profile
  merged: {
    name: String,
    headline: String,
    summary: String,
    primary_role: String,
    years_of_experience: Number,
    seniority_level: String,

    skills: [{
      name: String,
      source: String,           // linkedin, cv, manual, assessment
      confidence: Number,       // 0-1 for extracted skills
      verified: Boolean,
      verified_score: Number,   // 0-100
      verified_at: ISODate,
      assessment_id: UUID
    }],

    experience: [{
      // Merged from LinkedIn and CV
      title: String,
      company: String,
      start_date: ISODate,
      end_date: ISODate,
      source: String
    }]
  },

  // Spider Map (Assessment Results)
  spider_map: {
    technical_devops: {
      score: Number,            // 0-100
      assessed_at: ISODate,
      assessment_id: UUID,
      sub_dimensions: {
        kubernetes: Number,
        docker: Number,
        ci_cd: Number,
        terraform: Number,
        monitoring: Number,
        cloud_platforms: Number
      }
    },
    technical_backend: {
      score: Number,
      assessed_at: ISODate,
      assessment_id: UUID,
      sub_dimensions: {
        languages: Number,
        apis: Number,
        databases: Number,
        architecture: Number
      }
    },
    system_design: {
      score: Number,
      assessed_at: ISODate,
      assessment_id: UUID,
      sub_dimensions: {
        scalability: Number,
        trade_offs: Number,
        communication: Number
      }
    },
    soft_skills: {
      score: Number,
      assessed_at: ISODate,
      sub_dimensions: {
        communication: Number,
        teamwork: Number,
        problem_solving: Number,
        leadership: Number
      }
    },
    cognitive: {
      score: Number,
      assessed_at: ISODate,
      sub_dimensions: {
        logical_reasoning: Number,
        learning_ability: Number,
        adaptability: Number
      }
    },
    language: {
      score: Number,
      assessed_at: ISODate,
      sub_dimensions: {
        fluency: Number,
        vocabulary: Number,
        clarity: Number
      }
    }
  },

  // Job Matches
  job_matches: [{
    job_id: UUID,
    match_score: Number,        // 0-1
    missing_dimensions: [String],
    strong_dimensions: [String],
    calculated_at: ISODate
  }],

  // Discrepancies (CV vs LinkedIn vs Assessed)
  discrepancies: [{
    field: String,
    linkedin_value: Mixed,
    cv_value: Mixed,
    assessed_value: Mixed,
    flagged_at: ISODate,
    resolved: Boolean
  }],

  // Metadata
  completeness: Number,         // 0-100
  status: String,               // active, archived, deleted
  tags: [String],               // recruiter-added tags
  notes: [{
    author_id: UUID,
    content: String,
    created_at: ISODate
  }],

  created_at: ISODate,
  updated_at: ISODate
}
```

### Indexes

```javascript
// Primary lookups
db.profiles.createIndex({ user_id: 1 }, { unique: true });
db.profiles.createIndex({ linkedin_id: 1 }, { unique: true });
db.profiles.createIndex({ organization_id: 1 });

// LinkedIn lookup
db.profiles.createIndex({ "linkedin.id": 1 }, { sparse: true });

// Verification indexes
db.profiles.createIndex({ "verification.status": 1 });
db.profiles.createIndex({ "verification.identity_verified": 1 });

// Search indexes
db.profiles.createIndex({ "merged.skills.name": 1 });
db.profiles.createIndex({ "merged.primary_role": 1 });
db.profiles.createIndex({ "spider_map.technical_devops.score": -1 });
db.profiles.createIndex({ "spider_map.technical_backend.score": -1 });

// Job matching
db.profiles.createIndex({ "job_matches.job_id": 1 });
db.profiles.createIndex({ "job_matches.match_score": -1 });

// Full-text search
db.profiles.createIndex({
  "merged.name": "text",
  "merged.headline": "text",
  "merged.skills.name": "text"
});
```

---

## Collection: `assessments`

Stores assessment instances with full conversation transcripts.

### Schema

```javascript
{
  _id: ObjectId,
  assessment_id: UUID,          // Public ID

  // References
  user_id: UUID,
  organization_id: UUID,
  booking_id: UUID,
  config_id: UUID,
  template_id: UUID,

  // Configuration
  type: String,                 // devops, backend, soft_skills, etc.
  duration_minutes: Number,
  pass_threshold: Number,

  // Status
  status: String,               // scheduled, in_progress, completed, abandoned, no_show
  started_at: ISODate,
  completed_at: ISODate,
  agent_id: String,
  meeting_url: String,

  // Transcript
  transcript: [{
    role: String,               // ai, candidate
    content: String,
    timestamp: ISODate,
    audio_segment: {
      start_ms: Number,
      end_ms: Number,
      file_path: String
    },
    sentiment: {
      confidence: Number,       // 0-1
      stress: Number,           // 0-1
      enthusiasm: Number,       // 0-1
      markers: [String]         // hedging_words, confident, uncertain
    }
  }],

  // Questions Flow
  questions: [{
    question_id: UUID,
    topic: String,
    difficulty: String,
    question: String,
    asked_at: ISODate,
    response: String,
    response_duration_ms: Number,

    follow_ups: [{
      question: String,
      response: String,
      asked_at: ISODate
    }],

    scores: {
      accuracy: Number,         // 0-100
      depth: Number,
      communication: Number,
      problem_solving: Number
    },

    evaluator_notes: String     // LLM evaluation reasoning
  }],

  // Recording
  recording: {
    available: Boolean,
    video_path: String,
    audio_path: String,
    duration_seconds: Number,
    size_bytes: Number,
    retention_until: ISODate
  },

  // Scores (populated by scoring-service)
  scores: {
    overall: Number,            // 0-100
    passed: Boolean,

    dimensions: {
      technical_accuracy: {
        score: Number,
        weight: Number,
        reasoning: String
      },
      problem_solving: {
        score: Number,
        weight: Number,
        reasoning: String
      },
      communication: {
        score: Number,
        weight: Number,
        reasoning: String
      },
      depth: {
        score: Number,
        weight: Number,
        reasoning: String
      },
      adaptability: {
        score: Number,
        weight: Number,
        reasoning: String
      }
    },

    per_topic: {
      kubernetes: Number,
      docker: Number,
      ci_cd: Number
      // ... varies by assessment type
    },

    justification: String,      // Overall evaluation summary
    strengths: [String],
    improvement_areas: [String]
  },

  // Sentiment Summary
  sentiment_summary: {
    overall_confidence: Number,
    overall_stress: Number,
    overall_enthusiasm: Number,
    authenticity_score: Number,

    key_moments: [{
      timestamp: ISODate,
      type: String,             // high_confidence, hesitation, strong_answer
      description: String,
      question_index: Number
    }],

    patterns: {
      confidence_trend: String, // increasing, decreasing, stable
      stress_triggers: [String],
      strongest_topics: [String],
      weakest_topics: [String]
    }
  },

  // Brain Dump Detection (anti-cheating analysis)
  brain_dump_detection: {
    brain_dump_score: Number,    // 0-1, higher = more suspicious
    flagged: Boolean,
    suspicious_patterns: [String],  // e.g., "rapid_fire_answers", "keyword_stuffing"
    response_time_variance: Number,
    coherence_score: Number,     // How well answers flow logically
    analysis_notes: String,
    analyzed_at: ISODate
  },

  // Technical Metadata
  technical_info: {
    stt_model: String,
    tts_voice: String,
    llm_model: String,
    total_latency_ms: Number,
    avg_response_latency_ms: Number
  },

  // Metadata
  created_at: ISODate,
  scored_at: ISODate
}
```

### Indexes

```javascript
db.assessments.createIndex({ assessment_id: 1 }, { unique: true });
db.assessments.createIndex({ user_id: 1, status: 1 });
db.assessments.createIndex({ organization_id: 1, completed_at: -1 });
db.assessments.createIndex({ booking_id: 1 });
db.assessments.createIndex({ status: 1 });
db.assessments.createIndex({ type: 1, "scores.passed": 1 });
db.assessments.createIndex({ "scores.overall": -1 });
db.assessments.createIndex({ created_at: -1 });
```

---

## Collection: `templates`

Stores reusable assessment templates.

### Schema

```javascript
{
  _id: ObjectId,
  template_id: UUID,
  organization_id: UUID,        // null for system templates

  name: String,
  type: String,                 // devops, backend, soft_skills, etc.
  description: String,
  is_default: Boolean,
  is_system: Boolean,           // System-provided templates

  // Structure
  sections: [{
    name: String,
    duration_minutes: Number,
    order: Number,

    questions: [{
      question_id: UUID,
      topic: String,
      question: String,
      difficulty: String,       // easy, medium, hard
      expected_answer: String,
      evaluation_criteria: String,

      follow_up_triggers: [{
        condition: String,      // incomplete, incorrect, interesting
        follow_up: String
      }],

      scoring_criteria: {
        accuracy_weight: Number,
        depth_weight: Number,
        communication_weight: Number
      },

      tags: [String],
      order: Number
    }]
  }],

  // Scoring Configuration
  scoring: {
    dimensions: [{
      name: String,
      weight: Number,
      description: String
    }],
    pass_threshold: Number,
    score_ranges: {
      excellent: { min: 90, max: 100 },
      good: { min: 70, max: 89 },
      acceptable: { min: 50, max: 69 },
      needs_improvement: { min: 0, max: 49 }
    }
  },

  // Customization Options
  options: {
    duration_flexible: Boolean,
    skip_sections_allowed: Boolean,
    adaptive_difficulty: Boolean,
    show_timer_to_candidate: Boolean
  },

  // Metadata
  version: Number,
  cloned_from: UUID,
  created_at: ISODate,
  updated_at: ISODate,
  created_by: UUID,
  usage_count: Number
}
```

---

## Collection: `questions`

Question bank for building templates.

### Schema

```javascript
{
  _id: ObjectId,
  question_id: UUID,
  organization_id: UUID,        // null for system questions

  topic: String,
  subtopic: String,
  question: String,
  difficulty: String,
  expected_answer: String,
  evaluation_criteria: String,

  // Alternative phrasings
  variations: [String],

  // Follow-ups
  follow_ups: [{
    trigger: String,
    question: String
  }],

  // Usage statistics
  usage_count: Number,
  avg_score: Number,
  discrimination_index: Number, // How well it differentiates candidates

  tags: [String],
  is_system: Boolean,
  created_at: ISODate,
  updated_at: ISODate
}
```

---

## Collection: `jobs`

Job definitions for candidate matching.

### Schema

```javascript
{
  _id: ObjectId,
  job_id: UUID,
  organization_id: UUID,

  title: String,
  description: String,
  department: String,
  location: String,
  remote_policy: String,        // remote, hybrid, onsite

  // Requirements
  requirements: {
    experience_years: {
      min: Number,
      max: Number
    },
    education: String,

    required_skills: [{
      name: String,
      min_score: Number         // Minimum spider map score
    }],

    preferred_skills: [{
      name: String,
      weight: Number
    }],

    assessment_requirements: [{
      type: String,
      min_score: Number
    }]
  },

  // Matching weights
  matching_weights: {
    technical_devops: Number,
    technical_backend: Number,
    system_design: Number,
    soft_skills: Number,
    cognitive: Number,
    language: Number
  },

  // Status
  status: String,               // open, closed, draft
  created_at: ISODate,
  updated_at: ISODate,
  created_by: UUID
}
```

---

## Collection: `verification_documents`

Stores user verification documents and their status (mirrors PostgreSQL for document queries).

### Schema

```javascript
{
  _id: ObjectId,
  document_id: UUID,          // References PostgreSQL auth.verification_documents
  user_id: UUID,              // References auth.users
  linkedin_id: String,        // User's LinkedIn ID for quick lookup

  type: String,               // government_id, linkedin_profile, employment_verification,
                              // education_verification, certification
  status: String,             // pending, in_review, verified, rejected, expired

  document: {
    url: String,              // MinIO S3 path
    file_name: String,
    file_type: String,        // pdf, jpg, png
    file_size: Number,
    uploaded_at: ISODate
  },

  verification: {
    verified_at: ISODate,
    verified_by: UUID,        // Admin/system user ID
    verifier_notes: String,
    rejection_reason: String,
    expires_at: ISODate       // For time-limited verifications
  },

  // Extracted data (varies by document type)
  extracted_data: {
    // For government_id
    full_name: String,
    date_of_birth: ISODate,
    document_number: String,
    issuing_country: String,
    expiry_date: ISODate,

    // For employment_verification
    employer_name: String,
    job_title: String,
    employment_dates: {
      start: ISODate,
      end: ISODate
    },

    // For education_verification
    institution_name: String,
    degree: String,
    graduation_year: Number,

    // For certification
    certification_name: String,
    issuing_authority: String,
    credential_id: String,
    issue_date: ISODate,
    expiry_date: ISODate
  },

  created_at: ISODate,
  updated_at: ISODate
}
```

### Indexes

```javascript
db.verification_documents.createIndex({ document_id: 1 }, { unique: true });
db.verification_documents.createIndex({ user_id: 1 });
db.verification_documents.createIndex({ linkedin_id: 1 });
db.verification_documents.createIndex({ type: 1, status: 1 });
db.verification_documents.createIndex({ status: 1 });
db.verification_documents.createIndex({ "verification.expires_at": 1 },
  { expireAfterSeconds: 0, partialFilterExpression: { status: "verified" } });
```

---

## Collection: `agent_pod_metrics`

Stores detailed AI Agent Pod performance metrics and session history.

### Schema

```javascript
{
  _id: ObjectId,
  pod_id: UUID,               // References PostgreSQL agents.pods
  k8s_pod_name: String,       // Kubernetes pod name

  // Current metrics (synced from PostgreSQL)
  current_metrics: {
    total_sessions: Number,
    successful_sessions: Number,
    failed_sessions: Number,
    success_rate: Number,        // 0-1
    avg_latency_ms: Number,
    last_updated: ISODate
  },

  // Performance metrics
  performance: {
    avg_stt_latency_ms: Number,
    avg_tts_latency_ms: Number,
    avg_llm_latency_ms: Number,
    avg_total_response_latency_ms: Number,
    p95_response_latency_ms: Number,
    p99_response_latency_ms: Number
  },

  // Resource usage
  resource_usage: {
    cpu_percent: Number,
    memory_mb: Number,
    gpu_memory_mb: Number,       // For GPU-accelerated pods
    network_bytes_in: Number,
    network_bytes_out: Number,
    last_sampled: ISODate
  },

  // Historical performance
  performance_history: [{
    period_start: ISODate,
    period_end: ISODate,
    period_type: String,      // hourly, daily
    sessions_completed: Number,
    sessions_failed: Number,
    avg_latency_ms: Number,
    resource_avg: {
      cpu_percent: Number,
      memory_mb: Number
    }
  }],

  // Recent session details
  recent_sessions: [{
    session_id: UUID,
    assessment_id: UUID,
    status: String,           // completed, failed, error
    started_at: ISODate,
    ended_at: ISODate,
    duration_seconds: Number,
    error_details: String,
    metrics: {
      avg_stt_latency_ms: Number,
      avg_tts_latency_ms: Number,
      avg_llm_latency_ms: Number,
      total_turns: Number
    }
  }],

  // Health indicators
  health: {
    status: String,           // healthy, degraded, unhealthy
    last_heartbeat: ISODate,
    consecutive_failures: Number,
    error_rate_1h: Number,    // Error rate last hour
    error_rate_24h: Number    // Error rate last 24 hours
  },

  // Alerts and issues
  alerts: [{
    type: String,             // high_latency, frequent_errors, resource_exhaustion
    severity: String,         // warning, critical
    message: String,
    created_at: ISODate,
    resolved_at: ISODate
  }],

  created_at: ISODate,
  updated_at: ISODate
}
```

### Indexes

```javascript
db.agent_pod_metrics.createIndex({ pod_id: 1 }, { unique: true });
db.agent_pod_metrics.createIndex({ k8s_pod_name: 1 });
db.agent_pod_metrics.createIndex({ "current_metrics.success_rate": -1 });
db.agent_pod_metrics.createIndex({ "health.status": 1 });
db.agent_pod_metrics.createIndex({ "alerts.resolved_at": 1 },
  { partialFilterExpression: { "alerts.resolved_at": null } });
```

---

## Collection: `cost_of_living`

Stores cost of living data by location for salary adjustments in job matching.

### Schema

```javascript
{
  _id: ObjectId,

  // Location identification
  location: {
    country: String,
    country_code: String,     // ISO 3166-1 alpha-2
    region: String,           // State/province
    city: String,
    metro_area: String        // For metro-level granularity
  },

  // Cost of living index
  col_index: {
    overall: Number,          // Base index (100 = global average)
    housing: Number,
    food: Number,
    transportation: Number,
    healthcare: Number,
    utilities: Number,
    childcare: Number,
    entertainment: Number
  },

  // Salary data
  salary_data: {
    median_tech_salary: Number,
    currency_code: String,    // ISO 4217
    exchange_rate_to_usd: Number,
    last_salary_update: ISODate
  },

  // Adjustment factors
  adjustment_factors: {
    remote_work_factor: Number,  // Adjustment for remote work (e.g., 0.85 for 15% reduction)
    tax_burden_factor: Number,   // Effective tax impact
    purchasing_power_parity: Number
  },

  // Comparison helpers
  comparison: {
    vs_sf_bay_area: Number,    // Ratio compared to SF Bay Area (1.0 = same)
    vs_nyc: Number,            // Ratio compared to NYC
    vs_london: Number,         // Ratio compared to London
    tier: String               // tier_1, tier_2, tier_3 (cost categories)
  },

  // Metadata
  data_source: String,        // numbeo, expatistan, internal, etc.
  confidence_score: Number,   // 0-1, data reliability
  valid_from: ISODate,
  valid_until: ISODate,
  created_at: ISODate,
  updated_at: ISODate
}
```

### Indexes

```javascript
db.cost_of_living.createIndex({ "location.country_code": 1, "location.city": 1 });
db.cost_of_living.createIndex({ "location.metro_area": 1 });
db.cost_of_living.createIndex({ "col_index.overall": 1 });
db.cost_of_living.createIndex({ "comparison.tier": 1 });
db.cost_of_living.createIndex({ valid_until: 1 });

// Text search for location lookup
db.cost_of_living.createIndex({
  "location.city": "text",
  "location.region": "text",
  "location.metro_area": "text"
});
```

---

## Data Migration Helpers

### Profile Creation from LinkedIn

```javascript
async function createProfileFromLinkedIn(userId, linkedInData) {
  return await db.profiles.insertOne({
    user_id: userId,
    linkedin_id: linkedInData.id,  // Primary identifier
    linkedin: {
      id: linkedInData.id,
      profile_url: linkedInData.profileUrl,
      experience: linkedInData.positions,
      education: linkedInData.education,
      skills: linkedInData.skills,
      imported_at: new Date()
    },
    verification: {
      status: 'pending',
      verified_at: null,
      verified_documents: [],
      identity_verified: false,
      employment_verified: false,
      education_verified: false
    },
    merged: {
      name: linkedInData.name,
      headline: linkedInData.headline,
      skills: linkedInData.skills.map(s => ({
        name: s,
        source: 'linkedin',
        confidence: 1.0,
        verified: false
      }))
    },
    spider_map: {},
    completeness: 40,
    created_at: new Date(),
    updated_at: new Date()
  });
}
```

### Cost of Living Lookup

```javascript
async function getCostOfLivingFactor(candidateLocation, jobLocation) {
  const [candidateCOL, jobCOL] = await Promise.all([
    db.cost_of_living.findOne({
      "location.city": candidateLocation.city,
      "location.country_code": candidateLocation.countryCode
    }),
    db.cost_of_living.findOne({
      "location.city": jobLocation.city,
      "location.country_code": jobLocation.countryCode
    })
  ]);

  if (!candidateCOL || !jobCOL) {
    return { factor: 1.0, confidence: 0 };
  }

  const factor = candidateCOL.col_index.overall / jobCOL.col_index.overall;
  const avgConfidence = (candidateCOL.confidence_score + jobCOL.confidence_score) / 2;

  return {
    factor: factor,
    confidence: avgConfidence,
    candidateIndex: candidateCOL.col_index.overall,
    jobIndex: jobCOL.col_index.overall,
    currencyAdjustment: candidateCOL.salary_data.exchange_rate_to_usd /
                        jobCOL.salary_data.exchange_rate_to_usd
  };
}
```

---

*Document Version: 3.0*
*Last Updated: 2026-01-04*
*Owner: Architecture Team*
