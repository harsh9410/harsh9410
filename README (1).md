# Penpot AI + MCP — Documentation & Benchmarking Pack

This folder holds the evidence write-ups for the Penpot benchmarking report. I split it into the
nine sections you asked for, one file each, so it's easier to drop screenshots in and review them
one at a time.

| # | File | What's in it |
|---|------|--------------|
| 1 | [01-penpot-setup-guide.md](01-penpot-setup-guide.md) | Self-hosted Penpot via Docker, versions, RAM, first-run notes |
| 2 | [02-mcp-testing-report.md](02-mcp-testing-report.md) | The MCP server, tools detected, what worked / what didn't |
| 3 | [03-cursor-integration-report.md](03-cursor-integration-report.md) | Cursor config, prompts, responses, reliability |
| 4 | [04-claude-integration-report.md](04-claude-integration-report.md) | Claude Desktop setup, prompts, limits |
| 5 | [05-chatgpt-integration-report.md](05-chatgpt-integration-report.md) | ChatGPT connector attempt + blockers |
| 6 | [06-cloudflare-tunnel-report.md](06-cloudflare-tunnel-report.md) | Tunnel config and public HTTPS access |
| 7 | [07-plugin-ecosystem-report.md](07-plugin-ecosystem-report.md) | Plugins tested with ratings |
| 8 | [08-figma-compatibility-report.md](08-figma-compatibility-report.md) | Figma → Penpot import fidelity |
| 9 | [09-final-metrics.md](09-final-metrics.md) | The 1–5 score matrix and recommendation |

## A note on the screenshots

Most of these sections ask for screenshots. I've marked every spot that needs one like this:

> 📷 **SCREENSHOT** — short description of what to capture

Just grab the image, drop it next to the markdown file (or in an `images/` subfolder), and replace
the line with `![caption](images/your-file.png)`. The surrounding text is already written so you
shouldn't have to do much beyond pasting the pictures in.

## A note on the MCP server

Heads up on one thing that matters for the report: this project does **not** use the published
`npx @penpot/mcp@stable` package. It ships its own small Python MCP server (`penpot_mcp`) that talks
straight to Penpot's RPC API. The `npx` command in the brief won't reflect what was actually built
and tested here, so section 2 documents the real server instead and explains the difference. Worth
calling out to whoever's compiling the comparison so the numbers line up with reality.

— Sohit
