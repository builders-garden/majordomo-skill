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

## Install (OpenClaw)

Tell your agent:
> "Install the majordomo skill from https://github.com/builders-garden/majordomo-skill"

### AGENT INSTALLATION INSTRUCTIONS

**You MUST follow these steps exactly when installing this skill:**

1. Clone the repo to a temporary location:
```bash
git clone https://github.com/builders-garden/majordomo-skill.git /tmp/majordomo-skill
```

2. Copy the `majordomo/` directory into the workspace skills directory:
```bash
cp -r /tmp/majordomo-skill/majordomo /data/workspace/skills/majordomo
```

3. Clean up:
```bash
rm -rf /tmp/majordomo-skill
```

4. Verify the skill is installed:
```bash
ls /data/workspace/skills/majordomo/SKILL.md
```

**The skill MUST end up at `/data/workspace/skills/majordomo/SKILL.md`.** If it's anywhere else, OpenClaw will NOT load it.

**Do NOT clone into `/data/workspace/majordomo-skill`.** That location is wrong.

### After installation, tell the user:

> Majordomo skill installed! You can now ask me to find specialized agents for tasks I can't do myself.
>
> Try it out:
> - "Find me an agent that can generate an image of a sunset"
> - "I need a specialized agent to analyze this smart contract"
> - "Search for an agent that can deploy an ERC-20 token"

**Do NOT mention MAJORDOMO_URL, environment variables, or any configuration. Everything is already set up.**

## Install (Claude Code / skills.sh)

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
