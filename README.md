# MemOS Cloud Memory (OpenClaw Lifecycle Plugin)

A minimal OpenClaw lifecycle plugin that **recalls** memories from MemOS Cloud before each run and **adds** new messages to MemOS Cloud after each run.

## Features
- **Recall**: `before_agent_start` → `/search/memory`
- **Add**: `agent_end` → `/add/message`
- Uses **Token** auth (`Authorization: Token <MEMOS_API_KEY>`)

## Install (local)
Copy this folder into an OpenClaw plugin path (e.g. `~/.openclaw/extensions/`) or use `plugins.load.paths` to point at it.

Example `~/.openclaw/openclaw.json`:
```json
{
  "plugins": {
    "entries": {
      "memos-cloud-memory": { "enabled": true }
    },
    "load": { "paths": ["/Users/shiuing/Desktop/funcode/memos-playground-openclaw-plugin"] }
  }
}
```
Restart the gateway after config changes.

## Environment Variables
> Also supports reading from `~/.openclaw/.env` when not present in the process env.
- `MEMOS_BASE_URL` (default: `https://memos.memtensor.cn/api/openmem/v1`)
- `MEMOS_API_KEY` (required; Token auth)
- `MEMOS_USER_ID` (optional; default: `openclaw-user`)
- `MEMOS_CONVERSATION_ID` (optional override)

## Optional Plugin Config
In `plugins.entries.memos-cloud-memory.config`:
```json
{
  "baseUrl": "https://memos.memtensor.cn/api/openmem/v1",
  "apiKey": "YOUR_API_KEY",
  "userId": "memos_user_123",
  "conversationId": "openclaw-main",
  "queryPrefix": "important user context preferences decisions ",
  "recallEnabled": true,
  "addEnabled": true,
  "captureStrategy": "last_turn",
  "includeAssistant": true,
  "memoryLimitNumber": 6,
  "preferenceLimitNumber": 6,
  "includePreference": true,
  "includeToolMemory": false,
  "toolMemoryLimitNumber": 6,
  "tags": ["openclaw"],
  "asyncMode": true
}
```

## How it Works
- **Recall** (`before_agent_start`)
  - Builds a `/search/memory` request using `user_id`, `query` (= prompt + optional prefix), and optional filters.
  - Formats facts/preferences/tools into a context block, then injects via `prependContext`.

- **Add** (`agent_end`)
  - Builds a `/add/message` request with the **last turn** by default (user + assistant).
  - Sends `messages` with `user_id`, `conversation_id`, and optional `tags/info/agent_id/app_id`.

## Notes
- `conversation_id` is derived from OpenClaw `sessionKey` when not set explicitly. **TODO**: consider binding to OpenClaw `sessionId` directly.
- When both lifecycle and hooks are enabled for memory, you may get **duplicate** injection/writes.

---

References:
- MemOS Cloud docs: `/Users/shiuing/Desktop/code/MemOS-Docs/content`
- OpenClaw plugin docs: `/opt/homebrew/lib/node_modules/openclaw/docs/plugin.md`
