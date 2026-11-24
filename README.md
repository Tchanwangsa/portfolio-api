# portfolio-api

The API behind the live stats cards on [tchanwangsa.xyz](https://tchanwangsa.xyz), and the collector that reports coding-agent token usage to it. The site lives in [portfolio-v3](https://github.com/Tchanwangsa/portfolio-v3).

| Path | Contents |
| --- | --- |
| `portfolioapi/` | API handlers and storage |
| `api/`, `server/` | API entry points for Vercel and local use |
| `telemetry/` | Agentic coding collector |
| `internal/` | Snapshot format and settings loader shared by the API and collector |

## Develop

Needs Go 1.25 or newer.

```sh
cp server/.env.dev.example server/.env.dev          # then set STATS_PUBLISH_TOKEN
go run ./server -env-file server/.env.dev           # API on http://127.0.0.1:8787
go run ./telemetry run -env-file server/.env.dev    # publishes a snapshot to it
```

Check changes with `go test ./...`.

## API

| Route | Purpose |
| --- | --- |
| `GET /api/agentic-stats` | Latest token usage snapshot |
| `PUT /api/agentic-stats` | Publish a snapshot. Needs `Authorization: Bearer <STATS_PUBLISH_TOKEN>` |
| `GET /api/github-stats` | Commit times from the last seven days and the latest public commit, cached for 30 minutes |
| `GET /api/runs` | Runs from the last eight weeks and the latest run |
| `POST /api/runs` | Record a run. Needs `Authorization: Bearer <RUNS_WEBHOOK_TOKEN>` |
| `GET /healthz` | Returns `204` while the API is up |

The site groups commit and run times into the viewer's local days and weeks.

### Settings

| Variable | Purpose |
| --- | --- |
| `STATS_PUBLISH_TOKEN` | Required. At least 32 random characters |
| `STATS_ALLOWED_ORIGINS` | Comma-separated origins allowed to read the API from a browser |
| `GITHUB_USERNAME`, `GITHUB_TOKEN` | Turn on the GitHub route. Use a fine-grained token with **Public repositories** selected and no added permissions |
| `GITHUB_PRIVATE_TOKEN` | Optional. Fine-grained token for the private repositories to count. Only their commit times are published, rounded down to 15 minutes |
| `RUNS_WEBHOOK_TOKEN` | Turns on the runs routes. At least 32 random characters, different from the publish token |
| `BLOB_STORE_ID`, `BLOB_READ_WRITE_TOKEN` | Required on Vercel. Private Vercel Blob store for API data |
| `STATS_DATA_FILE` | Local storage path instead of Blob. Defaults to `data/agentic-stats.json`, with runs saved beside it |
| `STATS_LISTEN_ADDR` | Local server address. Defaults to `127.0.0.1:8787` |

### Strava runs

An IFTTT applet posts each new Strava activity to `POST /api/runs` with this body:

```json
{
  "name": "{{Name}}",
  "activityType": "{{ActivityType}}",
  "createdAt": "{{CreatedAt}}",
  "distanceMeters": "{{DistanceMeters}}",
  "elapsedTimeInSeconds": "{{ElapsedTimeInSeconds}}",
  "linkToActivity": "{{LinkToActivity}}",
  "routeMapImageUrl": "{{RouteMapImageUrl}}"
}
```

`createdAt` is wall-clock time where the run happened; the API gets its time zone from the start of the route map. Runs without a map, such as treadmill runs, use the latest mapped run's time zone, or Melbourne if there is none. The API ignores non-run activities and duplicate deliveries.

## Deploy

The Vercel project serves `https://api.tchanwangsa.xyz`: `api/portfolio.go` is the function, and `vercel.json` routes the paths above to it. Pushing to `main` deploys. The project needs a private Blob store connected to Production and the settings above.

## Telemetry

`telemetry/` reads local usage from [tokscale](https://github.com/junhoyeo/tokscale) and publishes a seven-day snapshot to the API: tokens per day, total tokens, session count, and most-used model. Nothing else leaves the machine.

Set it up on macOS:

```sh
npm install -g tokscale
cp telemetry/.env.example telemetry/.env    # then set STATS_PUBLISH_TOKEN
go run ./telemetry install -env-file telemetry/.env
```

`install` publishes once, copies the collector and its settings to `~/.local/share/portfolio-telemetry`, and schedules it with launchd to run every 15 minutes and at login.

| Command | Effect |
| --- | --- |
| `go run ./telemetry status` | Show schedule, settings, latest snapshot, and last error |
| `go run ./telemetry run` | Collect and publish once |
| `go run ./telemetry uninstall` | Stop the collector; keep settings so `install` resumes without flags |
| `go run ./telemetry uninstall -purge` | Stop the collector and delete its settings and data |

The site hides the agentic coding card three days after the last snapshot.
