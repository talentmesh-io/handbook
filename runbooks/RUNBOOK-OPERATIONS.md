# Talent Mesh Operations Runbook

## Overview

Operational procedures for managing Talent Mesh in production.

> **Infrastructure:** Runs on OpenOva platform (K3s cluster on Contabo VPS)

---

## Automated Operations

Most incidents are handled automatically via GitHub Actions. See [OpenOva Platform Runbook](https://github.com/openova-io/handbook/docs/runbooks/RUNBOOK-PLATFORM.md) for platform-level operations.

### Check Automation Status

```bash
# Recent auto-remediation
gh run list --repo talentmesh-io/platform --workflow auto-remediation.yaml --limit 10
```

---

## Service-Specific Procedures

### Auth Service

```bash
# Health check
curl -s http://auth-service.talentmesh-prod:5001/health | jq

# View logs
kubectl logs -l app=auth-service -n talentmesh-prod --tail=100

# JWT verification
kubectl exec -it deploy/auth-service -n talentmesh-prod -- \
  node -e "console.log(process.env.JWT_SECRET ? 'Set' : 'Missing')"
```

#### Session Issues

```bash
# Check Redis sessions
kubectl exec -it dragonfly-0 -n databases -- redis-cli keys "session:*" | wc -l

# Clear specific session
kubectl exec -it dragonfly-0 -n databases -- redis-cli del "session:$SESSION_ID"
```

---

### Assessment Service

```bash
# Health check
curl -s http://assessment-service.talentmesh-prod:5003/health | jq

# Active assessments
kubectl exec -it deploy/assessment-service -n talentmesh-prod -- \
  python -c "from app.db import db; print(db.assessments.count_documents({'status': 'in_progress'}))"
```

#### Stuck Assessments

```bash
# Find stuck assessments
kubectl exec -it deploy/assessment-service -n talentmesh-prod -- python -c "
from app.db import db
from datetime import datetime, timedelta
stuck = db.assessments.find({
    'status': 'in_progress',
    'updated_at': {'\$lt': datetime.utcnow() - timedelta(hours=2)}
})
for a in stuck:
    print(f\"{a['_id']}: {a['user_id']} - started {a['created_at']}\")
"

# Force complete
curl -X POST http://assessment-service.talentmesh-prod:5003/internal/assessments/$ID/force-complete \
  -H "X-Internal-Token: $INTERNAL_TOKEN"
```

---

### LLM Gateway Usage

Talent Mesh consumes the OpenOva LLM Gateway:

```bash
# Check LLM Gateway status
curl -s http://llm-gateway.platform-services:5005/health | jq

# Session pool status
curl -s http://llm-gateway.platform-services:5005/sessions | jq
```

---

### STT Service (whisper-rs)

```bash
# Health check
curl -s http://stt-service.talentmesh-prod:5006/health | jq

# Test transcription
curl -X POST http://stt-service.talentmesh-prod:5006/transcribe \
  -F "audio=@test.wav" | jq

# Check model loaded
kubectl logs -l app=stt-service -n talentmesh-prod | grep "model loaded"
```

---

### TTS Service (piper-rs)

```bash
# Health check
curl -s http://tts-service.talentmesh-prod:5007/health | jq

# Test synthesis
curl -X POST http://tts-service.talentmesh-prod:5007/synthesize \
  -d '{"text": "Hello, this is a test"}' \
  -o test.wav

# List voices
curl -s http://tts-service.talentmesh-prod:5007/voices | jq
```

---

### Signaling Service (WebRTC)

```bash
# Health check
curl -s http://signaling-service.talentmesh-prod:5009/health | jq

# Active sessions
curl -s http://signaling-service.talentmesh-prod:5009/sessions | jq

# Check STUNner
kubectl get pods -l app=stunner -n stunner-system
```

---

## Database Operations

### PostgreSQL (Auth Data)

```bash
# Test connection
kubectl exec -it talentmesh-postgres-1 -n databases -- \
  psql -U talentmesh -d auth -c "SELECT 1"

# Check connections
kubectl exec -it talentmesh-postgres-1 -n databases -- \
  psql -U talentmesh -d auth -c "SELECT count(*) FROM pg_stat_activity"
```

### MongoDB (Assessment Data)

```bash
# Check replica set
kubectl exec -it mongodb-0 -n databases -- mongosh --eval "rs.status()"

# Collection stats
kubectl exec -it mongodb-0 -n databases -- \
  mongosh talentmesh --eval "db.assessments.stats()"
```

---

## Common Issues

### High Error Rate

```bash
# Check errors
kubectl logs -l app=assessment-service -n talentmesh-prod --since=30m | grep ERROR
```

### High Latency

```bash
# LLM Gateway latency
curl -s http://llm-gateway.platform-services:5005/metrics | grep llm_request_duration

# STT/TTS latency
curl -s http://stt-service.talentmesh-prod:5006/metrics | grep stt_
curl -s http://tts-service.talentmesh-prod:5007/metrics | grep tts_
```

### AI Agent Pod Issues

```bash
# Pod pool status
curl -s http://agent-service.talentmesh-prod:5008/pods | jq

# Force release stuck pod
curl -X POST http://agent-service.talentmesh-prod:5008/pods/$POD_ID/release

# Scale AI Agent Pods
kubectl scale deployment/ai-agent-pod --replicas=5 -n talentmesh-prod
```

---

## Logging

### Change Log Level

```bash
# Auth Service (Node.js)
kubectl set env deployment/auth-service LOG_LEVEL=debug -n talentmesh-prod

# STT Service (Rust)
kubectl set env deployment/stt-service RUST_LOG=stt_service=debug -n talentmesh-prod

# Revert after debugging
kubectl set env deployment/auth-service LOG_LEVEL=info -n talentmesh-prod
```

### Loki Queries

```promql
# All errors
{namespace="talentmesh-prod"} |= "error" | json | level="error"

# Slow requests
{namespace="talentmesh-prod"} | json | response_time > 1000

# Correlation trace
{namespace="talentmesh-prod"} |= "correlation_id=abc123"
```

---

## Related

- [OpenOva Platform Runbook](https://github.com/openova-io/handbook/docs/runbooks/RUNBOOK-PLATFORM.md)
- [Talent Mesh Architecture](../architecture/SYSTEM_OVERVIEW.md)

---

*Document Version: 1.0*
*Owner: Talent Mesh Team*
