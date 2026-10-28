# Cafe Server (MCP)

FastMCP-based server for a cafe/order agent. Provides tools to list menu, place orders, and check order status with PostgreSQL persistence.

## Project structure
- `server/Code/src/` - application code
  - `main.py` - FastMCP entrypoint (HTTP transport via `--serve`)
  - `settings.py` - config from `.env`
  - `routes/` - FastMCP router wiring
  - `tools/` - MCP tools exposed to agents
  - `repositories/` - DB access (SQLAlchemy) and business logic
  - `models/` - Pydantic schemas
  - `migrations/` - Alembic migrations
  - `utils/` - helpers/exceptions/logging
  - `pyproject.toml` - dependencies (managed with uv)

## Setup

### Prerequisites
- Python 3.14+
- PostgreSQL
- [uv](https://docs.astral.sh/uv/) recommended

### Environment
Create `.env` in `server/Code/src/` with:
```env
DB_HOST=localhost
DB_PORT=5432
DB_NAME=your_db
DB_USERNAME=postgres
DB_PASSWORD=your_password
HOST=0.0.0.0
PORT=8080
```

### Install dependencies
From `server/Code/src/`:
```bash
uv sync
```
(Dependencies are declared in `pyproject.toml`; use `uv add <pkg>` to add new ones.)

### Run
HTTP server:
```bash
uv run main.py --serve
```
Server runs at http://localhost:8000/mcp

## MCP Tools
- `list_menu()` - returns menu items with prices
- `place_order(customer_id, items: List[OrderItem])` - places order; validates items exist in menu. Items require `item_name`, `quantity` (>0). Always call `list_menu` first.
- `order_status(order_id: int)` - retrieves order status

## Database
Uses SQLAlchemy with PostgreSQL (`postgresql+psycopg2://`). Models/migrations under `models/` and `migrations/`. Run migrations as needed with Alembic.

## Notes
- Price is calculated as `sum(item_price * quantity)`. 
- Orders are created atomically; order items are saved per item.
- If an item is not found in the menu, a friendly `CustomAppException` is raised.
