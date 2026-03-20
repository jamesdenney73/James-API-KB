# n8n - Integration Patterns

## Common Integration Scenarios

### 1. List and Filter Workflows

```python
import requests
import os

N8N_API_KEY = os.environ.get("N8N_API_KEY")
N8N_BASE_URL = os.environ.get("N8N_BASE_URL", "https://your-instance.app.n8n.cloud/api/v1")

def get_workflows(active_only=True):
    url = f"{N8N_BASE_URL}/workflows"
    
    headers = {
        "X-N8N-API-KEY": N8N_API_KEY,
        "accept": "application/json"
    }
    
    params = {}
    if active_only:
        params["active"] = "true"
    
    response = requests.get(url, headers=headers, params=params)
    return response.json()["data"]

# Usage
workflows = get_workflows()
for wf in workflows:
    print(f"{wf[\"id\"]}: {wf[\"name\"]} (active: {wf[\"active\"]})")
```

---

### 2. Trigger Workflow via Webhook

```python
import requests
import json

# Production webhook URL from n8n workflow
WEBHOOK_URL = "https://your-instance.app.n8n.cloud/webhook/abc123"

def trigger_workflow(data):
    response = requests.post(WEBHOOK_URL, json=data)
    return response.json()

# Usage
result = trigger_workflow({
    "contact_email": "user@example.com",
    "action": "send_welcome"
})
print(result)
```

---

### 3. Get Execution History

```python
import requests
import os

N8N_API_KEY = os.environ.get("N8N_API_KEY")
N8N_BASE_URL = os.environ.get("N8N_BASE_URL", "https://your-instance.app.n8n.cloud/api/v1")

def get_executions(workflow_id=None, status=None, limit=20):
    url = f"{N8N_BASE_URL}/executions"
    
    headers = {
        "X-N8N-API-KEY": N8N_API_KEY,
        "accept": "application/json"
    }
    
    params = {"limit": limit}
    if workflow_id:
        params["workflowId"] = workflow_id
    if status:
        params["status"] = status
    
    response = requests.get(url, headers=headers, params=params)
    return response.json()["data"]

def get_failed_executions(workflow_id):
    return get_executions(workflow_id=workflow_id, status="error")

# Usage
failed = get_failed_executions("1")
for execution in failed:
    print(f"Execution {execution[\"id\"]} failed at {execution[\"stoppedAt\"]}")
```

---

### 4. Create and Activate Workflow

```python
import requests
import os

N8N_API_KEY = os.environ.get("N8N_API_KEY")
N8N_BASE_URL = os.environ.get("N8N_BASE_URL", "https://your-instance.app.n8n.cloud/api/v1")

def create_workflow(name, nodes, connections):
    url = f"{N8N_BASE_URL}/workflows"
    
    headers = {
        "X-N8N-API-KEY": N8N_API_KEY,
        "accept": "application/json",
        "Content-Type": "application/json"
    }
    
    payload = {
        "name": name,
        "nodes": nodes,
        "connections": connections,
        "settings": {
            "executionOrder": "v1"
        }
    }
    
    response = requests.post(url, headers=headers, json=payload)
    return response.json()

def activate_workflow(workflow_id):
    url = f"{N8N_BASE_URL}/workflows/{workflow_id}"
    
    headers = {
        "X-N8N-API-KEY": N8N_API_KEY,
        "Content-Type": "application/json"
    }
    
    payload = {"active": True}
    response = requests.patch(url, headers=headers, json=payload)
    return response.json()

# Example: Simple webhook to HTTP request workflow
nodes = [
    {
        "parameters": {
            "httpMethod": "POST",
            "path": "webhook-endpoint"
        },
        "id": "webhook-node",
        "name": "Webhook",
        "type": "n8n-nodes-base.webhook",
        "typeVersion": 1,
        "position": [250, 300],
        "webhookId": "webhook-id"
    },
    {
        "parameters": {
            "url": "https://api.example.com/endpoint",
            "method": "POST"
        },
        "id": "http-node",
        "name": "HTTP Request",
        "type": "n8n-nodes-base.httpRequest",
        "typeVersion": 4,
        "position": [450, 300]
    }
]

connections = {
    "Webhook": {
        "main": [[{"node": "HTTP Request", "type": "main", "index": 0}]]
    }
}

# Create and activate
workflow = create_workflow("API Integration", nodes, connections)
activated = activate_workflow(workflow["id"])
print(f"Workflow {activated[\"id\"]} activated")
```

---

### 5. Manage Variables

```python
import requests
import os

N8N_API_KEY = os.environ.get("N8N_API_KEY")
N8N_BASE_URL = os.environ.get("N8N_BASE_URL", "https://your-instance.app.n8n.cloud/api/v1")

def get_variables():
    url = f"{N8N_BASE_URL}/variables"
    
    headers = {
        "X-N8N-API-KEY": N8N_API_KEY,
        "accept": "application/json"
    }
    
    response = requests.get(url, headers=headers)
    return response.json()["data"]

def create_variable(key, value):
    url = f"{N8N_BASE_URL}/variables"
    
    headers = {
        "X-N8N-API-KEY": N8N_API_KEY,
        "Content-Type": "application/json"
    }
    
    payload = {"key": key, "value": value}
    response = requests.post(url, headers=headers, json=payload)
    return response.json()

def update_variable(var_id, value):
    url = f"{N8N_BASE_URL}/variables/{var_id}"
    
    headers = {
        "X-N8N-API-KEY": N8N_API_KEY,
        "Content-Type": "application/json"
    }
    
    payload = {"value": value}
    response = requests.patch(url, headers=headers, json=payload)
    return response.json()

# Usage
create_variable("ENVIRONMENT", "production")
create_variable("API_VERSION", "v2")

variables = get_variables()
for var in variables:
    print(f"{var[\"key\"]}: {var[\"value\"]}")
```

---

### 6. User Management

```python
import requests
import os

N8N_API_KEY = os.environ.get("N8N_API_KEY")
N8N_BASE_URL = os.environ.get("N8N_BASE_URL", "https://your-instance.app.n8n.cloud/api/v1")

def create_user(email, first_name, last_name):
    url = f"{N8N_BASE_URL}/users"
    
    headers = {
        "X-N8N-API-KEY": N8N_API_KEY,
        "Content-Type": "application/json"
    }
    
    payload = {
        "email": email,
        "firstName": first_name,
        "lastName": last_name
    }
    
    response = requests.post(url, headers=headers, json=payload)
    return response.json()

def delete_user(user_id):
    url = f"{N8N_BASE_URL}/users/{user_id}"
    
    headers = {
        "X-N8N-API-KEY": N8N_API_KEY
    }
    
    response = requests.delete(url, headers=headers)
    return response.status_code == 200

# Usage
new_user = create_user("jane@example.com", "Jane", "Developer")
print(f"Created user: {new_user[\"id\"]}")
```

---

### 7. HTTP Request Node Configuration (for external APIs)

When building n8n workflows that call external APIs, use the HTTP Request node:

**Hatz AI Configuration in n8n**:
```json
{
  "method": "POST",
  "url": "https://ai.hatz.ai/v1/chat/completions",
  "authentication": "predefinedCredentialType",
  "nodeCredentialType": "httpHeaderAuth",
  "sendHeaders": true,
  "headerParameters": {
    "parameters": [
      {"name": "Content-Type", "value": "application/json"}
    ]
  },
  "sendBody": true,
  "bodyParameters": {
    "parameters": [
      {"name": "messages", "value": "={{ $json.messages }}"},
      {"name": "model", "value": "gpt-4o"}
    ]
  }
}
```

**Go High Level Configuration in n8n**:
```json
{
  "method": "POST",
  "url": "https://services.leadconnectorhq.com/contacts/",
  "authentication": "predefinedCredentialType",
  "sendHeaders": true,
  "headerParameters": {
    "parameters": [
      {"name": "Content-Type", "value": "application/json"},
      {"name": "Version", "value": "2021-07-28"}
    ]
  },
  "sendBody": true,
  "bodyParameters": {
    "parameters": [
      {"name": "locationId", "value": "={{ $json.locationId }}"},
      {"name": "email", "value": "={{ $json.email }}"},
      {"name": "firstName", "value": "={{ $json.firstName }}"}
    ]
  }
}
```

---

### 8. Complete API Client Class

```python
import requests
import os

class N8NClient:
    def __init__(self, base_url=None, api_key=None):
        self.base_url = base_url or os.environ.get("N8N_BASE_URL")
        self.api_key = api_key or os.environ.get("N8N_API_KEY")
        self.headers = {
            "X-N8N-API-KEY": self.api_key,
            "accept": "application/json",
            "Content-Type": "application/json"
        }
    
    def _request(self, method, endpoint, **kwargs):
        url = f"{self.base_url}{endpoint}"
        response = requests.request(method, url, headers=self.headers, **kwargs)
        response.raise_for_status()
        return response.json()
    
    # Workflows
    def list_workflows(self, active=None):
        params = {}
        if active is not None:
            params["active"] = str(active).lower()
        return self._request("GET", "/workflows", params=params)
    
    def get_workflow(self, workflow_id):
        return self._request("GET", f"/workflows/{workflow_id}")
    
    def update_workflow(self, workflow_id, data):
        return self._request("PATCH", f"/workflows/{workflow_id}", json=data)
    
    def delete_workflow(self, workflow_id):
        return self._request("DELETE", f"/workflows/{workflow_id}")
    
    # Executions
    def list_executions(self, workflow_id=None, status=None, limit=20):
        params = {"limit": limit}
        if workflow_id:
            params["workflowId"] = workflow_id
        if status:
            params["status"] = status
        return self._request("GET", "/executions", params=params)
    
    def get_execution(self, execution_id):
        return self._request("GET", f"/executions/{execution_id}")
    
    # Users
    def list_users(self):
        return self._request("GET", "/users")
    
    def create_user(self, email, first_name, last_name):
        return self._request("POST", "/users", json={
            "email": email,
            "firstName": first_name,
            "lastName": last_name
        })
    
    # Variables
    def list_variables(self):
        return self._request("GET", "/variables")
    
    def create_variable(self, key, value):
        return self._request("POST", "/variables", json={
            "key": key,
            "value": value
        })

# Usage
client = N8NClient()
workflows = client.list_workflows(active=True)
print(f"Active workflows: {len(workflows[\"data\"])}")
```

---

## Best Practices

1. **Use environment variables** for API keys and base URLs
2. **Handle rate limits** - implement backoff on 429 responses
3. **Use webhooks** for real-time triggers instead of polling
4. **Log execution IDs** for debugging failed workflows
5. **Set up error workflows** to handle failures gracefully
6. **Use variables** for configuration that changes between environments
7. **Test with webhook-test URLs** before going live
