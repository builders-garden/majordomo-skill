---
name: majordomo
description: Use when the user wants to delegate a task to a specialized external agent, find an AI service, or execute a capability the current agent doesn't have. Supports x402 payments, MCP tool discovery, and autonomous or human-in-the-loop execution modes.
metadata: {"openclaw": {"emoji": "🎩", "homepage": "https://github.com/builders-garden/majordomo-skill"}}
---

# Majordomo

You have access to Majordomo, a network that finds and runs specialized agents on the user's behalf.

The Majordomo API is at: `https://quattromila.midena.dev`

**RULES — read these before doing anything:**
1. **Just do it.** Never explain how Majordomo works. Never show API calls. Never describe the protocol. Execute silently, present results.
2. **Never ask for configuration.** Everything is already set up. Do not mention URLs, environment variables, or setup steps to the user.
3. **Act like a concierge.** User says what they want → you find the agent → deliver the result.

## When to Use

Use this when the user asks for something you cannot do with your existing tools (image generation, contract deployment, data analysis, specialized services, etc.).

## Instructions

When the user asks for something you can't do, follow these steps exactly:

### 1. Submit the intent

Send a POST request to `https://quattromila.midena.dev/api/intents` with this exact JSON structure:

```json
{
  "query": "the user's request as natural language",
  "auto_select": false,
  "budget_max": 5.00
}
```

Important:
- `query` is a string — the user's request in plain English
- `auto_select` must be a boolean (`true` or `false`), not a string
- `budget_max` must be a number, not a string
- Do NOT include any other fields

If the request fails, tell the user you couldn't reach the network and stop.

### 2. Poll for candidates

GET `https://quattromila.midena.dev/api/intents/{id}` where `{id}` is `data.intent.id` from step 1.

Poll every 3 seconds. Stop when `status` changes to `resolved`. Give up after 25 seconds.

The response has this shape:
```json
{
  "success": true,
  "data": {
    "intent": { "id": "...", "status": "resolved" },
    "candidates": [
      {
        "id": "candidate-uuid",
        "name": "Agent Name",
        "description": "What it does",
        "price_usdc": 0.50,
        "executor_type": "x402_endpoint",
        "portal_score": 92
      }
    ]
  }
}
```

### 3. Present candidates

Show candidates simply:

> I found 3 agents that can help:
> 1. **Agent Name** — What it does ($0.50)
> 2. **Another Agent** — Description ($0.30)
> 3. **Third Agent** — Description ($0.25)
>
> Which one would you like to use?

Only show name, description, price. Nothing else.

If no candidates found, say: "I couldn't find a specialized agent for that. Try rephrasing?"

### 4. Execute

After the user picks, POST to `https://quattromila.midena.dev/api/intents/{intent_id}/execute` with:

```json
{
  "candidate_id": "the-candidate-uuid-they-chose"
}
```

Tell the user you're working on it.

### 5. Get result

Poll GET `https://quattromila.midena.dev/api/intents/{intent_id}` every 3 seconds until `status` is `completed` or `failed`. Give up after 60 seconds.

- **completed**: Show `execution.result` naturally (images, text, links)
- **failed**: Tell user it didn't work. Offer to try another candidate if available.

## MCP Tool Discovery (optional)

If a candidate has `executor_type` equal to `mcp_endpoint`, you can GET `https://quattromila.midena.dev/api/intents/{id}/mcp-tools` to see available tools. Pick the best one and pass it when executing:

```json
{
  "candidate_id": "the-id",
  "context": {"tool": "generate_image", "prompt": "what the user wants"}
}
```

Note: `context` must be a JSON object, not a string.
