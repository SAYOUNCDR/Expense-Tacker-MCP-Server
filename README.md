# ExpenseTracker FastMCP server

Lightweight MCP server for tracking personal expenses using FastMCP and SQLite. This repo provides a small, local MCP server you can run on your machine for quick experiments, demos, or as the basis for a remote deployment.

Tags: expense-tracker, fastmcp, mcp, python, sqlite

---

## What this is

- A minimal MCP server exposing a few tools to record and query expenses.
- Stores data in a local SQLite database: `expenses.db` (created automatically).
- Ships a JSON resource `expense://categories` backed by `categories.json`.

Main features

- add_expense(date, amount, category, subcategory, note)
- list_expenses(start_date, end_date)
- update_expense(expense_id, ...)
- delete_expense(expense_id)
- summarize(start_date, end_date, category=None)
- categories resource (JSON)

## Quick start (Windows - cmd.exe)

1. Create a virtual environment and activate it (Windows `cmd.exe`):

First of all install uv if you don't have it already globally using:

```batch
pip install uvicorn
```

Then create and activate a virtual environment:

```batch

python -m venv .venv
.venv\Scripts\activate
```

2. Install dependencies. If you use `pip` directly, at minimum install FastMCP. If your project uses a lockfile or `pyproject.toml`, prefer that.

```batch
pip install fastmcp
# or, if you maintain requirements.txt:
pip install -r requirements.txt
```

Optional: if you're using the `uv` helper CLI (used in some FastMCP guides), you can add FastMCP via the `uv` CLI instead of pip. Only run the following if you already have the `uv` tool installed:

```batch
uv add fastmcp
```

3. Run the server in development mode (opens FastMCP studio if available):

```batch
uv run fastmcp dev main.py
```

4. Run the server for normal usage:

```batch
uv run fastmcp run main.py
```

Notes

- Database: `expenses.db` will be created next to `main.py` on first run.
- Edit `categories.json` to customize categories; the MCP `categories` resource reads it fresh on each call.

## Testing with FastMCP Studio / Claude Desktop

You can test and inspect the MCP server locally either using the FastMCP studio (dev mode) or by adding the server to Claude Desktop.

1. Run FastMCP studio (dev mode)

```batch
uv run fastmcp dev main.py
```

This runs the server in development mode and opens FastMCP's studio/inspector (if available) so you can call tools interactively.

2. Add the server to Claude Desktop (optional)

If you want Claude Desktop to manage and talk to your MCP server, install the Claude Desktop helper using the FastMCP/uv helper (only if you have the `uv` helper installed):

```batch
uv run fastmcp install claude-desktop main.py
```

Important notes when using Claude Desktop:

- After changing the server code, fully restart Claude Desktop so it reloads the MCP server. That means quitting the app completely (exit from the system tray / taskbar) and then reopening it.
- If Claude Desktop fails to load your MCP server, edit the Claude Desktop MCP config: open Claude Desktop, go to Settings → Developer → Edit config. You'll see a JSON like this:

```json
{
  "mcpServers": {
    "ExpenseTracker": {
      "command": "C:\\Users\\Sayoun Parui\\AppData\\Roaming\\Python\\Python313\\Scripts\\uv.exe", # here if u see  only  uv 
      "args": [
        "run",
        "--with",
        "fastmcp",
        "fastmcp",
        "run",
        "C:\\Users\\Sayoun Parui\\Desktop\\ExpenceTracker MCP Server\\main.py"
      ],
      "env": {},
      "transport": "stdio",
      "type": null,
      "cwd": null,
      "timeout": null,
      "description": null,
      "icon": null,
      "authentication": null
    }
  },
  "preferences": {
    "menuBarEnabled": false,
    "legacyQuickEntryEnabled": false
  }
}
```

- If the `command` value just shows `uv` (or a short name) you should replace it with the full path to the `uv` executable. To find the full path, open a Windows `cmd.exe` and run:

```batch
where uv
```

Copy the full path returned (for example `C:\Users\You\AppData\Roaming\Python\Python313\Scripts\uv.exe`) and paste it into the `command` field in the Claude config, replacing the short `uv` value. Save the config and restart Claude Desktop.

If `where uv` returns multiple results, pick the one that points to your desired Python environment (global or the one where you installed `uv`).

## Tools / API (what you can call)

- add_expense(date: str, amount: float, category: str, subcategory: str = "", note: str = "") -> {status, id}
- list_expenses(start_date: str, end_date: str) -> list of expense objects
- update_expense(expense_id: int, date, amount, category, subcategory, note) -> {status}
- delete_expense(expense_id: int) -> {status}
- summarize(start_date: str, end_date: str, category: Optional[str]) -> [{category, total_amount}]
- Resource: `expense://categories` — returns `categories.json` content (mime_type: application/json)

Example call (pseudo):

```python
# with FastMCP client or via the studio, call the `add_expense` tool:
result = mcp.call('add_expense', date='2025-10-01', amount=12.50, category='food', subcategory='snacks', note='coffee')
```

## Project layout

- `main.py` — MCP server implementation and tool definitions.
- `categories.json` — default categories/subcategories used by the resource.
- `expenses.db` — SQLite DB (auto-created).

## Deployment & next steps

This project is intentionally small so it's easy to convert to a remote service. Ideas for production-ready deployments:

- Move the DB to Postgres or another managed DB (RDS, Cloud SQL).
- Containerize the app and deploy to ECS, EKS, GKE, or App Services.
- Add authentication to MCP endpoints and secure the server behind a gateway.

Short-term suggestions:

- Add basic schema migration tooling (alembic/sqlalchemy or a simple migration script).
- Add input validation and richer date handling (ISO 8601 enforcement).
- Add unit tests for the tool functions.

## Contributing

Small contributions welcome. Suggested workflow:

1. Fork the repo
2. Create a feature branch
3. Open a PR with a short description of changes

Please keep changes small and focused. If you plan a large refactor (DB change, API redesign), open an issue first.
