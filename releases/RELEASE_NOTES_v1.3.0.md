# RedisAtlas — Release Notes v1.3.0

**Released:** April 2026  
**Distribution:** Docker image · macOS DMG · Windows NSIS · Linux AppImage/DEB/RPM

---

## Overview

RedisAtlas v1.3.0 is a major feature release delivering 28 fully implemented pages across every area of Redis operations — from day-to-day key management through deep diagnostic analysis, cluster topology inspection, stream processing, alerting, and team access control. This release also brings the full dependency stack up to their latest versions including Tailwind CSS 4, recharts 3, lucide-react 1.x, TypeScript 5.9, and TypeScript 6-ready configuration.

---

## Feature Highlights

### Real-Time Dashboard

Live metrics overview streaming directly from Redis over WebSocket. A status bar shows Redis version, connection mode (standalone / cluster / sentinel), and a memory utilization bar with percentage. Metric cards display ops/sec, total keys, memory used, hit rate, connected clients, evicted keys, and uptime — each with a delta trend arrow showing change since the last tick. A stacked area chart plots cache hits vs misses in real-time. All data updates continuously without manual refresh.

### Key Explorer

Full-featured key browser with cursor-based SCAN pagination safe for production databases of any size. Keys render in a virtualised list so performance stays consistent whether browsing 1,000 or 10,000,000 keys. Pattern filter with glob support (`user:*`, `session:??:*`). Checkbox multi-select for bulk delete and bulk TTL operations. Per-key actions: rename, set/clear TTL, view full value. Import keys from JSON, CSV, or Redis protocol files. Export selected keys or the entire matched set as JSON.

**Key Inspector (new in v1.3):** An "Inspect" slide-over panel reveals internal Redis metadata for any selected key — data type badge, memory encoding, idle seconds since last access, and type-specific detail:
- **Hash** — field count, sample of up to 20 field→value pairs
- **List** — length, first 10 and last 10 elements
- **Set** — cardinality, sample of up to 20 members
- **Sorted Set** — cardinality, min/max score, sample of up to 20 members with scores

### Analytics

Multi-section analytics page combining live stream data with historical trends.

- **Historical charts** — ops/sec, memory %, hit rate %, and connected clients over selectable 1h / 6h / 24h windows
- **Memory Growth Tracker** — linear regression over the historical window extrapolates a projected growth line with OOM ETA when Redis has a `maxmemory` limit set
- **Latency Percentiles (new in v1.3)** — fetches `LATENCY HISTORY` for every event reported by `LATENCY LATEST` and computes P50, P95, P99, and Max client-side. A colour-coded histogram bins all samples into six ranges (< 1 ms green → > 100 ms red)
- **Eviction Rate Monitor (new in v1.3)** — delta/sec calculation between WebSocket ticks replaces the previous cumulative counter. Reference lines at 100/s (orange) and 1,000/s (red) highlight spikes
- **Hit Rate** — donut chart showing hit/miss ratio with percentage label
- **Key Namespaces** — horizontal bar chart of top 10 prefixes, plus a scrollable list of all detected namespaces with proportional bars and copy buttons
- **Hot Keys** — LFU frequency sampling table with copy-to-clipboard
- **Command Breakdown** — switchable chart / table view of top commands by call count, average latency (µs), and share %; table columns are sortable

### Command Analytics

Dedicated real-time command analytics page with animated bar chart tracking the top 10 commands by current calls/sec. Sparklines for the top 5 commands over the last 20 samples. Summary cards for unique command count, total calls, and peak command rate.

### Diagnostics

Redis `INFO` section browser with multi-section navigation (server, clients, memory, persistence, stats, replication, cluster, keyspace, and all). Health indicators flag fragmentation, RDB/AOF state, and missing replicas. Full-text search within any section. BGSAVE and BGREWRITEAOF trigger buttons for admin users.

### Console

Interactive Redis CLI embedded in the browser. Multi-tab sessions, command history with arrow-key navigation, starred-command bookmarks, and inline documentation covering 30+ commands (syntax, description, example). History panel with search and filter.

### Slow Query Log

Slow query table with command text, execution time, timestamp, and client address. Colour-coded latency (yellow > threshold, red > 100 ms). Copy-to-clipboard for reproducing slow queries. Configurable fetch count.

### Pub/Sub Monitor

Subscribe to channels and patterns simultaneously. Live message feed with timestamps. Inline publish panel for testing message flows. Dynamic add/remove of subscriptions.

### Live Monitor

Real-time `MONITOR` command stream with colour-coded command types (reads = blue, writes = green, deletes = orange). Filter by command substring, pause/resume without dropping data, configurable buffer (100–2,000 rows), live commands/sec counter. Production-impact warning banner.

### Streams

Full Redis Streams management: browse stream keys, view and add entries, create and delete consumer groups, inspect consumers (idle time, inactive duration), view and ACK pending messages individually or in bulk. Stream metadata header shows length and group count.

### Memory Size Analysis

Scans up to a configurable sample size and ranks keys by `MEMORY USAGE`. Top-keys table with type badges. Namespace memory breakdown chart showing which prefixes consume the most RAM.

### Schema Discovery *(new in v1.3)*

Scans a configurable sample of keys (100 / 500 / 2k / 10k) and infers key naming templates by replacing UUID, numeric, and long alphanumeric segments with typed placeholders (`{uuid}`, `{id}`, `{key}`). Results display as a card grid — each card shows the template with placeholders highlighted in green, the occurrence count, data type pills, and an example key with a copy button. Useful for understanding keyspace structure in unfamiliar databases.

### Memory Fragmentation Analyzer *(new in v1.3)*

Reads `INFO memory` to compute the fragmentation ratio (`used_memory_rss / used_memory`), used memory, RSS memory, and wasted bytes. A large colour-coded gauge (green < 1.2 · yellow 1.2–1.5 · orange 1.5–2.0 · red > 2.0) gives an at-a-glance health signal. A contextual recommendation explains the state and suggests remediation (`MEMORY PURGE`, restart scheduling, swap investigation). A dual-axis area chart tracks the ratio and both memory values over up to 60 historical samples with optional 5-second auto-refresh.

### Config Viewer

Full `CONFIG GET *` output in a searchable, copy-friendly table. Filter by parameter name or value.

### Large Keys Detector

Scans a sample (5k–50k keys) and returns the top 25/50/100 largest keys by `MEMORY USAGE`. Horizontal bar chart for the top 15 with percentage-of-largest labels. Full sortable table with type badges.

### Hot Keys Detector

LFU-based access frequency sampling. Ranked list with frequency bars colour-coded by access intensity. Explains the `maxmemory-policy` LFU requirement inline.

### TTL Inspector

Configurable expiry window (1 min / 5 min / 10 min / 1 h / 1 day) returns all keys expiring within that window. Distribution histogram across 7 TTL buckets. Inline quick-extend buttons (1 h, 1 d, 1 w) and a custom seconds input. Persist action removes TTL entirely.

### Key Patterns

Hierarchical namespace tree built from the live keyspace. Expand prefix nodes to reveal sub-namespaces up to three levels deep. Each node shows key count, percentage of total, and a "Browse" shortcut that opens Explorer pre-filtered to that pattern.

### Cluster Topology

Full cluster topology view showing master nodes with their replica assignments, slot distribution, and connection status. Cluster INFO parameter table. Graceful standalone-mode detection with a clear explanatory message when `cluster_enabled = 0`.

### Replication

Role-aware view. On a master: replica table with address, state (online/offline), replication offset, and a byte-lag bar gauge. Redundancy alert when no replicas are connected. On a replica: master address, link status, replication offset, and relative lag. Replication ID displayed for PSYNC diagnostics.

### Persistence Health

RDB snapshot status — last save timestamp, age, duration, success/failure, BGSAVE-in-progress indicator, and a freshness gauge (fresh / aging / stale). AOF status — enabled state, current size vs base size, rewrite status. RPO estimate ("if Redis crashed now, you would lose X of data"). BGSAVE and BGREWRITEAOF trigger buttons.

### Client List

Live client table (5-second auto-refresh) with ID, address, name, DB, last command, age, and idle time. Human-readable flag badges (blocked, replica, master, pubsub, readonly). Dedicated blocked-clients alert panel. Filter by address, command, or ID; "Blocked Only" toggle.

### Key Watch

Keyspace notification event stream. Detects current notification flags (`KEA` etc.) and offers one-click enable. Watch rules combine a key pattern with a target DB. Real-time event feed with timestamp, DB index, event type, and key name. Virtual list keeps the feed responsive under high event volume.

### Key Diff

Point-in-time snapshot comparison tool. Capture the value of any key into a local history (up to 50 entries). Select any two snapshots for side-by-side diff with `+`/`-` line counts. Delete individual snapshots from history.

### Dataset Snapshot

Export an entire Redis DB as NDJSON (newline-delimited JSON) — one key per line with type, TTL, and value. Import NDJSON back with an optional overwrite flag. Restore report shows key counts for restored, skipped, and errored entries.

### Audit Log

Tamper-evident log of all user actions with timestamp, actor email, action type, resource type, resource ID, and client IP. Filter by action (create / update / delete / login / logout). Full-text search. Paginated 50 entries per page with colour-coded action badges.

### Alert Rules

Configurable threshold alerts on any key metric (ops/sec, memory %, connected clients, hit rate, evictions, key expiry rate, total keys). Operators: `>`, `<`, `>=`, `<=`, `==`. Cooldown and minimum-duration settings to suppress transient spikes. Notification channels: internal log or outbound webhook (Slack-compatible JSON payload). Enable/disable toggle per rule. Last-fired timestamp displayed.

### User Management

Role-based access control with `admin` and `viewer` roles. Admins have access to all write operations; viewers can browse but cannot modify data or connection settings. In the desktop build, a simplified profile editor handles single-user name and password changes. In the web/Docker build, a full user management table allows admins to create, edit, and delete accounts.

---

## Dependency Upgrades

All direct dependencies updated to their latest major versions:

| Package | Previous | Updated |
|---|---|---|
| tailwindcss | 3.4.17 | 4.x |
| recharts | 2.15.1 | 3.x |
| lucide-react | 0.483.0 | 1.x |
| eslint (frontend) | 9.x | 10.x |
| typescript | 5.8.2 | 5.9.3 |
| bcryptjs | 2.4.3 | 3.0.3 |
| uuid | 11.1.0 | 14.0.0 |
| @types/node | 22.x | 25.x |
| class-validator | 0.14.1 | 0.14.4 |

**Tailwind CSS 4 migration** — PostCSS config migrated to `@tailwindcss/postcss`, globals.css updated to `@import "tailwindcss"`, all deprecated utility classes renamed (`flex-shrink-0` → `shrink-0`, `outline-none` → `outline-hidden`, etc.) across the full source tree.

---

## Distribution

### Docker

```bash
docker pull redisatlas/redisatlas:1.3.0
docker pull redisatlas/redisatlas:latest
```

Runs nginx (ports 80/443), Next.js frontend, and NestJS backend under supervisord in a single container. Multi-arch: `linux/amd64` + `linux/arm64`.

```bash
docker run -p 8080:80 \
  -e JWT_SECRET=your-secret \
  -e ADMIN_PASSWORD=your-admin-password \
  redisatlas/redisatlas:1.3.0
```

### Desktop

Auto-updating Electron application. Downloads available from the GitHub Releases page:

| Platform | Format |
|---|---|
| macOS (Apple Silicon + Intel) | `.dmg`, `.zip` |
| Windows | NSIS installer `.exe`, portable `.exe` |
| Linux | `.AppImage`, `.deb`, `.rpm` |

The desktop app embeds the NestJS backend in-process and requires no separate server. Data is stored in the OS user-data directory and persists across updates.

---

## Environment Variables

| Variable | Default | Purpose |
|---|---|---|
| `JWT_SECRET` | random | JWT signing key — set a stable value for production |
| `ADMIN_PASSWORD` | — | Initial admin password on first boot |
| `DATABASE_URL` | embedded PGLite | External PostgreSQL URL for multi-instance deployments |
| `REDIS_CACHE_URL` | in-memory | Redis URL for server-side cache backend |
| `CORS_ORIGIN` | `*` | Restrict allowed origins |
| `PORT` | `3001` | NestJS API listen port |

---

## Upgrading from v1.0.x

No database migration is required. User accounts and connection profiles stored in the embedded PGLite database are forward-compatible.

Docker:
```bash
docker pull redisatlas/redisatlas:1.3.0
# restart your container with the new image
```

Desktop: The auto-updater will prompt on next launch. Or download the installer directly from GitHub Releases.

---

*RedisAtlas is open-source. Issues and contributions welcome at github.com/NexaLeaf/redisatlas.*
