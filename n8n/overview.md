# n8n - Platform Overview

## What is n8n?

n8n is a workflow automation platform that provides:
- **Visual Workflow Builder** - Drag-and-drop node-based automation
- **400+ Integrations** - Connect to virtually any API
- **Self-Hosted or Cloud** - Deploy anywhere or use n8n Cloud
- **API Access** - Programmatic control of workflows and executions
- **AI Integration** - Built-in AI capabilities

## Base URL

### Self-Hosted
```
<N8N_HOST>:<N8N_PORT>/<N8N_PATH>/api/v1
```

Example: `http://localhost:5678/api/v1`

### n8n Cloud
```
https://{instance}.app.n8n.cloud/api/v1
```

Example: `https://my-workflows.app.n8n.cloud/api/v1`

---

## Authentication

### API Key Authentication

n8n uses API keys passed in the request header:

```
X-N8N-API-KEY: your-api-key
```

### Creating an API Key

1. Log in to your n8n instance
2. Navigate to **Settings** → **n8n API**
3. Click **Create an API key**
4. Set a label and expiration date
5. (Enterprise only) Select scopes for restricted access
6. Copy the key immediately (shown only once)

**Important**: API access is not available during the free trial. You must upgrade to a paid plan.

---

## API Scopes (Enterprise)

Enterprise instances can restrict API key access with scopes:

| Scope | Access |
|-------|--------|
| Default | All resources (non-enterprise) |
| Custom | Specific resources per key |

Non-enterprise instances have full access to all API features.

---

## Example Request

### List Workflows

```bash
# Self-hosted
curl -X GET \
  "http://localhost:5678/api/v1/workflows?active=true" \
  -H "accept: application/json" \
  -H "X-N8N-API-KEY: your-api-key"

# n8n Cloud
curl -X GET \
  "https://your-instance.app.n8n.cloud/api/v1/workflows?active=true" \
  -H "accept: application/json" \
  -H "X-N8N-API-KEY: your-api-key"
```

---

## API Playground

Self-hosted n8n instances include a built-in API playground for testing endpoints.

Access at: `http://localhost:5678/api-playground`

---

## Using n8n API Node

n8n provides a dedicated **n8n API node** for calling the n8n API from within workflows.

### Use Cases:
- Trigger workflows from other workflows
- Get execution history
- Manage users programmatically

---

## Platform Documentation

- **API Overview**: https://docs.n8n.io/api/
- **API Reference**: https://docs.n8n.io/api/api-reference/
- **Authentication**: https://docs.n8n.io/api/authentication/
- **Main Docs**: https://docs.n8n.io/

---

## Capacity Limits

| Resource | Limit |
|----------|-------|
| Workflows | Unlimited |
| Workflow size | 5MB |
| Nodes per workflow | Unlimited |
| Execution history | Plan-dependent |

---

## Common Use Cases

1. **Trigger workflows programmatically** from external applications
2. **Monitor execution status** and handle failures
3. **Manage users and permissions** (Enterprise)
4. **Build custom dashboards** showing workflow metrics
5. **Integrate n8n with other tools** like Hatz AI and Go High Level
