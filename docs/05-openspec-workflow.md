# 05. 把 DESIGN.md 轉成 OpenSpec Workflow

## 什麼是 Spec-Driven Development

先寫「系統應該做什麼」（spec），再寫「怎麼做」（design）和「分幾步做」（tasks），最後才寫程式。OpenSpec 把這個流程固定成四個 artifact：

```text
proposal.md  ──→  specs/<capability>/spec.md  ──→  design.md  ──→  tasks.md  ──→  apply（實作）──→  archive
   Why/What          行為契約（SHALL + Scenario）      How + 取捨        可驗收的工作項
```

## 安裝與初始化（本 repo 實際執行的指令）

```bash
npm install -g @fission-ai/openspec@latest   # 本次使用 v2.0.0
openspec init --tools claude --language zh-TW .
openspec new change add-tb3-navigation-agent
# 撰寫 proposal → specs → design → tasks
openspec validate add-tb3-navigation-agent --strict
openspec status --change add-tb3-navigation-agent
```

驗證結果：

```text
Change 'add-tb3-navigation-agent' is valid
Progress: 4/4 artifacts complete
[x] proposal  [x] specs  [x] design  [x] tasks
add-tb3-navigation-agent     0/21 tasks
```

## 原始 DESIGN.md 各節 → OpenSpec 對照

原始檔案保存在 [`source/DESIGN.md`](../source/DESIGN.md)。

| 原 DESIGN.md 章節 | 轉到 OpenSpec 的位置 | 轉換方式 |
|---|---|---|
| §1 問題與需求（功能／非功能需求） | `proposal.md` → Why、What Changes | 需求拆成 4 個 capability |
| §1 功能需求：解析、列地點、澄清 | `specs/task-intent-parsing/` | 改寫成 SHALL／MUST + WHEN／THEN 情境 |
| §1 功能需求：查狀態 | `specs/robot-readiness/` | 同上 |
| §1 功能需求：檢查路徑、派送／取消、監控 | `specs/navigation-dispatch/` | 同上 |
| §1 功能需求：證據回報；非功能：可追溯 | `specs/mission-reporting/` | 同上 |
| §2 Architecture、§3 Components | `design.md` → Architecture | 模組表多加一欄「對應 capability」 |
| §4 Data Flow、§5 AI Model | `design.md` → Decisions D1–D3 | 改寫成「選擇／替代方案／理由」 |
| §6 Domain Knowledge（`domain_rules.md` K01–K07） | 散入各 spec 的 requirement | 例：K05「任務完成 ≠ 成功」→ `mission-reporting` 的 *Evidence-based arrival* |
| §7 Flowchart、§8 Algorithm | `design.md` → Algorithm | 原樣保留；「最多一次」寫成 Decision D4 |
| §9 Interfaces（`INTERFACES.md`） | `navigation-dispatch` 的 *Whitelisted tool gateway*；`mission-reporting` 的 *Structured report fields* | 工具與欄位變成可測試的要求 |
| §10 How to Run、§11 Results | `tasks.md` 各組驗證條件 | 每個 task 都附「怎麼驗證完成」 |
| §12 Limitations | `design.md` → Risks / Trade-offs、Non-Goals | 改成「[風險] → 對策」格式 |

## Spec 統計

| Capability | Requirements | Scenarios |
|---|:-:|:-:|
| `task-intent-parsing` | 4 | 7 |
| `robot-readiness` | 3 | 5 |
| `navigation-dispatch` | 6 | 8 |
| `mission-reporting` | 4 | 5 |
| **合計** | **17** | **25** |

每個 Scenario 都可以直接變成一個測試案例；原始 repo 的 8 個 mock 情境（`CLARIFY`、`NOT_READY`、`NO_PATH`、`FAILED`、`TIMEOUT`、`BUSY`、`CANCELED`、`ARRIVED`）都已涵蓋。

## 為什麼這樣轉有幫助

- **原本**：DESIGN.md 混合了需求、架構、演算法與狀態，看不出「什麼算做完」。
- **現在**：
  - spec 定義驗收標準（例如「去門口」必須回傳 CLARIFY 且 `goal_sent` 為 false）。
  - design 只記錄決策與取捨。
  - tasks 依相依順序排列，每項都有驗證方式。
- **期末**：實作完成後執行 `openspec archive`，specs 會合併成 `openspec/specs/` 的正式規格，成為期末報告「系統規格」章節的來源。

## 在 Claude Code 中使用

`openspec init --tools claude` 已在 `.claude/skills/` 安裝 OpenSpec skills（`openspec-propose`、`openspec-apply-change`、`openspec-archive-change` 等），之後可以直接請 Claude Code 依 `tasks.md` 逐項實作。
