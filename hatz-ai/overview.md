# Hatz AI - Platform Overview

## What is Hatz AI?

Hatz AI is an AI/ML platform that provides:
- **Chat Completions** - OpenAI-compatible LLM API
- **AI Agents** - Configurable AI assistants with tools
- **Workflows** - Multi-step AI pipelines
- **App Builder** - Build custom AI applications
- **File Processing** - Upload and process documents
- **Code Execution** - Run Python/Shell in sandbox

## Base URL

```
https://ai.hatz.ai
```

## Authentication

### API Key Authentication

Two methods for authentication:

#### Method 1: X-API-Key Header (Recommended)
```bash
curl https://ai.hatz.ai/v1/chat/completions \
  -H "X-API-Key: YOUR_API_KEY" \
  -H "Content-Type: application/json"
```

#### Method 2: Bearer Token
```bash
curl https://ai.hatz.ai/v1/chat/completions \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json"
```

### Getting an API Key

1. Access the Hatz Admin Dashboard
2. Navigate to Settings tab
3. Generate API Key
4. Contact help@hatz.ai for assistance

## API Versioning

All endpoints use `/v1/` prefix:
```
https://ai.hatz.ai/v1/chat/completions
https://ai.hatz.ai/v1/workflows/run
https://ai.hatz.ai/v1/app/list
```

## Rate Limits

- Managed via credit system
- 429 response indicates credit overage
- Check credit usage: https://www.hatz.ai/credit-usage-faq

## OpenAI Compatibility

Hatz AI is compatible with OpenAI SDK and OpenCode:

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
```

## Platform Documentation

- **API Documentation**: https://api-docs.hatz.ai/
- **API Reference**: https://api-docs.hatz.ai/reference
- **Main Documentation**: https://docs.hatz.ai/
