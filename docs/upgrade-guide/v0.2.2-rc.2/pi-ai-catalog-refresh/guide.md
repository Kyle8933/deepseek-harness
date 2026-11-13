---
kind: upgrade-guide
description: "Replace model ids that the refreshed pi-ai 1.5.2 catalog no longer lists."
---

# Model catalog refreshed to pi-ai 1.5.2

English | [中文](guide.zh.md)

## Change

`@deepseek-ai/dsh-llm-pi-ai` now depends on `@earendil-works/pi-ai@1.5.2`. Model discovery answers a route from that installed catalog without a network call, so every profile's model list follows the new catalog.

- Chat models change on 35 routes: 159 added, 130 removed. OpenRouter alone adds 40 and removes 33, including `deepseek/deepseek-r1`, `openai/gpt-3.5-turbo`, `openai/gpt-4-turbo`, `minimax/minimax-m2`, `minimax/minimax-m2.1`, `inception/mercury-2`, and the `:batch` entries. `github-copilot` drops from 36 to 27, `baseten` 22 to 15, `mistral` 40 to 34, and `xiaomi` 6 to 3.
- `qwen-token-plan-individual-cn` is a new route, listing 10 chat models under `QWEN_TOKEN_PLAN_CN_API_KEY`.
- `openai-codex` drops `gpt-5.5`. The `openai` route still lists it through Responses.
- A `providers.<route>.models` entry or `modelOverrides` key naming an id the catalog no longer lists fails profile load with `llm-pi-ai: provider "<route>" has no configured model "<id>"`; it is no longer ignored.
- The catalog still trails the provider: models released after pi-ai 1.5.2 stay unlisted until the next bump or an explicit `models` entry.

Streaming: pi-ai's `stream*` functions now report parsed tool arguments as incremental partial values instead of empty objects, and `parseStreamingJson` is gone. DSH's `argumentsDelta` fragments and finalized `arguments` strings are unchanged.

## Migration

1. Run `pnpm install` where profiles load from source, or install the new app build. Each provider package that declares pi-ai moves to `^1.5.2`, and `patches/@earendil-works+pi-ai@1.5.2.patch` replaces the 1.3.0 patch.
2. Open each `cordis.yml` profile, the Models settings page, and any settings file with `providers.<route>.models` or `modelOverrides`, and delete or replace ids that the catalog no longer lists. Loading the profile names the first stale id.
3. To keep using a model that the catalog omits, add it to `providers.<route>.models` with its `api`, `baseURL`, `contextWindow`, and `maxTokens`.
4. Custom plugins that import pi-ai directly: read the final arguments from the completion event and treat intermediate partials as prefixes of the finished value. Replace `parseStreamingJson` with `parseToolCallArguments`.
5. Confirm by starting `dsh`, opening Models settings for each route, and streaming one request per configured model.
