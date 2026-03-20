# Cross-Platform Integration Patterns

## Overview

This section covers integration patterns combining Hatz AI, Go High Level, and n8n for real-world automation scenarios.

---

## Pattern 1: n8n → Hatz AI → AI Processing

**Use Case**: Process data through Hatz AI for intelligent analysis.

### n8n Workflow Setup

**Trigger**: Any n8n trigger (webhook, schedule, manual)

**HTTP Request Node**:
```json
{
  "method": "POST",
  "url": "https://ai.hatz.ai/v1/chat/completions",
  "authentication": "genericCredentialType",
  "genericAuthType": "httpHeaderAuth",
  "sendHeaders": true,
  "headerParameters": {
    "parameters": [
      {"name": "Content-Type", "value": "application/json"}
    ]
  },
  "sendBody": true,
  "contentType": "json",
  "bodyParameters": {
    "parameters": [
      {"name": "model", "value": "gpt-4o"},
      {"name": "messages", "value": "=[{\"role\": \"user\", \"content\": \"{{$json.user_message}}\"}]"}
    ]
  }
}
```

**Credential Setup**: Create header auth credential with:
- Name: `X-API-Key`
- Value: Your Hatz AI API key

### Example Flow

1. **Webhook Trigger** - Receive data from external source
2. **Hatz AI Processing** - Analyze/transform data
3. **Function Node** - Extract relevant fields
4. **Output** - Send to next destination

---

## Pattern 2: n8n → Go High Level → Contact Creation

**Use Case**: Create or update contacts in GHL from external data sources.

### n8n HTTP Request Node for GHL

```json
{
  "method": "POST",
  "url": "https://services.leadconnectorhq.com/contacts/",
  "authentication": "genericCredentialType",
  "genericAuthType": "httpHeaderAuth",
  "sendHeaders": true,
  "headerParameters": {
    "parameters": [
      {"name": "Content-Type", "value": "application/json"},
      {"name": "Version", "value": "2021-07-28"}
    ]
  },
  "sendBody": true,
  "contentType": "json",
  "bodyParameters": {
    "parameters": [
      {"name": "locationId", "value": "={{$env.GHL_LOCATION_ID}}"},
      {"name": "email", "value": "={{$json.email}}"},
      {"name": "firstName", "value": "={{$json.firstName}}"},
      {"name": "lastName", "value": "={{$json.lastName}}"},
      {"name": "phone", "value": "={{$json.phone}}"},
      {"name": "source", "value": "api"},
      {"name": "tags", "value": "={{$json.tags}}"}
    ]
  }
}
```

---

## Pattern 3: GHL Webhook → n8n → Hatz AI → GHL Update

**Use Case**: Intelligent contact enrichment.

### Flow Diagram

```
GHL ContactCreated webhook
        ↓
     n8n receives webhook
        ↓
  Hatz AI analyzes contact data
     (categorize, score, enrich)
        ↓
     n8n processes AI response
        ↓
  GHL update contact with insights
```

### Step 1: Configure GHL Webhook

In GHL, set up webhook for `ContactCreate` event pointing to:
```
https://your-instance.app.n8n.cloud/webhook/ghl-contact-created
```

### Step 2: n8n Webhook Node

```json
{
  "parameters": {
    "httpMethod": "POST",
    "path": "ghl-contact-created",
    "responseMode": "responseNode"
  },
  "name": "GHL Webhook",
  "type": "n8n-nodes-base.webhook"
}
```

### Step 3: n8n HTTP Request to Hatz AI

```json
{
  "method": "POST",
  "url": "https://ai.hatz.ai/v1/chat/completions",
  "sendBody": true,
  "bodyParameters": {
    "parameters": [
      {"name": "model", "value": "gpt-4o"},
      {"name": "messages", "value": "=[{\"role\": \"system\", \"content\": \"Analyze this contact and determine lead quality and suggested tags.\"}, {\"role\": \"user\", \"content\": \"Contact: {{JSON.stringify($json)}}\"}]"}
    ]
  }
}
```

### Step 4: Update GHL Contact

```json
{
  "method": "PUT",
  "url": "=https://services.leadconnectorhq.com/contacts/{{$json.contactId}}",
  "sendBody": true,
  "bodyParameters": {
    "parameters": [
      {"name": "tags", "value": "={{$json.aiSuggestedTags}}"},
      {"name": "customFields", "value": "=[{\"id\": \"lead_score\", \"value\": {{$json.leadScore}}}]"}
    ]
  }
}
```

---

## Pattern 4: Hatz AI Agent with Code Execution → GHL API

**Use Case**: Hatz AI agent uses code execution tool to interact with GHL.

### Hatz AI Chat Completion Request

```bash
curl "https://ai.hatz.ai/v1/chat/completions" \
  -H "X-API-Key: $HATZ_API_KEY" \
  -H "Content-Type: application/json" \
  -d \'{
    "messages": [
      {
        "role": "user",
        "content": "Use Python to pull all contacts from Go High Level API (location ID: abc123) and create a summary CSV with name, email, and lead score"
      }
    ],
    "model": "agent-code-executor",
    "tools_to_use": ["daytona_code_execution"]
  }\'
```

### Agent Executes Python Code

The agent will generate and execute Python code like:

```python
import requests
import csv
import os

GHL_TOKEN = os.environ.get("GHL_TOKEN")
LOCATION_ID = "abc123"

headers = {
    "Authorization": f"Bearer {GHL_TOKEN}",
    "Version": "2021-07-28"
}

# Search contacts
response = requests.post(
    "https://services.leadconnectorhq.com/contacts/search",
    headers=headers,
    json={"locationId": LOCATION_ID, "pageSize": 100}
)

contacts = response.json()["contacts"]

# Create CSV
with open("/home/daytona/workspace/output/contacts_summary.csv", "w", newline="") as f:
    writer = csv.writer(f)
    writer.writerow(["Name", "Email", "Lead Score"])
    for c in contacts:
        score = next((f["value"] for f in c.get("customFields", []) if f["id"] == "lead_score"), "N/A")
        writer.writerow([
            f"{c.get(\"firstName\", \"\")} {c.get(\"lastName\", \"\")}",
            c.get("email", ""),
            score
        ])

print(f"Created CSV with {len(contacts)} contacts")
```

---

## Pattern 5: Scheduled n8n → GHL Opportunity Analysis → Hatz AI Report

**Use Case**: Daily pipeline analysis with AI-generated insights.

### n8n Workflow

**Node 1: Schedule Trigger**
```json
{
  "parameters": {
    "rule": {
      "interval": [{"field": "hours", "hoursAt": 9}]
    }
  }
}
```

**Node 2: Get GHL Opportunities**
```json
{
  "method": "GET",
  "url": "https://services.leadconnectorhq.com/opportunities/",
  "sendQuery": true,
  "queryParameters": {
    "parameters": [
      {"name": "locationId", "value": "={{$env.GHL_LOCATION_ID}}"},
      {"name": "status", "value": "open"}
    ]
  }
}
```

**Node 3: Format for Hatz AI**
```javascript
// Code node
const opportunities = $input.all();
const summary = {
  totalValue: 0,
  count: opportunities.length,
  byStage: {}
};

for (const opp of opportunities) {
  summary.totalValue += opp.json.monetaryValue || 0;
  const stage = opp.json.pipelineStageId || "unknown";
  summary.byStage[stage] = (summary.byStage[stage] || 0) + 1;
}

return {
  json: {
    prompt: \`Analyze this pipeline data and provide strategic recommendations:
    - Total Open Opportunities: \${summary.count}
    - Total Value: $\${summary.totalValue.toLocaleString()}
    - Distribution by Stage: \${JSON.stringify(summary.byStage)}
    
    Provide: 1) Pipeline health assessment, 2) Conversion recommendations, 3) Follow-up priorities\`
  }
};
```

**Node 4: Hatz AI Analysis**
```json
{
  "method": "POST",
  "url": "https://ai.hatz.ai/v1/chat/completions",
  "sendBody": true,
  "bodyParameters": {
    "parameters": [
      {"name": "model", "value": "gpt-4o"},
      {"name": "messages", "value": "=[{\"role\": \"user\", \"content\": \"{{$json.prompt}}\"}]"}
    ]
  }
}
```

**Node 5: Send Report (Email/Slack)**
```json
{
  "method": "POST",
  "url": "https://hooks.slack.com/services/YOUR/SLACK/WEBHOOK",
  "sendBody": true,
  "bodyParameters": {
    "parameters": [
      {"name": "text", "value": "={{$json.choices[0].message.content}}"}
    ]
  }
}
```

---

## Pattern 6: AI-Powered Lead Qualification

**Use Case**: Automatically qualify leads from GHL forms using Hatz AI.

### Flow

```
GHL Form Submission webhook
        ↓
     n8n receives form data
        ↓
  Hatz AI qualifies lead (BANT analysis)
     - Budget
     - Authority  
     - Need
     - Timeline
        ↓
     n8n processes qualification score
        ↓
  GHL update contact + create opportunity if qualified
        ↓
  GHL send follow-up SMS if unqualified
```

### Hatz AI Qualification Prompt

```json
{
  "model": "gpt-4o",
  "messages": [
    {
      "role": "system",
      "content": "You are a lead qualification expert. Analyze form submissions using BANT methodology. Return JSON with: { qualified: boolean, score: 1-10, reasoning: string, tags: string[], nextAction: string }"
    },
    {
      "role": "user",
      "content": "Form data: {{$json.formFields}}"
    }
  ]
}
```

---

## Pattern 7: Multi-Platform Data Sync

**Use Case**: Keep data synchronized across platforms.

### Example: Contact created in GHL → Sync to other systems

```
GHL ContactCreate webhook
        ↓
     n8n orchestrates sync
        ↓
  ┌────┴────┬─────────┐
  ↓         ↓         ↓
Hatz AI   CRM X    Database
(log)    (create)  (insert)
```

### n8n Parallel Execution

```javascript
// Use Split In Batches node for parallel processing
// Each output goes to different destination
```

---

## Pattern 8: Error Handling Across Platforms

**Use Case**: Robust error handling and retry logic.

### n8n Error Workflow

```json
{
  "parameters": {
    "errorOutput": "error"
  },
  "name": "Error Handler",
  "type": "n8n-nodes-base.errorTrigger"
}
```

### Error Notification to Hatz AI

```json
{
  "method": "POST",
  "url": "https://ai.hatz.ai/v1/chat/completions",
  "bodyParameters": {
    "parameters": [
      {"name": "model", "value": "gpt-4o"},
      {"name": "messages", "value": "=[{\"role\": \"user\", \"content\": \"Analyze this error and suggest fixes: {{$json}}\"}]"}
    ]
  }
}
```

---

## Environment Variables Template

For secure credential management across all three platforms:

```bash
# n8n environment
N8N_API_KEY=n8n_xxxxx
N8N_BASE_URL=https://your-instance.app.n8n.cloud/api/v1

# Hatz AI
HATZ_API_KEY=hatz_xxxxx

# Go High Level
GHL_TOKEN=ghl_xxxxx
GHL_LOCATION_ID=abc123
GHL_WEBHOOK_SECRET=secret_xxxxx
```

---

## Best Practices for Cross-Platform Integrations

1. **Use webhooks for real-time** - More efficient than polling
2. **Implement idempotency** - Handle duplicate webhooks
3. **Centralize credentials** - Use n8n credentials manager
4. **Log all API calls** - Essential for debugging
5. **Handle rate limits** - Different platforms have different limits
6. **Use environment variables** - Never hardcode credentials
7. **Monitor execution history** - Watch for failures
8. **Build error workflows** - Automatic error handling
