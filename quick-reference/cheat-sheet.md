# API Expert Cheat Sheet

Quick reference for vibe coders building integrations.

---

## Authentication Quick Reference

### Hatz AI
```bash
# Header format
-H "X-API-Key: YOUR_KEY"
# OR
-H "Authorization: Bearer YOUR_KEY"
```

### Go High Level
```bash
# Required headers
-H "Authorization: Bearer YOUR_TOKEN"
-H "Version: 2021-07-28"
```

### n8n
```bash
# Header format
-H "X-N8N-API-KEY: YOUR_KEY"
```

---

## Base URLs

| Platform | Base URL |
|----------|----------|
| Hatz AI | `https://ai.hatz.ai/v1` |
| Go High Level | `https://services.leadconnectorhq.com` |
| n8n Cloud | `https://{instance}.app.n8n.cloud/api/v1` |
| n8n Self-hosted | `http://localhost:5678/api/v1` |

---

## Hatz AI Quick Commands

### Chat Completion
```bash
curl "https://ai.hatz.ai/v1/chat/completions" \
  -H "X-API-Key: $KEY" \
  -H "Content-Type: application/json" \
  -d \'{"messages":[{"role":"user","content":"Hello"}],"model":"gpt-4o"}\'
```

### List Models
```bash
curl "https://ai.hatz.ai/v1/chat/models" -H "X-API-Key: $KEY"
```

### Use an Agent
```bash
curl "https://ai.hatz.ai/v1/chat/completions" \
  -H "X-API-Key: $KEY" \
  -H "Content-Type: application/json" \
  -d \'{"messages":[{"role":"user","content":"Search for AI news"}],"model":"agent-SEARCH_AGENT_ID"}\'
```

### Upload File
```bash
curl "https://ai.hatz.ai/v1/files/upload" \
  -H "X-API-Key: $KEY" \
  -F "file=@document.pdf"
```

---

## Go High Level Quick Commands

### Create Contact
```bash
curl -X POST "https://services.leadconnectorhq.com/contacts/" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Version: 2021-07-28" \
  -H "Content-Type: application/json" \
  -d \'{"locationId":"LOC_ID","email":"test@example.com","firstName":"John"}\'
```

### Search Contacts
```bash
curl -X POST "https://services.leadconnectorhq.com/contacts/search" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Version: 2021-07-28" \
  -H "Content-Type: application/json" \
  -d \'{"locationId":"LOC_ID","query":"john"}\'
```

### Send SMS
```bash
curl -X POST "https://services.leadconnectorhq.com/conversations/messages" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Version: 2021-07-28" \
  -H "Content-Type: application/json" \
  -d \'{"locationId":"LOC_ID","contactId":"C_ID","type":"SMS","message":"Hello!"}\'
```

### Get Opportunities
```bash
curl "https://services.leadconnectorhq.com/opportunities/?locationId=LOC_ID&status=open" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Version: 2021-07-28"
```

---

## n8n Quick Commands

### List Workflows
```bash
curl "https://instance.app.n8n.cloud/api/v1/workflows" \
  -H "X-N8N-API-KEY: $KEY"
```

### List Active Workflows
```bash
curl "https://instance.app.n8n.cloud/api/v1/workflows?active=true" \
  -H "X-N8N-API-KEY: $KEY"
```

### Get Executions
```bash
curl "https://instance.app.n8n.cloud/api/v1/executions?limit=10" \
  -H "X-N8N-API-KEY: $KEY"
```

### Trigger Webhook
```bash
curl -X POST "https://instance.app.n8n.cloud/webhook/WEBHOOK_ID" \
  -H "Content-Type: application/json" \
  -d \'{"data":"value"}\'
```

---

## Hatz AI Tools Reference

| Tool | Use Case |
|------|----------|
| `google_search` | Web search |
| `firecrawl_scrape` | Scrape webpage |
| `daytona_code_execution` | Run Python/Shell |
| `google_maps_*` | Location services |
| `google_weather_*` | Weather data |

**Enable Tools**:
```json
{
  "tools_to_use": ["google_search", "firecrawl_scrape"],
  "auto_tool_selection": true
}
```

---

## Go High Level Rate Limits

| Limit Type | Value |
|------------|-------|
| Burst | 100 requests / 10 sec |
| Daily | 200,000 requests / day |

**Check Remaining**:
```
X-RateLimit-Daily-Remaining
X-RateLimit-Remaining
```

---

## GHL Webhook Events (Most Common)

| Event | Trigger |
|-------|---------|
| `ContactCreate` | New contact |
| `OpportunityCreate` | New deal |
| `InvoicePaid` | Payment received |
| `InboundMessage` | SMS/Email received |
| `AppointmentCreate` | Booking made |

---

## Python Client Snippets

### Hatz AI
```python
import requests

def hatz_chat(message, model="gpt-4o"):
    r = requests.post(
        "https://ai.hatz.ai/v1/chat/completions",
        headers={"X-API-Key": API_KEY},
        json={"messages": [{"role": "user", "content": message}], "model": model}
    )
    return r.json()["choices"][0]["message"]["content"]
```

### Go High Level
```python
def ghl_create_contact(email, first_name, location_id):
    r = requests.post(
        "https://services.leadconnectorhq.com/contacts/",
        headers={
            "Authorization": f"Bearer {GHL_TOKEN}",
            "Version": "2021-07-28",
            "Content-Type": "application/json"
        },
        json={
            "locationId": location_id,
            "email": email,
            "firstName": first_name
        }
    )
    return r.json()
```

### n8n
```python
def n8n_list_workflows():
    r = requests.get(
        f"{N8N_BASE_URL}/workflows",
        headers={"X-N8N-API-KEY": N8N_KEY}
    )
    return r.json()["data"]
```

---

## HTTP Status Codes

| Code | Meaning | Action |
|------|---------|--------|
| 200 | Success | Process response |
| 201 | Created | New resource created |
| 400 | Bad Request | Check payload |
| 401 | Unauthorized | Check credentials |
| 403 | Forbidden | Check permissions |
| 404 | Not Found | Check resource ID |
| 422 | Validation Error | Check data format |
| 429 | Rate Limited | Wait and retry |
| 500 | Server Error | Retry later |

---

## Environment Variables Template

```bash
# .env file
HATZ_API_KEY=hatz_xxxxxxxx
GHL_TOKEN=ghl_xxxxxxxx
GHL_LOCATION_ID=abc123
N8N_API_KEY=n8n_xxxxxxxx
N8N_BASE_URL=https://your-instance.app.n8n.cloud/api/v1
```

---

## Documentation Links

| Platform | URL |
|----------|-----|
| Hatz AI API | https://api-docs.hatz.ai/ |
| GHL Marketplace | https://marketplace.gohighlevel.com/docs/ |
| n8n API | https://docs.n8n.io/api/ |
