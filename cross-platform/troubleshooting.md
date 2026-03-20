# Cross-Platform Troubleshooting Guide

## Common Errors by Platform

---

## Hatz AI Errors

### 401 Unauthorized

**Symptoms**:
- `{"error": {"message": "Invalid API key", "type": "invalid_request_error"}}`

**Causes**:
- Wrong API key
- API key expired
- Using wrong header format

**Solutions**:
```python
# Correct header format
headers = {
    "X-API-Key": "your-api-key",  # Not "Authorization: Bearer"
    "Content-Type": "application/json"
}
```

**Check**:
1. Verify API key in Hatz dashboard
2. Ensure no extra spaces in key
3. Check for typos in header name

---

### 429 Rate Limited

**Symptoms**:
- `{"error": {"message": "Rate limit exceeded"}}`

**Causes**:
- Credit overage
- Too many requests in short time

**Solutions**:
- Check credit usage: https://www.hatz.ai/credit-usage-faq
- Implement exponential backoff
- Consider upgrading plan

**Backoff Implementation**:
```python
import time
import random

def call_with_backoff(func, max_retries=5):
    for attempt in range(max_retries):
        response = func()
        if response.status_code != 429:
            return response
        
        wait_time = (2 ** attempt) + random.random()
        time.sleep(wait_time)
    
    raise Exception("Max retries exceeded")
```

---

### 400 Bad Request

**Symptoms**:
- `{"error": {"message": "Invalid request body"}}`

**Common Causes**:
- Missing required field `messages`
- Invalid `model` parameter
- Malformed JSON

**Debug**:
```python
import json

# Validate your payload before sending
payload = {
    "messages": [{"role": "user", "content": "test"}],
    "model": "gpt-4o"
}

print(json.dumps(payload, indent=2))  # Check structure
```

---

## Go High Level Errors

### 401 Unauthorized

**Symptoms**:
- `{"error": "Unauthorized"}`

**Causes**:
- Expired token
- Wrong token format
- Missing headers

**Solutions**:
```python
# Correct headers for GHL
headers = {
    "Authorization": "Bearer your-token",  # Must include "Bearer"
    "Version": "2021-07-28",
    "Content-Type": "application/json"
}
```

**For OAuth tokens**:
- Check if token expired (typically 1 hour)
- Use refresh token to get new access token

```python
def refresh_access_token(refresh_token, client_id, client_secret):
    response = requests.post(
        "https://services.leadconnectorhq.com/oauth/token",
        data={
            "client_id": client_id,
            "client_secret": client_secret,
            "grant_type": "refresh_token",
            "refresh_token": refresh_token
        }
    )
    return response.json()["access_token"]
```

---

### 422 Unprocessable Entity

**Symptoms**:
- `{"message": "Invalid request data"}`

**Common Causes**:
- Missing `locationId`
- Invalid phone number format
- Duplicate email (for create operations)

**Solutions**:
```python
# Ensure phone numbers are in E.164 format
def format_phone(phone):
    # Remove all non-numeric characters
    cleaned = "".join(filter(str.isdigit, phone))
    
    # Add country code if missing
    if not cleaned.startswith("1") and len(cleaned) == 10:
        cleaned = "1" + cleaned
    
    return f"+{cleaned}"

# Usage
formatted = format_phone("555-123-4567")  # +15551234567
```

---

### Rate Limit Errors (429)

**Symptoms**:
- `{"error": "Rate limit exceeded"}`

**GHL Limits**:
- 100 requests per 10 seconds (burst)
- 200,000 requests per day

**Monitor Headers**:
```python
def check_rate_limit(response):
    headers = response.headers
    return {
        "daily_limit": headers.get("X-RateLimit-Limit-Daily"),
        "daily_remaining": headers.get("X-RateLimit-Daily-Remaining"),
        "burst_max": headers.get("X-RateLimit-Max"),
        "burst_remaining": headers.get("X-RateLimit-Remaining")
    }
```

---

## n8n Errors

### 401 Invalid API Key

**Symptoms**:
- `{"message": "Invalid API key"}`

**Causes**:
- Wrong API key
- API key expired
- Using wrong instance URL

**Solutions**:
1. Verify API key in n8n Settings → n8n API
2. Check base URL matches your instance
3. Ensure key hasn\'t been revoked

---

### 403 Forbidden

**Symptoms**:
- `{"message": "Access denied"}`

**Causes**:
- API access not available on free trial
- Insufficient scope (Enterprise)

**Solutions**:
- Upgrade from free trial
- Check API key scopes in Enterprise

---

### Workflow Execution Errors

**Symptoms**:
- Workflow shows "Error" status
- Node has red error badge

**Debug in n8n**:
1. Click on the error node to see error details
2. Check execution history for full stack trace
3. Use "Execute Node" to test individual nodes

**Common Fixes**:
```javascript
// Add error handling with Try-Catch pattern
// Use "Continue On Fail" in node settings
// Add "If" node to check for valid responses
```

---

## Cross-Platform Debugging

### Request Logging

Log all API requests for debugging:

```python
import requests
import json
from datetime import datetime

def logged_request(method, url, **kwargs):
    timestamp = datetime.now().isoformat()
    
    print(f"\n[{timestamp}] {method} {url}")
    if "json" in kwargs:
        print(f"Request: {json.dumps(kwargs[\"json\"], indent=2)}")
    
    response = requests.request(method, url, **kwargs)
    
    print(f"Response: {response.status_code}")
    try:
        print(f"Body: {json.dumps(response.json(), indent=2)}")
    except:
        print(f"Body: {response.text}")
    
    return response
```

---

### Webhook Debugging

When webhooks aren\'t working:

**Checklist**:
1. [ ] Webhook URL is correct (test vs production)
2. [ ] Credentials are passed correctly
3. [ ] Server is publicly accessible (not localhost)
4. [ ] Response returns 200 status
5. [ ] JSON parsing is correct

**Test Webhook Locally**:
```bash
# Use ngrok for local testing
ngrok http 3000

# Webhook URL becomes
https://abc123.ngrok.io/webhook/endpoint
```

---

### Timeout Issues

**Symptoms**:
- Request hangs indefinitely
- Timeout errors

**Solutions**:
```python
# Always set timeouts
response = requests.get(url, timeout=30)  # 30 second timeout

# With retries
from requests.adapters import HTTPAdapter
from urllib3.util.retry import Retry

session = requests.Session()
retry = Retry(total=3, backoff_factor=1, status_forcelist=[500, 502, 503, 504])
adapter = HTTPAdapter(max_retries=retry)
session.mount("https://", adapter)
```

---

## Error Decision Matrix

| Error Code | Hatz AI | Go High Level | n8n | Action |
|------------|---------|---------------|-----|--------|
| 401 | Invalid API key | Invalid/expired token | Invalid API key | Check credentials |
| 403 | Feature restricted | Scope issue | Trial limit | Check permissions |
| 404 | Resource not found | Contact/record not found | Workflow not found | Verify IDs |
| 422 | Invalid payload | Validation error | Invalid data | Check request body |
| 429 | Credit overage | Rate limited | N/A | Implement backoff |
| 500 | Server error | Server error | Server error | Retry with backoff |
| 502/503 | Temporary outage | Temporary outage | Temporary outage | Wait and retry |

---

## Quick Diagnostics

### Test Hatz AI Connection
```bash
curl -s -o /dev/null -w "%{http_code}" \
  "https://ai.hatz.ai/v1/chat/models" \
  -H "X-API-Key: $HATZ_API_KEY"
# Expect: 200
```

### Test Go High Level Connection
```bash
curl -s -o /dev/null -w "%{http_code}" \
  "https://services.leadconnectorhq.com/contacts/?locationId=test" \
  -H "Authorization: Bearer $GHL_TOKEN" \
  -H "Version: 2021-07-28"
# Expect: 200 or 401 (if token invalid)
```

### Test n8n Connection
```bash
curl -s -o /dev/null -w "%{http_code}" \
  "https://your-instance.app.n8n.cloud/api/v1/workflows" \
  -H "X-N8N-API-KEY: $N8N_API_KEY"
# Expect: 200
```

---

## Getting Help

### Hatz AI
- Documentation: https://docs.hatz.ai/
- Credit FAQ: https://www.hatz.ai/credit-usage-faq
- Email: help@hatz.ai

### Go High Level
- API Docs: https://marketplace.gohighlevel.com/docs/
- Support: https://help.gohighlevel.com/

### n8n
- Docs: https://docs.n8n.io/
- API Docs: https://docs.n8n.io/api/
- Community: https://community.n8n.io/
