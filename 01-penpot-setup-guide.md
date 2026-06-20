# 1. Penpot Setup Guide

This is the self-hosted Penpot stack running locally through Docker Compose. Everything below was
captured from the machine the project actually runs on, so the versions and resource numbers are
real, not estimates.

## My environment

| Thing | Value |
|-------|-------|
| OS | Ubuntu 24.04 (kernel 6.17.0-35-generic, x86_64) |
| Docker | 29.4.1 (build 055a478) |
| Docker Compose | v5.1.3 |
| Node.js | v22.22.2 |
| CPU | 12 cores |
| RAM | 16 GB total (15 GiB visible) |

> 📷 **SCREENSHOT** — `docker --version` and `node --version` in the terminal. (For reference the
> output is `Docker version 29.4.1, build 055a478` and `v22.22.2`.)

A quick note on Docker Desktop: I'm running this on Linux with the Docker Engine + Compose plugin
directly rather than Docker Desktop, so there's no Desktop GUI window to screenshot. If the report
specifically needs the Desktop UI, that has to be done on a Mac/Windows box. On Linux the equivalent
"is it running" evidence is the `docker compose ps` output below.

> 📷 **SCREENSHOT** — Docker Desktop dashboard *(only if you re-run this on Mac/Windows; on this
> Linux host it doesn't apply — use the compose ps shot instead).*

## The stack

It's a five-service stack defined in [`self-host/docker-compose.yml`](../self-host/docker-compose.yml):
frontend, backend, exporter, Postgres, and Redis. Only the frontend is published to the host, on
port **9001**, and it reverse-proxies the backend and exporter internally.

Start it:

```bash
cd self-host
cp .env.example .env      # set PENPOT_DATABASE_PASSWORD and generate PENPOT_SECRET_KEY
cd ..
make up                   # == docker compose up -d
```

### `docker compose ps`

Here's the actual output with all five containers up:

```
NAME              IMAGE                       SERVICE           STATUS                    PORTS
penpot-backend    penpotapp/backend:latest    penpot-backend    Up 17 minutes
penpot-exporter   penpotapp/exporter:latest   penpot-exporter   Up 17 minutes
penpot-frontend   penpotapp/frontend:latest   penpot-frontend   Up 17 minutes             0.0.0.0:9001->8080/tcp
penpot-postgres   postgres:15                 penpot-postgres   Up 17 minutes (healthy)   5432/tcp
penpot-redis      redis:7.2                   penpot-redis      Up 17 minutes             6379/tcp
```

> 📷 **SCREENSHOT** — your terminal showing `docker compose ps` with all five rows "Up". The output
> above is exactly what it looks like.

The three Penpot images aren't tiny, worth noting for anyone sizing a box:

```
penpotapp/exporter   2.5 GB
penpotapp/backend    1.31 GB
penpotapp/frontend   940 MB
```

## Penpot running on http://localhost:9001

`curl -o /dev/null -w "%{http_code}" http://localhost:9001` returns **200**, and the RPC API at
`http://localhost:9001/api/rpc/command/get-profile` also responds, so both the UI and the API layer
are live.

> 📷 **SCREENSHOT** — the Penpot landing page in a browser at `http://localhost:9001`.

## Login screen

> 📷 **SCREENSHOT** — the Penpot login/register screen.

One gotcha I hit: SMTP isn't configured in this stack, so the email-verification step on first
registration won't send a real email. The verification link shows up in the backend logs instead.
Grab it with:

```bash
make logs    # == docker compose logs -f, then look for the verification URL
```

That's the main "setup error" worth recording — it's not really an error, just a self-host quirk.
Once you click that link the account is verified and login works normally.

> 📷 **SCREENSHOT** — the backend log line containing the verification link (token can be blurred).

## Dashboard / home screen

After login you land on the team dashboard.

> 📷 **SCREENSHOT** — the Penpot dashboard / home screen after logging in.

## Test project created

I created a throwaway project + file to confirm the whole thing works end to end (this is also what
the MCP server reads later).

> 📷 **SCREENSHOT** — a test project with at least one design file inside it.

## Setup errors encountered

Honestly it was pretty smooth. The only things worth logging:

1. **No SMTP → email verification link lives in the logs** (covered above). Expected for a local
   self-host, not a real bug.
2. **Image pull is heavy** — about 4.7 GB across the three Penpot images plus Postgres/Redis, so the
   first `make up` takes a while on a slow connection. After that it's cached.
3. **`PENPOT_SECRET_KEY` must be set** before first boot. Generate one with
   `python3 -c "import secrets; print(secrets.token_urlsafe(64))"`. If you skip it the stack will
   start but sessions behave oddly.

## Approximate RAM usage

With the full stack up plus a browser and editor open, the machine sat around **9.4 GiB used of
16 GB**. The Penpot containers themselves are the lighter part of that; Postgres and the JVM backend
are the main consumers. I'd say **plan for ~3–4 GB headroom for the stack alone** on an otherwise
idle box, more if you're exporting large files.

> 📷 **SCREENSHOT** *(optional)* — `docker stats` or `free -h` showing memory while the stack runs.

## Summary

Self-hosting Penpot through this compose stack was straightforward. Five containers, one published
port, one .env to fill in, and the only friction is the SMTP-less verification step that's easy once
you know to check the logs. Setup ease: I'd give it a 4/5 (see section 9).
