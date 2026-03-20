# Go High Level - API Endpoints

## Contacts API

### Search Contacts (Recommended)

**Endpoint**: `POST /contacts/search`

**Description**: Search contacts with filtering. Use this instead of deprecated V1 Get Contacts.

**Headers**:
```
Authorization: Bearer YOUR_TOKEN
Version: 2021-07-28
Content-Type: application/json
```

**Request Body**:
```json
{
  "locationId": "REQUIRED_LOCATION_ID",
  "query": "search term",
  "filters": [
    {
      "field": "email",
      "operator": "eq",
      "value": "user@example.com"
    }
  ],
  "page": 0,
  "pageSize": 25
}
```

**Example**:
```bash
curl -X POST "https://services.leadconnectorhq.com/contacts/search" \
  -H "Authorization: Bearer $GHL_TOKEN" \
  -H "Version: 2021-07-28" \
  -H "Content-Type: application/json" \
  -d \'{
    "locationId": "abc123",
    "query": "john",
    "page": 0,
    "pageSize": 25
  }\'
```

---

### Get Contact by ID

**Endpoint**: `GET /contacts/{contactId}`

**Example**:
```bash
curl "https://services.leadconnectorhq.com/contacts/ocQHyuzHvysMo5N5VsXc" \
  -H "Authorization: Bearer $GHL_TOKEN" \
  -H "Version: 2021-07-28"
```

---

### Create Contact

**Endpoint**: `POST /contacts/`

**Request Body**:
```json
{
  "locationId": "LOCATION_ID",
  "firstName": "John",
  "lastName": "Doe",
  "email": "john@example.com",
  "phone": "+1234567890",
  "source": "api",
  "tags": ["lead", "website"],
  "customFields": [
    {
      "id": "custom_field_id",
      "value": "custom value"
    }
  ]
}
```

**Example**:
```bash
curl -X POST "https://services.leadconnectorhq.com/contacts/" \
  -H "Authorization: Bearer $GHL_TOKEN" \
  -H "Version: 2021-07-28" \
  -H "Content-Type: application/json" \
  -d \'{
    "locationId": "abc123",
    "firstName": "Jane",
    "lastName": "Smith",
    "email": "jane@example.com",
    "phone": "+1555123456"
  }\'
```

---

### Update Contact

**Endpoint**: `PUT /contacts/{contactId}`

**Request Body**:
```json
{
  "firstName": "Updated Name",
  "tags": ["new-tag"],
  "customFields": [
    {"id": "field_id", "value": "updated value"}
  ]
}
```

**Example**:
```bash
curl -X PUT "https://services.leadconnectorhq.com/contacts/ocQHyuzHvysMo5N5VsXc" \
  -H "Authorization: Bearer $GHL_TOKEN" \
  -H "Version: 2021-07-28" \
  -H "Content-Type: application/json" \
  -d \'{"firstName": "Updated John"}\'
```

---

### Delete Contact

**Endpoint**: `DELETE /contacts/{contactId}`

**Example**:
```bash
curl -X DELETE "https://services.leadconnectorhq.com/contacts/ocQHyuzHvysMo5N5VsXc" \
  -H "Authorization: Bearer $GHL_TOKEN" \
  -H "Version: 2021-07-28"
```

---

### Contact Object Structure

```json
{
  "id": "ocQHyuzHvysMo5N5VsXc",
  "locationId": "C2QujeCh8ZnC7al2InWR",
  "firstName": "John",
  "lastName": "Doe",
  "email": "john@example.com",
  "phone": "+1234567890",
  "timezone": "America/Chicago",
  "country": "US",
  "source": "website",
  "dateAdded": "2026-03-20T10:00:00.255Z",
  "dateUpdated": "2026-03-20T12:00:00.000Z",
  "customFields": [
    {"id": "company", "value": "Acme Corp"}
  ],
  "tags": ["lead", "website"],
  "businessId": "business-123"
}
```

---

## Opportunities API

### Get Opportunities

**Endpoint**: `GET /opportunities/`

**Query Parameters**:
- `locationId` (required)
- `status` (optional): open, won, lost, abandoned
- `contactId` (optional)

**Example**:
```bash
curl "https://services.leadconnectorhq.com/opportunities/?locationId=abc123&status=open" \
  -H "Authorization: Bearer $GHL_TOKEN" \
  -H "Version: 2021-07-28"
```

---

### Create Opportunity

**Endpoint**: `POST /opportunities/`

**Request Body**:
```json
{
  "locationId": "LOCATION_ID",
  "contactId": "CONTACT_ID",
  "name": "Deal Name",
  "pipelineId": "PIPELINE_ID",
  "pipelineStageId": "STAGE_ID",
  "monetaryValue": 1000,
  "status": "open"
}
```

---

### Update Opportunity

**Endpoint**: `PUT /opportunities/{opportunityId}`

**Request Body**:
```json
{
  "status": "won",
  "pipelineStageId": "new_stage_id"
}
```

---

## Conversations API

### Get Conversations

**Endpoint**: `GET /conversations/`

**Query Parameters**:
- `locationId` (required)
- `contactId` (optional)
- `limit`, `offset` (pagination)

**Example**:
```bash
curl "https://services.leadconnectorhq.com/conversations/?locationId=abc123" \
  -H "Authorization: Bearer $GHL_TOKEN" \
  -H "Version: 2021-07-28"
```

---

### Send Message

**Endpoint**: `POST /conversations/messages`

**Request Body**:
```json
{
  "locationId": "LOCATION_ID",
  "contactId": "CONTACT_ID",
  "type": "SMS",
  "message": "Hello from the API!"
}
```

**Message Types**: SMS, Email

**Example**:
```bash
curl -X POST "https://services.leadconnectorhq.com/conversations/messages" \
  -H "Authorization: Bearer $GHL_TOKEN" \
  -H "Version: 2021-07-28" \
  -H "Content-Type: application/json" \
  -d \'{
    "locationId": "abc123",
    "contactId": "contact-123",
    "type": "SMS",
    "message": "Your appointment is confirmed!"
  }\'
```

---

## Calendars API

### Get Calendars

**Endpoint**: `GET /calendars/`

**Example**:
```bash
curl "https://services.leadconnectorhq.com/calendars/?locationId=abc123" \
  -H "Authorization: Bearer $GHL_TOKEN" \
  -H "Version: 2021-07-28"
```

---

### Get Events

**Endpoint**: `GET /calendars/events`

**Query Parameters**:
- `locationId` (required)
- `calendarId` (optional)
- `startTime`, `endTime` (ISO dates)

---

### Create Appointment

**Endpoint**: `POST /calendars/events/appointments`

**Request Body**:
```json
{
  "locationId": "LOCATION_ID",
  "calendarId": "CALENDAR_ID",
  "contactId": "CONTACT_ID",
  "startTime": "2026-03-25T10:00:00Z",
  "endTime": "2026-03-25T11:00:00Z",
  "title": "Consultation Call",
  "notes": "Initial consultation"
}
```

---

## Payments API

### List Invoices

**Endpoint**: `GET /invoices/`

**Example**:
```bash
curl "https://services.leadconnectorhq.com/invoices/?locationId=abc123" \
  -H "Authorization: Bearer $GHL_TOKEN" \
  -H "Version: 2021-07-28"
```

---

### Create Invoice

**Endpoint**: `POST /invoices/`

**Request Body**:
```json
{
  "locationId": "LOCATION_ID",
  "contactId": "CONTACT_ID",
  "items": [
    {
      "name": "Service",
      "amount": 100
    }
  ],
  "dueDate": "2026-04-01"
}
```

---

## Custom Fields API

### Get Custom Fields

**Endpoint**: `GET /locations/{locationId}/customFields`

**Example**:
```bash
curl "https://services.leadconnectorhq.com/locations/abc123/customFields" \
  -H "Authorization: Bearer $GHL_TOKEN" \
  -H "Version: 2021-07-28"
```

---

## Error Responses

| Status Code | Description |
|-------------|-------------|
| 400 | Bad Request - invalid parameters |
| 401 | Unauthorized - invalid token |
| 403 | Forbidden - insufficient scope |
| 404 | Not Found |
| 422 | Unprocessable Entity - validation error |
| 429 | Rate Limited |

**Error Response Format**:
```json
{
  "error": "Invalid request",
  "message": "Missing required field: locationId"
}
```

---

## API Reference Links

- Full documentation: https://marketplace.gohighlevel.com/docs/
- API Reference: https://marketplace.gohighlevel.com/docs/api-reference
