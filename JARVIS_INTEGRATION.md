# Scrapling — Jarvis Integration

Fork of [D4Vinci/Scrapling](https://github.com/D4Vinci/Scrapling) vendored at `tools/scrapling/`.
Our fork: [chadronbryant/Scrapling](https://github.com/chadronbryant/Scrapling). Upstream pullable via `git remote upstream`.

## What this gives Jarvis

- **HTTP scraping** with TLS fingerprint impersonation (`curl_cffi`).
- **Stealth browser fetches** (`patchright`) that bypass Cloudflare Turnstile and most anti-bot walls.
- **Dynamic/full-browser scraping** (Playwright 1.58) for JS-heavy SPAs.
- **Adaptive selectors** that survive site redesigns.
- **MCP server** exposing `get`, `bulk_get`, `fetch`, `bulk_fetch`, `stealthy_fetch`, `bulk_stealthy_fetch`, `dynamic_fetch`, `bulk_dynamic_fetch` tools (from `scrapling.core.ai:ScraplingMCPServer`).

## Layout

```
tools/scrapling/
├── .venv/                  # Python 3.14 venv with scrapling[all] + playwright browsers
├── scrapling/              # Upstream source (editable install)
├── JARVIS_INTEGRATION.md   # this file
└── ... (upstream files)
```

Wrapper lives at `scripts/scrape.py` (repo root) and re-execs into `.venv` automatically.

## Three call paths

### 1. CLI (direct)

```bash
# Basic HTTP fetch + selector
python3 scripts/scrape.py get https://example.com --selector "h1::text" --json

# Stealth mode (patchright)
python3 scripts/scrape.py get https://protected.example.com --stealth --json

# Full browser (Playwright)
python3 scripts/scrape.py get https://spa.example.com --dynamic --selector ".product-name::text"

# Interactive scraping shell (upstream)
python3 scripts/scrape.py shell
```

### 2. MCP (preferred for Claude/Jarvis)

Registered in `.mcp.json` as the `scrapling` server. Boots via:
```
tools/scrapling/.venv/bin/scrapling mcp
```

After a Claude Code reload the `mcp__scrapling__*` tools become available. Use these in preference to calling the CLI when working in a Claude session.

### 3. n8n webhook

Template at `Jarvis/ai-max-services/n8n-workflows/jarvis-scrape-webhook.json`.

**Not yet operational** — ai-max does not have Scrapling installed. To activate:
1. On ai-max: `ssh chadron@100.85.167.60 'cd jarvis/tools/scrapling && python3 -m venv .venv && source .venv/bin/activate && pip install -e ".[all]" && scrapling install'`
2. Import the workflow JSON into n8n at `http://100.85.167.60:5678` (or via `mcp__n8n-max__execute_workflow` after publishing).
3. Test: `curl -X POST http://100.85.167.60:5678/webhook/jarvis-scrape -d '{"url":"https://example.com","selector":"h1::text"}' -H 'Content-Type: application/json'`

## Keeping the fork in sync with upstream

```bash
cd tools/scrapling
git fetch upstream
git merge upstream/main   # or rebase, then push to origin
```

Do this quarterly or when upstream ships a bypass for a new anti-bot system we care about.

## Why fork instead of just pip-installing

- **Auditable**: every upstream change passes through our repo history.
- **Patchable**: if an anti-bot bypass breaks, we can fix in-tree without waiting on upstream.
- **Offline-capable**: the brain mesh stays functional even if PyPI or GitHub is unreachable.
- **Pinned**: the version shipped to ai-max/jarvis-brain/MBP is identical, determined by our commit SHA.

## Guardrails

- **Respect robots.txt and ToS.** Scrapling includes `protego` for robots parsing — use it on any new scraper.
- **Never scrape production ServiceNow.** We have MCP tools for that (`napaanesthesia*`).
- **Rate-limit.** Default fetchers are polite; don't override without reason.
- **PII.** If a scraped page might contain PII, route output through `snow_pii_redact` or an equivalent redactor before storing.

## Upstream

- License: BSD (per `tools/scrapling/LICENSE`)
- Docs: https://scrapling.readthedocs.io/
- Version pinned: `0.4.6` (see `pyproject.toml`)
