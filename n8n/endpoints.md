# n8n - API Endpoints

## Workflows API

### List Workflows

**Endpoint**: `GET /workflows`

**Query Parameters**:
- `active` (optional): Filter by active status (true/false)
- `limit` (optional): Number of results
- `cursor` (optional): Pagination cursor

**Example**:
```bash
curl -X GET \
  "https://your-instance.app.n8n.cloud/api/v1/workflows?active=true" \
  -H "accept: application/json" \
  -H "X-N8N-API-KEY: your-api-key"
```

**Response**:
```json
{
  "data": [
    {
      "id": "1",
      "name": "My Workflow",
      "active": true,
      "createdAt": "2026-03-20T10:00:00.000Z",
      "updatedAt": "2026-03-20T12:00:00.000Z"
    }
  ],
  "nextCursor": null
}
```

---

### Get Workflow

**Endpoint**: `GET /workflows/{id}`

**Example**:
```bash
curl -X GET \
  "https://your-instance.app.n8n.cloud/api/v1/workflows/1" \
  -H "accept: application/json" \
  -H "X-N8N-API-KEY: your-api-key"
```

---

### Create Workflow

**Endpoint**: `POST /workflows`

**Request Body**:
```json
{
  "name": "New Workflow",
  "nodes": [...],
  "connections": {...},
  "settings": {}
}
```

**Example**:
```bash
curl -X POST \
  "https://your-instance.app.n8n.cloud/api/v1/workflows" \
  -H "accept: application/json" \
  -H "X-N8N-API-KEY: your-api-key" \
  -H "Content-Type: application/json" \
  -d \'{
    "name": "My New Workflow",
    "nodes": [],
    "connections": {}
  }\'
```

---

### Update Workflow

**Endpoint**: `PATCH /workflows/{id}`

**Request Body**:
```json
{
  "name": "Updated Name",
  "active": true
}
```

---

### Delete Workflow

**Endpoint**: `DELETE /workflows/{id}`

**Example**:
```bash
curl -X DELETE \
  "https://your-instance.app.n8n.cloud/api/v1/workflows/1" \
  -H "X-N8N-API-KEY: your-api-key"
```

---

### Transfer Workflow

**Endpoint**: `POST /workflows/{id}/transfer`

**Description**: Transfer workflow ownership to another user.

---

## Executions API

### List Executions

**Endpoint**: `GET /executions`

**Query Parameters**:
- `workflowId` (optional): Filter by workflow
- `status` (optional): Filter by status (success, error, running, waiting)
- `limit` (optional): Number of results
- `cursor` (optional): Pagination cursor

**Example**:
```bash
curl -X GET \
  "https://your-instance.app.n8n.cloud/api/v1/executions?workflowId=1&limit=10" \
  -H "accept: application/json" \
  -H "X-N8N-API-KEY: your-api-key"
```

**Response**:
```json
{
  "data": [
    {
      "id": "100",
      "workflowId": "1",
      "status": "success",
      "mode": "manual",
      "startedAt": "2026-03-20T10:00:00.000Z",
      "stoppedAt": "2026-03-20T10:00:05.000Z"
    }
  ]
}
```

---

### Get Execution

**Endpoint**: `GET /executions/{id}`

**Example**:
```bash
curl -X GET \
  "https://your-instance.app.n8n.cloud/api/v1/executions/100" \
  -H "accept: application/json" \
  -H "X-N8N-API-KEY: your-api-key"
```

---

### Delete Execution

**Endpoint**: `DELETE /executions/{id}`

---

## Users API

### List Users

**Endpoint**: `GET /users`

**Example**:
```bash
curl -X GET \
  "https://your-instance.app.n8n.cloud/api/v1/users" \
  -H "accept: application/json" \
  -H "X-N8N-API-KEY: your-api-key"
```

---

### Create User

**Endpoint**: `POST /users`

**Request Body**:
```json
{
  "email": "user@example.com",
  "firstName": "John",
  "lastName": "Doe",
  "password": "securepassword"
}
```

**Example**:
```bash
curl -X POST \
  "https://your-instance.app.n8n.cloud/api/v1/users" \
  -H "accept: application/json" \
  -H "X-N8N-API-KEY: your-api-key" \
  -H "Content-Type: application/json" \
  -d \'{
    "email": "user@example.com",
    "firstName": "John",
    "lastName": "Doe"
  }\'
```

---

### Get User

**Endpoint**: `GET /users/{id}`

---

### Update User

**Endpoint**: `PATCH /users/{id}`

**Request Body**:
```json
{
  "firstName": "Updated First Name"
}
```

---

### Delete User

**Endpoint**: `DELETE /users/{id}`

---

## Variables API

### List Variables

**Endpoint**: `GET /variables`

**Example**:
```bash
curl -X GET \
  "https://your-instance.app.n8n.cloud/api/v1/variables" \
  -H "accept: application/json" \
  -H "X-N8N-API-KEY: your-api-key"
```

---

### Create Variable

**Endpoint**: `POST /variables`

**Request Body**:
```json
{
  "key": "API_KEY",
  "value": "secret-value"
}
```

---

### Update Variable

**Endpoint**: `PATCH /variables/{id}`

---

### Delete Variable

**Endpoint**: `DELETE /variables/{id}`

---

## Credentials API

### List Credentials

**Endpoint**: `GET /credentials`

**Example**:
```bash
curl -X GET \
  "https://your-instance.app.n8n.cloud/api/v1/credentials" \
  -H "accept: application/json" \
  -H "X-N8N-API-KEY: your-api-key"
```

---

### Create Credential

**Endpoint**: `POST /credentials`

**Request Body**:
```json
{
  "name": "My API Key",
  "type": "httpHeaderAuth",
  "data": {
    "name": "Authorization",
    "value": "Bearer token123"
  }
}
```

---

## Audit API

### Generate Audit

**Endpoint**: `POST /audit`

**Request Body**:
```json
{
  "filters": {
    "action": "workflow.run",
    "userId": "1"
  },
  "pagination": {
    "limit": 100
  }
}
```

**Example**:
```bash
curl -X POST \
  "https://your-instance.app.n8n.cloud/api/v1/audit" \
  -H "accept: application/json" \
  -H "X-N8N-API-KEY: your-api-key" \
  -H "Content-Type: application/json" \
  -d \'{"filters": {"action": "workflow.run"}}\'
```

---

## Webhook Endpoints (Production)

n8n workflows can create webhook endpoints for external triggers:

```
POST /webhook/{webhook-id}
POST /webhook-test/{webhook-id}
```

The `-test` version is for testing within the editor.

---

## Quick Reference Table

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/workflows` | GET | List workflows |
| `/workflows/{id}` | GET | Get workflow |
| `/workflows` | POST | Create workflow |
| `/workflows/{id}` | PATCH | Update workflow |
| `/workflows/{id}` | DELETE | Delete workflow |
| `/executions` | GET | List executions |
| `/executions/{id}` | GET | Get execution |
| `/users` | GET | List users |
| `/users` | POST | Create user |
| `/users/{id}` | DELETE | Delete user |
| `/variables` | GET | List variables |
| `/credentials` | GET | List credentials |
| `/audit` | POST | Generate audit |

---

## Error Responses

| Status Code | Description |
|-------------|-------------|
| 400 | Bad Request |
| 401 | Invalid API key |
| 403 | Forbidden (feature not available during trial) |
| 404 | Resource not found |

---

## API Reference Link

Full API reference: https://docs.n8n.io/api/api-reference/
