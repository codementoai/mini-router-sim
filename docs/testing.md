# Testing

This project includes API tests using pytest and httpx.

## Dependencies

Install the backend dependencies, including test libraries:

```powershell
cd backend
python -m venv .venv
.\.venv\Scripts\Activate
pip install -r requirements.txt
```

The backend `requirements.txt` includes:

- `fastapi`
- `uvicorn`
- `sqlalchemy`
- `aiosqlite`
- `pydantic`
- `pytest`
- `httpx`
- `pytest-asyncio`

## Test files

- `tests/test_api.py` — API integration tests using FastAPI's ASGI app and HTTPX.

## Run tests

From the repository root:

```powershell
pytest -q
```

## What the tests cover

The existing tests verify:

- basic status API availability
- routing table behavior
- OSPF neighbor creation
- OSPF route learning via `/api/ospf/run`
- cleanup of neighbors after the test

## Notes

The tests run against the application in-memory through the ASGI app, so no separate server process is required.
