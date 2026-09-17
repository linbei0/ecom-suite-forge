# ecom-suite-forge

[English](README.md) · [简体中文](README.zh-CN.md)

[![skills.sh](https://skills.sh/b/linbei0/ecom-suite-forge)](https://skills.sh/linbei0/ecom-suite-forge)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

Turn one set of viral e-commerce product images (爆款套图) into a single reusable, product-agnostic **suite template (套图模版)**.

The skill reverse-engineers the *visual grammar* of a winning image set — shot roles, composition, camera, lighting, background, color system, text strategy — and rewrites it as placeholder-driven prompt templates. Swap in a different product later and the suite still works.

> [!NOTE]
> This is an [Agent Skill](https://agentskills.io). It forges templates only — it does not generate images and does not depend on any specific app, backend, or provider.

## What it does

Given a folder of 6–12 viral product images of the same product, the skill produces one `*.suite.json` file containing:

- an **L1/L2 category** and free-text `leaf` product name,
- a **conversion-ordered storyboard** mapping each source image onto a fixed 8-role funnel vocabulary (`HERO`, `PAIN_POINT`, `COMPARISON`, `SCENE`, `DETAIL`, `TRUST`, `VARIANT`, `CTA`),
- a **Campaign Style Lock** reused verbatim across every shot,
- one **English `promptTemplate` per shot** with hex colors, numeric product occupancy and a negative list.

At generation time the flow is:

```
user product photo (product truth)
   + suite template (N storyboard shots)
        -> an agent rewrites each shot with the user's product facts
   -> N final image prompts
        -> a professional, high-CTR product image set
```

## Install

```bash
npx skills add linbei0/ecom-suite-forge
```

Install a specific agent or a global scope:

```bash
npx skills add linbei0/ecom-suite-forge -a claude-code -a opencode
npx skills add linbei0/ecom-suite-forge -g -y
```

Works with any Agent Skills–compatible agent (Claude Code, Cursor, Codex, OpenCode, Gemini CLI, and more).

<details>
<summary>Manual install</summary>

Copy this repository into your agent's skills directory, e.g.:

- project: `.agents/skills/ecom-suite-forge/`
- global: `~/.config/opencode/skills/ecom-suite-forge/` or `~/.claude/skills/ecom-suite-forge/`

`SKILL.md` must stay at the skill folder root; keep `references/` and `assets/` alongside it.

</details>

## When it triggers

Use it when you have a folder or batch of 爆款电商图片 and want to 拆解 / 提炼 / 复刻 them into a reusable 套图模版, or when you are building and expanding a 套图库.

> [!IMPORTANT]
> One suite = one main product. A suite needs multiple shots to be meaningful — the minimum useful set is about 5, 6–12 is ideal. This skill never invents certifications, ratings, sales counts, or efficacy claims; unverifiable proof becomes a `proof placeholder`.

## Output

A single file named `<l1-slug>-<leaf-slug>.suite.json`, for example:

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

The full field contract lives in [`references/suite-schema.md`](references/suite-schema.md); a complete worked example is [`assets/suite-template.example.json`](assets/suite-template.example.json).

## How it works

The skill runs seven phases in order and gates the result before output:

1. **Intake** — confirm the whole set and the main product.
2. **Read & classify** — inspect every image, pick exact `L1`/`L2`, write a free-text `leaf`.
3. **Decompose each image** — fill the per-image analysis worksheet.
4. **Derive the Campaign Style Lock** — one whole-set visual contract.
5. **Write each shot's prompt template** — English, hex colors, numeric occupancy, negative list.
6. **De-identify + QA gate** — strip brand, logos, faces and source copy; all P0 checks must pass.
7. **Output** — write the suite JSON and report assumptions.

## Repository layout

```
SKILL.md                              # entry point: scope, iron rules, workflow
references/
  category-taxonomy.md                # built-in L1/L2 classification list
  shot-playbook.md                    # per-shot purpose, composition, camera, lighting
  prompt-contract.md                  # prompt iron rules + Campaign Style Lock
  de-identification.md                # what to strip from the source, and the audit
  suite-schema.md                     # suite JSON schema (v1)
  quality-checklist.md                # P0/P1/P2 gate
assets/
  analysis-worksheet.md               # per-image analysis worksheet
  suite-template.example.json         # complete example suite
skills.sh.json                        # skills.sh repository page config
README.md                             # English guide (this file)
README.zh-CN.md                       # Simplified Chinese guide
```

## Core rules

- **Detach from the source.** Only reusable visual grammar survives; brand, model identity, logos, and source copy are stripped.
- **Order is a funnel**, not a pile of images: click → benefit → pain → proof → scene → detail → trust → variant → close.
- **Colors are hex**, never adjectives; product occupancy is a number; every prompt ends with a concrete negative list.
- **One style lock, reused verbatim** — the set stays consistent while angles, backgrounds and purposes vary.
- **No invented facts.** Unverifiable proof is left as a placeholder.

## License

Released under the [MIT License](LICENSE).
