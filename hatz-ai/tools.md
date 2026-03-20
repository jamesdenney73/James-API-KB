# Hatz AI - Available Tools

## Overview

Hatz AI agents can use built-in tools to extend their capabilities. Tools are enabled via `tools_to_use` parameter or `auto_tool_selection` in the completions API.

## Web Search & Scraping Tools

### google_search
**Description**: Search Google via Custom Search API.

**Use Case**: Finding current information, news, websites, documentation.

**Example Request**:
```json
{
  "messages": [{"role": "user", "content": "What are the latest features in Python 3.12?"}],
  "model": "gpt-4o",
  "tools_to_use": ["google_search"]
}
```

---

### firecrawl_search
**Description**: Search the web using Firecrawl.

**Use Case**: Web search with structured results.

---

### firecrawl_scrape
**Description**: Scrape webpage content and return clean markdown.

**Use Case**: Extract content from specific URLs.

**Example**:
```json
{
  "messages": [
    {"role": "user", "content": "Scrape https://docs.python.org/3/whatsnew/3.12.html and summarize"}
  ],
  "model": "gpt-4o",
  "tools_to_use": ["firecrawl_scrape"]
}
```

---

### tavily_search
**Description**: Web search via Tavily API.

**Use Case**: Real-time web search with detailed results.

---

## Code Execution Tools

### daytona_code_execution
**Description**: Execute Python scripts and shell commands in a sandboxed environment.

**Capabilities**:
- Run Python code with full standard library
- Execute shell commands
- Create and manipulate files
- Install and use preinstalled packages

**Preinstalled Packages**:
```
pandas, numpy, matplotlib, scipy, scikit-learn
openpyxl, xlrd, XlsxWriter
pillow, python-docx, python-pptx
lxml, defusedxml, reportlab, fpdf2
pypdf, pdfplumber, pypdfium2, pdf2image
pytesseract, markitdown
requests, six
```

**System Tools**: LibreOffice, Pandoc, Poppler, qpdf, tesseract, nodejs, npm

**Use Case**: Data processing, file generation, calculations, API calls from code.

**Example**:
```json
{
  "messages": [
    {"role": "user", "content": "Write a Python script to generate a CSV with 100 random records"}
  ],
  "model": "agent-code-executor",
  "tools_to_use": ["daytona_code_execution"]
}
```

---

### execute_python_code
**Description**: Execute Python code directly.

**Use Case**: Quick Python calculations, data transformations.

---

## Location & Maps Tools

### google_maps_search_places
**Description**: Search for places using Google Maps API.

**Use Case**: Find businesses, locations, points of interest.

---

### google_maps_get_directions
**Description**: Get directions between locations.

**Use Case**: Route planning, travel estimates.

---

### google_maps_get_distance_matrix
**Description**: Calculate distances and times between multiple locations.

**Use Case**: Logistics, delivery planning, commute analysis.

---

## Weather Tools

### google_weather_current
**Description**: Get current weather for a location.

**Use Case**: Real-time weather data.

---

### google_weather_forecast
**Description**: Get weather forecast for a location.

**Use Case**: Planning, outdoor activities.

---

## Using Tools in API Calls

### Explicit Tool Selection
```bash
curl "https://ai.hatz.ai/v1/chat/completions" \
  -H "X-API-Key: $HATZ_API_KEY" \
  -H "Content-Type: application/json" \
  -d \'{
    "messages": [
      {"role": "user", "content": "Search for the latest news about AI"}
    ],
    "model": "gpt-4o",
    "tools_to_use": ["google_search"]
  }\'
```

### Automatic Tool Selection
```bash
curl "https://ai.hatz.ai/v1/chat/completions" \
  -H "X-API-Key: $HATZ_API_KEY" \
  -H "Content-Type: application/json" \
  -d \'{
    "messages": [
      {"role": "user", "content": "What is the weather in Chicago and find nearby restaurants"}
    ],
    "model": "gpt-4o",
    "auto_tool_selection": true
  }\'
```

### Multiple Tools
```bash
curl "https://ai.hatz.ai/v1/chat/completions" \
  -H "X-API-Key: $HATZ_API_KEY" \
  -H "Content-Type: application/json" \
  -d \'{
    "messages": [
      {"role": "user", "content": "Search for a webpage and scrape its content"}
    ],
    "model": "gpt-4o",
    "tools_to_use": ["google_search", "firecrawl_scrape"]
  }\'
```

---

## Tool Results in Response

When tools are used, the response includes tool calls:

```json
{
  "choices": [
    {
      "message": {
        "role": "assistant",
        "content": null,
        "tool_calls": [
          {
            "id": "call_abc123",
            "type": "function",
            "function": {
              "name": "google_search",
              "arguments": "{\"query\": \"latest AI news\"}"
            }
          }
        ]
      }
    }
  ]
}
```

---

## Best Practices

1. **Use auto_tool_selection** for general queries where the best tool isn\'t obvious
2. **Specify tools_to_use** when you want precise control over tool usage
3. **File processing** requires uploading files first, then referencing in `file_uuids`
4. **Code execution** is sandboxed - files must be saved to output directory
5. **Rate limits** apply to tool usage - monitor credit consumption
