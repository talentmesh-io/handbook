# ADR-002: Polyglot Database Strategy

## Status
**Accepted**

## Date
2026-01-01

## Context

Talent Mesh handles diverse data types with different requirements:
- User accounts and sessions require ACID guarantees
- Candidate profiles and assessments need flexible schemas
- Real-time agent state needs sub-millisecond access
- Event streams need durability and replay capability

We need to choose database technologies that match each use case according to CAP theorem trade-offs.

## Decision

We will adopt a **polyglot persistence** strategy with purpose-specific databases:

### PostgreSQL (CP - Consistency + Partition Tolerance)
- **Use cases**: Auth, sessions, bookings, audit logs
- **Rationale**: ACID compliance, referential integrity
- **Data**: Users, organizations, schedules

### MongoDB (AP - Availability + Partition Tolerance)
- **Use cases**: Profiles, assessments, templates, transcripts
- **Rationale**: Flexible schemas, document model fits domain
- **Data**: Candidate profiles, assessment results, templates

### Redis (CP with high availability)
- **Use cases**: Sessions, caching, agent state, rate limiting
- **Rationale**: Sub-millisecond latency, rich data structures
- **Data**: Session tokens, rate limits, agent pool state

### Apache Kafka
- **Use cases**: Event streaming, audit trail, service communication
- **Rationale**: Durable event log, replay capability, decoupling
- **Data**: Domain events, audit events, assessment events

### MinIO (S3-compatible)
- **Use cases**: Object storage for files
- **Rationale**: S3-compatible, self-hosted, no vendor lock-in
- **Data**: CVs, recordings, exports

## Consequences

### Positive
- **Optimized for use case**: Each database excels at its purpose
- **CAP theorem alignment**: Right consistency model per data type
- **Scalability**: Can scale databases independently
- **Performance**: No compromise on latency for any use case
- **Flexibility**: MongoDB handles evolving schemas well

### Negative
- **Operational complexity**: Five different data stores to manage
- **Cross-database queries**: No joins across databases
- **Eventual consistency**: Must handle async data propagation
- **Learning curve**: Team needs multiple database skills
- **Backup complexity**: Different backup strategies per database

### Mitigations
- Use managed database services in production (RDS, Atlas, ElastiCache)
- Implement clear data ownership per service
- Use event sourcing for cross-service data needs
- Comprehensive documentation for each database
- Unified monitoring with Prometheus/Grafana

## CAP Theorem Analysis

```
                    Consistency
                         /\
                        /  \
                       /    \
                      /  CP  \
                     / PostgreSQL
                    /   Redis   \
                   /------------\
                  /              \
                 /       CA       \
                /   (Impossible    \
               /   in distributed  \
              /      systems)       \
             /----------------------\
            /                        \
           /           AP             \
          /         MongoDB            \
         /____________________________\
        Availability        Partition Tolerance
```

## Data Ownership

| Service | Primary DB | Data |
|---------|------------|------|
| Auth | PostgreSQL | users, sessions, refresh_tokens |
| User | MongoDB | profiles, education, experience |
| Assessment | MongoDB | assessments, transcripts, scores |
| Scheduling | PostgreSQL | bookings, availability |
| Agent | Redis | pool state, assignments |
| All | Kafka | events |

## Alternatives Considered

### Single Database (PostgreSQL only)
- **Pros**: Simpler operations, strong consistency everywhere
- **Cons**: JSON queries slower, no sub-ms caching, no event replay
- **Rejected**: Cannot meet latency requirements for agent state

### Single Database (MongoDB only)
- **Pros**: Flexible everywhere, good JSON support
- **Cons**: Weaker transactions, no ACID for critical data
- **Rejected**: Cannot guarantee consistency for auth/payments

### CockroachDB (Distributed SQL)
- **Pros**: Distributed ACID, PostgreSQL compatible
- **Cons**: Higher latency than Redis, overkill for sessions
- **Rejected**: Still need Redis for sub-ms access

## References

- [DATA_ARCHITECTURE.md](../03-architecture/DATA_ARCHITECTURE.md)
- [ENTITY_RELATIONSHIP.md](../05-data-model/ENTITY_RELATIONSHIP.md)
- [DOCUMENT_SCHEMAS.md](../05-data-model/DOCUMENT_SCHEMAS.md)
