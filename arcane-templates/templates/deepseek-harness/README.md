# DeepSeek Harness

DeepSeek Harness (DSH) is an agent harness distributed as the npm package
`@deepseek-ai/dsh`. DSH officially only listens on loopback, so this image
(`smanx/deepseek-harness`) bundles a Node reverse proxy that forwards
HTTP and WebSocket traffic from the container network to DSH inside the
container.

## Before deploying

1. Set `DSH_PROXY_USERNAME` and `DSH_PROXY_PASSWORD` in Arcane's project
   environment. Basic auth is only enabled when **both** values are set; if
   either is empty, access is completely open, which is not recommended for
   a LAN-reachable AI agent endpoint.
2. Verify that `DSH_PROXY_HOST_PORT` (default `3080`) is free on the host.
3. Deploy the project and open `http://<host>:${DSH_PROXY_HOST_PORT}/` in a
   browser.
4. Configure your LLM API key (for example a DeepSeek key) from the DSH
   UI after deployment.

## Storage

DSH session and configuration data lives at `/root/.dsh` inside the
container and is persisted to `DSH_DATA_PATH`
(`/var/apps/deepseek-harness` by default). Back up that directory if your
sessions matter; removing it resets DSH to a fresh state.

## Networking

The proxy listens on `0.0.0.0:${DSH_PROXY_PORT}` inside the container and
forwards to DSH on `127.0.0.1:${DSH_PORT}`. WebSocket realtime channels are
forwarded automatically. The two ports must differ. If you change
`DSH_PROXY_PORT`, keep the host port mapping in sync.

## Images

`DSH_IMAGE_TAG` defaults to the slim `latest` runtime image. For
troubleshooting inside the container, use `devtools-min-latest`
(curl, jq, git, ...) or `devtools-latest` (plus vim, tmux, python3,
build tools). Matching versioned tags are available for reproducible
deployments. A GHCR mirror (`ghcr.io/smanx/deepseek-harness`) exists with
identical tags if Docker Hub pulls are restricted.

## Source

- [smanx/deepseek-harness on Docker Hub](https://hub.docker.com/r/smanx/deepseek-harness)
- [deepseek-ai/deepseek-harness on GitHub](https://github.com/deepseek-ai/deepseek-harness)
