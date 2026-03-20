# James-API-KB

> Knowledge Base for an **API Expert Agent** designed to help vibe coders build API-driven integrations.

---

## Overview

This Knowledge Base enables an AI agent to provide accurate, actionable guidance for API integrations across three platforms:

| Platform | Description |
|----------|-------------|
| **[Hatz AI](./hatz-ai/overview.md)** | AI/ML platform with chat completions, agents, workflows, and app builder |
| **[Go High Level](./go-high-level/overview.md)** | Marketing automation CRM with contacts, conversations, opportunities, and webhooks |
| **[n8n](./n8n/overview.md)** | Workflow automation platform with visual node-based programming |

---

## Knowledge Base Structure

```
James-API-KB/
├── README.md                              # This file - master index
│
├── hatz-ai/
│   ├── overview.md                        # Platform overview, base URL, authentication
│   ├── endpoints.md                       # All API endpoints with request/response examples
│   ├── tools.md                           # Available AI tools (search, code execution, etc.)
│   └── patterns.md                        # Integration patterns and code snippets
│
├── go-high-level/
│   ├── overview.md                        # Platform overview, OAuth, rate limits
│   ├── endpoints.md                       # Core API endpoints (contacts, opportunities, etc.)
│   ├── webhooks.md                        # 50+ webhook events and handler setup
│   └── patterns.md                        # Integration patterns and code snippets
│
├── n8n/
│   ├── overview.md                        # Platform overview, API keys, cloud vs self-hosted
│   ├── endpoints.md                       # API endpoints (workflows, executions, users)
│   └── patterns.md                        # Client code and workflow automation patterns
│
├── cross-platform/
│   ├── integration-patterns.md            # Multi-platform integration flows
│   └── troubleshooting.md                 # Error handling, debugging, and retry logic
│
└── quick-reference/
    └── cheat-sheet.md                     # Quick lookup tables for fast reference
```

---

## Quick Navigation

### By Platform

**Hatz AI**
- [Overview & Authentication](./hatz-ai/overview.md)
- [API Endpoints](./hatz-ai/endpoints.md)
- [AI Tools Reference](./hatz-ai/tools.md)
- [Integration Patterns](./hatz-ai/patterns.md)

**Go High Level**
- [Overview & Authentication](./go-high-level/overview.md)
- [API Endpoints](./go-high-level/endpoints.md)
- [Webhooks Reference](./go-high-level/webhooks.md)
- [Integration Patterns](./go-high-level/patterns.md)

**n8n**
- [Overview & Authentication](./n8n/overview.md)
- [API Endpoints](./n8n/endpoints.md)
- [Integration Patterns](./n8n/patterns.md)

### Cross-Platform
- [Integration Patterns](./cross-platform/integration-patterns.md)
- [Troubleshooting Guide](./cross-platform/troubleshooting.md)

### Quick Reference
- [Cheat Sheet](./quick-reference/cheat-sheet.md)

---

## How to Use This KB

1. **Start with platform overviews** - Understand authentication, base URLs, and rate limits
2. **Reference endpoints** - Get specific API calls, parameters, and response formats
3. **Use patterns** - Copy-paste integration examples for common use cases
4. **Check troubleshooting** - Debug errors and understand platform-specific issues
5. **Quick reference** - Fast lookup tables for everyday operations

---

## Target Audience

- **Vibe coders** building API-first integrations
- Developers needing quick, accurate API guidance
- Integration specialists connecting multiple platforms

---

## Official Documentation Sources

| Platform | Documentation URL |
|----------|-------------------|
| Hatz AI | https://api-docs.hatz.ai/ |
| Go High Level | https://marketplace.gohighlevel.com/docs/ |
| n8n | https://docs.n8n.io/api/ |

---

## Last Updated

2026-03-20

---

## License

[Apache-2.0](./LICENSE)