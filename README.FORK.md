# Fork: vulkanfry/CLIProxyAPI

Fork of [router-for-me/CLIProxyAPI](https://github.com/router-for-me/CLIProxyAPI) with **Codex → xAI/Grok Responses** compatibility.

Default branch: **`main-xai-codex`**

## Why this fork exists

Codex (including multi_agent_v2, tools, MCP namespaces, shell history) speaks OpenAI Responses wire format. xAI Grok accepts a **subset**. Without adaptation you get:

```text
422 ModelInput (agent_message, shell_call, custom_tool_call, web_search_call, ...)
400 Maximum tools limit reached (namespace MCP fan-out > 200)
422 missing field environment (type=shell tool)
400 MCP server URL resolves to internal address
```

## Compatibility matrix (normalized before upstream)

### Input items
| Codex / OpenAI item | xAI handling |
|---|---|
| `message` | keep; strip phase/passthrough; `output_text`→`input_text` in history |
| `agent_message` | → `message` (author/recipient prefix; drop encrypted blob) |
| `function_call` | rebuild clean; null args→`{}`; fold `namespace` into name |
| `function_call_output` | string output (arrays/objects flattened) |
| `custom_tool_call*` | → `function_call*` |
| `local_shell_call` / `shell_call` | → `function_call` (+ output) |
| `web_search_call` / `image_generation_call` / `tool_search_call` | → clean `function_call` (id→call_id) |
| `compaction` / `reasoning` | keep (sanitize encrypted_content) |
| `context_compaction` | → `compaction` when possible |
| `compaction_trigger` / `additional_tools` / `item_reference` | drop |
| unknown | drop (avoid 422) |

### Tools
| Tool type | Handling |
|---|---|
| `function` / `web_search` / `code_interpreter` | keep |
| `namespace` | flatten nested tools; prefix `namespace.name` |
| `custom` | → `function` (+ input param schema) |
| `shell` | → `function` named shell (xAI needs environment) |
| `tool_search` / `image_generation` / `computer_use_preview` | drop |
| `mcp` | drop (private URL / validation issues) |
| after flatten | **cap 190** tools (prefer core/web_search; drop excess MCP) |

## Install into Quotio

```bash
git clone -b main-xai-codex https://github.com/vulkanfry/CLIProxyAPI.git
cd CLIProxyAPI
go build -o CLIProxyAPI ./cmd/server

UP="$HOME/Library/Application Support/Quotio/proxy/upstream"
mkdir -p "$UP/v7.2.61-custom-tool-fix"
cp CLIProxyAPI "$UP/v7.2.61-custom-tool-fix/CLIProxyAPI"
chmod +x "$UP/v7.2.61-custom-tool-fix/CLIProxyAPI"
ln -sfn "$UP/v7.2.61-custom-tool-fix" "$UP/current"
# restart Quotio / kill CLIProxyAPI on :18317 so it reloads
```

## Upstream sync

`.github/workflows/sync-upstream.yml` daily + manual:
- FF `main` from `router-for-me/CLIProxyAPI`
- rebase `main-xai-codex`
- open issue on conflict

Actions → **Sync upstream** → Run workflow.
