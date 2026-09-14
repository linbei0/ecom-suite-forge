# ecom-suite-forge 电商套件锻造

[English](README.md) · [简体中文](README.zh-CN.md)

[![skills.sh](https://skills.sh/b/linbei0/ecom-suite-forge)](https://skills.sh/linbei0/ecom-suite-forge)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

将一组爆款电商产品图（爆款套图）逆向提炼为一个可复用、与产品无关的 **套图模版（suite template）**。

本 Skill 会逆向拆解一组成功套图的 *视觉语法* —— 分镜角色、构图、机位、光线、背景、配色系统、文案策略 —— 并将其改写为占位符驱动的 Prompt 模版。之后换成任意产品，套图依然适用。

> [!NOTE]
> 这是一个 [Agent Skill](https://agentskills.io)。它只锻造模版 —— 不生成图片，也不依赖任何特定应用、后端或 Provider。

## 它能做什么

输入同一产品的 6–12 张爆款产品图，Skill 会产出单个 `*.suite.json` 文件，其中包含：

- 一个 **L1/L2 品类** 和一个自由文本的 `leaf` 产品名；
- 一张**按转化顺序排列的分镜表**，把每张源图映射到固定的 8 角色漏斗词汇表（`HERO`、`PAIN_POINT`、`COMPARISON`、`SCENE`、`DETAIL`、`TRUST`、`VARIANT`、`CTA`）；
- 一套**逐镜原样复用的 Campaign Style Lock**；
- 每个分镜一条**英文 `promptTemplate`**，包含 hex 颜色、数值化的产品占比、明确的留白与 negative list。

生成阶段的流程如下：

```
用户产品图（product truth）
   + 套图模版（N 个分镜）
        -> 由 agent 结合用户产品事实重写每个分镜
   -> N 条最终生图 Prompt
        -> 一套专业、高点击率的产品图
```

## 安装

```bash
npx skills add linbei0/ecom-suite-forge
```

安装到指定 agent 或使用全局作用域：

```bash
npx skills add linbei0/ecom-suite-forge -a claude-code -a opencode
npx skills add linbei0/ecom-suite-forge -g -y
```

适用于任何兼容 Agent Skills 的 agent（Claude Code、Cursor、Codex、OpenCode、Gemini CLI 等）。

<details>
<summary>手动安装</summary>

把本仓库拷贝进你的 agent skills 目录，例如：

- 项目级：`.agents/skills/ecom-suite-forge/`
- 全局级：`~/.config/opencode/skills/ecom-suite-forge/` 或 `~/.claude/skills/ecom-suite-forge/`

`SKILL.md` 必须位于 skill 目录根部；`references/` 与 `assets/` 需与其同级保留。

</details>

## 触发时机

当你手头有一组或一批爆款电商图片，想将它们拆解 / 提炼 / 复刻为可复用的套图模版，或在搭建与扩充套图库时使用。

> [!IMPORTANT]
> 一个套图对应一个主产品。套图需要多个分镜才有意义 —— 可用下限约 5 张，6–12 张最佳。本 Skill 绝不虚构认证、评分、销量或功效宣称；无法验证的证明一律保留为 `proof placeholder`。

## 产出

单个文件，命名为 `<l1-slug>-<leaf-slug>.suite.json`，例如：

```jsonc
{
  "schemaVersion": 1,
  "kind": "ecom.suite",
  "id": "suite-hufugehu-jiemianru",
  "name": "氨基酸温和不紧绷洁面乳款",
  "category": { "l1": "护肤个护", "l2": "面部护理", "leaf": "氨基酸温和洁面乳", "leafKeywords": ["洁面乳", "氨基酸", "温和"] },
  "styleLock": { "lockText": "Campaign Style Lock: ..." },
  "shots": [
    { "shotId": "shot-01", "order": 1, "shotRole": "HERO", "promptTemplate": "Product photography of {product}. ..." }
  ],
  "provenance": { "sourceKind": "viral-reference-set", "sourceImageCount": 8, "detached": true }
}
```

完整字段契约见 [`references/suite-schema.md`](references/suite-schema.md)；完整示例见 [`assets/suite-template.example.json`](assets/suite-template.example.json)。

## 工作方式

Skill 依次运行七个阶段，并在输出前对结果做质量门禁：

1. **Intake** —— 确认整套图片与主产品。
2. **Read & classify** —— 逐张查看图片，选定精确的 `L1`/`L2`，写出自由文本 `leaf`。
3. **Decompose each image** —— 填写逐图分析工作表。
4. **Derive the Campaign Style Lock** —— 提炼一份全套装通用的视觉契约。
5. **Write each shot's prompt template** —— 英文、hex 颜色、数值化占比、negative list。
6. **De-identify + QA gate** —— 剥离品牌、Logo、人脸与源图文案；所有 P0 检查必须通过。
7. **Output** —— 写出套图 JSON 并说明假设。

## 仓库结构

```
SKILL.md                              # 入口：范围、铁律、工作流
references/
  category-taxonomy.md                # 内置 L1/L2 分类清单
  shot-playbook.md                    # 各分镜的目的、构图、机位、光线
  prompt-contract.md                  # Prompt 铁律 + Campaign Style Lock
  de-identification.md                # 需要剥离的源图元素与审计规则
  suite-schema.md                     # 套图 JSON Schema（v1）
  quality-checklist.md                # P0/P1/P2 门禁
assets/
  analysis-worksheet.md               # 逐图分析工作表
  suite-template.example.json         # 完整示例套图
skills.sh.json                        # skills.sh 仓库页配置
README.md                             # 英文说明
README.zh-CN.md                       # 简体中文说明（本文件）
```

## 核心规则

- **与源图解耦。** 只保留可复用的视觉语法；品牌、模特身份、Logo 与源图文案一律剥离。
- **顺序即漏斗**，而非图片堆叠：点击 → 卖点 → 痛点 → 证明 → 场景 → 细节 → 信任 → 变体 → 收单。
- **颜色用 hex**，不用形容词；产品占比与留白用数字；每条 Prompt 以具体的 negative list 结尾。
- **一套 style lock，原样复用** —— 在机位、背景与目的变化时保持一致。
- **不虚构事实。** 无法验证的证明保留为占位符。

## 许可证

基于 [MIT License](LICENSE) 发布。
