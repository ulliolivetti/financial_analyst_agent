# Financial analyst agent

Agent workspace connected to the existing personal investment tracker via its
local MCP server. The initial scope is recorded balances/history and financial
operations; financial-advice skills can be developed separately later.

```text
OpenCode session in this repository
    ↕ stdio MCP
../investment_tracker/integrations/mcp
    ↕ HTTP http://127.0.0.1:8000/api
Existing FastAPI tracker → SQLite
```

## Expected checkout layout

```text
dev/
├── financial_analyst_agent/       This repository
│   ├── AGENTS.md                  Agent operating rules
│   └── opencode.json              MCP connection
└── investment_tracker/            Existing tracker checkout
    ├── app/
    └── integrations/mcp/          MCP code, dependencies and tests
```

The tracker repository is hosted at
https://github.com/ulliolivetti/personal_investment_tracker; its local folder here
is named `investment_tracker`. The relative path in `opencode.json` assumes this
sibling layout. Change the `--directory` argument to the correct relative or
absolute MCP package path on another computer.

## Setup and use (PowerShell)

Run from this repository:

```powershell
# Install the tracker's MCP dependencies using its lockfile.
uv sync --project ../investment_tracker/integrations/mcp --locked

# Start the tracker in a separate terminal if it is not already running.
# Its existing .env and database remain in its own directory.
uv run --directory ../investment_tracker uvicorn app.main:app --host 127.0.0.1 --port 8000

# Verify OpenCode connects to the MCP, then open the agent workspace.
opencode mcp list
opencode
```

OpenCode launches the stdio MCP process automatically. It does not start the
tracker API. A connected MCP can list tools even while the API is unavailable;
verify the complete connection by asking it to list accounts.

Try these read-only prompts first:

- “List my accounts and their tracking methods.”
- “Show my current account's recorded balance.”
- “Show the last five operations in my PEA.”

To record an operation, specify the account, date, amount and relevant details.
Financial tool calls act on the real tracker: use a separate test tracker if you
want to experiment with writes rather than record actual transactions.

## Available tools

`list_accounts`, `get_account_balance`, `list_recent_operations`,
`list_instruments`, `record_income`, `record_expense`, `record_bank_interest`,
`record_transfer`, `record_investment_operation`.

The implementation and detailed contracts belong to the tracker repository's
`integrations/mcp/README.md`. No MCP code or database is duplicated here.
No API credential is required by the current local-only tracker. The configuration
does not install automatic startup tasks, add API authentication or sandbox the agent.

## Troubleshooting

- **MCP fails to start:** check `uv` is on PATH and the configured sibling path exists;
  run the sync command above and check `/mcps` in OpenCode.
- **MCP connected, tool request fails:** check the tracker API is running at
  `http://127.0.0.1:8000`; use `/docs` in a browser to verify reachability.
- **Different API port/host:** update `TRACKER_API_URL` in `opencode.json` (origin
  only, without `/api`). Keep the unauthenticated API local/private.
- **Changed MCP source:** reconnect/restart its process to load the edits.
- **Uncertain write outcome:** inspect history before deciding whether to retry;
  the current API has no idempotency protection.

## Existing reference documents

`PROJECT_MASTER_PLAN.md`, `FINANCIAL_EQUATIONS.md` and
`FINANCIAL_EQUATIONS.html` are preserved historical planning/reference material.
They do not describe all current tracker behavior and must not be treated as live
holdings, the current schema, or implemented API contracts. Use MCP results and
the tracker implementation for current information.
