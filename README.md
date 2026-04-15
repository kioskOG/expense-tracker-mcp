# 💰 Expense Tracker MCP Server

A simple **FastMCP-based remote server** for tracking expenses using SQLite. This project exposes MCP tools to add, list, and summarize expenses, along with a categories resource.

---

## 🚀 Features

* Add expense entries
* List expenses by date range
* Summarize expenses by category
* Dynamic categories via JSON resource
* Lightweight SQLite database
* MCP-compatible (works with Inspector, Claude, Cursor, etc.)

---

## 📁 Project Structure

```
.
├── main.py            # MCP server implementation
├── expenses.db        # SQLite database (auto-created)
├── categories.json    # Expense categories
└── README.md
├── pyproject.toml
└── uv.lock
```

---

## ⚙️ Requirements

* Python 3.11+
* uv (recommended) OR pip

```bash
mkdir expense-tracker-remote-mcp-server
cd expense-tracker-remote-mcp-server
uv init .
```

Install dependencies:

```bash
pip install fastmcp
# or
uv add fastmcp
```

---

## 🧠 Available MCP Tools

### ➕ add_expense

Add a new expense entry.

**Input:**

* `date` (str)
* `amount` (float)
* `category` (str)
* `subcategory` (optional)
* `note` (optional)

---

### 📋 list_expenses

List expenses between two dates.

**Input:**

* `start_date` (str)
* `end_date` (str)

---

### 📊 summarize

Summarize expenses by category.

**Input:**

* `start_date` (str)
* `end_date` (str)
* `category` (optional)

---

## 📦 MCP Resource

### expense://categories

Returns categories from `categories.json`.

---

## ▶️ Running the Server

You can run the MCP server in multiple ways:

### Option 1: Direct Python

```bash
uv run main.py
```

---

### Option 2: FastMCP Run (Recommended for Remote Server)

```bash
uv run fastmcp run main.py \
  --transport http \
  --host 0.0.0.0 \
  --port 8000 \
  --reload
```

Server will be available at:

```
http://localhost:8000/mcp
```

---

## 🔍 Running MCP Inspector (UI)

Run inspector in a separate terminal:

```bash
uv run fastmcp dev inspector main.py --reload
```

Then open:

```
http://localhost:6274
```

Use the generated token if required.

---

## 🧪 Testing the Server

### ⚠️ Important

The `/mcp` endpoint is **not a REST API**. It uses:

* JSON-RPC
* Server-Sent Events (SSE)
* Session-based communication

Opening it in a browser will result in errors like:

```
Not Acceptable: Client must accept text/event-stream
```

---

### 🟢 Option 1: MCP Inspector (Recommended)

Start inspector:

```bash
uv run fastmcp dev inspector main.py --reload
```

Open:

```
http://localhost:6274
```

Connect to:

```
http://localhost:8000/mcp
```

👉 You can now interact with tools visually.

---

### 🔵 Option 2: curl (Manual MCP Flow)

#### Step 1: Initialize session

```shell
curl -i \
  -H "Accept: application/json, text/event-stream" \
  -H "Content-Type: application/json" \
  -X POST http://localhost:8000/mcp \
  -d '{
    "jsonrpc": "2.0",
    "id": 1,
    "method": "initialize",
    "params": {
      "protocolVersion": "2024-11-05",
      "capabilities": {},
      "clientInfo": {
        "name": "curl-client",
        "version": "1.0"
      }
    }
  }'
```

👉 Copy the `mcp-session-id` from response headers.

---

#### Step 2: Call a tool

```shell
curl -N \
  -H "Accept: application/json, text/event-stream" \
  -H "Content-Type: application/json" \
  -H "mcp-session-id: YOUR_SESSION_ID" \
  -X POST http://localhost:8000/mcp \
  -d '{
    "jsonrpc": "2.0",
    "id": 2,
    "method": "tools/call",
    "params": {
      "name": "add_expense",
      "arguments": {
        "date": "2026-04-15",
        "amount": 500,
        "category": "food",
        "note": "lunch"
      }
    }
  }'
```

---

### 🧠 Understanding MCP Response

Example response:

```text
event: message
data: { ... }
```

Key fields:

* `content` → Human-readable output (for LLMs)
* `structuredContent` → Machine-readable output

---

### 🔍 Verify Database Entry

```bash
sqlite3 expenses.db "SELECT * FROM expenses;"
```

---

## 🌐 Using as Remote MCP Server

This server can be used with MCP-compatible clients like:

* Claude Desktop
* Cursor
* Custom MCP clients

Example configuration:

```json
{
  "mcpServers": {
    "expense-tracker": {
      "url": "http://localhost:8000/mcp"
    }
  }
}
```

---

## 🗄️ Database

* Uses SQLite
* Auto-creates `expenses.db`
* No external DB required

---

## 🛠️ Development Tips

* Modify `categories.json` without restarting server
* Use `--reload` for auto-restart
* Change port if 8000 is occupied

