# Fork: vulkanfry/CLIProxyAPI

Fork of [router-for-me/CLIProxyAPI](https://github.com/router-for-me/CLIProxyAPI) with Codex multi-agent / xAI Responses compatibility patches.

## Patch branch

Use **`main-xai-codex`** (or `fix/xai-custom-tool-modelinput`).

### What it fixes

When Codex `multi_agent_v2` subagents call Grok via local Quotio/CLIProxyAPI:

1. **`agent_message` + `encrypted_content`** → mapped to normal `message` (xAI has no `agent_message` ModelInput variant)
2. **`custom_tool_call*` / `shell_call` / hosted tool history** → `function_call*`
3. **MCP namespace fan-out** past xAI’s tool limit → cap at 190 (prefer core/web_search)
4. Drop unsupported tool types like `computer_use_preview`

Without this, typical errors:

```text
422 ... untagged enum ModelInput
400 ... Maximum tools limit reached ... maximum is 200
```

## Install over Quotio upstream binary

```bash
git clone -b main-xai-codex https://github.com/vulkanfry/CLIProxyAPI.git
cd CLIProxyAPI
go build -o CLIProxyAPI ./cmd/server

UP="$HOME/Library/Application Support/Quotio/proxy/upstream"
mkdir -p "$UP/v7.2.61-custom-tool-fix"
cp CLIProxyAPI "$UP/v7.2.61-custom-tool-fix/CLIProxyAPI"
chmod +x "$UP/v7.2.61-custom-tool-fix/CLIProxyAPI"
ln -sfn "$UP/v7.2.61-custom-tool-fix" "$UP/current"
# restart CLIProxyAPI / Quotio
```

## Upstream sync

GitHub Action `.github/workflows/sync-upstream.yml`:

- daily cron + manual `workflow_dispatch`
- fast-forwards `main` from `router-for-me/CLIProxyAPI`
- rebases `main-xai-codex` onto upstream; force-with-lease push
- on conflict, opens/comments an issue labeled `upstream-sync`

Trigger manually: **Actions → Sync upstream → Run workflow**.
