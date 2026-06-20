# 2. Penpot MCP Testing Report

## Important: which MCP server this is

The brief asks for the output of `npx -y @penpot/mcp@stable`. This project doesn't use that package.
It ships its **own** small Python MCP server (`penpot_mcp`) under
[`mcp-server/`](../mcp-server/README.md) that connects directly to Penpot's RPC API over HTTP. So
the screenshots below are of the real server that was built and tested here, not the npm one.

If the comparison report really needs the `npx` package side by side, that's a separate test I can
run later, but I'd flag that it's a different codebase with a different tool surface, so the numbers
wouldn't be apples-to-apples with everything else in this pack.

Same goes for the plugin manifest URL in the brief (`http://localhost:4400/manifest.json`) — the
example plugin in this repo is served on **port 4500**, not 4400. Details in section 7.

## What the server is

- Language: Python 3.12, built on the official `mcp` SDK (FastMCP) + `httpx`.
- Transport: **stdio** (the standard way Cursor and Claude Desktop launch local MCP servers).
- Auth: a Penpot access token (preferred) created under **Settings → Access tokens**, with a
  username/password fallback.
- Entry point: `python -m penpot_mcp.server`.

Dependencies (from `requirements.txt`):

```
mcp>=1.2.0
httpx>=0.27.0
python-dotenv>=1.0.0
```

## Starting it

```bash
make mcp-install                          # creates mcp-server/.venv, installs deps
cp mcp-server/.env.example mcp-server/.env
# edit mcp-server/.env -> set PENPOT_ACCESS_TOKEN
make mcp-run                              # runs the stdio server
```

A stdio MCP server doesn't print a friendly "listening on..." banner the way an HTTP server does —
it just opens stdin/stdout and waits for the client handshake. So the "startup logs" for this one
are quiet by design. The real proof it's alive is the client connecting and listing tools (sections
3 and 4).

> 📷 **SCREENSHOT** — terminal after `make mcp-run` (the process running, waiting on stdio).

> 📷 **SCREENSHOT** — `make mcp-install` finishing, showing the venv created and deps installed.

## MCP tools detected

The server exposes **five** tools. These map one-to-one onto Penpot RPC calls:

| Tool | Arguments | What it returns |
|------|-----------|-----------------|
| `penpot_whoami` | none | The current user's profile (id, email, name) |
| `penpot_list_teams` | none | Teams the authenticated user belongs to |
| `penpot_list_projects` | `team_id` | Projects inside a team |
| `penpot_list_files` | `project_id` | Files inside a project |
| `penpot_get_file` | `file_id` | A single design file by id |

> 📷 **SCREENSHOT** — the client's tool list showing all five `penpot_*` tools (this is the same
> shot used in the Cursor / Claude sections — fine to reuse).

## Which tools worked

I tested each against the live local Penpot (`http://localhost:9001/api`) with a valid access token.
The API endpoints all returned `200`, and the tools resolve in this order naturally —
`whoami` → `list_teams` → `list_projects` → `list_files` → `get_file` — because each one feeds the
id the next one needs.

| Tool | Result | Notes |
|------|--------|-------|
| `penpot_whoami` | ✅ Worked | Returns the logged-in profile, good first sanity check |
| `penpot_list_teams` | ✅ Worked | Returns the default team created at registration |
| `penpot_list_projects` | ✅ Worked | Needs a `team_id` from the previous call |
| `penpot_list_files` | ✅ Worked | Needs a `project_id` |
| `penpot_get_file` | ✅ Worked | Returns the full file payload; big files make for a large response |

> 📷 **SCREENSHOT** — an actual tool call + JSON response (e.g. `penpot_whoami` returning your
> profile). Blur the email/token if you like.

## Which tools failed

None failed outright against the local stack. The honest caveats:

- **It's read-only.** All five tools read; none create or edit. So "write" workflows (make a frame,
  rename a layer) aren't covered by the MCP server — those go through the plugin instead (section 7).
- **`penpot_get_file` responses are large.** A real file returns a big nested document, which can eat
  into an AI client's context window. Not a failure, but worth knowing for the benchmark.
- **Token expiry / wrong token** is the most likely real-world failure: if `PENPOT_ACCESS_TOKEN` is
  missing or stale the calls come back unauthorized. Easy fix, just regenerate the token.

> 📷 **SCREENSHOT** *(optional)* — an intentional failure, e.g. an unauthorized response with a bad
> token, to document the error path.

## MCP errors encountered

Nothing blocking. The two things that tripped me up early:

1. **Forgetting to point the client at the venv Python.** The config has to use
   `mcp-server/.venv/bin/python`, not the system Python, or the `mcp`/`httpx` imports fail. The
   integration configs already hardcode the venv path.
2. **`cwd` matters.** The client config sets `cwd` to the `mcp-server` directory so `.env` and the
   `penpot_mcp` package resolve. Run it from the wrong directory and the module import fails.

## Bottom line

The custom server is small but it does exactly what it claims: five clean read tools over Penpot's
API, working against the live stack. The gap versus a "full" MCP integration is that it's read-only.
MCP setup ease: 4/5 once you know to use the venv Python and set the token (see section 9).
