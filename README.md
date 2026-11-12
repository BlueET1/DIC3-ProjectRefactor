# DIC3 — Project Proposal Refactor using OpenSpec

**專題**：以 TurtleBot3 Burger 為平台之室內導航與任務決策 AI Agent
**課程**：物聯網數據分析與應用 · DIC3 個人作業
**作者**：陳至恆（第 7 組）
**原始專題 repo**：<https://github.com/hank050392/tb3-navigation-agent>

> 本 repo 以 OpenSpec（spec-driven development）重構期中提案。內容為提案與規格，**尚無實驗結果**；所有效益都是待驗證的預期。

## 核心一句話

> **LLM 負責理解與協調；地點白名單、狀態檢查、Nav2 執行與事件紀錄負責判斷與證據。**

## 作業要求對照

| # | 作業要求 | 本 repo 對應 |
|---|---|---|
| 1 | 安裝 OpenSpec，熟悉 spec-driven development | [`openspec/`](openspec/)、[05 OpenSpec Workflow](docs/05-openspec-workflow.md)（含實際指令與 validate 結果） |
| 2 | 釐清專題 A–F 摘要、5 cores、3 trade-offs、10 details | [01 Abstract A–F](docs/01-abstract-A-to-F.md)、[02 Cores / Trade-offs / Details](docs/02-cores-tradeoffs-details.md) |
| 3 | 將 design.md 轉成 OpenSpec workflow | [`openspec/changes/add-tb3-navigation-agent/`](openspec/changes/add-tb3-navigation-agent/)：proposal → specs → design → tasks |
| 4 | Introduction 重構 | [03 Introduction](docs/03-introduction.md) |
| 5 | Related Work 重構 | [04 Related Work](docs/04-related-work.md)（新增 4 篇文獻與研究缺口比較表） |

## Repo 結構

```text
.
├── README.md
├── docs/
│   ├── 01-abstract-A-to-F.md          # A–F、300 字摘要、IEEE 風格 Title／Abstract
│   ├── 02-cores-tradeoffs-details.md  # 5 核心 · 3 取捨 · 10 細節 → 對應 spec
│   ├── 03-introduction.md             # 重構後緒論（F1–F3 ↔ 機制 ↔ RQ1–RQ3）
│   ├── 04-related-work.md             # 重構後相關研究 + 研究缺口表 + References
│   └── 05-openspec-workflow.md        # DESIGN.md → OpenSpec 對照與指令紀錄
├── openspec/
│   ├── config.yaml
│   ├── specs/                         # archive 後的正式規格（目前為空）
│   └── changes/add-tb3-navigation-agent/
│       ├── proposal.md
│       ├── specs/
│       │   ├── task-intent-parsing/spec.md
│       │   ├── robot-readiness/spec.md
│       │   ├── navigation-dispatch/spec.md
│       │   └── mission-reporting/spec.md
│       ├── design.md
│       └── tasks.md
├── source/                            # 轉換來源（自原始專題 repo 複製）
│   ├── DESIGN.md
│   ├── INTERFACES.md
│   └── domain_rules.md
└── .claude/skills/                    # openspec init 產生的 Claude Code skills
```

## 重現驗證

```bash
npm install -g @fission-ai/openspec@latest
openspec validate add-tb3-navigation-agent --strict
openspec show add-tb3-navigation-agent
```

## 重構重點摘要

1. **主軸統一**：A→F 全部圍繞「語言合理 ≠ 動作可行」，三個失效模式 F1–F3 對應三個機制與 RQ1–RQ3。
2. **規格化**：17 條 requirement、25 個 WHEN／THEN scenario，每個都可以直接變成測試案例。
3. **文獻補強**：新增 KnowNo（何時該問）、RoboGuard（LLM 機器人安全護欄）、ROS-LLM、LM-Nav，並以比較表說明研究缺口。
4. **誠實揭露**：不宣稱首次提出 LLM 導航；尚無實驗數據。

## AI 使用聲明

本 repo 的重構文字與規格由 AI（Claude Code）協助整理，內容來自原始專題文件。新增文獻 [13]–[16] 已核對 arXiv 編號與發表場合。作者負責檢查內容正確性。
