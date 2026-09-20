# Obsidian Server

MCP server that exposes two Obsidian vaults — `home` and `work` — to LLM
clients over streamable HTTP. Clients read, search, write, edit, and delete
vault files, and run a git sync workflow (commit / pull / push) against the
`main` branch of each vault's GitHub repo.

## Prerequisites

- Two GitHub repos (one per vault), each with a `main` branch.
- A GitHub token with **read + write** on both repos (a fine-grained PAT
  scoped to the two repos with *Contents: Read and write* is enough).

## Configuration

| Var | Purpose |
|---|---|
| `VAULTS` | `home=<home-repo-url>,work=<work-repo-url>` — clone URLs |
| `GH_TOKEN` | GitHub token with read/write on both repos |
| `MCP_AUTH_TOKEN` | Static bearer token MCP clients must send; required |
| `MCP_PORT` | Host port, default `8080` |
| `LOG_LEVEL` | `debug \| info \| warn \| error \| silent`, default `info` |

On first run both vaults are cloned into the `vaults-data` volume; restarts
fast-forward-pull.

## Ports

The container listens on `8080`. The port is bound to `127.0.0.1` on the
host: the server speaks plain HTTP and is meant to be reached through a
reverse proxy or `tailscale serve`, e.g.

```sh
sudo tailscale serve --bg --https=443 --set-path=/mcp http://127.0.0.1:8080
```

The MCP endpoint is then `https://<tailscale-hostname>/mcp`.

## MCP client config

```json
{
  "mcpServers": {
    "obsidian": {
      "url": "https://<tailscale-hostname>/mcp",
      "headers": { "Authorization": "Bearer <MCP_AUTH_TOKEN>" }
    }
  }
}
```

Locally: `http://127.0.0.1:<MCP_PORT>/mcp`. The token travels only in the
`Authorization` header, never in the URL.

## Security notes

- `GH_TOKEN` and `MCP_AUTH_TOKEN` must be set in the Arcane project
  environment; the server refuses to start without `MCP_AUTH_TOKEN`.
- Keep the port bound to `127.0.0.1` and terminate TLS at a reverse proxy
  or Tailscale HTTPS. Do not expose `8080` publicly.
- The image is public — never bake secrets into it; all secrets arrive at
  runtime via the environment.

## Upstream

- Image: https://hub.docker.com/r/thijs/obsidian-server
- MCP spec: https://modelcontextprotocol.io
