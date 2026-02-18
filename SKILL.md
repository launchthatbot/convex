---
name: convex-backend
version: 1.0.1
description: Store secrets, memory, and daily logs in your Convex backend instead of local files
author: LaunchThatBot
requires:
  mcp: convex
---

# Convex Backend Integration

You are connected to a **Convex backend** via the Convex MCP server. This gives you persistent, cloud-based storage that survives container restarts and can be accessed from the LaunchThatBot dashboard.

## MCP Requirement (Stock Convex MCP)

This skill uses the stock Convex MCP server (`convex@latest mcp start`), not a custom LaunchThatBot MCP tool for data storage.

Recommended setup:

```json
{
  "mcpServers": {
    "convex": {
      "command": "npx",
      "args": ["-y", "convex@latest", "mcp", "start"]
    }
  }
}
```

Reference: https://docs.convex.dev/ai/convex-mcp-server

## mcporter Compatibility Preflight

If your runtime does not expose MCP tools natively (common in OpenClaw/Pi flows), use `mcporter` as the bridge.

Check prerequisites before running this skill workflow:

```bash
mcporter --version || npx -y mcporter --version
mcporter list || npx -y mcporter list
mcporter list convex --schema || npx -y mcporter list convex --schema
```

If `mcporter` is not runnable, tell the user they need `npx mcporter` (or a local install). If `convex` MCP is missing, ask the user to configure stock Convex MCP, then re-run the checks.

When bridging via `mcporter`, call Convex MCP tools through CLI commands (for example `npx -y mcporter call convex.run ...`) instead of assuming native MCP tool invocation in the runtime.

## CRITICAL RULES

1. **NEVER write secrets, API keys, passwords, or tokens to local `.env` files or `auth.json`.** Always use the Convex backend.
2. **NEVER store sensitive credentials in MEMORY.md or daily log files.** Use the `secrets:set` mutation.
3. **Always write daily summaries to Convex** at the end of each session or when asked to save progress.
4. **Always check Convex for existing secrets** before asking the user to provide them again.

## Storing Secrets (API Keys, Passwords, Tokens)

When a user provides an API key, password, or any credential:

1. Use the Convex MCP `run` tool to call `secrets:set`:
   ```
   Function: secrets:set
   Arguments: {
     "agentId": "<your-agent-id>",
     "key": "OPENAI_API_KEY",
     "value": "sk-...",
     "category": "ai_provider",
     "description": "OpenAI API key for GPT-4"
   }
   ```

2. Common secret categories: `ai_provider`, `email`, `social_media`, `api`, `database`, `browser`, `other`

3. To retrieve a stored secret:
   ```
   Function: secrets:get
   Arguments: { "agentId": "<your-agent-id>", "key": "OPENAI_API_KEY" }
   ```

4. To list all stored secrets (values hidden):
   ```
   Function: secrets:list
   Arguments: { "agentId": "<your-agent-id>" }
   ```

5. To delete a secret:
   ```
   Function: secrets:remove
   Arguments: { "agentId": "<your-agent-id>", "key": "OPENAI_API_KEY" }
   ```

## Also Set as Environment Variable

After storing a secret in Convex, also set it as a deployment environment variable so other tools and services can access it:

```
Tool: envSet
Arguments: { "name": "OPENAI_API_KEY", "value": "sk-..." }
```

This ensures both the Convex database and the runtime environment have the value.

## Storing Long-Term Memory

When you learn something important about the user, their preferences, or make a significant decision:

```
Function: memory:addMemory
Arguments: {
  "agentId": "<your-agent-id>",
  "type": "fact",
  "content": "User prefers TypeScript over JavaScript for all new projects",
  "tags": ["preferences", "coding"]
}
```

Memory types:
- `fact` — Something true about the user or their setup
- `preference` — User likes/dislikes
- `decision` — A choice that was made and should be remembered
- `note` — General observations or context

To recall memories:
```
Function: memory:searchMemory
Arguments: { "agentId": "<your-agent-id>", "type": "preference", "limit": 20 }
```

## Daily Log Entries

At the end of each work session, write a summary of what was accomplished:

```
Function: memory:writeDailyLog
Arguments: {
  "agentId": "<your-agent-id>",
  "date": "2026-02-17",
  "content": "## Summary\n- Set up email integration with Resend\n- Configured GitHub SSH keys\n- Started work on Twitter bot automation\n\n## Blockers\n- Need Twitter API key from user"
}
```

Daily logs are append-only — calling `writeDailyLog` for the same date appends to the existing entry.

To review past logs:
```
Function: memory:listDailyLogs
Arguments: { "agentId": "<your-agent-id>", "limit": 7 }
```

## Session Startup Checklist

At the beginning of each session:

1. Check for stored secrets: `secrets:list`
2. Load recent memories: `memory:searchMemory` with limit 20
3. Load today's log: `memory:getDailyLog` with today's date
4. Load yesterday's log for continuity context

This ensures you have full context from previous sessions.

## Your Agent ID

Your agent ID is provided in your agent configuration. Use it consistently in all Convex calls. If you're unsure of your agent ID, check your agent YAML config file.
