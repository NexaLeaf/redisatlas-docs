# RedisAtlas v1.1.0 — Release Notes

**Release date:** 2026-04-20  
**Branch:** `feat/v1-2` → `main`  
**Tag:** `v1.1.0`

---

## What's New

### Developer Tools

**Key Value Diff Viewer** (`/diff`)  
Compare any two Redis key values side-by-side, or compare the same key at two points in time. Snapshots are captured manually and stored in localStorage (up to 50). Diff is computed client-side using the Myers LCS algorithm and displayed as a unified +/- view with colour-coded lines.

**Full Dataset Snapshot — Export & Restore** (`/snapshot`)  
Export your entire Redis database as a portable `.ndjson` file (key, type, value, TTL per line). Restore to any Redis instance with overwrite or skip-on-conflict strategy. Snapshot and restore are streamed server-side; no memory ceiling on large databases.

**CLI Playground — Multi-Tab & Inline Docs** (`/console`)  
The CLI console now supports multiple independent tabs, each with its own command history. 35 Redis commands are documented inline — start typing a known command and a docs bar appears with the signature, description, and complexity. Tab bar lives at the top; add, close, and switch tabs without losing history.

---

### Observability

**Command Usage Analytics** (`/command-analytics`)  
Live leaderboard of Redis command call rates (calls/sec), updated every 10 seconds. Top 5 commands show SVG sparklines tracking rate history. The full table shows total calls, average latency (µs/call), and share of total traffic. Sortable by calls or latency.

**Prometheus Metrics Endpoint**  
All RedisAtlas-collected Redis metrics are now exposed at `GET /api/connections/:id/metrics/prometheus` in Prometheus exposition format, scrapeable by any Prometheus-compatible collector.

**Grafana Dashboard Bundle**  
Pre-built Grafana dashboard JSON (`docs/grafana/redisatlas-overview.json`) covering memory, ops/sec, hit rate, connected clients, evictions, keys, latency, and replication lag. Uses `$DS_REDIS_PROMETHEUS` datasource variable and `$connection` filter variable for multi-cluster support. Import directly into Grafana via the dashboard JSON import UI.

---

### Dashboard & UI

**Dashboard status bar**  
The connection header now shows Redis version (pill), mode (standalone / cluster / sentinel), an inline memory usage progress bar, live ops/sec counter, and a pulsing green live indicator dot.

**Metric card trend indicators**  
Each metric card on the dashboard now shows a ▲/▼ trend arrow and delta value comparing the current WebSocket tick to the previous tick, colour-coded green (improving) or red (degrading).

**Upgraded memory chart**  
The Memory chart uses a filled area chart with a green gradient. The Ops/Clients chart now has dual Y-axes so large ops counts no longer flatten the clients line.

**Hit vs. Misses chart**  
A new stacked area chart on the dashboard shows cache hits (green) and misses (red) over time, built from the live WebSocket stream.

**Analytics improvements**
- Command Breakdown section now has a **Chart / Table** toggle. The table view is sortable by Calls or Avg µs/call and shows each command's share of total traffic.
- Command bar chart is now fully responsive (was broken on narrow screens due to a fixed `width=600` prop).
- Latency event table rows are now colour-coded: `bg-red-500/10` for events ≥ 10 ms, `bg-yellow-500/10` for 1–10 ms, no background for < 1 ms.

---

### Backend Architecture (NestJS Refactor)

This release includes a full NestJS infrastructure refactor. No API routes changed; this is internal structure only.

| Area | Change |
|---|---|
| `common/decorators/` | New `@CurrentUser()` and `@ConnectionId()` param decorators; `@Roles()` moved from `auth/` |
| `common/filters/` | `HttpExceptionFilter` moved here; new `AllExceptionsFilter` catches unhandled errors and returns `{ statusCode: 500, code: "INTERNAL_ERROR" }` |
| `common/guards/` | `AuthGuard`, `RolesGuard`, `GuardsModule` moved from `auth/guards/` |
| `common/interceptors/` | New `LoggingInterceptor` (per-request log with timing and X-Request-ID); new `TransformInterceptor` (wraps all JSON responses in `{ data, meta: { timestamp, requestId } }`); `@SkipTransform()` opt-out decorator |
| `common/middleware/` | New `RequestIdMiddleware` — attaches `X-Request-ID` header to every request |
| `common/pipes/` | New `ParseConnectionIdPipe` — validates `connectionId` params are non-empty |
| `keys/` | `CommandController` and `PubSubController` split into separate files |
| `*/dto/` | All DTOs moved into `dto/` subfolders within their modules |
| `main.ts` | Global interceptors and correct filter order (`AllExceptions → HttpException`) registered |

**Frontend API envelope**: All JSON API responses are now wrapped in `{ data: T, meta: { timestamp, requestId } }`. The `apiFetch()` helper in `api.ts` auto-unwraps `.data` — no call sites changed.

---

### ESLint Setup

Both `apps/backend` and `apps/frontend` now have working ESLint configurations (`eslint.config.js`, flat config format). Zero errors on both. The lint scripts in `package.json` are updated to work with ESLint v9+ flat config.

---

## Bug Fixes

- Fixed ternary-as-expression lint error in `explorer/page.tsx`
- Fixed unused `Header` import in `metrics.controller.ts`
- Fixed stale DTO import path in `auth.service.ts`
- Resolved dual `rxjs` version conflict between workspace root and `apps/backend` (removed duplicate local install)

---

## Upgrading

**Docker** (auto via tag):
```bash
docker pull redisatlas/redisatlas:1.1.0
docker pull redisatlas/redisatlas:latest
```

**Desktop** — download the installer for your platform from the GitHub Release assets:
- macOS: `RedisAtlas-1.1.0.dmg` / `RedisAtlas-1.1.0-mac.zip`
- Windows: `RedisAtlas-Setup-1.1.0.exe` / `RedisAtlas-1.1.0-win-portable.exe`
- Linux: `RedisAtlas-1.1.0.AppImage` / `redisatlas_1.1.0_amd64.deb` / `redisatlas-1.1.0.x86_64.rpm`

**Self-hosted from source**:
```bash
git pull && git checkout v1.1.0
npm run install:all
npm run build
```

---

## Steps to Publish This Release

1. Commit the 4 remaining modified frontend files on `feat/v1-2`
2. Merge `feat/v1-2` → `main` (PR or direct)
3. Bump version in `package.json`, `apps/frontend/package.json`, `apps/backend/package.json`, `apps/desktop/package.json` to `1.1.0`
4. Tag `v1.1.0` on `main` → triggers Docker multi-arch build automatically
5. Run `desktop-build` workflow manually (`workflow_dispatch`) to produce platform installers and GitHub Release assets
