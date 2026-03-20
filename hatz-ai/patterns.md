# Hatz AI - Integration Patterns

## Common Integration Scenarios

### 1. Simple Chat Completion

**Use Case**: Basic AI-powered responses.

```python
import requests
import os

API_KEY = os.environ.get("HATZ_API_KEY")

response = requests.post(
    "https://ai.hatz.ai/v1/chat/completions",
    headers={
        "X-API-Key": API_KEY,
        "Content-Type": "application/json"
    },
    json={
        "messages": [
            {"role": "user", "content": "Explain REST APIs in simple terms"}
        ],
        "model": "gpt-4o",
        "stream": False
    }
)

result = response.json()
print(result["choices"][0]["message"]["content"])
```

---

### 2. Streaming Response Handler

**Use Case**: Real-time response display.

```python
import requests
import json
import os

API_KEY = os.environ.get("HATZ_API_KEY")

response = requests.post(
    "https://ai.hatz.ai/v1/chat/completions",
    headers={
        "X-API-Key": API_KEY,
        "Content-Type": "application/json"
    },
    json={
        "messages": [
            {"role": "user", "content": "Write a long story about a robot"}
        ],
        "model": "gpt-4o",
        "stream": True
    },
    stream=True
)

for line in response.iter_lines():
    if line:
        line = line.decode("utf-8")
        if line.startswith("data: "):
            data = line[6:]
            if data == "[DONE]":
                break
            chunk = json.loads(data)
            if chunk["choices"][0].get("delta", {}).get("content"):
                print(chunk["choices"][0]["delta"]["content"], end="", flush=True)
```

---

### 3. Multi-Turn Conversation

**Use Case**: Maintaining conversation context.

```python
import requests
import os

API_KEY = os.environ.get("HATZ_API_KEY")
conversation_history = []

def chat(user_message):
    # Add user message to history
    conversation_history.append({
        "role": "user",
        "content": user_message
    })
    
    response = requests.post(
        "https://ai.hatz.ai/v1/chat/completions",
        headers={
            "X-API-Key": API_KEY,
            "Content-Type": "application/json"
        },
        json={
            "messages": conversation_history,
            "model": "gpt-4o"
        }
    )
    
    assistant_message = response.json()["choices"][0]["message"]["content"]
    
    # Add assistant response to history
    conversation_history.append({
        "role": "assistant",
        "content": assistant_message
    })
    
    return assistant_message

# Example usage
print(chat("My name is James"))
print(chat("What is my name?"))  # Should remember "James"
```

---

### 4. Document Processing Pipeline

**Use Case**: Upload document, ask questions about it.

```python
import requests
import os
import time

API_KEY = os.environ.get("HATZ_API_KEY")

# Step 1: Upload file
with open("report.pdf", "rb") as f:
    upload_response = requests.post(
        "https://ai.hatz.ai/v1/files/upload",
        headers={"X-API-Key": API_KEY},
        files={"file": f}
    )

file_uuid = upload_response.json()["file_uuid"]

# Step 2: Wait for processing
status = "processing"
while status != "ready":
    time.sleep(2)
    status_response = requests.get(
        f"https://ai.hatz.ai/v1/files/{file_uuid}/status",
        headers={"X-API-Key": API_KEY}
    )
    status = status_response.json()["status"]

# Step 3: Query with file context
response = requests.post(
    "https://ai.hatz.ai/v1/chat/completions",
    headers={
        "X-API-Key": API_KEY,
        "Content-Type": "application/json"
    },
    json={
        "messages": [
            {"role": "user", "content": "Summarize the key points of this document"}
        ],
        "model": "gpt-4o",
        "file_uuids": [file_uuid]
    }
)

print(response.json()["choices"][0]["message"]["content"])
```

---

### 5. Using AI Agents with Tools

**Use Case**: Leverage pre-configured agents with tools enabled.

```python
import requests
import os

API_KEY = os.environ.get("HATZ_API_KEY")

# Use an agent with web search capability
response = requests.post(
    "https://ai.hatz.ai/v1/chat/completions",
    headers={
        "X-API-Key": API_KEY,
        "Content-Type": "application/json"
    },
    json={
        "messages": [
            {"role": "user", "content": "What are the top 5 AI news stories today?"}
        ],
        "model": "agent-web-search",  # Agent ID
        "auto_tool_selection": True
    }
)

print(response.json()["choices"][0]["message"]["content"])
```

---

### 6. OpenAI SDK Integration

**Use Case**: Use Hatz AI with OpenAI SDK.

```python
from openai import OpenAI
import os

client = OpenAI(
    base_url="https://ai.hatz.ai/v1/openai",
    api_key=os.environ.get("HATZ_API_KEY")
)

# Use standard OpenAI SDK methods
response = client.chat.completions.create(
    model="gpt-4o",
    messages=[
        {"role": "system", "content": "You are a helpful coding assistant."},
        {"role": "user", "content": "Write a Python function to sort a list"}
    ]
)

print(response.choices[0].message.content)
```

---

### 7. Workflow Execution Pattern

**Use Case**: Run multi-step AI workflows.

```python
import requests
import os
import time

API_KEY = os.environ.get("HATZ_API_KEY")

# Start workflow
start_response = requests.post(
    "https://ai.hatz.ai/v1/workflows/run",
    headers={
        "X-API-Key": API_KEY,
        "Content-Type": "application/json"
    },
    json={
        "workflow_id": "data-analysis-workflow",
        "input": {
            "data_source": "sales_2026.csv",
            "analysis_type": "trend"
        }
    }
)

job_id = start_response.json()["job_id"]

# Poll for completion
while True:
    status_response = requests.get(
        f"https://ai.hatz.ai/v1/workflows/{job_id}",
        headers={"X-API-Key": API_KEY}
    )
    
    job_status = status_response.json()["status"]
    
    if job_status == "complete":
        result = status_response.json()["result"]
        print("Workflow completed:", result)
        break
    elif job_status == "failed":
        print("Workflow failed:", status_response.json()["error"])
        break
    
    time.sleep(5)
```

---

### 8. App Builder Query

**Use Case**: Query a custom-built Hatz AI app.

```python
import requests
import os

API_KEY = os.environ.get("HATZ_API_KEY")

response = requests.post(
    "https://ai.hatz.ai/v1/app/customer-support-app/query",
    headers={
        "X-API-Key": API_KEY,
        "Content-Type": "application/json"
    },
    json={
        "query": "How do I reset my password?",
        "context": {
            "user_id": "user-123",
            "account_type": "premium"
        }
    }
)

print(response.json()["response"])
```

---

## Error Handling Pattern

```python
import requests
import os

API_KEY = os.environ.get("HATZ_API_KEY")

def safe_chat(message, model="gpt-4o"):
    try:
        response = requests.post(
            "https://ai.hatz.ai/v1/chat/completions",
            headers={
                "X-API-Key": API_KEY,
                "Content-Type": "application/json"
            },
            json={
                "messages": [{"role": "user", "content": message}],
                "model": model
            },
            timeout=30
        )
        
        if response.status_code == 401:
            return "Error: Invalid API key"
        elif response.status_code == 429:
            return "Error: Rate limited - check credit usage"
        elif response.status_code >= 400:
            return f"Error: {response.json().get('error', {}).get('message', 'Unknown error')}"
        
        return response.json()["choices"][0]["message"]["content"]
        
    except requests.exceptions.Timeout:
        return "Error: Request timed out"
    except requests.exceptions.RequestException as e:
        return f"Error: {str(e)}"

# Usage
result = safe_chat("Hello, AI!")
print(result)
```

---

## Best Practices

1. **Environment Variables**: Store API keys in environment variables, never hardcode
2. **Timeouts**: Always set request timeouts to avoid hanging
3. **Retry Logic**: Implement retry for transient errors (5xx responses)
4. **Streaming**: Use streaming for long responses to improve UX
5. **History Management**: Keep conversation history reasonable (truncate old messages)
6. **File Cleanup**: Delete processed files when no longer needed
7. **Credit Monitoring**: Monitor usage via Hatz dashboard
