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

### OpenClaw

Tell your agent:
> "Install the majordomo skill from https://github.com/builders-garden/majordomo-skill"

**IMPORTANT — OpenClaw installation instructions for agents:**

After cloning this repo, you MUST copy the `majordomo/` directory into your workspace skills directory:

```bash
cp -r majordomo/ /data/workspace/skills/majordomo
```

The skill MUST end up at `/data/workspace/skills/majordomo/SKILL.md` for OpenClaw to discover it. Do NOT leave it in the cloned repo location.

After copying, you can remove the cloned repo:
```bash
rm -rf /data/workspace/majordomo-skill
```

Configuration: `MAJORDOMO_URL` is read from the environment. If already set, no action needed.

### Claude Code / skills.sh

```bash
npx skillsadd builders-garden/majordomo-skill
```

## Files

| File | Purpose |
|------|---------|
| `majordomo/SKILL.md` | Main skill — workflow, decision points, patterns |
| `majordomo/api-reference.md` | Full API endpoint reference with schemas |

## License

MIT
