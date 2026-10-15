# Mintlify Starter Kit

Use the starter kit to get your docs deployed and ready to customize.

Click the green **Use this template** button at the top of this repo to copy the Mintlify starter kit. The starter kit contains examples with11

- Guide pages
- Navigation
- Customizations
- API reference pages
- Use of popular components

**[Follow the full quickstart guide](https://starter.mintlify.com/quickstart)**

## Development

Install the [Mintlify CLI](https://www.npmjs.com/package/mint) to preview your documentation changes locally. To install, use the following command:

```
npm i -g mint
```

Use Node.js 22 LTS for local development. `mint dev` does not support Node 25+.

Run the following command at the root of your documentation, where your `docs.json` is located:

```
mint dev
```

View your local preview at `http://localhost:3000`.

## Publishing changes

Install our GitHub app from your [dashboard](https://dashboard.mintlify.com/settings/organization/github-app) to propagate changes from your repo to your deployment. Changes are deployed to production automatically after pushing to the default branch.

## Need help?

### Troubleshooting

- If your dev environment isn't running: Run `mint update` to ensure you have the most recent version of the CLI.
- If a page loads as a 404: Make sure you are running in a folder with a valid `docs.json`.

### Resources
- [Mintlify documentation](https://mintlify.com/docs)

## 生成类模型页标题结构

写新的图像、视频、3D、TTS 或音乐生成模型页时遵循本节。完整约定在仓库根目录 [`AGENTS.md`](AGENTS.md)。

### 适用范围

用于 `api-manual` 下的图像、视频、3D、TTS、音乐**生成模型页**，例如 `image-series/nano-banana.mdx`、`video-series/sora-2.mdx`、`3d-series/hunyuan-3d-3-1.mdx`、`music-series/elevenlabs-v4-tts.mdx`、`music-series/elevenlabs-music.mdx`。

以下页面保持各自现有模板，不要套用本节的 `##` 顺序：

- Chat：`chat-series/`（chat completions、Claude messages、responses）
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
