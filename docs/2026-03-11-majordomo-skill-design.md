# Majordomo Skill — Design Spec

**Date:** 2026-03-11
**Status:** Draft
**Author:** Matteo + Claude

## Problem

AI agents (Claude Code, Cursor, Copilot, etc.) lack built-in knowledge of the Majordomo intent resolution protocol. When a user asks an agent to perform a task the agent can't do natively — e.g., "generate an image of a sunset" or "deploy this contract" — the agent has no way to discover and delegate to specialized external agents.

## Solution

A skills.sh-compatible skill (`majordomo`) that teaches AI agents how to act as **requesters** in the Majordomo network: submit natural-language intents, browse ranked candidates, execute via x402 payment, and return results.

## Audience

AI coding agents (Claude Code, Cursor, Copilot, etc.) that need to **use** Majordomo at runtime to delegate tasks to specialized agents.

## Scope

**In scope:**
- Submitting intents to Majordomo API
- Polling for resolver candidates
- Presenting candidates to user (manual mode) or auto-selecting (autonomous mode)
- Executing a chosen candidate via Servex (x402)
- Polling for execution results
- MCP tool discovery for `mcp_endpoint` candidates
- Configurable endpoint (local dev + production)

**Out of scope:**
- Building resolvers or executors
- Registering agents in the network
- Wallet setup or key management
- Majordomo deployment or infrastructure

## Architecture

### Skill Structure

```
majordomo-skill/
├── SKILL.md              # Main skill document
├── api-reference.md      # Full API endpoint reference
├── README.md             # For humans browsing the repo
└── docs/
    └── 2026-03-11-majordomo-skill-design.md  # This file
```

### Frontmatter

```yaml
name: majordomo
description: Use when the user wants to delegate a task to a specialized external agent, find an AI service, or execute a capability the current agent doesn't have
```

### Configuration

The agent reads the Majordomo endpoint from the `MAJORDOMO_URL` environment variable:
- **Default:** `http://localhost:3000` (local Servex proxy)
- **Production:** User sets `MAJORDOMO_URL` to the hosted instance

The skill assumes the agent has a wallet/keys already configured for x402 payments.

### Authentication

- **Intent submission** (`POST /api/intents`, `GET /api/intents/:id`): No authentication required. These are public endpoints.
- **Execution** (`POST /api/intents/:id/execute`): Goes through Servex (x402 reverse proxy). Servex handles the 402 challenge/response and payment signing transparently — the agent just makes a normal HTTP request and Servex manages the x402 flow using the configured wallet.
- **No API keys or JWTs needed.** The payment itself is the authorization.

### Core Workflow

```
1. User asks agent to do something it can't do natively
2. Agent submits intent: POST /api/intents { query, auto_select?, budget_max? }
3. Agent polls GET /api/intents/:id until status is "resolved"
4. Decision point:
   a. auto_select=true → Majordomo auto-picks best candidate, moves to execution
   b. auto_select=false → Agent presents top candidates to user, user picks one
5. Agent executes: POST /api/intents/:id/execute { candidate_id }
6. Agent polls GET /api/intents/:id until status is "completed" or "failed"
7. Agent returns result to user (or handles failure/refund)
```

### Status State Machine

```
pending → resolving → resolved → executing → completed | failed | refunded
```

- `pending`: Intent created, not yet broadcast to resolvers
- `resolving`: Broadcast to resolvers, awaiting candidates
- `resolved`: Candidates received and ranked, ready for selection
- `executing`: Candidate selected, execution in progress
- `completed`: Execution succeeded, `execution.result` contains the output
- `failed`: Execution failed, `execution.error` contains details, refund issued
- `refunded`: Payment returned to requester

### Error Handling

| Failure | Agent Behavior |
|---------|---------------|
| No candidates found (resolution timeout) | Inform user no specialized agents were found for this task. Suggest rephrasing the query. Do NOT retry automatically. |
| Execution fails | Refund is automatic (100%). Inform user of the error. Offer to try the next-best candidate if one exists. |
| Network/polling error (5xx) | Retry poll up to 3 times with 2s backoff. If still failing, inform user and provide the intent ID for manual follow-up. |
| Candidate no longer available | The execute endpoint returns an error. Inform user and offer to re-submit the intent. |

### Budget and Pricing

- `budget_max` is in **USDC** (e.g., `5.00` = $5 USDC)
- Candidates with `price_usdc` above `budget_max` are filtered out during resolution
- If no candidates are within budget, the intent resolves with an empty candidate list
- Default budget varies by Majordomo instance; always set explicitly for predictable behavior

### MCP Tool Discovery

When a candidate has `executor_type: "mcp_endpoint"`:
1. Agent calls `GET /api/intents/:id/mcp-tools` to list available tools
2. Agent can present tools to user or select the most relevant one
3. Tool selection is passed in the `context` field of `POST /api/intents/:id/execute` (e.g., `{ "candidate_id": "...", "context": "Use the generate_image tool with prompt: ..." }`)
4. Execution still goes through the normal `POST /api/intents/:id/execute` endpoint — MCP tool invocation is handled server-side by Majordomo

### Execution Modes

| Mode | Trigger | Behavior |
|------|---------|----------|
| **Autonomous** | Cost is below `MAJORDOMO_AUTO_BUDGET` (default: $1 USDC) AND user has not indicated preference for manual mode | Sets `auto_select: true`, executes without confirmation |
| **Human-in-the-loop** | Default, or when cost/risk is high | Presents candidates with scores and pricing, waits for user choice |

### Autonomous Mode Guardrails

- `MAJORDOMO_AUTO_BUDGET` env var sets the max cost for autonomous execution (default: `1.00` USDC)
- If candidate price exceeds this threshold, always fall back to human-in-the-loop
- Tasks involving signing transactions, sensitive data, or irreversible actions should never be autonomous regardless of cost

### Polling Strategy

- Poll interval: 2 seconds
- Resolution timeout: 20 seconds (matches Majordomo's resolver timeout)
- Execution timeout: 60 seconds (matches Majordomo's executor timeout)
- Agent should inform user if waiting takes longer than expected

### Fee Awareness

The skill teaches agents that:
- Total fee is 5% (3.5% Majordomo + 1.5% resolver)
- Price shown on candidates is the executor price before fees
- Failed executions get 100% refund

## SKILL.md Content Outline

1. **Overview** — What Majordomo is, when to use it
2. **Configuration** — `MAJORDOMO_URL` env var, wallet prereqs
3. **When to Use** — Symptoms that trigger the skill
4. **Core Workflow** — Flowchart + step-by-step
5. **MCP Tool Discovery** — Advanced flow for MCP executors
6. **Quick Reference** — Endpoint table
7. **Common Mistakes** — Pitfalls to avoid

## api-reference.md Content Outline

1. **Create Intent** — `POST /api/intents` with full request/response schema
2. **Get Intent** — `GET /api/intents/:id` with status progression
3. **Execute** — `POST /api/intents/:id/execute` with payment notes
4. **Get Result** — `GET /api/intents/:id/result`
5. **MCP Tools** — `GET /api/intents/:id/mcp-tools`
6. **Status Codes & Errors** — Common error responses
7. **Types** — Key TypeScript types (Intent, Candidate, Execution)

## Success Criteria

- An AI agent with this skill installed can successfully delegate a task through Majordomo
- The skill triggers correctly when the agent encounters a task it can't do natively
- Both autonomous and manual modes work as expected
- MCP tool discovery works for `mcp_endpoint` candidates
- The skill is installable via `npx skillsadd builders-garden/majordomo-skill`
