# @launchthatbot/convex-backend

OpenClaw skill package that configures long-term memory, daily logs, and secret storage through a Convex backend.

## Source of truth

- Monorepo package path: `packages/launchthatbot-convex-backend`
- Mirror repository: `https://github.com/launchthatbot/convex`
- Sync workflow: `.github/workflows/sync-skill-mirrors.yml`

## What this skill enforces

- Do not store secrets in local plaintext files.
- Use Convex `secrets:*` mutations for API keys and tokens.
- Persist memory and daily logs in Convex so context survives restarts.

## Support

- Website: https://launchthatbot.com
- Discord: https://discord.gg/launchthatbot
