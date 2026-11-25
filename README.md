# portfolio-api

[![Go](https://img.shields.io/badge/go-00ADD8?style=for-the-badge&logo=go&logoColor=white)](https://go.dev/)
[![Vercel](https://img.shields.io/badge/vercel-000000?style=for-the-badge&logo=vercel&logoColor=white)](https://vercel.com/)

## About

The API behind the stats cards on [tchanwangsa.xyz](https://tchanwangsa.xyz), served at `api.tchanwangsa.xyz`. It also holds `telemetry`, a macOS collector that publishes my coding-agent token usage to the API.

## Run it

```sh
cp server/.env.dev.example server/.env.dev    # then set STATS_PUBLISH_TOKEN
go run ./server -env-file server/.env.dev     # http://127.0.0.1:8787
```

To run the collector on a Mac:

```sh
npm install -g tokscale
cp telemetry/.env.example telemetry/.env      # then set STATS_PUBLISH_TOKEN
go run ./telemetry install -env-file telemetry/.env
```

`go run ./telemetry status` shows what it's doing, and `go run ./telemetry uninstall` stops it.
