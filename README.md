# Vara MCP Server

Machine-callable surface for the **Real Vara** sensory architecture (dual-track, multi-timescale, Sentinel-gated, Veil/Vault-mediated).

This package wraps the operational Vara pipeline so it can be registered as a custom MCP connector in Grok (or any MCP-compatible client). It exposes both **live scanning** and **access to previously gathered signals**, enabling continuous Field Intel Report production.

## Goals

- Make the real Vara scan architecture callable (`vara_run_scan`)
- Provide read access to historical scans and the Vault / Veil corpus
- Preserve governance contracts (G1/G2/G3, consecutive recurrence, observation-aware Vault)
- Emit Operator-Tier Field Intel Reports as first-class outputs
- Remain deployable to public HTTPS endpoints (required by Grok custom connectors)

## Directory Layout

```
vara-mcp-server/
├── README.md
├── requirements.txt
├── vercel.json             # Vercel routing + function limits
├── api/
│   └── index.py            # Vercel Python entry (exposes FastAPI app)
├── src/
│   ├── __init__.py
│   ├── server.py           # FastAPI / MCP surface
│   ├── tools.py            # Tool implementations
│   ├── schemas.py          # JSON Schema definitions
│   └── config.py           # Paths & defaults
├── schemas/
│   └── tools.json
├── docs/
│   ├── ARCHITECTURE.md
│   └── DEPLOY.md
└── data/
    └── README.md
```

## Deploy on Vercel (your requested path)

1. **Vendor the Vara package + data** into the repo (e.g. `vendor/vara/` containing `vara_scan.py`, `vault_signals.json`, `vara_output/`, etc.). Vercel functions cannot see your local skill paths.
2. Push to GitHub.
3. Import the project in the Vercel dashboard (or `vercel` CLI).
4. Set environment variables in the Vercel project:
   - `VARA_PACKAGE_PATH` = `/var/task/vendor/vara` (or the path you chose)
   - `VARA_DATA_ROOT`   = `/var/task/vendor/vara` (or separate data path)
5. Deploy. Note the production HTTPS URL.
6. In **Grok → Connectors → New → Custom** paste that URL.

### Vercel realities (important)

| Tool | Vercel suitability |
|------|--------------------|
| `vara_list_scans` | Excellent |
| `vara_get_scan` | Excellent |
| `vara_query_signals` | Excellent |
| `vara_get_veil_state` | Excellent |
| `vara_generate_fir` | Excellent |
| `vara_run_scan` | Risky on free/hobby (network-heavy, can hit duration limits). Better on Pro + Fluid or on a long-running host. |

`vercel.json` is already configured with `maxDuration: 60` and 1 GB memory. Raise further on Pro if needed.

For production-grade live scans, the recommended split is:
- Vercel → query + FIR tools (always on)
- Fly / Railway → full live `vara_run_scan` (or a queue that Vercel triggers)

## Local / tunnel quick start

```bash
cd vara-mcp-server
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt

export VARA_PACKAGE_PATH=/path/to/vara-scan/assets/vara
export VARA_DATA_ROOT=/path/to/vara-scan/assets/vara

python -m src.server
# then: ngrok http 8000
```

## Tools

| Tool | Description |
|------|-------------|
| `vara_run_scan` | Full dual-track / multi-timescale Real Vara scan |
| `vara_list_scans` | Historical scan inventory |
| `vara_get_scan` | Full past report by ID |
| `vara_query_signals` | Filtered Vault (+ optional Veil) query |
| `vara_get_veil_state` | Current hold + trajectories |
| `vara_generate_fir` | Operator-Tier Field Intel Report markdown |

## Governance

- Live scans still run through Sentinel and Veil/Vault.
- Historical reads are read-only.
- Config hashes and full provenance are returned for auditability.

See `docs/ARCHITECTURE.md` and `docs/DEPLOY.md` for full detail.
# Vara_mcp
