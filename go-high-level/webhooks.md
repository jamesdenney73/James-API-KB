# Go High Level - Webhooks

## Overview

Go High Level provides 50+ webhook events for real-time notifications. Webhooks are configured through the OAuth flow or marketplace app installation.

## Webhook Configuration

### Setup via OAuth

Include webhook-related scopes and provide webhook URLs during app setup.

### Verify Webhook Signature

GHL sends webhooks with a signature header for verification:

```python
import hmac
import hashlib

def verify_webhook(payload, signature, secret):
    expected = hmac.new(
        secret.encode(),
        payload.encode(),
        hashlib.sha256
    ).hexdigest()
    return hmac.compare_digest(f"sha256={expected}", signature)
```

---

## Contact Webhooks

| Event Type | Trigger |
|------------|---------|
| `ContactCreate` | New contact created |
| `ContactUpdate` | Contact updated |
| `ContactDelete` | Contact deleted |
| `ContactTagUpdate` | Contact tags changed |
| `ContactDndUpdate` | DND status changed |

### ContactCreate Payload Example

```json
{
  "type": "ContactCreate",
  "locationId": "abc123",
  "contactId": "contact-456",
  "data": {
    "id": "contact-456",
    "firstName": "John",
    "lastName": "Doe",
    "email": "john@example.com",
    "phone": "+1234567890",
    "source": "website",
    "tags": ["lead"],
    "dateAdded": "2026-03-20T10:00:00Z"
  }
}
```

---

## Opportunity Webhooks

| Event Type | Trigger |
|------------|---------|
| `OpportunityCreate` | New opportunity created |
| `OpportunityUpdate` | Opportunity updated |
| `OpportunityDelete` | Opportunity deleted |
| `OpportunityStatusUpdate` | Status changed (open/won/lost) |
| `OpportunityPipelineUpdate` | Pipeline or stage changed |

### OpportunityCreate Payload Example

```json
{
  "type": "OpportunityCreate",
  "locationId": "abc123",
  "opportunityId": "opp-789",
  "data": {
    "id": "opp-789",
    "contactId": "contact-456",
    "name": "Big Deal",
    "pipelineId": "pipeline-123",
    "pipelineStageId": "stage-456",
    "monetaryValue": 5000,
    "status": "open"
  }
}
```

---

## Appointment Webhooks

| Event Type | Trigger |
|------------|---------|
| `AppointmentCreate` | New appointment booked |
| `AppointmentUpdate` | Appointment updated |
| `AppointmentDelete` | Appointment cancelled |
| `AppointmentStatusUpdate` | Status changed |

---

## Message Webhooks

| Event Type | Trigger |
|------------|---------|
| `InboundMessage` | Contact sends message (SMS/Email) |
| `OutboundMessage` | User sends message |
| `ConversationStatusUpdate` | Conversation status changed |

### InboundMessage Payload Example

```json
{
  "type": "InboundMessage",
  "locationId": "abc123",
  "contactId": "contact-456",
  "conversationId": "conv-123",
  "data": {
    "message": "I want to schedule a call",
    "type": "SMS",
    "from": "+1234567890",
    "timestamp": "2026-03-20T10:00:00Z"
  }
}
```

---

## Invoice Webhooks

| Event Type | Trigger |
|------------|---------|
| `InvoiceCreate` | Invoice created |
| `InvoiceUpdate` | Invoice updated |
| `InvoiceDelete` | Invoice deleted |
| `InvoicePaid` | Invoice paid |
| `InvoicePaymentFailed` | Payment failed |

### InvoicePaid Payload Example

```json
{
  "type": "InvoicePaid",
  "locationId": "abc123",
  "invoiceId": "inv-123",
  "data": {
    "id": "inv-123",
    "contactId": "contact-456",
    "amount": 500,
    "paidAt": "2026-03-20T10:00:00Z",
    "paymentMethod": "card"
  }
}
```

---

## Task Webhooks

| Event Type | Trigger |
|------------|---------|
| `TaskCreate` | Task created |
| `TaskUpdate` | Task updated |
| `TaskDelete` | Task deleted |
| `TaskComplete` | Task marked complete |

---

## User Webhooks

| Event Type | Trigger |
|------------|---------|
| `UserCreate` | New user added |
| `UserUpdate` | User updated |
| `UserDelete` | User removed |

---

## Form Submission Webhooks

| Event Type | Trigger |
|------------|---------|
| `FormSubmission` | Form submitted |

### FormSubmission Payload Example

```json
{
  "type": "FormSubmission",
  "locationId": "abc123",
  "formId": "form-123",
  "data": {
    "contactId": "contact-456",
    "formName": "Contact Form",
    "fields": {
      "name": "John Doe",
      "email": "john@example.com",
      "message": "I have a question"
    },
    "submittedAt": "2026-03-20T10:00:00Z"
  }
}
```

---

## Workflow Webhooks

| Event Type | Trigger |
|------------|---------|
| `WorkflowStart` | Workflow triggered |
| `WorkflowComplete` | Workflow finished |
| `WorkflowError` | Workflow failed |

---

## Complete Webhook Event List

### Contact Events
- `ContactCreate`
- `ContactUpdate`
- `ContactDelete`
- `ContactTagUpdate`
- `ContactDndUpdate`

### Opportunity Events
- `OpportunityCreate`
- `OpportunityUpdate`
- `OpportunityDelete`
- `OpportunityStatusUpdate`
- `OpportunityPipelineUpdate`

### Appointment Events
- `AppointmentCreate`
- `AppointmentUpdate`
- `AppointmentDelete`
- `AppointmentStatusUpdate`

### Message Events
- `InboundMessage`
- `OutboundMessage`
- `ConversationStatusUpdate`

### Invoice Events
- `InvoiceCreate`
- `InvoiceUpdate`
- `InvoiceDelete`
- `InvoicePaid`
- `InvoicePaymentFailed`

### Task Events
- `TaskCreate`
- `TaskUpdate`
- `TaskDelete`
- `TaskComplete`

### User Events
- `UserCreate`
- `UserUpdate`
- `UserDelete`

### Form Events
- `FormSubmission`

### Workflow Events
- `WorkflowStart`
- `WorkflowComplete`
- `WorkflowError`

---

## Webhook Handler Pattern

```python
from flask import Flask, request, jsonify
import os

app = Flask(__name__)
WEBHOOK_SECRET = os.environ.get("GHL_WEBHOOK_SECRET")

@app.route("/webhook/ghl", methods=["POST"])
def handle_ghl_webhook():
    payload = request.json
    event_type = payload.get("type")
    
    # Verify signature (implement verify_webhook function)
    # signature = request.headers.get("X-GHL-Signature")
    # if not verify_webhook(request.data, signature, WEBHOOK_SECRET):
    #     return jsonify({"error": "Invalid signature"}), 401
    
    handlers = {
        "ContactCreate": handle_contact_create,
        "ContactUpdate": handle_contact_update,
        "OpportunityCreate": handle_opportunity_create,
        "InvoicePaid": handle_invoice_paid,
        "InboundMessage": handle_inbound_message,
    }
    
    handler = handlers.get(event_type)
    if handler:
        return handler(payload)
    
    return jsonify({"status": "unhandled", "type": event_type}), 200

def handle_contact_create(payload):
    contact = payload["data"]
    # Process new contact
    print(f"New contact: {contact[\"email\"]}")
    return jsonify({"status": "processed"}), 200

def handle_invoice_paid(payload):
    invoice = payload["data"]
    # Process paid invoice
    print(f"Invoice paid: {invoice[\"id\"]}")
    return jsonify({"status": "processed"}), 200

if __name__ == "__main__":
    app.run(port=3000)
```

---

## Best Practices

1. **Verify signatures** to ensure webhooks are from GHL
2. **Return 200 quickly** - process webhooks asynchronously
3. **Handle duplicates** - webhooks may be sent multiple times
4. **Log all events** for debugging and audit trails
5. **Use queue system** for high-volume webhook processing
