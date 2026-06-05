# codex-proxy on Dokploy

A minimal docker-compose to run [`icebear0828/codex-proxy`](https://github.com/icebear0828/codex-proxy)
behind your Dokploy/Traefik with a domain. Gives you an OpenAI-compatible
endpoint (`/v1/chat/completions`, `/v1/models`, ...) backed by your ChatGPT login.

## Deploy

1. Point DNS for `openai.debugblackbox.com` at your Traefik / Dokploy host.
2. In Dokploy: **Create → Compose**, link this repo.
3. Set env vars: copy `.env.example` to `.env` (set `CODEX_ARCH`, optionally `CODEX_JWT_TOKEN`).
4. Deploy. Then open `https://openai.debugblackbox.com` and **log in** (OAuth via
   the dashboard, or paste `CODEX_JWT_TOKEN` in `.env` to skip the interactive flow).
5. Point your apps at `https://openai.debugblackbox.com/v1` with the API key the proxy issues.

## Three things that will actually bite you

- **Architecture.** Mac Mini = Apple Silicon → `CODEX_ARCH=arm64`. If the
  `:latest` image is amd64-only it'll run emulated; check with
  `docker compose exec codex-proxy uname -m`. If it says `x86_64`, set
  `CODEX_ARCH=x64`.
- **Traefik label names.** The compose assumes a Dokploy-style Traefik:
  network `dokploy-network`, entrypoint `websecure`, cert resolver
  `letsencrypt`. If your Traefik uses different names, edit the labels in
  `docker-compose.yml` to match (or add the domain via Dokploy's **Domains**
  tab, which injects correct labels automatically).
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
