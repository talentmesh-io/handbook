# LLM Integration Specification

## Overview

TalentMesh uses the **OpenOva LLM Gateway** platform service for all LLM capabilities. This document describes how TalentMesh services integrate with the LLM Gateway.

> **Note:** The LLM Gateway implementation is managed by OpenOva. For full technical details, see [OpenOva LLM Gateway Service](https://github.com/openova-io/handbook/blob/main/services/LLM_GATEWAY.md).

---

## Integration Points

### Service Endpoint

```
http://llm-gateway.platform-services.svc.cluster.local:5005
```

### Endpoints Used by TalentMesh

| Endpoint | Purpose |
|----------|---------|
| `POST /v1/chat/completions` | Main chat completions API (OpenAI-compatible) |
| `GET /health` | Health checks for readiness probes |

---

## Usage in TalentMesh Services

### Assessment Service

The Assessment Service uses the LLM Gateway for:
- Generating adaptive interview questions
- Analyzing candidate responses
- Maintaining conversational context

```python
import openai

client = openai.OpenAI(
    base_url="http://llm-gateway.platform-services:5005/v1",
    api_key="not-required"
)

def generate_question(context: dict, candidate_response: str) -> str:
    response = client.chat.completions.create(
        model="claude-sonnet",
        messages=[
            {
                "role": "system",
                "content": f"You are an AI interviewer assessing {context['role']}..."
            },
            {
                "role": "user",
                "content": candidate_response
            }
        ]
    )
    return response.choices[0].message.content
```

### Scoring Service

The Scoring Service uses the LLM Gateway for:
- Evaluating technical accuracy
- Generating competency assessments

```python
def evaluate_response(question: str, answer: str, rubric: dict) -> dict:
    response = client.chat.completions.create(
        model="claude-sonnet",
        messages=[
            {
                "role": "system",
                "content": f"Evaluate this answer using the rubric: {rubric}"
            },
            {
                "role": "user",
                "content": f"Question: {question}\nAnswer: {answer}"
            }
        ]
    )
    return parse_evaluation(response.choices[0].message.content)
```

---

## Configuration

### Kubernetes Deployment

```yaml
# k8s/base/assessment-service/deployment.yaml
spec:
  template:
    spec:
      containers:
      - name: assessment-service
        env:
        - name: LLM_BASE_URL
          value: "http://llm-gateway.platform-services:5005/v1"
        - name: LLM_MODEL
          value: "claude-sonnet"
        - name: LLM_TIMEOUT
          value: "60"
```

### Application Configuration

```python
# config.py
import os

LLM_CONFIG = {
    "base_url": os.environ.get("LLM_BASE_URL", "http://llm-gateway.platform-services:5005/v1"),
    "model": os.environ.get("LLM_MODEL", "claude-sonnet"),
    "timeout": int(os.environ.get("LLM_TIMEOUT", "60")),
    "max_tokens": int(os.environ.get("LLM_MAX_TOKENS", "4096")),
}
```

---

## Error Handling

### Expected Errors

| HTTP Code | Meaning | TalentMesh Action |
|-----------|---------|-------------------|
| 504 | LLM timeout | Retry with backoff, show user message |
| 503 | LLM unavailable | Queue request, show maintenance message |
| 429 | Rate limited | Exponential backoff |

### Retry Strategy

```python
import asyncio
from tenacity import retry, stop_after_attempt, wait_exponential

@retry(
    stop=stop_after_attempt(3),
    wait=wait_exponential(multiplier=1, min=1, max=10)
)
async def call_llm(messages: list) -> str:
    response = client.chat.completions.create(
        model="claude-sonnet",
        messages=messages
    )
    return response.choices[0].message.content
```

---

## Latency Requirements

| Use Case | Target Latency | Notes |
|----------|---------------|-------|
| Interview question generation | < 1s | Real-time conversation |
| Response evaluation | < 2s | Background processing acceptable |
| Batch scoring | < 5s per item | Async processing |

---

## References

- [OpenOva LLM Gateway Service](https://github.com/openova-io/handbook/blob/main/services/LLM_GATEWAY.md)
- [ADR-009: LLM Integration](../adrs/ADR-009-CLAUDE-CLI-WRAPPER.md)
- [AI/ML Specifications](./AI_ML_SPECIFICATIONS.md)

---

*Document Version: 4.0*
*Last Updated: 2026-01-12*
*Owner: AI Team*
