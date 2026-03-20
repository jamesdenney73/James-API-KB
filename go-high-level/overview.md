# Go High Level - Platform Overview

## What is Go High Level?

Go High Level (GHL) is a marketing automation and CRM platform that provides:
- **Contact Management** - Store and manage contacts with custom fields
- **Conversations** - SMS, email, and phone communications
- **Calendars** - Appointment scheduling and booking
- **Opportunities** - Sales pipeline and deal tracking
- **Payments** - Invoicing and payment processing
- **Webhooks** - Real-time event notifications
- **Workflows** - Multi-automation triggers

## Base URL

```
https://services.leadconnectorhq.com
```

## Authentication

### Option 1: Private Integration Token

**Best For**:
- Internal integrations only
- Single sub-account access
- No webhooks or marketplace listing needed

**How to Get**:
1. Navigate to Settings → Integrations
2. Create Private Integration
3. Select sub-account
4. Copy the token

**Usage**:
```bash
curl https://services.leadconnectorhq.com/contacts/ \
  -H "Authorization: Bearer YOUR_PRIVATE_TOKEN" \
  -H "Version: 2021-07-28"
```

---

### Option 2: OAuth 2.0 Flow

**Best For**:
- Public Marketplace apps
- Multiple location/account access
- Webhooks required
- Third-party applications

**OAuth Flow**:
1. Direct user to authorization URL
2. User grants permissions
3. Receive callback with authorization code
4. Exchange code for access token

**Authorization URL**:
```
https://marketplace.gohighlevel.com/oauth/chooselocation?response_type=code&client_id=YOUR_CLIENT_ID&redirect_uri=YOUR_REDIRECT_URI&scope=contacts.readonly
```

**Token Exchange**:
```bash
curl -X POST https://services.leadconnectorhq.com/oauth/token \
  -d "client_id=YOUR_CLIENT_ID" \
  -d "client_secret=YOUR_CLIENT_SECRET" \
  -d "grant_type=authorization_code" \
  -d "code=AUTHORIZATION_CODE" \
  -d "redirect_uri=YOUR_REDIRECT_URI"
```

**Token Response**:
```json
{
  "access_token": "ACCESS_TOKEN",
  "token_type": "Bearer",
  "expires_in": 3600,
  "refresh_token": "REFRESH_TOKEN",
  "scope": "contacts.readonly",
  "userType": "Account",
  "locationId": "LOCATION_ID"
}
```

**Refresh Token**:
```bash
curl -X POST https://services.leadconnectorhq.com/oauth/token \
  -d "client_id=YOUR_CLIENT_ID" \
  -d "client_secret=YOUR_CLIENT_SECRET" \
  -d "grant_type=refresh_token" \
  -d "refresh_token=REFRESH_TOKEN"
```

---

### Required Headers

All API requests require:

| Header | Description |
|--------|-------------|
| `Authorization` | Bearer token: `Bearer YOUR_TOKEN` |
| `Version` | API version (e.g., `2021-07-28`) |

```bash
Authorization: Bearer YOUR_TOKEN
Version: 2021-07-28
```

---

## API Versioning

- **V1**: Deprecated, end-of-support
- **V2**: Current version with OAuth and enhanced features

Always include version header:
```
Version: 2021-07-28
```

---

## Rate Limits

### Burst Limit
**100 requests per 10 seconds** per resource

### Daily Limit
**200,000 requests per day** per resource

### Rate Limit Headers

| Header | Description |
|--------|-------------|
| `X-RateLimit-Limit-Daily` | Daily limit |
| `X-RateLimit-Daily-Remaining` | Remaining daily requests |
| `X-RateLimit-Interval-Milliseconds` | Interval for burst calculation |
| `X-RateLimit-Max` | Burst limit |
| `X-RateLimit-Remaining` | Remaining burst requests |

### Handling Rate Limits

```python
import time
import requests

def rate_limited_request(url, headers):
    response = requests.get(url, headers=headers)
    
    if response.status_code == 429:
        # Read retry-after header or wait
        time.sleep(10)
        return rate_limited_request(url, headers)
    
    return response
```

---

## Scopes

Common OAuth scopes:

| Scope | Permission |
|-------|------------|
| `contacts.readonly` | Read contacts |
| `contacts.write` | Create/update contacts |
| `opportunities.readonly` | Read opportunities |
| `opportunities.write` | Create/update opportunities |
| `conversations.readonly` | Read conversations |
| `conversations.write` | Send messages |
| `calendars.readonly` | Read calendars |
| `calendars.write` | Manage appointments |
| `payments.readonly` | Read payments |
| `payments.write` | Process payments |

---

## Platform Documentation

- **API Documentation**: https://marketplace.gohighlevel.com/docs/
- **Support Article**: https://help.gohighlevel.com/support/solutions/articles/48001060529-highlevel-api-documentation
