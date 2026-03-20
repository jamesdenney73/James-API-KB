# Hatz AI - API Endpoints

## Chat Completions

### Create Chat Completion

**Endpoint**: `POST /v1/chat/completions`

**Description**: Generate AI responses using LLMs or AI agents.

**Headers**:
```
X-API-Key: YOUR_API_KEY
Content-Type: application/json
```

**Request Body**:

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `messages` | array | Yes | List of message objects with role and content |
| `model` | string | No | Model ID (e.g., "gpt-4o") or agent ID (e.g., "agent-123") |
| `stream` | boolean | No | Enable SSE streaming (default: false) |
| `file_uuids` | array | No | File UUIDs to include in context |
| `tools_to_use` | array | No | List of tool names to enable |
| `auto_tool_selection` | boolean | No | Auto-select relevant tools |
| `store` | boolean | No | Store conversation for analytics |
| `include_reasoning` | boolean | No | Include reasoning in response |
| `metadata` | object | No | Custom metadata |

**Message Object**:
```json
{
  "role": "system" | "user" | "assistant" | "tool",
  "content": "string or array of content parts",
  "name": "string (optional)",
  "tool_call_id": "string (for tool role)"
}
```

**Example - Basic Chat**:
```bash
curl "https://ai.hatz.ai/v1/chat/completions" \
  -H "X-API-Key: $HATZ_API_KEY" \
  -H "Content-Type: application/json" \
  -d \'{
    "messages": [
      {"role": "user", "content": "Explain quantum computing in one paragraph"}
    ],
    "model": "gpt-4o",
    "stream": false
  }\'
```

**Example - Using an Agent**:
```bash
curl "https://ai.hatz.ai/v1/chat/completions" \
  -H "X-API-Key: $HATZ_API_KEY" \
  -H "Content-Type: application/json" \
  -d \'{
    "messages": [
      {"role": "user", "content": "Search for the latest AI news"}
    ],
    "model": "agent-web-search-agent",
    "auto_tool_selection": true
  }\'
```

**Example - Streaming Response**:
```bash
curl "https://ai.hatz.ai/v1/chat/completions" \
  -H "X-API-Key: $HATZ_API_KEY" \
  -H "Content-Type: application/json" \
  -d \'{
    "messages": [{"role": "user", "content": "Write a short story"}],
    "model": "gpt-4o",
    "stream": true
  }\'
```

**Example - With File Context**:
```bash
curl "https://ai.hatz.ai/v1/chat/completions" \
  -H "X-API-Key: $HATZ_API_KEY" \
  -H "Content-Type: application/json" \
  -d \'{
    "messages": [
      {"role": "user", "content": "Summarize this document"}
    ],
    "model": "gpt-4o",
    "file_uuids": ["file-abc123", "file-def456"]
  }\'
```

**Response Structure**:
```json
{
  "id": "chatcmpl-abc123",
  "object": "chat.completion",
  "created": 1234567890,
  "model": "gpt-4o",
  "choices": [
    {
      "index": 0,
      "message": {
        "role": "assistant",
        "content": "Response text here..."
      },
      "finish_reason": "stop"
    }
  ],
  "usage": {
    "prompt_tokens": 10,
    "completion_tokens": 50,
    "total_tokens": 60
  }
}
```

---

### List Available Models

**Endpoint**: `GET /v1/chat/models`

**Description**: Get list of available LLM models.

**Example**:
```bash
curl "https://ai.hatz.ai/v1/chat/models" \
  -H "X-API-Key: $HATZ_API_KEY"
```

**Response**:
```json
{
  "data": [
    {"id": "gpt-4o", "object": "model"},
    {"id": "gpt-4o-mini", "object": "model"},
    {"id": "claude-3-5-sonnet", "object": "model"}
  ]
}
```

---

### List Available Agents

**Endpoint**: `GET /v1/chat/agents`

**Description**: Get list of configured AI agents available in your organization.

**Example**:
```bash
curl "https://ai.hatz.ai/v1/chat/agents" \
  -H "X-API-Key: $HATZ_API_KEY"
```

**Use in Chat**: Set `model` to `agent-{agent_id}` in completions.

---

## Files API

### Get Presigned Upload URL

**Endpoint**: `POST /v1/files/upload-url`

**Description**: Get a presigned URL for file upload (for browser/mobile clients).

**Request Body**:
```json
{
  "filename": "document.pdf",
  "content_type": "application/pdf"
}
```

**Response**:
```json
{
  "upload_url": "https://...",
  "file_uuid": "file-abc123"
}
```

---

### Upload File Direct

**Endpoint**: `POST /v1/files/upload`

**Description**: Upload file directly (multipart form data).

**example**:
```bash
curl "https://ai.hatz.ai/v1/files/upload" \
  -H "X-API-Key: $HATZ_API_KEY" \
  -F "file=@document.pdf"
```

---

### Check File Status

**Endpoint**: `GET /v1/files/{file_uuid}/status`

**Description**: Poll for file processing status.

**Statuses**:
- `uploading` - File is being uploaded
- `processing` - File is being processed
- `ready` - File is ready for use
- `failed` - Processing failed

**Example**:
```bash
curl "https://ai.hatz.ai/v1/files/file-abc123/status" \
  -H "X-API-Key: $HATZ_API_KEY"
```

---

## Workflows API

### Run a Workflow

**Endpoint**: `POST /v1/workflows/run`

**Description**: Execute a multi-step AI workflow.

**Request Body**:
```json
{
  "workflow_id": "workflow-123",
  "input": {
    "data": "your input data"
  }
}
```

**Response**:
```json
{
  "job_id": "job-abc123"
}
```

---

### Get Workflow Status

**Endpoint**: `GET /v1/workflows/{job_id}`

**Description**: Poll for workflow execution status.

**Statuses**:
- `pending` - Job is queued
- `running` - Job is executing
- `complete` - Job finished successfully
- `failed` - Job failed

**Example**:
```bash
curl "https://ai.hatz.ai/v1/workflows/job-abc123" \
  -H "X-API-Key: $HATZ_API_KEY"
```

**Response (Complete)**:
```json
{
  "id": "job-abc123",
  "status": "complete",
  "result": {
    "output": "Workflow result data..."
  },
  "created_at": "2026-03-20T10:00:00Z",
  "completed_at": "2026-03-20T10:00:30Z"
}
```

---

## App Builder API

### List Apps

**Endpoint**: `GET /v1/app/list`

**Description**: Get list of available apps.

**Example**:
```bash
curl "https://ai.hatz.ai/v1/app/list" \
  -H "X-API-Key: $HATZ_API_KEY"
```

---

### Query an App

**Endpoint**: `POST /v1/app/{app_id}/query`

**Description**: Send a query to a configured app.

**Request Body**:
```json
{
  "query": "Your question or request",
  "context": {}
}
```

**Example**:
```bash
curl "https://ai.hatz.ai/v1/app/app-123/query" \
  -H "X-API-Key: $HATZ_API_KEY" \
  -H "Content-Type: application/json" \
  -d \'{"query": "Analyze this data"}\'
```

---

## OpenAI-Compatible Endpoints

### OpenAI Responses

**Endpoint**: `POST /v1/openai/responses`

**Base URL for OpenAI SDK**: `https://ai.hatz.ai/v1/openai`

Use this for tools that expect OpenAI-compatible API.

**Python Example**:
```python
from openai import OpenAI

client = OpenAI(
    base_url="https://ai.hatz.ai/v1/openai",
    api_key="YOUR_HATZ_API_KEY"
)

response = client.chat.completions.create(
    model="gpt-4o",
    messages=[{"role": "user", "content": "Hello!"}]
)
print(response.choices[0].message.content)
```

---

## Error Responses

| Status Code | Description |
|-------------|-------------|
| 400 | Bad request - invalid parameters |
| 401 | Unauthorized - invalid API key |
| 404 | Not found - resource doesn\'t exist |
| 429 | Rate limited - credit overage |
| 500 | Server error |

**Error Response Format**:
```json
{
  "error": {
    "message": "Error description",
    "type": "invalid_request_error",
    "code": "invalid_api_key"
  }
}
```
