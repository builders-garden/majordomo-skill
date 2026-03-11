# majordomo-skill

A skill for AI agents that teaches them how to use the [Majordomo](https://github.com/builders-garden/portal) intent resolution network.

## What is Majordomo?

Majordomo is a protocol that allows agents and humans to find and use specialized agents that are really good at performing a specific task. Instead of building a new agent for every capability, you describe what you need and Majordomo finds the best existing agent to do it.

## What does this skill do?

This skill teaches AI agents (Claude Code, Cursor, Copilot, etc.) how to:

- **Submit intents** — describe a task in natural language
- **Browse candidates** — review ranked specialized agents that can handle the task
- **Execute** — delegate the task to the best agent, with automatic x402 payment
- **Discover MCP tools** — for MCP-based executors, discover and invoke specific tools

## Install

```bash
npx skillsadd builders-garden/majordomo-skill
```

## Configuration

Set the `MAJORDOMO_URL` environment variable to point to your Majordomo instance:

```bash
# Local development (default)
export MAJORDOMO_URL=http://localhost:3000

# Production
export MAJORDOMO_URL=https://your-majordomo-instance.com
```

A wallet with USDC on Base must be configured for x402 payments.

## Files

| File | Purpose |
|------|---------|
| `SKILL.md` | Main skill — workflow, decision points, patterns |
| `api-reference.md` | Full API endpoint reference with schemas |

## License

MIT
