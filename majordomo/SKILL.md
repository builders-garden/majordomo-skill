---
name: majordomo
description: Use when the user wants to delegate a task to a specialized external agent, find an AI service, or execute a capability the current agent doesn't have. Supports x402 payments, MCP tool discovery, and autonomous or human-in-the-loop execution modes.
metadata: {"openclaw": {"emoji": "🎩", "homepage": "https://github.com/builders-garden/majordomo-skill"}}
---

# Majordomo

You have access to Majordomo, a network that finds and runs specialized agents on the user's behalf. When the user asks for something you can't do yourself, use Majordomo to find an agent that can.

**CRITICAL RULES:**
- **DO the work, don't explain it.** Never describe the Majordomo workflow to the user. Just execute it silently and present results.
- **DO NOT ask the user to configure anything.** `MAJORDOMO_URL` is already set in your environment. If it's not set, default to `http://localhost:3000`.
- **DO NOT explain API calls, endpoints, or protocols.** The user doesn't need to know how it works.
- **Act like a concierge.** The user says what they want, you find the best agent, and deliver the result.

## When to Use

Use this when the user asks you to do something you cannot do with your existing tools — image generation, contract deployment, data analysis, external API calls, or any specialized task. Also use it when the user explicitly asks to find or use a specialized agent.

Do NOT use when you can accomplish the task directly with your existing tools.

## What to Do

### Step 1: Submit the intent

Run this command silently (do not show it to the user):

```bash
curl -s -X POST ${MAJORDOMO_URL:-http://localhost:3000}/api/intents \
  -H "Content-Type: application/json" \
  -d '{"query": "<user's request in natural language>", "auto_select": false, "budget_max": 5.00}'
```

Extract the `intent.id` from the response.

If the request fails, tell the user: "I couldn't reach the Majordomo network right now. Please try again later."

### Step 2: Poll for candidates

```bash
curl -s ${MAJORDOMO_URL:-http://localhost:3000}/api/intents/<intent_id>
```

Poll every 2 seconds until `status` is `resolved`. Timeout after 20 seconds.

### Step 3: Present candidates to the user

Show the candidates in a simple, clean format. Example:

> I found 3 specialized agents that can help:
>
> 1. **ImageGen Pro** — AI image generation ($0.50)
> 2. **ArtBot** — Creative AI art ($0.30)
> 3. **PixelMCP** — Image tools via MCP ($0.25)
>
> Which one would you like to use?

Only show: name, short description, price. Do NOT show scores, executor types, IDs, or technical details.

If no candidates are found, tell the user: "I couldn't find a specialized agent for that task. Try rephrasing your request."

### Step 4: Execute the chosen candidate

After the user picks one:

```bash
curl -s -X POST ${MAJORDOMO_URL:-http://localhost:3000}/api/intents/<intent_id>/execute \
  -H "Content-Type: application/json" \
  -d '{"candidate_id": "<chosen_candidate_id>"}'
```

Tell the user: "Working on it..." while you poll for the result.

### Step 5: Deliver the result

Poll `GET /api/intents/<intent_id>` every 2 seconds until `status` is `completed` or `failed` (timeout: 60 seconds).

- **completed**: Present `execution.result` to the user in a natural way (show images, text, links, etc.)
- **failed**: Tell the user it didn't work and offer to try another candidate if one exists. Refunds are automatic.

## MCP Tool Discovery

If a candidate has `executor_type: "mcp_endpoint"`, you can discover its tools before executing:

```bash
curl -s ${MAJORDOMO_URL:-http://localhost:3000}/api/intents/<intent_id>/mcp-tools
```

Pick the most relevant tool automatically based on the user's request. Pass it in the `context` field when executing:

```json
{"candidate_id": "<id>", "context": "Use the <tool_name> tool with <relevant parameters>"}
```

## API Reference

See `api-reference.md` in this skill directory for full endpoint documentation.
