# Development guide: OpenGMB (StandardTouch fork)

This fork of [synectus/OpenGMB](https://github.com/synectus/OpenGMB) runs at https://gmb1.standardtouch.com. The upstream project is MIT licensed, and its [TRADEMARKS.md](../TRADEMARKS.md) allows changed versions as long as they don't use "OpenGMB" in a product or company name.

## Layout

- `apps/web`: Next.js 16 web app (UI, server actions, API routes).
- `apps/mcp`: MCP server (tools for AI clients).
- `apps/worker`: background jobs (sync, publishing, reports, schedules).
- `packages/core`: business logic shared by all apps (actions, permissions, guardrails, audit log, geo-grid drivers).
- `packages/db`: database schema and migrations (PostgreSQL, Drizzle).
- `deploy/selfhost`: Docker Compose stack used in production.

## Setup

Requires Node 22 (see `.nvmrc`) and pnpm 10.15.0.

```bash
pnpm install
pnpm dev          # all apps, through turbo
pnpm test         # vitest
pnpm typecheck
pnpm lint
```

## Branding

The name and tagline shown in the app are set in code, not in settings:

- `apps/web/lib/theme/brand.ts`: the top-left header. Default text is "OpenGMB by StandardTouch". With a custom logo or colour it reads "Powered by StandardTouch".
- `apps/web/lib/host-branding.ts`: the header on an agency's own address. Reads "Powered by StandardTouch".
- `apps/web/lib/branding-form.ts`: the settings preview. It checks for the default text, so keep it in sync with `brand.ts`.
- `apps/web/messages/pages/assistant.ts`: the assistant button and panel title ("Ask AI Assistant").
- `packages/core/src/ai/assistant.ts`: the assistant's system prompt, which still says "You are Ask OpenGMB". Change it only if the assistant's replies should use the new name.

Tests that check these strings: `apps/web/lib/theme/brand.test.ts`. Change them with the code.

## Configuration

Copy `deploy/selfhost/.env.example` to `.env` for local runs. Never commit `.env`.

Required secrets are `POSTGRES_PASSWORD`, `BETTER_AUTH_SECRET`, `ENCRYPTION_KEY`, and `AUDIT_CHECKPOINT_KEY`. Generate each with `openssl rand`. `ENCRYPTION_KEY` encrypts stored Google tokens and DataForSEO keys, so losing it makes them unreadable.

Optional integrations:

- Google: `GOOGLE_CLIENT_ID`, `GOOGLE_CLIENT_SECRET`, and `GOOGLE_BUSINESS_API_MODE` (`mock` or `live`). Redirect URIs must be `<WEB_URL>/api/google/callback` and `<WEB_URL>/api/auth/callback/google`.
- DataForSEO for rank tracking: `DATAFORSEO_LOGIN` and `DATAFORSEO_PASSWORD`. A key saved under Rank tracking → Usage and key takes precedence and is billed at cost.
- AI: `ANTHROPIC_API_KEY`, plus `OPENAI_API_KEY` and `PIXABAY_API_KEY` for images.
- Email: `RESEND_API_KEY` and `EMAIL_FROM`.

## Production deploy (Dokploy)

- Dokploy project `OpenGMB`, service `opengmb` (Compose).
- Source: Git, `https://github.com/hamzahali/OpenGMB-ST.git`, branch `standardtouch-branding`, compose path `./deploy/selfhost/docker-compose.yml`.
- Domain: `gmb1.standardtouch.com` → `web` service, port 3000, HTTPS via Let's Encrypt.
- Environment: set in Dokploy's Environment tab. The file is created on each deploy.
- Deploy: Dokploy → General → Deploy → Confirm. A first build takes several minutes. A build can fail with `next/font/google` errors when the server can't download Google Fonts. Redeploy first; the failure has been temporary.
- Migrations run automatically before the app starts. Take a backup first (see `SETUP.md`).

## Workflow

- Work on `standardtouch-branding` (or a new branch). Keep upstream changes merged in through rebase or a merge commit, not by pushing to upstream.
- Before a deploy, run `pnpm test` and `pnpm typecheck`.
- Google writes are off by default. Turn them on only after testing read-only features and the approval flow.

## Reference

- `SETUP.md`: full self-host setup, Google Cloud steps, backups and upgrades.
- `CONTRIBUTING.md`: upstream contribution guidelines.
- `SECURITY.md`: how to report vulnerabilities.
