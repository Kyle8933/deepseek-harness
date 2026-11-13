---
kind: upgrade-guide
description: "替换 pi-ai 1.5.2 刷新后的目录不再列出的模型 id。"
---

# 模型目录刷新到 pi-ai 1.5.2

[English](guide.md) | 中文

## 变更

`@deepseek-ai/dsh-llm-pi-ai` 现在依赖 `@earendil-works/pi-ai@1.5.2`。模型发现直接从这一已安装的目录作答，不发起网络调用，因此每个路线的模型列表都随新目录变化。

- 35 个路线的对话模型发生变化：新增 159 个，移除 130 个。仅 OpenRouter 就新增 40 个、移除 33 个，其中被移除的 id 包括 `deepseek/deepseek-r1`、`openai/gpt-3.5-turbo`、`openai/gpt-4-turbo`、`minimax/minimax-m2`、`minimax/minimax-m2.1`、`inception/mercury-2` 以及 `:batch` 各项。`github-copilot` 从 36 个减到 27 个，`baseten` 从 22 个减到 15 个，`mistral` 从 40 个减到 34 个，`xiaomi` 从 6 个减到 3 个。
- `qwen-token-plan-individual-cn` 是新增路线，在 `QWEN_TOKEN_PLAN_CN_API_KEY` 下列出 10 个对话模型。
- `openai-codex` 移除 `gpt-5.5`。`openai` 路线仍通过 Responses 列出它。
- `providers.<route>.models` 条目或 `modelOverrides` 键指向目录不再列出的 id 时，路线配置加载以 `llm-pi-ai: provider "<route>" has no configured model "<id>"` 失败；该条目不再被静默忽略。
- 目录仍滞后于提供方：pi-ai 1.5.2 之后发布的模型在下次升级或显式 `models` 条目之前不会出现在列表中。

流式：pi-ai 的 `stream*` 函数现在把解析后的工具参数报告为增量部分值，而不再报告空对象，`parseStreamingJson` 已被移除。DSH 的 `argumentsDelta` 片段与最终 `arguments` 字符串保持不变。

## 迁移

1. 从源码加载路线配置时运行 `pnpm install`，否则安装新的应用构建。每个声明 pi-ai 的提供方包都改为 `^1.5.2`，`patches/@[EMAIL]` 取代 1.3.0 的补丁文件。
2. 逐个打开 `cordis.yml` 配置文件、模型设置页面，以及任何含 `providers.<route>.models` 或 `modelOverrides` 的设置文件，删除或替换目录不再列出的 id。加载配置时会指出第一个失效的 id。
3. 继续使用目录未收录的模型时，在 `providers.<route>.models` 中补上 `api`、`baseURL`、`contextWindow` 和 `maxTokens`。
4. 直接导入 pi-ai 的自定义插件：从完成事件读取最终参数，并把中间部分值当作最终值的前缀。把 `parseStreamingJson` 换成 `parseToolCallArguments`。
5. 确认方式：启动 `dsh`，打开各路线对应的模型设置页面，并为每个已配置模型各发起一次流式请求。
