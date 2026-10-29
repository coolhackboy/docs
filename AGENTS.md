# PoYo API 文档

写新的 API 文档前先读本文件。生成类模型页的标题、请求体、OpenAPI 和 `docs.json` 都以这里为准。

## 生成类模型页标题结构

### 适用范围

用于 `api-manual` 下的图像、视频、3D、TTS、音乐**生成模型页**，例如 `image-series/nano-banana.mdx`、`video-series/sora-2.mdx`、`3d-series/hunyuan-3d-3-1.mdx`、`music-series/elevenlabs-v4-tts.mdx`、`music-series/elevenlabs-music.mdx`。

以下页面保持各自现有模板，不要套用本节的 `##` 顺序：

- LLM：`chat-series/`，按下文「LLM 页面结构」写
- Moderation：`moderation-series/`
- 音乐动作页：`generate-music`、`generate-lyrics`、`extend-music`、`add-vocals`、`add-instrumental`、`upload-and-cover-audio`、`upload-and-extend-audio`、vocal / style / cover / mashup / wav 转换等非「一个模型一张生成页」的页面
- Task management、account、file upload

### 文件

每个模型页一对文件，文件名 kebab-case，两边同名：

```
api-manual/{series}/{name}.mdx
api-manual/{series}/{name}.json
```

`series` 用已有目录：`image-series`、`video-series`、`3d-series`、`music-series`。

在 `docs.json` 里登记两处：

1. `api.openapi` 追加 `"/api-manual/{series}/{name}.json"`。
2. 对应分组的 `navigation` `pages` 追加 `"api-manual/{series}/{name}"`（不要带扩展名）。

`input` 里的参数名用 snake_case。API Key 占位符写 `PoYo_API_KEY`。请求体只有三个顶层字段：必填 `model`、可选 `callback_url`、以及 `input`。业务参数全部放进 `input`。

OpenAPI 使用 3.0.0，`servers` 为 `https://api.poyo.ai`。提交成功响应是 `code: 200`，`data` 里有 `task_id`、`status: not_started`、`created_time`。参数错误和未授权分别是 400、401，body 为 `error.message` 与 `error.type`。参数说明写在 `description` 里；已经有 `example` 字段时，description 里不要再写一遍 Example。

URL 参数（包括 URL 数组的 `items`）使用 `type: string` 和 `format: uri`。公开可访问及 HTTP/HTTPS 要求写在 `description` 中。

生成类模型页的 OpenAPI 不使用 `pattern` 展示正则限制，包括 URL 协议和 `prompt` 非空白校验，否则 Mintlify 会在参数下额外显示 `Pattern`。这些要求写在 `description` 中，保留 `minLength`、`maxLength`、`format` 等结构化约束。提交前检查整份 OpenAPI（包括嵌套字段和数组 `items`），确保没有残留的 `pattern`。

### Frontmatter

`title` 只写模型名，例如 `Nano Banana`、`Sora 2`、`Kling 3.0`。不要在标题后追加 Generation、Video Generation、Image Generation 或 API。侧栏已经挂在 Image / Video / Music / 3D 分组下。

```yaml
---
title: "Nano Banana"
description: "一句话说明这个模型做什么"
openapi: "/api-manual/image-series/nano-banana.json POST /api/generate/submit"
---
```

`openapi` 路径、HTTP 方法与 JSON 里的 path 保持一致。

### 正文开头

不要写与 frontmatter `title` 重复的一级标题。Mintlify 会渲染 `title`。

正文第一块是现有任务提示，原文保持不变：

```mdx
<Tip>
1. After submission, a `task_id` will be returned. If you provided a `callback_url`, when the task status becomes `finished` or `failed`, a POST request will be sent to the `callback_url`.
2. Regardless of whether `callback_url` is provided, you can retrieve the response result through the unified [Query Task Status](/api-manual/task-management/status) endpoint.
</Tip>
```

紧接着一段简短介绍：这个模型能生成什么、适合什么场景。不要再写 `# 模型名`。

### 二级标题顺序

按下面顺序写。某一节没有内容就整节省略，不要留空标题。

1. `## Models` — 每个对外 model id 一条 bullet。只有一个模型也用这个标题，并只列一条。不要用 Available Model、Available Models 或 Model。

   ```mdx
   ## Models

   - **nano-banana** — Text-to-image and image-to-image generation
   - **nano-banana-edit** — Advanced image editing
   ```

2. `## Features` — 这个模型能做什么。用它替换 Key Features、Capabilities、Generation Modes。这里写能力，不写互斥流程。

3. `## Input modes` — 仅当存在互斥工作流时才写，例如 text-to-video 与 image-to-video、text-to-image 与 image-to-image。说明每种模式何时使用、依赖哪些输入。只有一条路径时省略本节。

4. `## Required parameters` — `input` 里必填字段。每条写 `` `param_name` (type): 说明 ``。

5. `## Optional parameters` — `input` 里可选字段，同样用 snake_case。没有可选参数就省略。

6. 领域小节 — 仅在需要时，放在可选参数之后、示例之前。可用标题只有：`## Size`、`## Resolution`、`## Output`、`## Output Formats`、`## Voice Presets`。不要把这些小节放到 `## Models` 前面。没有对应约束就不要加。

7. `## Examples` — 按场景给出请求要点或与 OpenAPI examples 对应的说明。OpenAPI JSON 仍负责可执行示例；这里不要空标题。

8. `## Notes` — 限制、互斥、失败条件和调用时必须知道的行为。没有就省略。

### 不要写的内容

不要加 Pricing、Billing、Credit Billing，也不要写预估积分数。积分数量放在定价站。任务状态接口里的响应字段（例如 `credits_amount`）可以保留，那是 API 字段，不是定价说明。

不要在 MDX 里重复 OpenAPI 已经生成的大段 cURL / SDK 示例，除非 `## Examples` 需要点出模式差异。

## LLM 页面结构

`chat-series/` 下的侧栏分组叫 LLM Model，分三类页面：

- `overview.mdx`：全量模型表，列 model id、厂商、支持的 API 格式。不挂 `openapi`。
- API 格式页：`chat-completions`、`responses`、`claude-messages`、`gemini-native-format`，每种协议一页，与模型无关。
- 厂商页：一个厂商一对 `{provider}.mdx` / `{provider}.json`，例如 `openai`、`anthropic`、`deepseek`。

模型列表和每个模型支持的格式以前端为准：模型来自 `poyoapi/data/models.json`（`tasks` 含 `Chat`），格式规则来自 `poyoapi/components/tool/chat-playground/index.tsx` 的 endpoint 选择逻辑。

新增 LLM 模型时：

1. 在 `overview.mdx` 表里加一行。
2. 在对应厂商页 `## Models` 加一条，并把 id 加进该厂商 JSON 里 `model` 的 `enum`。
3. 没有对应厂商页时新建一对文件，并在 `docs.json` 的 `api.openapi` 和 LLM Model 下的 `Models` 分组登记。

厂商页规则：

- `title` 写厂商名，例如 `OpenAI`、`Anthropic`、`xAI`。
- playground 用一个该厂商全部模型都支持的格式，`openapi` 指向厂商 JSON。JSON 复制对应格式页的 schema，只把 `model` 改成该厂商的 `enum` 并给 `default`，不写 `example`。Gemini Native 的 `model` 在路径参数里。
- 二级标题顺序：`## Models`（每条注明支持的格式或「only」限制）、`## Examples`（对应 JSON examples）、`## Notes`。
- 不写价格。不要复制格式页的大段参数说明，链接过去即可。
