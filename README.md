# codex-proxy on Dokploy

A minimal docker-compose to run [`icebear0828/codex-proxy`](https://github.com/icebear0828/codex-proxy)
behind your Dokploy/Traefik with a domain. Gives you an OpenAI-compatible
endpoint (`/v1/chat/completions`, `/v1/models`, ...) backed by your ChatGPT login.

## Deploy

1. Point a DNS record (e.g. `codex.example.com`) at your Dokploy host.
2. In Dokploy: **Create → Compose**, link this repo (or paste the compose).
3. Set env vars: copy `.env.example` to `.env` and set at least `DOMAIN`.
4. Deploy. Then open `https://<DOMAIN>` and **log in** (OAuth via the dashboard,
   or paste `CODEX_JWT_TOKEN` in `.env` to skip the interactive flow).
5. Point your apps at `https://<DOMAIN>/v1` with any API key the proxy issues.

## Three things that will actually bite you

- **Architecture.** Mac Mini = Apple Silicon → `CODEX_ARCH=arm64`. If the
  `:latest` image is amd64-only it'll run emulated; check with
  `docker compose exec codex-proxy uname -m`. If it says `x86_64`, set
  `CODEX_ARCH=x64`.
- **Cert resolver name.** This compose assumes Dokploy's Traefik resolver is
  named `letsencrypt`. If your Dokploy uses a different name, change it in the
  compose labels — or skip the labels and add the domain via the Dokploy
  **Domains** tab, which injects the correct ones automatically.
- **Login on a headless host.** The OAuth flow wants a browser. Easiest remote
  path: log in once via the web dashboard at `https://<DOMAIN>`, or paste a
  `CODEX_JWT_TOKEN`. The `data/` volume persists the session so you only do
  this once.

## Notes

- State (auth, accounts, quota) lives in `./data` and `./config` — back these up.
- This routes only the `8080` web/API port publicly. The proxy's internal
  OAuth callback port (`1455`) is intentionally **not** exposed.
- ⚠️ This uses your ChatGPT subscription as an API, which OpenAI's ToS doesn't
  sanction — keep it single-user and don't share the endpoint.
