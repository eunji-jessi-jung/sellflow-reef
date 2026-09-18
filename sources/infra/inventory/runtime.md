# Runtime — Inventory (inventory-api)

> From `Dockerfile`, `requirements.txt`, `.env.sample`, `app/config.py`, `app/db.py` and
> `alembic.ini`. Tier 4 (code reading).

| | |
|---|---|
| Stack | FastAPI 0.104.1, uvicorn 0.24.0, pymysql 1.1.0 |
| Version | `3.2.1` (`FastAPI(version=...)`) |
| Image | `python:3.9-slim`, `pip install -r requirements.txt`, `CMD uvicorn app.main:app --host 0.0.0.0 --port 8000` |
| Port | **8000** in the Dockerfile; order-service's `.env.example` expects inventory on 8082 locally |
| DB | `app/db.py` hardcodes `database="sellflow_order"` — the shared order instance, not a separate inventory DB |
| Migrations | two systems: `sql/V1__stock.sql` (Flyway-style, unmanaged) and `alembic/` (chain broken — base revision `1c22` missing). See `sources/schemas/inventory/schema.md`. |
| Deploy | no CI workflow in the repo |

**Env vars actually read:** `DB_HOST` (default `localhost`), `DB_USER` (default `sellflow`) in
`app/db.py`; `DB_URL` and `LOG_LEVEL` in `app/config.py`. `app/config.py`'s `DB_URL` is never
consumed by `app/db.py`, so `.env.sample`'s `mysql://inventory:@localhost:3306/inventory` has no
effect — the service connects to `sellflow_order` as user `sellflow`.

`RESTOCKABLE_REASONS = {"01", "02"}` is duplicated in both `app/main.py` and `app/config.py`;
`main.py` uses its own copy, so the config value is dead. This set is the single point where the
cancel-reason-to-restock policy lives, and it is not shared with order-service's `CancelReason`
enum in any machine-readable form.
