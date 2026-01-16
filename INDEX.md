# TalentMesh Handbook Index

This index provides a complete listing of all documentation in the TalentMesh handbook.

---

## Business

| Document | Description |
|----------|-------------|
| [Business Case](./business/BUSINESS_CASE.md) | Value proposition, market analysis, financial model |
| [Product Vision](./business/PRODUCT_VISION.md) | Product strategy and roadmap |
| [Stakeholder Personas](./business/STAKEHOLDER_PERSONAS.md) | User personas and use cases |

---

## Requirements

| Document | Description |
|----------|-------------|
| [User Stories](./requirements/USER_STORIES.md) | Epics and user stories |

---

## Architecture

| Document | Description |
|----------|-------------|
| [System Overview](./architecture/SYSTEM_OVERVIEW.md) | High-level application architecture |

---

## Technical Specifications

| Document | Description |
|----------|-------------|
| [AI/ML Specifications](./technical/AI_ML_SPECIFICATIONS.md) | LLM integration, scoring algorithms |
| [LLM Integration](./technical/CLAUDE_WRAPPER_SPEC.md) | OpenOva LLM Gateway usage |
| [WebRTC Mesh Spec](./technical/WEBRTC_MESH_SPEC.md) | Real-time video/audio architecture |
| [Tech Stack](./technical/TECH_STACK.md) | Languages, frameworks, dependencies |

---

## API Specifications

| Document | Description |
|----------|-------------|
| [API Overview](./api-specs/API_OVERVIEW.md) | API design principles |
| [OpenAPI Specs](./api-specs/openapi/) | REST API definitions |
| [AsyncAPI Specs](./api-specs/asyncapi/) | Event-driven API definitions |

---

## Data Model

| Document | Description |
|----------|-------------|
| [Domain Model](./data-model/DOMAIN_MODEL.md) | Core domain concepts |
| [Entity Relationship](./data-model/ENTITY_RELATIONSHIP.md) | Database schema |

---

## Architecture Decision Records (ADRs)

| ADR | Title | Status |
|-----|-------|--------|
| [ADR-002](./adrs/ADR-002-polyglot-database-strategy.md) | Polyglot Database Strategy | Accepted |
| [ADR-005](./adrs/ADR-005-whisper-cpp-stt.md) | Whisper.cpp STT | Accepted |
| [ADR-006](./adrs/ADR-006-LINKEDIN-ONLY-AUTH.md) | LinkedIn Only Auth | Accepted |
| [ADR-007](./adrs/ADR-007-event-driven-architecture.md) | Event Driven Architecture | Accepted |
| [ADR-009](./adrs/ADR-009-CLAUDE-CLI-WRAPPER.md) | LLM Integration via OpenOva | Accepted |
| [ADR-010](./adrs/ADR-010-WEBRTC-DIRECT-ASSESSMENT.md) | WebRTC Direct Assessment | Accepted |
| [ADR-015](./adrs/ADR-015-piper-tts.md) | Piper TTS | Accepted |
| [ADR-033](./adrs/ADR-033-DATABASE-CICD-MIGRATIONS.md) | Database CI/CD Migrations | Accepted |

---

## Security

| Document | Description |
|----------|-------------|
| [Security Architecture](./security/SECURITY_ARCHITECTURE.md) | Authentication, authorization, data protection |

---

## Runbooks

| Document | Description |
|----------|-------------|
| [Operations Runbook](./runbooks/RUNBOOK-OPERATIONS.md) | Service-specific operational procedures |

---

## UI/UX

| Document | Description |
|----------|-------------|
| [User Flows](./ui-ux/) | Application user flows and wireframes |

---

## Platform Documentation

TalentMesh runs on the **OpenOva Platform**. For infrastructure, deployment, and platform services documentation, see:

| Resource | Description |
|----------|-------------|
| [OpenOva Index](https://github.com/openova-io/INDEX.md) | Complete documentation index |
| [OpenOva Handbook](https://github.com/openova-io/handbook) | Platform documentation |
| [Platform Tech Stack](https://github.com/openova-io/handbook/blob/main/docs/specs/SPEC-PLATFORM-TECH-STACK.md) | Technology stack |
| [LLM Gateway](https://github.com/openova-io/handbook/blob/main/docs/specs/SPEC-LLM-GATEWAY.md) | LLM service specification |
| [DNS Failover](https://github.com/openova-io/handbook/blob/main/docs/specs/SPEC-DNS-FAILOVER.md) | High availability |
| [Platform Runbook](https://github.com/openova-io/handbook/blob/main/docs/runbooks/RUNBOOK-PLATFORM.md) | Platform operations |

---

## Document Conventions

- **ADRs** follow the [ADR template](https://adr.github.io/)
- **API specs** use [OpenAPI 3.1](https://spec.openapis.org/oas/v3.1.0) and [AsyncAPI 2.6](https://www.asyncapi.com/docs/specifications/v2.6.0)
- **Diagrams** use [Mermaid](https://mermaid.js.org/)

---

*Last Updated: 2026-01-12*
*Owner: Product Team*
