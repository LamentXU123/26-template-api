# Validation record

Date: 2026-09-21. Environment: Windows x64, Bun 1.4.2, dependencies pinned in `bun.lock`, real disposable MongoDB launched by `mongodb-memory-server`.

| Check | Result |
| --- | --- |
| `bun run compile` | Passed: source and test TypeScript projects |
| `bun run check` | Passed: Biome formatting, import ordering and lint |
| `bun run test` | **61 passed, 0 failed**, 8 test files, 244 expectations |
| Reported in-process coverage | 97.01% lines, 94.41% functions |
| Production entrypoint | Passed over actual loopback HTTP with `NODE_ENV=production` |
| Restart persistence | Created an event, restarted the API against the same MongoDB, read the event back |
| Production credentials | Configured user accepted; default development token rejected |
| Calendar interoperability check | Export parsed by independent `ical.js`; Unicode, escaping and UTC instants roundtrip |
| Compose YAML | Parsed with Bun's YAML parser; API/database services and named volume present |
| Base-image availability | `oven/bun:1.4.2-slim` and `mongo:8.3.4` tags confirmed active through Docker Hub API |
| Docker build / Compose startup | **Not run: no Docker engine/CLI in Windows or the installed WSL distribution** |

Coverage reports measure code loaded by the main test process. The separate server subprocess is behavior-tested but is not included in that in-process coverage denominator. Coverage is evidence of exercised paths, not a claim that every edge case is covered.

The tests cover successful and rejected CRUD, ownership on list/detail/update/delete/export, all endpoint authentication requirements, forbidden fields, malformed dates/IDs, nonblank and bounded text, numeric query bounds, adjacent/spanning intervals, null clearing, stale and competing updates, empty/private exports, export size limits and OpenAPI registration. The production test verifies application restart persistence; it does not substitute for a Docker volume restart test.

Nine additional regression cases cover CORS preflight and subsequent authenticated PATCH/DELETE requests, rejected UTC-year overflow/underflow on creation, unchanged data after a rejected PATCH, consistent validation of list/export bounds, and calendar parsing at supported year boundaries. The preflight and out-of-range validation failures were reproduced before the fix. For early years, calendar fields are compared directly because the parser's JavaScript Date conversion maps years 0–99 into 1900–1999; the parsed iCalendar fields themselves retain the correct year.

## Remaining environment-specific verification

On a machine with Docker Engine and Compose v2:

```sh
cp .env.example .env
docker compose config --quiet
docker compose up --build -d
docker compose ps
curl http://localhost:3000/health
```

Use the `.env` tokens to create an event. Restart with `docker compose restart api`, then verify it can still be read. Also stop/start MongoDB to verify the named volume in that environment. Import an exported `.ics` file into your preferred calendar client. No claim is made that those manual environment/client checks have already been performed.
