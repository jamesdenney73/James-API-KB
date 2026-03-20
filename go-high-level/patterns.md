# Go High Level - Integration Patterns

## Common Integration Scenarios

### 1. Create Contact with Custom Fields

```python
import requests
import os

GHL_TOKEN = os.environ.get("GHL_TOKEN")
LOCATION_ID = os.environ.get("GHL_LOCATION_ID")

def create_contact(email, first_name, last_name, phone=None, tags=None, custom_fields=None):
    url = "https://services.leadconnectorhq.com/contacts/"
    
    headers = {
        "Authorization": f"Bearer {GHL_TOKEN}",
        "Version": "2021-07-28",
        "Content-Type": "application/json"
    }
    
    payload = {
        "locationId": LOCATION_ID,
        "firstName": first_name,
        "lastName": last_name,
        "email": email,
        "source": "api"
    }
    
    if phone:
        payload["phone"] = phone
    if tags:
        payload["tags"] = tags
    if custom_fields:
        payload["customFields"] = custom_fields
    
    response = requests.post(url, headers=headers, json=payload)
    
    if response.status_code == 200:
        return response.json()
    else:
        raise Exception(f"Failed to create contact: {response.text}")

# Usage
contact = create_contact(
    email="jane@example.com",
    first_name="Jane",
    last_name="Smith",
    phone="+1555123456",
    tags=["lead", "website"],
    custom_fields=[
        {"id": "company_field_id", "value": "Acme Corp"}
    ]
)
print(f"Created contact: {contact[\"id\"]}")
```

---

### 2. Search and Update Contact

```python
import requests
import os

GHL_TOKEN = os.environ.get("GHL_TOKEN")
LOCATION_ID = os.environ.get("GHL_LOCATION_ID")

def find_contact_by_email(email):
    url = "https://services.leadconnectorhq.com/contacts/search"
    
    headers = {
        "Authorization": f"Bearer {GHL_TOKEN}",
        "Version": "2021-07-28",
        "Content-Type": "application/json"
    }
    
    payload = {
        "locationId": LOCATION_ID,
        "filters": [
            {
                "field": "email",
                "operator": "eq",
                "value": email
            }
        ]
    }
    
    response = requests.post(url, headers=headers, json=payload)
    data = response.json()
    
    if data.get("contacts"):
        return data["contacts"][0]
    return None

def update_contact_tags(contact_id, tags_to_add):
    url = f"https://services.leadconnectorhq.com/contacts/{contact_id}"
    
    headers = {
        "Authorization": f"Bearer {GHL_TOKEN}",
        "Version": "2021-07-28",
        "Content-Type": "application/json"
    }
    
    # Get current contact to preserve existing tags
    contact = requests.get(url, headers=headers).json()
    existing_tags = contact.get("tags", [])
    new_tags = list(set(existing_tags + tags_to_add))
    
    payload = {"tags": new_tags}
    
    response = requests.put(url, headers=headers, json=payload)
    return response.json()

# Usage
contact = find_contact_by_email("john@example.com")
if contact:
    update_contact_tags(contact["id"], ["customer", "2026"])
    print("Tags updated")
```

---

### 3. Send SMS Message

```python
import requests
import os

GHL_TOKEN = os.environ.get("GHL_TOKEN")
LOCATION_ID = os.environ.get("GHL_LOCATION_ID")

def send_sms(contact_id, message):
    url = "https://services.leadconnectorhq.com/conversations/messages"
    
    headers = {
        "Authorization": f"Bearer {GHL_TOKEN}",
        "Version": "2021-07-28",
        "Content-Type": "application/json"
    }
    
    payload = {
        "locationId": LOCATION_ID,
        "contactId": contact_id,
        "type": "SMS",
        "message": message
    }
    
    response = requests.post(url, headers=headers, json=payload)
    
    if response.status_code == 200:
        return response.json()
    else:
        raise Exception(f"Failed to send SMS: {response.text}")

# Usage
result = send_sms("contact-123", "Hi! Your appointment is confirmed for tomorrow at 2pm.")
print("SMS sent")
```

---

### 4. Create Opportunity in Pipeline

```python
import requests
import os

GHL_TOKEN = os.environ.get("GHL_TOKEN")
LOCATION_ID = os.environ.get("GHL_LOCATION_ID")

def create_opportunity(contact_id, name, value, pipeline_id, stage_id):
    url = "https://services.leadconnectorhq.com/opportunities/"
    
    headers = {
        "Authorization": f"Bearer {GHL_TOKEN}",
        "Version": "2021-07-28",
        "Content-Type": "application/json"
    }
    
    payload = {
        "locationId": LOCATION_ID,
        "contactId": contact_id,
        "name": name,
        "pipelineId": pipeline_id,
        "pipelineStageId": stage_id,
        "monetaryValue": value,
        "status": "open"
    }
    
    response = requests.post(url, headers=headers, json=payload)
    
    if response.status_code == 200:
        return response.json()
    else:
        raise Exception(f"Failed to create opportunity: {response.text}")

# Usage
opp = create_opportunity(
    contact_id="contact-123",
    name="Enterprise Deal - Acme Corp",
    value=15000,
    pipeline_id="pipeline-456",
    stage_id="stage-789"
)
print(f"Created opportunity: {opp[\"id\"]}")
```

---

### 5. Book Appointment

```python
import requests
import os
from datetime import datetime, timedelta

GHL_TOKEN = os.environ.get("GHL_TOKEN")
LOCATION_ID = os.environ.get("GHL_LOCATION_ID")

def book_appointment(contact_id, calendar_id, start_time, duration_minutes=60, title=None, notes=None):
    url = "https://services.leadconnectorhq.com/calendars/events/appointments"
    
    headers = {
        "Authorization": f"Bearer {GHL_TOKEN}",
        "Version": "2021-07-28",
        "Content-Type": "application/json"
    }
    
    end_time = start_time + timedelta(minutes=duration_minutes)
    
    payload = {
        "locationId": LOCATION_ID,
        "calendarId": calendar_id,
        "contactId": contact_id,
        "startTime": start_time.isoformat(),
        "endTime": end_time.isoformat()
    }
    
    if title:
        payload["title"] = title
    if notes:
        payload["notes"] = notes
    
    response = requests.post(url, headers=headers, json=payload)
    
    if response.status_code == 200:
        return response.json()
    else:
        raise Exception(f"Failed to book appointment: {response.text}")

# Usage
appointment_time = datetime(2026, 3, 25, 14, 0)  # March 25, 2026 at 2pm
result = book_appointment(
    contact_id="contact-123",
    calendar_id="calendar-456",
    start_time=appointment_time,
    duration_minutes=30,
    title="Initial Consultation",
    notes="Discussed during website inquiry"
)
```

---

### 6. Get Pipeline Data

```python
import requests
import os

GHL_TOKEN = os.environ.get("GHL_TOKEN")
LOCATION_ID = os.environ.get("GHL_LOCATION_ID")

def get_open_opportunities():
    url = f"https://services.leadconnectorhq.com/opportunities/"
    
    headers = {
        "Authorization": f"Bearer {GHL_TOKEN}",
        "Version": "2021-07-28"
    }
    
    params = {
        "locationId": LOCATION_ID,
        "status": "open"
    }
    
    response = requests.get(url, headers=headers, params=params)
    return response.json()

def get_pipeline_value():
    opportunities = get_open_opportunities()
    
    total_value = 0
    for opp in opportunities.get("opportunities", []):
        total_value += opp.get("monetaryValue", 0)
    
    return total_value

# Usage
pipeline_value = get_pipeline_value()
print(f"Open pipeline value: ${pipeline_value:,.2f}")
```

---

### 7. Rate Limit Handler

```python
import requests
import os
import time

GHL_TOKEN = os.environ.get("GHL_TOKEN")

class GHLClient:
    def __init__(self, token):
        self.token = token
        self.base_url = "https://services.leadconnectorhq.com"
        self.headers = {
            "Authorization": f"Bearer {token}",
            "Version": "2021-07-28"
        }
    
    def request(self, method, endpoint, **kwargs):
        url = f"{self.base_url}{endpoint}"
        
        response = requests.request(method, url, headers=self.headers, **kwargs)
        
        # Check rate limit headers
        remaining = int(response.headers.get("X-RateLimit-Remaining", 100))
        
        if response.status_code == 429:
            # Rate limited - wait and retry
            wait_time = 10 - (remaining * 0.1) if remaining == 0 else 10
            time.sleep(wait_time)
            return self.request(method, endpoint, **kwargs)
        
        if remaining < 20:
            # Approaching limit, slow down
            time.sleep(0.5)
        
        return response

# Usage
client = GHLClient(GHL_TOKEN)
response = client.request("GET", "/contacts/contact-123")
print(response.json())
```

---

### 8. Webhook Handler with Flask

```python
from flask import Flask, request, jsonify
import requests
import os

app = Flask(__name__)

# Store state for processed events (use Redis in production)
processed_events = set()

@app.route("/webhook/ghl", methods=["POST"])
def handle_webhook():
    payload = request.json
    event_type = payload.get("type")
    event_id = payload.get("id") or payload.get("contactId")
    
    # Prevent duplicate processing
    event_key = f"{event_type}:{event_id}"
    if event_key in processed_events:
        return jsonify({"status": "duplicate"}), 200
    
    processed_events.add(event_key)
    
    handlers = {
        "ContactCreate": on_contact_create,
        "InvoicePaid": on_invoice_paid,
        "OpportunityStatusUpdate": on_opportunity_update,
    }
    
    handler = handlers.get(event_type)
    if handler:
        handler(payload)
    
    return jsonify({"status": "received"}), 200

def on_contact_create(payload):
    contact = payload["data"]
    print(f"New contact created: {contact[\"email\"]}")
    # Add your logic here

def on_invoice_paid(payload):
    invoice = payload["data"]
    print(f"Invoice paid: {invoice[\"id\"]} - ${invoice[\"amount\"]}")

def on_opportunity_update(payload):
    opp = payload["data"]
    if opp.get("status") == "won":
        print(f"Deal won: {opp[\"name\"]} - ${opp[\"monetaryValue\"]}")

if __name__ == "__main__":
    app.run(port=3000)
```

---

## Best Practices

1. **Use locationId** - Required for all requests, store it securely
2. **Handle rate limits** - Monitor headers and implement backoff
3. **Use search endpoint** - V1 Get Contacts is deprecated
4. **Verify webhooks** - Check signatures for security
5. **Handle duplicates** - Webhooks may be sent multiple times
6. **Use environment variables** - Never hardcode tokens
7. **Log API calls** - For debugging and audit trails
