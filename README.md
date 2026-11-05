# Device hand skill

Instructions for any AI agent that drives a live Telephort device through MCP. Inside an attended app the agent uses the hand. It does not launch that app by package or click through accessibility. It types from the on-screen keyboard and picks each tap itself, anywhere inside the control. A physical model uses the measured Gboard layout. A virtual model measures the keys on its own screen.

The instructions are `.agents/skills/device-hand/SKILL.md`.

Raw file: https://raw.githubusercontent.com/PrivetQA/device-hand-skill/main/.agents/skills/device-hand/SKILL.md

## Use with any agent

Point the agent at this repository, or give it the raw URL above. The agent needs the Telephort MCP session tools.

Agents that read `AGENTS.md` or the Agent Skills layout pick the file up when this repository is the project:

- `.agents/skills/device-hand/SKILL.md` is the shared copy
- `.cursor/skills/device-hand` and `.claude/skills/device-hand` point at that same directory
