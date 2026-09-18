# Runtime — Delivery (delivery-bff)

> From `package.json`, `.env.template`, `tsconfig.json` and `src/`. Tier 4 (code reading).

| | |
|---|---|
| Stack | Node + TypeScript 5.3, Express 4.18, axios 1.6 |
| Version | `1.3.0` (package.json) |
| Node version | README says Node 18, service registry says Node 16 — unresolved; no `engines` field, no `.nvmrc`, no Dockerfile |
| Port | 8083, hardcoded in `src/index.ts` (`app.listen(8083)`) |
| Build / run | `tsc` then `node dist/index.js` |
| DB | none (matches the registry) |
| Deploy | no CI workflow, no Dockerfile in the repo |

**Env vars.** `.env.template` declares `ORDER_API_BASE`, `CARRIER_API_BASE` and `RETRY_COUNT`.
The code reads **none of them**: `src/deliveryStatus.ts` reads `process.env.CARRIER_API` (a
different name, defaulting to `https://api.carrier.example`) and hardcodes `MAX_RETRY = 3`;
`src/generated/orderApi.ts` issues a relative `fetch('/api/v1/orders/...')` with no base URL at all.
Every declared configuration knob is inert.

## Outbound behaviour

- Carrier tracking: `GET ${CARRIER_API}/tracking/{ordNo}`, 3s timeout, 3 attempts, 500ms×attempt
  backoff. On exhaustion it returns HTTP 200 with `{status: 'PREPARING', stale: true}` and no
  alert (`3회 실패 시 포기한다. 별도 알림은 없다.`). Carrier outages are therefore invisible to
  monitoring and surface to users as "준비중".
- order-service cancel: via the 2022-generated client, path `/api/v1/orders/{ordNo}/cancel`, which
  does not match today's order-service route. `requestCancel` is exported but called from nowhere
  in this repo.

## Ownership

Unsettled. `services.yaml`: `owner_team: TODO   # 물류팀 이관 논의 중 (2025-11~), 확정 전`, while
`README.md` and `package.json` both claim 커머스본부 물류팀.
