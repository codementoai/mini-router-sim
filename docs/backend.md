# Backend

The backend is implemented in `backend/` using FastAPI and SQLAlchemy with asynchronous SQLite support.

## Key files

- `backend/main.py` — FastAPI app with route handlers and static frontend mounting
- `backend/database.py` — async SQLAlchemy engine and session factory
- `backend/models.py` — ORM models for interfaces, routes, and config entries
- `backend/routeros.py` — router simulation logic, OSPF neighbor handling, routing table computation
- `backend/schemas.py` — Pydantic models for request/response validation

## Architecture

On startup, the backend:

1. Creates the SQLite database tables via SQLAlchemy models
2. Loads persisted interfaces, routes, and config from the database into the in-memory router state

The backend exposes REST APIs for management and simulation endpoints.

## Router simulation

`backend/routeros.py` contains the `RouterOS` class, which maintains:

- `interfaces`: active virtual interfaces
- `static_routes`: persisted route entries loaded from SQLite
- `ospf_routes`: in-memory learned routes from the OSPF simulation
- `routes`: active merged routing table used for forwarding
- `config`: key/value configuration stored in memory and persisted as needed
- `ospf_neighbors`: a neighbor database containing advertised networks and path costs

### Routing behavior

- Static routes are persisted in SQLite and merged into the active routing table.
- The routing table is sorted by longest prefix and then lowest metric.
- Packet forwarding uses the active merged routing table to find the best route.

### OSPF simulation

This is a simple, in-memory simulation:

- Neighbors are added via API and given an ID, cost, and list of advertised networks
- When OSPF is run, the router learns new routes from each neighbor advertisement
- Learned routes are stored in `ospf_routes` and merged into the active routing table
- The router selects the best route using longest-prefix matching, then lowest `metric` among equal prefixes

This makes the simulation behave like a shortest-path selection among OSPF paths. The backend now supports multi-hop neighbor links, so route learning uses a link-state graph of neighbors and computes lowest-cost paths through intermediate routers.

For example, if `r2` is linked to `r3`, and `r3` advertises a network, the router can learn that route through `r2` if the total path cost is lower than the direct cost to `r3`.

## Persistence

Persistent data is stored in `backend/routeros.db` by default.

- `interfaces` table stores virtual interface definitions
- `routes` table stores static routing entries
- `configs` table stores arbitrary key/value configuration

## Running the backend

From the repository root:

```powershell
cd backend
python -m venv .venv
.\.venv\Scripts\Activate
pip install -r requirements.txt
uvicorn backend.main:app --reload --host 0.0.0.0 --port 8000
```

Then open `http://localhost:8000` to access the frontend UI.
