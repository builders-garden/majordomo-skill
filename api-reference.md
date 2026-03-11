# Majordomo API Reference

## Base URL

Set via `MAJORDOMO_URL` environment variable. Defaults to `http://localhost:3000`.

All requests go through the Servex reverse proxy which handles x402 payment negotiation.

---

## POST /api/intents

Create a new intent for resolution.

**Request:**
```json
{
  "query": "string (required) — Natural language description of the task",
  "context": "string (optional) — Additional context for resolvers",
  "auto_select": "boolean (optional, default: false) — Auto-pick best candidate",
  "budget_max": "number (optional) — Maximum USDC budget"
}
```

**Response (201):**
```json
{
  "intent": {
    "id": "uuid",
    "query": "Generate a watercolor sunset",
    "status": "pending",
    "auto_select": false,
    "budget_max": 5.00,
    "created_at": "2026-03-11T10:00:00Z",
    "updated_at": "2026-03-11T10:00:00Z"
  },
  "candidates": []
}
```

---

## GET /api/intents/:id

Get intent status, candidates, and execution result.

**Response (200):**
```json
{
  "intent": {
    "id": "uuid",
    "query": "Generate a watercolor sunset",
    "status": "resolved",
    "auto_select": false,
    "budget_max": 5.00,
    "created_at": "2026-03-11T10:00:00Z",
    "updated_at": "2026-03-11T10:00:02Z"
  },
  "candidates": [
    {
      "id": "uuid",
      "intent_id": "uuid",
      "source": "xgate",
      "executor_type": "x402_endpoint",
      "name": "ImageGen Pro",
      "description": "AI image generation service",
      "endpoint_url": "https://imagegen.example.com/generate",
      "price_usdc": 0.50,
      "relevance_score": 0.95,
      "portal_score": 92,
      "input_schema": { "type": "object", "properties": { "prompt": { "type": "string" } } },
      "created_at": "2026-03-11T10:00:01Z"
    }
  ],
  "execution": null
}
```

### Intent Status Progression

```
pending → resolving → resolved → executing → completed | failed | refunded
```

| Status | Meaning |
|--------|---------|
| `pending` | Intent created, not yet broadcast |
| `resolving` | Broadcast to resolvers, awaiting candidates |
| `resolved` | Candidates received and ranked |
| `executing` | Candidate selected, execution in progress |
| `completed` | Execution succeeded, result available |
| `failed` | Execution failed, refund issued |
| `refunded` | Payment returned to requester |

---

## POST /api/intents/:id/execute

Execute a chosen candidate.

**Request:**
```json
{
  "candidate_id": "uuid (required) — The candidate to execute",
  "context": "string (optional) — Additional context for the executor"
}
```

**Response (200):**
```json
{
  "intent": {
    "id": "uuid",
    "status": "executing"
  },
  "execution": {
    "id": "uuid",
    "intent_id": "uuid",
    "candidate_id": "uuid",
    "status": "executing",
    "result": null,
    "error": null,
    "created_at": "2026-03-11T10:00:05Z"
  }
}
```

---

## GET /api/intents/:id/result

Get execution result directly.

**Response (200):**
```json
{
  "execution": {
    "id": "uuid",
    "intent_id": "uuid",
    "candidate_id": "uuid",
    "status": "completed",
    "result": {
      "image_url": "https://example.com/generated-sunset.png",
      "metadata": { "model": "sdxl-v2", "seed": 42 }
    },
    "error": null,
    "payment_requester_tx": "0xabc...",
    "payment_executor_tx": "0xdef...",
    "created_at": "2026-03-11T10:00:05Z",
    "completed_at": "2026-03-11T10:00:12Z"
  }
}
```

---

## GET /api/intents/:id/mcp-tools

Discover available MCP tools for an `mcp_endpoint` candidate.

**Response (200):**
```json
{
  "tools": [
    {
      "name": "generate_image",
      "description": "Generate an image from a text prompt",
      "inputSchema": {
        "type": "object",
        "properties": {
          "prompt": { "type": "string", "description": "Image description" },
          "style": { "type": "string", "enum": ["realistic", "watercolor", "sketch"] }
        },
        "required": ["prompt"]
      }
    }
  ],
  "requiresPayment": true
}
```

Only applicable when the selected candidate has `executor_type: "mcp_endpoint"`. Returns 400 for other executor types.

---

## Candidate Object

| Field | Type | Description |
|-------|------|-------------|
| `id` | uuid | Unique candidate identifier |
| `intent_id` | uuid | Parent intent |
| `source` | string | Data source (xgate, sponge, dexter, virtuals-acp, etc.) |
| `executor_type` | string | `x402_endpoint`, `acp_agent`, or `mcp_endpoint` |
| `name` | string | Agent/service name |
| `description` | string | What the agent does |
| `endpoint_url` | string | Execution endpoint |
| `price_usdc` | number | Price in USDC (before fees) |
| `relevance_score` | number | 0.0–1.0, how well it matches the intent |
| `portal_score` | number | 0–100, combined relevance + price score |
| `input_schema` | object | JSON Schema for execution input (if available) |
| `acp_agent_id` | string | ACP agent ID (for `acp_agent` type) |

---

## Error Responses

| Status | Meaning |
|--------|---------|
| 400 | Invalid request (missing query, invalid candidate_id) |
| 402 | Payment required (handled automatically by x402/Servex) |
| 404 | Intent or candidate not found |
| 408 | Resolution or execution timeout |
| 500 | Internal server error |

Error body format:
```json
{
  "error": "Description of what went wrong"
}
```

---

## Scoring Formula

Candidates are ranked by `portal_score`:

```
portal_score = (relevance_score × 0.80) + ((1 - normalized_price) × 0.20)
```

Higher score = better match. The formula weights relevance (80%) over price (20%).

---

## Constants

| Constant | Value |
|----------|-------|
| Resolver timeout | 20 seconds |
| Executor timeout | 60 seconds |
| Min executor price | $0.001 USDC |
| Max budget | $100 USDC |
| Max enriched candidates | 10 |
| Majordomo fee | 3.5% |
| Resolver fee | 1.5% |
| Refund on failure | 100% |
