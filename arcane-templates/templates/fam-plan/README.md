# Fam Plan

Fam Plan is a self-hosted finance planner for a household. It imports bank and
card statement files, keeps a ledger of transactions with categories and labels,
and shows dashboards for spending, budgets, goals, and receipts. Agents use the
same data through a token-protected `POST /mcp` endpoint.

One container serves everything: the built Vue single-page app, the JSON API, and
the MCP endpoint. The app is PHP 8.4 (Symfony) behind FrankenPHP, and the queue
worker for background work runs inside the same container.

## Before deploying

1. Set `APP_SECRET` in the Arcane project environment. Generate one with
   `openssl rand -hex 32`; do not commit the real value.
2. Set `MCP_API_TOKEN` if agents will call the MCP endpoint.
3. Point `FAM_PLAN_DATA_PATH` at persistent storage and make sure the directory
   exists; the SQLite database lives in it.
4. Keep `FAM_PLAN_PORT` (default `18000`) free on the host.
5. Deploy the project, then run the database migrations once:

   ```sh
   docker compose exec fam-plan php bin/console doctrine:migrations:migrate --no-interaction
   ```

   Recreate the service afterwards so it picks up the new schema.

6. Open `http://<host>:${FAM_PLAN_PORT}`.

## Configuration

| Var | Purpose |
|---|---|
| `FAM_PLAN_IMAGE_TAG` | Image tag, default `latest` |
| `FAM_PLAN_CONTAINER_NAME` | Container name |
| `FAM_PLAN_PORT` | Host port mapped to container port `80` |
| `FAM_PLAN_DATA_PATH` | Host directory for the SQLite database |
| `APP_SECRET` | Required; Symfony secret for sessions and URL signing |
| `MCP_API_TOKEN` | Bearer token required by `POST /mcp` |
| `MESSENGER_TRANSPORT_DSN` | Queue transport, default `doctrine://default` |
| `SYMFONY_TRUSTED_PROXIES` | Set when running behind a reverse proxy |
| `MAX_REQUESTS` | FrankenPHP worker restart threshold, default `500` |

The image ships committed defaults in its `.env` file. Real environment
variables win over that file, so set every value above in the Arcane project
environment instead of relying on the baked defaults.

## Ports

The container listens on `80` and serves the SPA, the API (`/api/*`, `/auth/*`),
and the MCP endpoint (`/mcp*`) on that single port. The mapping publishes it on
`0.0.0.0`, so keep access restricted by the network.

## Storage

```text
/app/var/data   SQLite database (app.db) and the queue table
```

Back up `FAM_PLAN_DATA_PATH`; deleting it resets the app to an empty ledger.
Fixtures are only for development and are loaded from the source checkout, not
from the image.

## MCP

The endpoint is a JSON endpoint, not a spec-compliant streamable HTTP server.
Requests must carry the token in the header:

```sh
curl -s -X POST http://<host>:${FAM_PLAN_PORT}/mcp \
  -H 'Content-Type: application/json' \
  -H 'Authorization: Bearer <MCP_API_TOKEN>' \
  -d '{"tool":"tools.list","args":{}}'
```

Useful tools include `finance.dashboard`, `transaction.list`,
`finance.spending_by_category`, `finance.cashflow`, `review.next_transaction`,
and `review.apply_decision`.

## Security notes

- There is no login yet; the app serves all routes to anonymous users. Keep it
  on the LAN or behind Tailscale, and do not expose `FAM_PLAN_PORT` publicly.
- `APP_SECRET` and `MCP_API_TOKEN` are secrets: keep them in the Arcane project
  environment, never in committed files.

## Image

`thijs/fam-plan` is built from `Dockerfile.prod` in the Fam Plan source
repository by `scripts/deploy-production-image.sh`, which publishes the given
tag, a per-release tag, and `latest`. Use a versioned tag when reproducible
updates matter.

## Source

- [thijs/fam-plan on Docker Hub](https://hub.docker.com/r/thijs/fam-plan)
