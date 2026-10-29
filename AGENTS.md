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

## 中文文档（zh）

英文是源语言，文件在仓库根目录，URL 不带前缀。中文文件放在 `zh/` 下，路径与英文一一对应，例如 `api-manual/image-series/nano-banana.mdx` 对应 `zh/api-manual/image-series/nano-banana.mdx`。翻译人工完成，不用脚本调用 LLM。

新增或修改英文页时，同步改对应中文页（MDX 和 OpenAPI JSON），并在 `docs.json` 的 `en` 和 `zh` 两份导航里都登记。zh 导航里的页面路径带 `zh/` 前缀。

OpenAPI JSON 也有中文版，放在 `zh/` 下的同名路径（如 `zh/api-manual/image-series/nano-banana.json`），并在 `api.openapi` 里登记。中文页 frontmatter 的 `openapi` 指向中文 JSON，例如 `"/zh/api-manual/image-series/nano-banana.json POST /api/generate/submit"`，方法和 path 不变。中文 JSON 与英文 JSON 结构完全一致，只翻译 `description`、`summary`、`title` 这几个字段的字符串；`enum`、`example`、`examples[].value`、`default`、`operationId`、`tags`、属性名都不改。改英文 JSON 时同步改中文 JSON。

### OpenAPI 公共字段

所有接口共用以下写法，新增 JSON 时照抄：

- `securitySchemes` 的 `description` 写清创建 API Key 的入口：**Dashboard → API Key**，链接 `https://poyo.ai/dashboard/api-key`（中文版写 **仪表盘 → API 密钥**，链接 `https://poyo.ai/zh/dashboard/api-key`）。
- 响应外层的 `code`（integer）写完整 `enum`：`[200, 400, 401, 402, 403, 404, 408, 429, 500, 502, 503]`，`description` 列出每个值的含义并链接错误码页。
- 任务 `status` 写完整 `enum`：`["not_started", "running", "finished", "failed"]`，`description` 列出每个值的含义。提交接口的 `example` 仍为 `not_started`，并说明提交时总是这个值。

### 翻译规则

- Frontmatter：只翻译 `title`、`description`、`sidebarTitle`。`openapi`、`icon` 等其他字段原样保留。标题是模型名、厂商名或产品名时不翻译（如 `Nano Banana`、`OpenAI`）。
- 代码块（含行内代码）整块原样保留，包括代码注释和 JSON 示例里的 prompt。
- 不翻译：model id、参数名、字段名、枚举值、HTTP 方法与状态码、URL、产品与厂商名、`task_id` / `callback_url` 等 API 术语。
- 站内链接加 `/zh` 前缀：`/api-manual/...`、`/integrations/...`、`/platform-security/...`、`/index` 分别改成 `/zh/api-manual/...` 等。外部链接不改。
- 锚点链接：标题翻译后，同步把 `#anchor` 改成中文标题生成的 slug（小写，去掉标点，空格变 `-`，中文字符保留）。
- MDX 组件标签和属性名不变，只翻译属性里给人看的文字（如 `<Card title="...">` 的 `title`）。`href`、`icon` 不变。
- 表格、列表、标题层级和段落顺序与英文一致，不增删内容。
- 用简体中文，中英文之间不强制加空格，标点用全角。
- OpenAPI 字符串里的 Markdown（行内代码、列表、代码块、链接）保持结构，站内链接同样加 `/zh` 前缀。`Range: 0-1` 这类取值说明照译，数字和单位不变。

### 固定译法

开头任务提示的两种英文写法都统一译为：

```mdx
<Tip>
1. 提交后会返回 `task_id`。如果提供了 `callback_url`，当任务状态变为 `finished` 或 `failed` 时，系统会向 `callback_url` 发送 POST 请求。
2. 无论是否提供 `callback_url`，都可以通过统一的[查询任务状态](/zh/api-manual/task-management/status)接口获取结果。
</Tip>
```

二级标题：Models → 模型；Features → 功能；Input modes → 输入模式；Required parameters → 必填参数；Optional parameters → 可选参数；Size → 尺寸；Resolution → 分辨率；Output → 输出；Output Formats → 输出格式；Voice Presets → 预设音色；Examples → 示例；Notes → 注意事项。

术语：task → 任务；callback / webhook → 回调 / Webhook；text-to-image → 文生图；image-to-image → 图生图；text-to-video → 文生视频；image-to-video → 图生视频；prompt → 提示词；aspect ratio → 宽高比；API Key 保持 `API Key`；endpoint → 接口；playground 保持 Playground；credits → 积分；agent 保持 `Agent`，AI agent 写 `AI Agent`，不译成「智能体」或「代理」（「代理」只用于网络代理 proxy）。
